.ONESHELL:

.PHONY:
xml:
	mmark rfc.md > rfc.xml

.PHONY:
txt: xml
	xml2rfc --v3 rfc.xml

.PHONY:
html: xml
	xml2rfc --html --v3 rfc.xml

devserver: xml txt
	while true;
	do
		inotifywait rfc.md
		mmark rfc.md > rfc.xml
		xml2rfc --v3 rfc.xml
	done

all: xml txt
