# Binary Spec
- [4 bytes] - **magic**. 
- [4 bytes] - **version**. (LE uint32).
- [2 bytes] - **major_version**. (LE uint16).
- [2 bytes] - **minor_version**. (LE uint16).
- [* bytes] - **dynamic_header**. See [DynamicHeader](#dynamicheader).
- [32 bytes] - **sha256**. `SHA256(header)`. (SHA256 hash from beginning of file up to **dynamic_header**, inclusive)
- [32 bytes] - **hmac_sha256**.  `HMAC-SHA256(header)` with key `SHA512(0xff * 8 || SHA512(master_seed || transformed_key || 0x01)))`
- [* bytes] - **payload**. The encrypted (and possibly compressed) payload containing XML data and another header. See [Payload](#payload)

## DynamicHeader
A series of items with the following structure:

- [1 byte] - **type**.
- [4 bytes] - **length**. (LE uint32).
- [**length** bytes] - **data**.

Where **type** denotes one of the following item types
- `0x00` - **end**. The last header item. **data** is 0 bytes.
- `0x01` - **comment**.
- `0x02` - **cipher_id**. Payload decryption method. **data** is 32 byte id.
  - AES256 - `31c1f2e6bf714350be5805216afc5aff`
  - twofish - `ad68f29f576f4bb9a36ad47af965346c`
  - chacha20 - `d6038a2b8b6f4cb5a524339a31dbb59a`
- `0x03` - **compression_flag**. Denotes whether payload is compessed (after decryption). 0 = uncompressed payload.  1 = gzipped payload (wbits `16 + 15`). **data** LE uint8 and 3 padding bytes. 
- `0x04` - **master_seed**. Bytes used in payload decryption and header validation.  `data` is 32 bytes
- `0x07` - **encryption_iv**. Intialization vector used for payload decryption. **data** is 16 bytes.
- `0x11` - **kdf_parameters**. Key derivation function parameters for computing **transformed_key**. See [KDFParameters](#KDFParameters). **data** is [VariantDictionary](#variantdictionary)
- `0x12` - **plugin_data**. Unencrypted data provided by plugins. **data** is [VariantDictionary](#variantdictionary).

## VariantDictionary
A data structure new to KDBX4 for storing [KDFParameters](#kdfparameters) and custom plugin data.

- [2 bytes] - **version**.
- [* bytes] - **items** - dictionary items. See below.
- [1 bytes] - `0x00`. Null byte.

**items** is a series of items with the following structure:

- [1 byte] - **type**.
- [4 bytes] - **key_length**. (LE uint32).
- [**key_len** bytes] - **key**. (UTF8 encoded string).
- [4 bytes] - **value_length**. (LE uint32).
- [**value_len** bytes] - **value**.

**type** determines the data type of **value** and is one of the following:
- `0x04` - LE uint32
- `0x05` - LE uint64
- `0x08` - Boolean
- `0x0C` - LE int32
- `0x0D` - LE int64
- `0x18` - UTF8 string
- `0x42` - bytes

## KDFParameters
**kdf_parameters** is a VariantDictionary. It always has an item with key `$UUID` that denotes the key derivation method used to find **transformed_key**.

**value** will take one of the following values:
- `c9d9f39a628a4460bf740d08c18a4fea` - AESKDF. See [AESKDF](#aeskdf)
- `ef636ddf8c29444b91f7a9a403e30a0c` - Argon2D. See [Argon2D](#argon2d)

The above KDF methods operate on **key_composite**, which is computed from the provided password and/or keyfile data.

**key_composite** = SHA256(SHA256(`password`) || SHA256(`keyfile_data`))

#### Argon2d
The new default key derivation method for KDBX4.

**kdf_parameters** will have additional items with the following keys

- `I` - Aargon2d time cost. (**value** is LE uint64).
- `M` - Argon2d memory cost. (**value** is LE uint64).
- `P` - Argon2d parallelism. (**value** is LE uint32).
- `S` - Argon2d salt. (**value** is bytes).
- `V` -  Argon2d version. (**value** is LE uint32).

**transformed_key** is computed as argon2d(**key_composite**) with time cost **kdf_parameters.I**, memory cost **kdf_parameters.M**, parallelism **kdf_parameters.P**, salt **kdf_parameters.S**, version **kdf_parameters.V**

#### AESKDF
The old key derivation method used in KDBX3.

**kdf_parameters** will have additional items with the following keys

- `R` - AESKDF Rounds. (**value** is LE uint64).
- `S` - AESKDF Key. (**value** is bytes).

```
transformed_key = key_composite
for i=1 to kdf_parameters.R
    transformed_key = AESECB(transformed_key)
```

Where AESECB is initialized with key **kdf_parameters.S** and IV `0x00 * 16`.

## Payload
A series of block encrypted with the method specified by **cipher_id**.  Each block has the following structure:

- [32 bytes] - **hmac_hash**. `HMAC-SHA256(i || block_len || block_data)` with key `SHA512(i || SHA512(master_seed || transformed_key || 0x01))`. *i* is the current zero-indexed block index expressed as a LE uint64.
- [4 bytes] - **block_len**. (LE uint32).
- [**block_len** bytes] - **block_data**. A block encrypted with the method specified by **cipher_id**. 

The **block_data**s should be concatenated before using the decryption method specified by **cipher_id**. See [AES256](#aes256), [twofish](#twofish), and [chacha20](#chacha20) for decryption.

If **compression_flag** is 1, the payload is gzip compressed and can be decompressed using zlib with window bits set to `16 + 15`.

The decrypted payload has the following structure:

- [* bytes] - **inner_header**. See [InnerHeader](#innerheader).
- [* bytes] - **xml_data**. Raw XML data containing DB info.

#### InnerHeader
A series of items with the following structure:

- [1 byte] - **type**.
- [4 bytes] - **length**. (LE uint32).
- [**length** bytes] - **data**.

Where **type** denotes one of the following item types:
- `0x00` - **end**. The last header item. (**data** 0 bytes).
- `0x01` - **inner_random_stream_id**. Used in unprotecting XML data. (**data** 4 bytes).
- `0x02` - **inner_random_stream_key**. Used in unprotecting XML data. (**data** 64 bytes).
- `0x03` - **binary_data**. Binary plugin data prefixed by a Boolean byte which denotes whether this data should be memory protected.

#### AES256
Decrypt concatenated **block_data** using AES256-CBC with key **master_key** and IV **encryption_iv**.

#### twofish
Decrypt concatenated **block_data** using Twofish CBC with key **master_key** and IV **encryption_iv**.

#### chacha20
Decrypt concatenated **block_data** using ChaCha20 cipher with key **master_key** and nonce **encryption_iv**



