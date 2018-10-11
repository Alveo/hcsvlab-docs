# Voyant Server Corpus Upload

## Server URL

voyant server runs on http://mydomain.com/voyant/

So to upload file to create new corpus:

## Request

POST http://mydomain.com/voyant/trombone

*** Body ***

|parameter	|value|
|-----------|--------------------------------------------
|tool 		|corpus.CorpusCreator
|upload		|file handler (text file or zipped text file)

## Response (json)
```
{
	"version": "5.5", 
	"voyantVersion": "2.4", 
	"voyantBuild": "M5pr5", 
	"duration": 232, 
	"stepEnabledCorpusCreator": {
		"storedId": "7078cea65fa37c07bd390e8570086d9b"
	}
}
```

*storedId* is the newly created corpus ID

## Final URL

So the final corpus url is:

http://mydomain.com/voyant/?corpus=7078cea65fa37c07bd390e8570086d9b

