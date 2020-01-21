%%%
Title = "KeePass RFC"
area = "Security"

[seriesInfo]
name="Internet-Draft"
value="draft-keepass-01"
stream="IETF"
status="informational"

date = 2019-09-09T00:00:00Z

[[author]]
initials="E."
surname="Widloski"
fullname="E. Widloski"
%%%

.# Abstract

This is a small test document!!

{mainmatter}

# KeePass Head (style 1)
   
{align="center"}
     Length    Name                  Data Type
     -------   -------------------   -------------
           4   magic1
           4   magic2                LE uint32
           2   major_version         LE uint16
           2   minor_version         LE uint16
           *   header_item           HeaderItem
          32   header_integrity  
          32   header_authenticity   
           *   payload

magic1:
: 4 unique bytes for filetype identification

magic2:
: 4 unique bytes for filetype identification

major_version:
: int representing database major version (e.g. '4')

minor_version:
: int representing database major version (e.g. '0')

header_items:
: variable length item containing information about encryption, compression, key derivation or plugin data (see Section (#HeaderItem))

header_integrity:
: SHA256 hash of header for detecting unintentional corruptions.  Computed as `SHA256(m)` where m is all bytes starting from the beginning of file to the last byte of the last HeaderItem (inclusive).

header_authenticity:
: HMAC-SHA256 hash of header for verifying database authenticity.  Computed as `HMAC-SHA256(K, m)` where m is all bytes starting from the beginning of the file to the last byte of the last HeaderItem (inclusive).  K is the secret key, computed as `SHA512(\xff * 8 || SHA512(master_seed || transformed_key || \x01))` where || represents concatenation and * represents byte duplication.  master_seed and transformed_key can be found in header_items.

# KeePass Head (style 2)
          
| Length 1 | Name           | Type          | Meaning                                                                                                                                                                                              |
|      --- | ---            | ---           | ---                                                                                                                                                                                                  |
|        4 | magic1         |               | 4 unique bytes for filesystem identification                                                                                                                                                         |
|        4 | magic2         |               | 4 unique bytes for filesystem identification                                                                                                                                                         |
|        2 | major_version  | LE uint16     | int representing database major version (e.g. '4')                                                                                                                                                   |
|        2 | minor_version  | LE uint16     | int representing database minor version (e.g. '0')                                                                                                                                                   |
|        * | dynamic_header | DynamicHeader | holds encryption and compression information about the payload variable length item containing information about encryption, compression, key derivation or plugin data (see Section (#HeaderItem)) |


# KeePass Head (style 3)

{align="center"}
| Length | Name           | Type          |
|    --- | ---            | ---           |
|      4 | magic1         |               |
|      4 | magic2         | LE uint32     |
|      2 | major_version  | LE uint16     |
|      2 | minor_version  | LE uint16     |
|      * | dynamic_header | DynamicHeader |
|     32 | sha256         |               |
|     32 | hmac_sha256    |               |
|      * | payload        |               |
Table: testing foo

magic1:
: 4 unique bytes for filetype identification

magic2:
: 4 unique bytes for filetype identification

major_version:
: int representing database major version (e.g. '4')

minor_version:
: int representing database major version (e.g. '0')

header_item:
: variable length item containing information about encryption, compression, key derivation or plugin data (see Section (#HeaderItem))
          

## HeaderItem {#HeaderItem}

foobar

# KeePass Head (style 4)
   
{align="center"}
     Length    Name                  Data Type
     -------   -------------------   -------------
           4   magic1
           4   magic2                LE uint32
           2   major_version         LE uint16
           2   minor_version         LE uint16
           *   header_item           HeaderItem
          32   header_integrity  
          32   header_authenticity   
           *   payload
           
- `magic` - 4 bytes. used for filetype identification

magic1:
: 4 unique bytes for filetype identification

magic2:
: 4 unique bytes for filetype identification

major_version:
: int representing database major version (e.g. '4')

minor_version:
: int representing database major version (e.g. '0')

header_items:
: variable length item containing information about encryption, compression, key derivation or plugin data (see Section (#HeaderItem))

header_integrity:
: SHA256 hash of header for detecting unintentional corruptions.  Computed as `SHA256(m)` where m is all bytes starting from the beginning of file to the last byte of the last HeaderItem (inclusive).

header_authenticity:
: HMAC-SHA256 hash of header for verifying database authenticity.  Computed as `HMAC-SHA256(K, m)` where m is all bytes starting from the beginning of the file to the last byte of the last HeaderItem (inclusive).  K is the secret key, computed as `SHA512(\xff * 8 || SHA512(master_seed || transformed_key || \x01))` where || represents concatenation and * represents byte duplication.  master_seed and transformed_key can be found in header_items.
