### API Methods
### Introduction

The HCS vLab API provides access to resources from the virtual lab platform in a machine readable format (JSON).    At present the API supports retrieving item lists, items, documents and collection level metadata; it does not support creating item lists.  The assumption at the moment is that a user will create an item list via the Web Application and then can write code to access the item list and individual items for processing.  

### Using the API

All requests to the API need to be authenticated via an API key which can be downloaded from the Web Application (under the user menu at the top right).  The API key is specific to a user and should not generally be included in code that is published.  (We are developing a standard configuration file location that can be used to store this).  
The API Key needs to be specified as X-API-KEY in the request header.
The API uses the same URLs as the main web application in most cases but is able to return data in JSON format when an appropriate Accept header is sent with the request.  If the Accept header is set to 'application/json' or is left blank, then the response will be sent back as JSON.   Alternatively, a '.json' extension can be added to the end of the URL to force JSON format.    A web browser will generally have a very broad Accept header that will include text/html and so will get an HTML response.

**Example**

	Where <host> is the vlab host, eg http://ic2-hcsvlab-staging1-vm.intersect.org.au/
	$ curl -H "X-API-KEY: WUqhKgM25PJuzivjdvGt" <host>/item_lists.json
	$ curl -H "X-API-KEY: WUqhKgM25PJuzivjdvGt" -H "Accept: application/json" <host>item_lists

### HTTP Response Codes

<table><tbody>
<tr>
<th> Scenario </th>
<th> HTTP Response Code </th>
</tr>
<tr>
<td> User is unathenticated </td>
<td> 401 </td>
</tr>
<tr>
<td> User is unauthorised </td>
<td> 403 </td>
</tr>
<tr>
<td> User is authorised and the requested data is returned </td>
<td> 200 </td>
</tr>
<tr>
<td> User is authorised but the requested data is not found </td>
<td> 404 </td>
</tr>
<tr>
<td> Request uses bad syntax/missing required parameters </td>
<td> 400 </td>
</tr>
</tbody></table>

### API Description

<table></tbody>
<tr>
<th> Description </th>
<th> URL </th>
<th> Method </th>
<th> Returns </th>
<th> Notes </th>
</tr>
<tr> 
<td> Get API version </td>
<td> /version </td>
<td> GET </td>
<td> API version </td>
<td />
</tr>
<tr>
<td> Example Response </td>
<td colspan=4> {"API version":"V2.0"} </td>
</tr>
<tr> 
<td> Get item lists </td>
<td> /item_lists </td>
<td> GET </td>
<td> Item Lists of the user (name, url, number of items) </td>
<td />
</tr>
<tr>
<td> Example Response </td>
<td colspan=4> [
{
"name":"Jared's Item List 1",
"item_list_url":"http://ic2-hcsvlab-staging2-vm.intersect.org.au/item_lists/5",
"num_items":1
},
{
"name":"Jared's Item List 2",
"item_list_url":"http://ic2-hcsvlab-staging2-vm.intersect.org.au/item_lists/15",
"num_items":3
},
{
"name":"Jared's Item List 3",
"item_list_url":"http://ic2-hcsvlab-staging2-vm.intersect.org.au/item_lists/16",
"num_items":3
}
] </td>
</tr>
<tr> 
<td> Get Items from Item List </td>
<td> /item_lists/{id} </td>
<td> GET </td>
<td> Specified Item List of the user (name, list of items) </td>
<td>
Item list can be retrieved as JSON, ZIP or WARC format. The JSON format will only return the URL for the items in the Item list as shown below. The ZIP and WARC file will create a package containing the documents and metadata of the items in the Item list.
<br>The Zip file will respect the BagIt structure.
<br>Requests example:
<br>	curl -H "X-API-KEY: <key>" -H "Accept: application/<format>" <server>/item_lists/{id}
<br>format= json or zip or warc
</td>
</tr>
<tr>
<td> Example Response </td>
<td colspan=4> {
"name":"Jared's Item List 2",
"num_items":3,
"items":[
"http://ic2-hcsvlab-staging2-vm.intersect.org.au/catalog/hcsvlab:17862",
"http://ic2-hcsvlab-staging2-vm.intersect.org.au/catalog/hcsvlab:21953",
"http://ic2-hcsvlab-staging2-vm.intersect.org.au/catalog/hcsvlab:22175"
]
} </td>
</tr>
<tr> 
<td> Get item metadata </td>
<td> /catalog/{item_id} </td>
<td> GET </td>
<td> Item Metadata, primary text URL & document URLs </td>
<td />
</tr>
<tr>
<td> Example Response </td>
<td colspan=4> {
"catalog_url":"http://ic2-hcsvlab-staging2-vm.intersect.org.au/catalog/hcsvlab:22175",
"metadata":{
"Title":"Boy Drowns Trying to Save Pet Dog",
"Created":"8 September 1986",
"Identifier":"A08g",
"Corpus":"ace",
"Word Count":"110",
"Mode":"written",
"Language (ISO 639-3 Code)":"eng",
"Genre":"Press Reportage",
"Audience":"mass_market",
"Communication Setting":"popular",
"Publication Status":"published",
"Source":"The Sun 8 September 1986 (2001 words)",
"Written Mode":"print",
"Publisher":"Publisher - John Fairfax and Sons Ltd",
"Documents":"A08g#Original, A08g#Raw, A08g#Text",
"Type":"Original, Raw, Text",
"Extent":"579959, 739, 631",
"date_group":"1980 - 1989"
},
"primary_text_url":"http://ic2-hcsvlab-staging2-vm.intersect.org.au/catalog/hcsvlab:22175/primary_text",
"documents":[
{
"url":"http://ic2-hcsvlab-staging2-vm.intersect.org.au/catalog/hcsvlab:22175/document/ace_a.txt",
"type":"Original",
"size":null
},
{
"url":"http://ic2-hcsvlab-staging2-vm.intersect.org.au/catalog/hcsvlab:22175/document/A08g-raw.txt",
"type":"Raw",
"size":"739 B"
},
{
"url":"http://ic2-hcsvlab-staging2-vm.intersect.org.au/catalog/hcsvlab:22175/document/A08g-plain.txt",
"type":"Text",
"size":"631 B"
}
]
} </td>
</tr>
<tr> 
<td> Get primary text </td>
<td> /catalog/{item_id}/primary_text </td>
<td> GET </td>
<td> Content of item's primary_text </td>
<td> This starts a download using the primary_text content with the filename as the label attached during ingest. </td>
</tr>
<tr> 
<td> Get document </td>
<td> /catalog/{item_id}/document/{filename}
<br>Example:
<br>/catalog/hcsvlab:5031/document/MECG4MB_Sanitised-raw.txt </td>
<td> GET </td>
<td> Specified document that belongs to the Item </td>
<td> This sends the file as listed in the SOURCE metadata. 
<br>Note: The filename is being passed as a url parameter, which allows us to have the unified devise error handling that comes with a .json response </td>
</tr>
<tr> 
<td> Get annotation context </td>
<td> /schema/json-ld </td>
<td> GET </td>
<td> Context for the annotations </td>
<td> Static conext linked from the annotations API call </td>
</tr>
<tr>
<td> Example Response </td>
<td colspan=4> {
"@context":
{
"@base":"http://purl.org/dada/schema/0.2/",
"annotations":{
"@id":"http://purl.org/dada/schema/0.2/annotations",
"@container":"@list"},
"commonProperties":
{"@id":"http://purl.org/dada/schema/0.2/commonProperties"},
"type":{"@id":"http://purl.org/dada/schema/0.2/type"},
"start":{"@id":"http://purl.org/dada/schema/0.2/start"},
"end":{"@id":"http://purl.org/dada/schema/0.2/end"},
"label":{"@id":"http://purl.org/dada/schema/0.2/label"},
"annotates":{"@id":"http://purl.org/dada/schema/0.2/annotates"}
}
} </td>
</tr>
<tr> 
<td> Get annotations </td>
<td> /catalog/{item_id}/annotations
<br>/catalog/{item_id}/annotations?type=phonetic&label=s </td>
<td> GET </td>
<td> Annotations from this item's annotation file </td>
<td> Queries the item's annotation file for all annotations (label, start/end times). Type and label can be passed in to narrow the query </td>
</tr>
<tr>
<td> Example Response </td>
<td colspan=4> {
"@context":"http://localhost:3000/schema/json-ld","commonProperties":{"annotates":"http://localhost:3000/catalog/hcsvlab:4/document/S1224s1.wav"}
"annotations":
[
 {
 "@type":"MillisecondAnnotation",
"@id":"http://ns.ausnc.org.au/corpora/mitcheldelbridge/annotation/26047",
"type": "phonetic",
 "label":"s",
 "start":6.6895,
 "end":6.7145
 },
 { 
 "type": "phonetic",
 "label":"s",
 "start":7.7895,
 "end":7.8145
 }
]
} </td>
</tr>
<tr> 
<td> Get items' documents and metadata </td>
<td> /catalog/download_items </td>
<td> POST </td>
<td> Download a file in the specified format containing the item's documents and metadata </td>
<td> This is a post request that requires a JSON set of items sent with it. Hence, cannot be replicated through a browser but through curl this can be done with something akin to:
<br>curl -H "X-API-KEY: <key>" -H "Accept: application/<format>" -H "Content-Type: application/json" -X POST -d '{"items":["<host>/catalog/hcsvlab:2","<host>/catalog/hcsvlab:4"]}' <host>/catalog/download_items
<br>Allowed formats = zip, warc.
<br>If format is not specified, zip will be set by default
<br>You can also specify the file format using as follow:
<br>curl -H "X-API-KEY: <key>" -H "Content-Type: application/json" -X POST -d '{"items":["<host>/catalog/hcsvlab:2","<host>/catalog/hcsvlab:4"]}' <host>/catalog/download_items.<format>
<br>curl -H "X-API-KEY: <key>" -H "Content-Type: application/json" -X POST -d '{"items":["<host>/catalog/hcsvlab:2","<host>/catalog/hcsvlab:4"]}' <host>/catalog/download_items?format=<format>
</td>
</tr>
<tr>
<td> Example Input </td>
<td colspan=4> {
"items":[
"http://localhost:3000/catalog/hcsvlab:2253",
"http://localhost:3000/catalog/hcsvlab:2258",
"http://localhost:3000/catalog/hcsvlab:2267",
"http://localhost:3000/catalog/hcsvlab:2271"]
} </td>
</tr>
<tr> 
<td> Get collection </td>
<td> /collections/{collection_id} </td>
<td> GET </td>
<td> Collection url, name and metadata </td>
<td />
</tr>
<tr>
<td> Example Response </td>
<td colspan=4> {
"collection_url":"http://localhost:3000/collections/hcsvlab:1086",
"collection_name":"monash",
"metadata": {
 "RDF_type":"Collection",
 "Is Located At":"Monash University, Victoria, Australia",
 "Custodian":"simon Musgrave",
 "Rights":"All rights reserved to Monash UNiversity",
 "Subject":"2004 - Linguistics",
 "Title":"Monash Corpus of Spoken English"
... (other fields)
}
} </td>
</tr>
<tr> 
<td> Search Metadata </td>
<td> /catalog/search?metadata=<query>
<br>/catalog/search?metadata=HCSvLab_collection_facet:cooee </td>
<td> GET </td>
<td> List of search results </td>
<td />
</tr>
<tr>
<td> Example Response </td>
<td colspan=4> {
"num_results":4,
"items":[
"http://localhost:3000/catalog/hcsvlab:2253",
"http://localhost:3000/catalog/hcsvlab:2258",
"http://localhost:3000/catalog/hcsvlab:2267",
"http://localhost:3000/catalog/hcsvlab:2271"]
} </td>
</tr>
<tr> 
<td> Add to item list </td>
<td> /item_lists
<br>/item_lists?name=ItemListA </td>
<td> POST </td>
<td> Result of item list operation (error/success) </td>
<td> This is a post request that requires a JSON set of items sent with it. Hence, cannot be replicated through a browser but through curl this can be done with something akin to:
<br>curl -H "X-API-KEY: <key>" -H "Content-Type: application/json" -H "Accept: application/json" -X POST -d '{"num_results":2,"items":["<host>/catalog/hcsvlab:2","<host>/catalog/hcsvlab:4"]}' <host>/item_lists?name=A
<br>Note: the name doesn't have to be sent in as a parameter attached to the URL, it can also be sent in as part of the JSON
</td>
</tr>
<tr>
<td> Example Input </td>
<td colspan=4> {
"name":"New Item List",
"num_results":4,
"items":[
"http://localhost:3000/catalog/hcsvlab:2253",
"http://localhost:3000/catalog/hcsvlab:2258",
"http://localhost:3000/catalog/hcsvlab:2267",
"http://localhost:3000/catalog/hcsvlab:2271"]
}
<br>OR if name is specified as url parameter as shown in the notes above
<br>{
"items":[
"http://localhost:3000/catalog/hcsvlab:2253",
"http://localhost:3000/catalog/hcsvlab:2258",
"http://localhost:3000/catalog/hcsvlab:2267",
"http://localhost:3000/catalog/hcsvlab:2271"]
} </td>
</tr>
<tr>
<td> Example Response </td>
<td colspan=4> Success:
<br>{"success":"2 items added to existing item list A"}
<br>Failure:
<br>{"error":"name parameter not found"}
<br>{"error":"items parameter not found"}
<br>or
<br>{"error":"items parameter not an array"}
<br>or
<br>{"error":"invalid-json"} </td>
</tr>











