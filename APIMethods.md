<!-- MarkdownTOC -->

- API Methods
	- Introduction
	- Using the API
	- HTTP Response Codes
	- Item List
		- Read Item Lists
		- Read Items From Item List
		- Delete Item List
		- Share Item List
		- Unshare Item List
		- Clear Item List
		- Add To Item List
		- Rename Item List
	- Item
		- Read Item Metadata
		- Update Item
		- Delete Item From Owned Collection
		- Read Primary Item Text
		- Read Items' Documents And Metadata
	- Document
		- Read Document
	- Annotation
		- Read Annotation Context
		- Read Annotations
		- Read Annotation Types
		- Upload Annotation
	- Search
		- Search Metadata
		- Query Metadata And Annotations Using SPARQL
	- Collection
		- Read Collection
		- Create Collection
		- Edit Collection
		- Add Items With Documents To An Owned Collection
		- Add Document To Owned Item
		- Delete Document From Owned Item
	- Annotation Contribution
		- Create Contribution
		- Read Contribution List
		- Read Single Contribution
		- Update Contribution
		- Add Document To Contribution
		- Delete Contribution
		- Export Contribution
	- Speaker
		- Read Speaker List
		- Create Speaker
		- Read Single Speaker
		- Update Speaker
		- Delete Speaker
	- Others
		- Read API Version
		- Read Licences

<!-- /MarkdownTOC -->


# API Methods

## Introduction

The Alveo API provides access to resources from the virtual lab platform in a machine readable format (JSON).    At present the API supports retrieving item lists, items, documents and collection level metadata; it does not support creating item lists.  The assumption at the moment is that a user will create an item list via the Web Application and then can write code to access the item list and individual items for processing.  

## Using the API

All requests to the API need to be authenticated via an API key which can be downloaded from the Web Application (under the user menu at the top right).  The API key is specific to a user and should not generally be included in code that is published.  (We are developing a standard configuration file location that can be used to store this).  
The API Key needs to be specified as X-API-KEY in the request header.
The API uses the same URLs as the main web application in most cases but is able to return data in JSON format when an appropriate Accept header is sent with the request.  If the Accept header is set to 'application/json' or is left blank, then the response will be sent back as JSON.   Alternatively, a '.json' extension can be added to the end of the URL to force JSON format.    A web browser will generally have a very broad Accept header that will include text/html and so will Read an HTML response.

**Examples**

Command line:

	Where <host> is the vlab host, e.g. https://app.alveo.edu.au/
	$ curl -H "X-API-KEY: WUqhKgM25PJuzivjdvGt" <host>/item_lists.json
	$ curl -H "X-API-KEY: WUqhKgM25PJuzivjdvGt" -H "Accept: application/json" <host>item_lists

Python
```python
import pycurl
import cStringIO
import json

api_key = <YOUR_X_API_KEY>
url = <YOUR_URL> # e.g.: https://app.alveo.edu.au/item_lists/1

curl = pycurl.Curl()
buf = cStringIO.StringIO()
curl.setopt(curl.URL, url)
curl.setopt(pycurl.HTTPHEADER, ['X-API-KEY: ' + api_key, 'Accept: application/json'])
curl.setopt(curl.WRITEFUNCTION, buf.write)
curl.perform()
response = buf.getvalue()
status = curl.getinfo(pycurl.HTTP_CODE)	
response = json.loads(string)
```

Ruby
```ruby
require 'net/http'
require 'json'

api_key = <YOUR_X_API_KEY>
url = <YOUR_URL> # e.g.: https://app.alveo.edu.au/item_lists/1

sampleUrl = URI(url)
req = Net::HTTP::Get.new(sampleUrl)
req['X-API-KEY'] = <YOUR_X_API_KEY>
req['Accept'] = "application/json"

response = Net::HTTP.start(sampleUrl.hostname, sampleUrl.port) {|http|
  http.request(req)
}

jsonResponse = JSON.parse(res.body)
```

## HTTP Response Codes

Scenario	|HTTP Response Code	
---			|:---:
User is unathenticated |401
User is unauthorised |403
User is authorised and the requested data is returned |200
User is authorised but the requested data is not found |404
Request uses bad syntax/missing required parameters |400


## Item List

### Read Item Lists
<table>
<tr>
<th> Description </th>
<th> URL </th>
<th> Method </th>
<th> Returns </th>
<th> Notes </th>
</tr>

<tr> 
<td> Read item lists </td>
<td> /item_lists </td>
<td> Read </td>
<td> Item Lists of the user (name, url, number of items) </td>
<td />
</tr>
<tr>
<td> Example Response </td>
<td colspan=4> 
<pre>
<CODE>
[
	{
		"name":"Jared's Item List 1",
		"item_list_url":"https://app.alveo.edu.au/item_lists/5",
		"num_items":1
	},
	{
		"name":"Jared's Item List 2",
		"item_list_url":"https://app.alveo.edu.au/item_lists/15",
		"num_items":3
	},
	{
		"name":"Jared's Item List 3",
		"item_list_url":"https://app.alveo.edu.au/item_lists/16",
		"num_items":3
	}
] 
</CODE>
</pre>
</td>
</tr>
</table>

### Read Items From Item List
<table>
<tr>
<th> Description </th>
<th> URL </th>
<th> Method </th>
<th> Returns </th>
<th> Notes </th>
</tr>

<tr> 
<td> Read Items from Item List </td>
<td> /item_lists/{id} </td>
<td> Read </td>
<td> Specified Item List of the user (name, list of items) </td>
<td>
Item list can be retrieved as JSON, ZIP or WARC format. The JSON format will only return the URL for the items in the Item list as shown below. The ZIP and WARC file will create a package containing the documents and metadata of the items in the Item list.
<br>The Zip file will respect the BagIt structure.
<br>Requests example:
<br>	curl -H "X-API-KEY: &lt;key&gt;" -H "Accept: application/&lt;format&gt;" &lt;server&gt;/item_lists/{id}
<br>format= json or zip or warc
</td>
</tr>
<tr>
<td> Example Response </td>
<td colspan=4>
<pre>
<CODE>
{
	"name":"Jared's Item List 2",
	"num_items":3,
	"items":[
		"https://app.alveo.edu.au/catalog/cooee/2-037",
		"https://app.alveo.edu.au/catalog/cooee/2-038",
		"https://app.alveo.edu.au/catalog/cooee/2-040"
	]
} 
</CODE>
</pre>
</td>
</tr>
</table>

### Delete Item List
<table>
<tr>
<th> Description </th>
<th> URL </th>
<th> Method </th>
<th> Returns </th>
<th> Notes </th>
</tr>

<tr> 
<td> Delete Item List </td>
<td> /item_lists/{id} </td>
<td> DELETE </td>
<td> Specified Item List of the user (name, list of items) </td>
<td>
Item list can be deleted via HTTP DELETE mehod with the same endpoint as Read items from item list
<br>Requests example:
<br>	curl -X DELETE -H "X-API-KEY: &lt;key&gt;" -H "Accept: application/json" &lt;server&gt;/item_lists/{id}
</td>
</tr>
<tr>
<td> Example Response </td>
<td colspan=4>
<pre>
<CODE>
{
	"success":"item list test deleted successfully"
} 
</CODE>
</pre>
</td>
</tr>
</table>

### Share Item List
<table>
<tr>
<th> Description </th>
<th> URL </th>
<th> Method </th>
<th> Returns </th>
<th> Notes </th>
</tr>
<tr> 
<td> Share Item List </td>
<td> /item_lists/{id}/share </td>
<td> POST </td>
<td> Specified Item List of the user (name, list of items) </td>
<td>
Item list can be shareed via HTTP POST mehod
<br>Requests example:
<br>	curl -X POST -d "" -H "X-API-KEY: &lt;key&gt;" -H "Accept: application/json" &lt;server&gt;/item_lists/{id}/share
</td>
</tr>
<tr>
<td> Example Response </td>
<td colspan=4>
<pre>
<CODE>
{
	"success":"Item list test is shared. Any user in the application will be able to see it."
} 
</CODE>
</pre>
</td>
</tr>
</table>

### Unshare Item List
<table>
<tr>
<th> Description </th>
<th> URL </th>
<th> Method </th>
<th> Returns </th>
<th> Notes </th>
</tr>
<tr> 
<td> Unshare Item List </td>
<td> /item_lists/{id}/unshare </td>
<td> POST </td>
<td> Specified Item List of the user (name, list of items) </td>
<td>
Item list can be unshareed via HTTP POST mehod
<br>Requests example:
<br>	curl -X POST -d "" -H "X-API-KEY: &lt;key&gt;" -H "Accept: application/json" &lt;server&gt;/item_lists/{id}/unshare
</td>
</tr>
<tr>
<td> Example Response </td>
<td colspan=4>
<pre>
<CODE>
{
	"success":"Item list test is not being shared anymore."
} 
</CODE>
</pre>
</td>
</tr>
</table>

### Clear Item List
<table>
<tr>
<th> Description </th>
<th> URL </th>
<th> Method </th>
<th> Returns </th>
<th> Notes </th>
</tr>
<tr> 
<td> Clear Item List </td>
<td> /item_lists/{id}/clear </td>
<td> POST </td>
<td> Specified Item List of the user (name, list of items) </td>
<td>
Item list can be unshareed via HTTP POST mehod
<br>Requests example:
<br>	curl -X POST -d "" -H "X-API-KEY: &lt;key&gt;" -H "Accept: application/json" &lt;server&gt;/item_lists/{id}/clear
</td>
</tr>
<tr>
<td> Example Response </td>
<td colspan=4>
<pre>
<CODE>
{
	"success":"2 cleared from item list test"
} 
</CODE>
</pre>
</td>
</tr>
</table>

### Add To Item List
<table>
<tr>
<th> Description </th>
<th> URL </th>
<th> Method </th>
<th> Returns </th>
<th> Notes </th>
</tr>

<tr> 
<td> Add to item list </td>
<td> /item_lists
<br>/item_lists?name=ItemListA </td>
<td> POST </td>
<td> Result of item list operation (error/success) </td>
<td> This is a post request that requires a JSON set of items sent with it. Hence, cannot be replicated through a browser but through curl this can be done with something akin to:
<br>curl -H "X-API-KEY: <key>" -H "Content-Type: application/json" -H "Accept: application/json" -X POST -d '{"num_results":2,"items":["<host>/catalog/cooee/2-015","<host>/catalog/cooee/2-021"]}' <host>/item_lists?name=A
<br>Note: the name doesn't have to be sent in as a parameter attached to the URL, it can also be sent in as part of the JSON
</td>
</tr>
<tr>
<td> Example Input </td>
<td colspan=4>
<pre>
<CODE>
{
	"name":"New Item List",
	"num_results":4,
	"items":[
		"https://app.alveo.edu.au/catalog/cooee/2-036",
		"https://app.alveo.edu.au/catalog/cooee/2-037",
		"https://app.alveo.edu.au/catalog/cooee/2-038",
		"https://app.alveo.edu.au/catalog/cooee/2-040"
	]
}
</CODE>
	<br>OR if name is specified as url parameter as shown in the notes above
<CODE>
{
	"items":[
		"https://app.alveo.edu.au/catalog/cooee/2-036",
		"https://app.alveo.edu.au/catalog/cooee/2-037",
		"https://app.alveo.edu.au/catalog/cooee/2-038",
		"https://app.alveo.edu.au/catalog/cooee/2-040"
	]
} 
</CODE>
</pre>
</td>
</tr>
<tr>
<td> Example Response </td>
<td colspan=4>
<pre>
<CODE>
Success:
	{"success":"2 items added to existing item list A"}
Failure:
	{"error":"name parameter not found"}
	or
	{"error":"items parameter not found"}
	or
	{"error":"items parameter not an array"}
	or
	{"error":"invalid-json"} 
</CODE>
</pre>
</td>
</tr>
</table>

### Rename Item List
<table>
<tr>
<th> Description </th>
<th> URL </th>
<th> Method </th>
<th> Returns </th>
<th> Notes </th>
</tr>

<tr>
<td> Rename Item List </td>
<td> /item_lists/{item_list_id} </td>
<td> PUT </td>
<td> Item list if successful/error message if failure </td>
<td> This is a put request that takes the new name of the item list. Example curl command:
<br>curl -H "X-API-KEY:&lt;api_key&gt;" -H "Accept: application/json" -X PUT -d '{"name":"&lt;new name&gt;"}' &lt;host&gt;/item_lists/:id
</td>
</tr>
<tr>
<td> Example Response </td>
<td colspan=4>
<pre>
<CODE>
Success:
{
	"name":"Jared's Item List 2",
	"num_items":3,
	"items":[
		"https://app.alveo.edu.au/catalog/cooee/2-037",
		"https://app.alveo.edu.au/catalog/cooee/2-038",
		"https://app.alveo.edu.au/catalog/cooee/2-040"
	]
} 
Failure:
	{"error":"name can't be blank"}
	or
	{"error":"name too long"}
	or
	{"error":"couldn't rename item list"}
</CODE>
</pre>
</td>
</tr>
</table>

## Item

### Read Item Metadata
<table>
<tr>
<th> Description </th>
<th> URL </th>
<th> Method </th>
<th> Returns </th>
<th> Notes </th>
</tr>

<tr> 
<td> Read item metadata </td>
<td> /catalog/{collection_name}/{item_id} </td>
<td> Read </td>
<td> Item Metadata, primary text URL & document URLs </td>
<td />
</tr>
<tr>
<td> Example Response </td>
<td colspan=4>
<pre>
<CODE>
{
	"@context": "https://app.alveo.edu.au/schema/json-ld",
	"alveo:catalog_url": "https://app.alveo.edu.au/catalog/ace/A08g",
	"alveo:metadata": {
		"dcterms:title": "Boy Drowns Trying to Save Pet Dog",
		"dcterms:created": "8 September 1986",
		"dcterms:identifier": "A08g",
		"dcterms:isPartOf": "ace",
		"ausnc:itemwordcount": "110",
		"ausnc:mode": "written",
		"ausnc:speech_style": "unspecified",
		"ausnc:interactivity": "unspecified",
		"ausnc:communication_context": "unspecified",
		"olac:discourse_type": "unspecified",
		"olac:language": "eng",
		"ace:genre": "Press Reportage",
		"ausnc:audience": "mass_market",
		"ausnc:communication_setting": "popular",
		"ausnc:communication_medium": "unspecified",
		"ausnc:publication_status": "published",
		"ausnc:source": "The Sun 8 September 1986 (2001 words)",
		"ausnc:written_mode": "print",
		"dcterms:publisher": "Publisher - John Fairfax and Sons Ltd",
		"ausnc:document": "A08g#Text, A08g#Original, A08g#Raw",
		"dcterms:type": "Text, Original, Raw",
		"dcterms:extent": "631, 579959, 739",
		"alveo:date_group": "1980 - 1989",
		"alveo:display_document": "A08g#Text",
		"alveo:indexable_document": "A08g#Text",
		"alveo:full_text": "BOY DROWNS TRYING TO SAVE PET DOG A BOY drowned...",
		"alveo:handle": "ace:A08g",
		"alveo:sparqlEndpoint": "https://app.alveo.edu.au/sparql/ace"
	},
	"alveo:primary_text_url": "https://app.alveo.edu.au/catalog/ace/A08g/primary_text.json",
	"alveo:annotations_url": "https://app.alveo.edu.au/catalog/ace/A08g/annotations.json",
	"alveo:documents": [{
		"alveo:url": "https://app.alveo.edu.au/catalog/ace/A08g/document/A08g-plain.txt",
		"dcterms:type": "Text",
		"alveo:size": "631 B",
		"rdf:type": "http://xmlns.com/foaf/0.1/Document",
		"dcterms:identifier": "A08g-plain.txt",
		"dcterms:title": "A08g#Text",
		"dcterms:extent": "631",
		"dcterms:type": "Text"
	}, {
		"alveo:url": "https://app.alveo.edu.au/catalog/ace/A08g/document/ace_a.txt",
		"dcterms:type": "Original",
		"alveo:size": null,
		"rdf:type": "http://xmlns.com/foaf/0.1/Document",
		"dcterms:identifier": "ace_a.txt",
		"dcterms:title": "A08g#Original",
		"dcterms:extent": "579959",
		"dcterms:type": "Original"
	}, {
		"alveo:url": "https://app.alveo.edu.au/catalog/ace/A08g/document/A08g-raw.txt",
		"dcterms:type": "Raw",
		"alveo:size": "739 B",
		"rdf:type": "http://xmlns.com/foaf/0.1/Document",
		"dcterms:identifier": "A08g-raw.txt",
		"dcterms:title": "A08g#Raw",
		"dcterms:extent": "739",
		"dcterms:type": "Raw"
	}]
}
</CODE>
</pre>
</td>
</tr>
</table>

### Update Item
<table>
<tr>
<th> Description </th>
<th> URL </th>
<th> Method </th>
<th> Returns </th>
<th> Notes </th>
</tr>
<tr>
<td>Update item</td>
<td>/catalog/{collection_name}/{item_name}</td>
<td>PUT</td>
<td>Result of operation (error/success)</td>
<td>Notes:
<ol>
<li>Users are only authorised to update an item from collection which they own.</li>
<li>This is a PUT request. Hence, cannot be replicated through a browser but through curl this can be done with something akin to the following.
<ul>
<li><code>curl -H "X-API-KEY: &lt;key&gt;" -H "Content-Type: application/json" -H "Accept: application/json" -X PUT -d '{"metadata": &lt;item_metadata&gt;}' &lt;server&gt;/catalog/&lt;collection_id&gt;/&lt;item_id&gt;</code></li>
</ul>
</li>
</ol>
</td>
</tr>
<tr>
<td>Example Input</td>
<td colspan=4> 
The following is an example of expected input for &lt;item_metadata&gt;:
<ul>
<li><code>{"http://ns.ausnc.org.au/schemas/ausnc_md_model/mode":"An updated test mode"}</code></li>
<li><code>{"@context":{"ausnc":"http://ns.ausnc.org.au/schemas/ausnc_md_model/"}, "ausnc:speech_style":"An updated speech style"}</code></li>
</ul>
</td>
</tr>
<tr>
<td>Example Response</td>
<td colspan=4> 
<br>Success:
<br>{"success":"Updated item &lt;item_id&gt; in collection &lt;collection_id&gt;"}
<br>Failure:
<br>{"error":"Requested collection not found"}
<br>or
<br>{"error":"Requested item not found"}
<br>or
<br>{"error":"User is unauthorised"}
<br>or
<br>{"error":"Invalid metadata"}
</td>
</tr>
</table>

### Delete Item From Owned Collection
<table>
<tr>
<th> Description </th>
<th> URL </th>
<th> Method </th>
<th> Returns </th>
<th> Notes </th>
</tr>
<tr>
<td>Delete item from owned collection</td>
<td>/catalog/{collection_id}/{item_id}</td>
<td>DELETE</td>
<td>Result of operation (error/success)</td>
<td>Notes:
<ol>
<li>Users are only authorised to delete an item from collection which they own.</li>
<li>Deleting an item from a collection also deletes all of that items documents and the corresponding document audits.</li>
<li>This is a DELETE request. Hence, cannot be replicated through a browser but through curl this can be done with something akin to the following.
<ul>
<li><code>curl -H "X-API-KEY: &lt;api_key&gt;" -H "Accept: application/json" -X DELETE &lt;server&gt;/catalog/&lt;collection_id&gt;/&lt;item_id&gt;</code></li>
</ul>
</li>
</ol>
</td>
</tr>
<tr>
<td>Example Response</td>
<td colspan=4> 
<br>Success:
<br>{"success":"Deleted the item &lt;item_id&gt; (and its documents) from collection &lt;collection_id&gt;"}
<br>Failure:
<br>{"error":"User is unauthorised"}
<br>or
<br>{"error":"Requested collection not found"}
<br>or
<br>{"error":"Requested item not found"}
</td>
</tr>
</table>

### Read Primary Item Text
<table>
<tr>
<th> Description </th>
<th> URL </th>
<th> Method </th>
<th> Returns </th>
<th> Notes </th>
</tr>

<tr> 
<td> Read primary item text </td>
<td> /catalog/{collection_name}/{item_id}/primary_text </td>
<td> Read </td>
<td> Content of item's primary_text </td>
<td> This starts a download using the primary_text content with the filename as the label attached during ingest. </td>
</tr>
</table>

### Read Items' Documents And Metadata
<table>
<tr>
<th> Description </th>
<th> URL </th>
<th> Method </th>
<th> Returns </th>
</tr>
<tr>
<td> Read items' documents and metadata </td>
<td> /catalog/download_items </td>
<td> POST </td>
<td> Download a file in the specified format containing the item's documents and metadata </td>
</tr>
<tr>
<td></td>	
<td colspan="4"> 
<ul>
<li>This is a post request that requires a JSON set of items sent with it. Hence, cannot be replicated through a browser but through curl this can be done with something akin to:
<br><code>curl -H "X-API-KEY: &lt;key&gt;" -H "Accept: application/&lt;format&gt;" -H "Content-Type: application/json" -X POST -d '{"items":["&lt;host&gt;/catalog/cooee/2-015","&lt;host&gt;/catalog/cooee/2-021"]}' &lt;host&gt;/catalog/download_items</code>
</li>
<li>Allowed formats = zip, warc.
<br>If format is not specified, zip will be set by default
<br>You can also specify the file format using as follow:
<br><code>curl -H "X-API-KEY: &lt;key&gt;" -H "Content-Type: application/json" -X POST -d '{"items":["&lt;host&gt;/catalog/cooee/2-015","&lt;host&gt;/catalog/cooee/2-021"]}' &lt;host&gt;/catalog/download_items.&lt;format&gt;</code>
<br><code>curl -H "X-API-KEY: &lt;key&gt;" -H "Content-Type: application/json" -X POST -d '{"items":["&lt;host&gt;/catalog/cooee/2-015","&lt;host&gt;/catalog/cooee/2-021"]}' &lt;host&gt;/catalog/download_items?format=&lt;format&gt;</code>
</li>
<li>To save the retrieved zip file the API call output needs to be redirected to a file. This can be achieved using the '&gt;' character at the end of the API call similar to the following:
<br><code>"X-API-KEY: &lt;key&gt;" -H "Accept: application/&lt;format&gt;" -H "Content-Type: application/json" -X POST -d '{"items":["&lt;host&gt;/catalog/cooee/2-015","&lt;host&gt;/catalog/cooee/2-021"]}' &lt;host&gt;/catalog/download_items &gt; items.zip</code>
</li>
<li>A document filter can be applied to limit the type of item documents downloaded. This is used by providing a glob to the JSON parameter <code>doc_filter</code>. If no glob filter is provided then this API call will download all item documents by default.</li>
<li>Note that Glob is used for filtering rather than Regular Expressions. This affects usage as the syntax for glob and regex are similar but different (i.e. a valid regex is not always a valid glob).
Basic glob syntax can be found on wikipedia: https://en.wikipedia.org/wiki/Glob_(programming)
The exact syntax as used in implementation: http://ruby-doc.org/core-2.2.0/File.html#method-c-fnmatch
Some example globs are:
*.wav
*.txt
*-raw.txt
myDocument.*
*.{wav,mp3}
</li>
</td>
</tr>
<tr>
<td> Example Input </td>
<td colspan=4>
<pre>
<CODE>
{
	"items":[
		"https://app.alveo.edu.au/catalog/cooee/2-036",
		"https://app.alveo.edu.au/catalog/cooee/2-037",
		"https://app.alveo.edu.au/catalog/cooee/2-038",
		"https://app.alveo.edu.au/catalog/cooee/2-040"
	],
	"doc_filter":"*.txt"
} 
</CODE>
<li>Another way to download, using Aspera Connect secure download, (Aspera will prompt user to download Aspera Connect if user doesn't already have it)
Example Aspera transfer download API call:
</li>
<code>
curl -H "X-API-KEY: &lt;key&gt;" -H "Content-Type: application/json" -X POST -d '{"doc_filter":"*.txt"}' &lt;serverurl&gt;/item_lists/3/aspera_transfer_spec
</code>
</pre>
</td>
</tr>
</table>

## Document

### Read Document
<table>
<tr>
<th> Description </th>
<th> URL </th>
<th> Method </th>
<th> Returns </th>
<th> Notes </th>
</tr>

<tr> 
<td> Read document </td>
<td> /catalog/{collection_name}/{item_id}/document/{filename}
<br>Example:
<br>/catalog/ace/A08g/document/A08g-raw.txt </td>
<td> Read </td>
<td> Specified document that belongs to the Item </td>
<td> This sends the file as listed in the SOURCE metadata. 
<br>Note: The filename is being passed as a url parameter, which allows us to have the unified devise error handling that comes with a .json response </td>
</tr>
</table>

## Annotation

### Read Annotation Context
<table>
<tr>
<th> Description </th>
<th> URL </th>
<th> Method </th>
<th> Returns </th>
<th> Notes </th>
</tr>
<tr> 
<td> Read annotation context </td>
<td> /schema/json-ld </td>
<td> Read </td>
<td> Context for the annotations </td>
<td> Static conext linked from the annotations API call </td>
</tr>
<tr>
<td> Example Response </td>
<td colspan=4>
<pre>
<CODE>
{
	"@context": {
		"@base":"http://purl.org/dada/schema/0.2/",
		"annotations":{
			"@id":"http://purl.org/dada/schema/0.2/annotations",
			"@container":"@list"
		},
		"commonProperties": {"@id":"http://purl.org/dada/schema/0.2/commonProperties"},
		"type":{"@id":"http://purl.org/dada/schema/0.2/type"},
		"start":{"@id":"http://purl.org/dada/schema/0.2/start"},
		"end":{"@id":"http://purl.org/dada/schema/0.2/end"},
		"label":{"@id":"http://purl.org/dada/schema/0.2/label"},
		"annotates":{"@id":"http://purl.org/dada/schema/0.2/annotates"}
	}
}
</CODE>
</pre>
</td>
</tr>
</table>

### Read Annotations
<table>
<tr>
<th> Description </th>
<th> URL </th>
<th> Method </th>
<th> Returns </th>
<th> Notes </th>
</tr>

<tr> 
<td> Read annotations </td>
<td>/catalog/{collection_name}/{item_id}/annotations
<br>/catalog/{collection_name}/{item_id}/annotations.json?type=maus:phonetic&label=s</td>
<td> Read </td>
<td> Annotations from this item's annotation file </td>
<td> Queries the item's annotation file for all annotations (label, start/end times). Type and label can be passed in to narrow the query </td>
</tr>
<tr>
<td> Example Response </td>
<td colspan=4>
<pre>
<CODE>
{
	@context: "https://app.alveo.edu.au/schema/json-ld",
	commonProperties: {
		alveo: annotates: "https://app.alveo.edu.au/catalog/mitcheldelbridge/S1224s1/document/S1224s1.wav"
	},
	alveo: annotations: [{@
		id: "http://ns.ausnc.org.au/corpora/mitcheldelbridge/annotation/302021",
		label: "s",
		type: "http://ns.ausnc.org.au/schemas/annotation/maus/phonetic",
		@type: "dada:SecondAnnotation",
		end: "2.21",
		start: "2.18"
	}, {@
		id: "http://ns.ausnc.org.au/corpora/mitcheldelbridge/annotation/302024",
		label: "s",
		type: "http://ns.ausnc.org.au/schemas/annotation/maus/phonetic",
		@type: "dada:SecondAnnotation",
		end: "3.31",
		start: "3.28"
	}]
}
</CODE>
</pre>
</td>
</tr>
</table>

### Read Annotation Types
<table>
<tr>
<th> Description </th>
<th> URL </th>
<th> Method </th>
<th> Returns </th>
<th> Notes </th>
</tr>

<tr> 
<td> Read annotation types </td>
<td> /catalog/{collection_name}/{item_id}/annotations/types
<td> Read </td>
<td> Annotation types from this item's annotations </td>
<td> Queries the item's annotations to return all the different annotation types for the item </td>
</tr>
<tr>
<td> Example Response </td>
<td colspan=4>
<pre>
<code>	
{
  item_url: "https://app.alveo.edu.au/catalog/cooee:4-340",
  annotation_types: [
    "http://ns.ausnc.org.au/schemas/annotation/cooee/ellipsis",
    "http://ns.ausnc.org.au/schemas/annotation/cooee/pageno"
  ]
}
</code>
</pre>
</td>
</tr>
</table>

### Upload Annotation
<table>
<tr>
<th> Description </th>
<th> URL </th>
<th> Method </th>
<th> Returns </th>
<th> Notes </th>
</tr>

<tr> 
	<td> Upload Annotation </td>
	<td> /catalog/{collection_name}/{item_id}/annotations</td>
	<td> POST </td>
	<td> Result of item list operation (error/success) </td>
	<td> This is a post request that requires a file sent with it. Hence, cannot be replicated through a browser but through curl this can be done with something akin to:
<br>curl -H "X-API-KEY:&lt;api_key&gt;" -H "Accept: application/json" -F file=@&lt;path_to_file&gt; &lt;host&gt;/catalog/:id/annotations
	</td>
</tr>
<tr>
	<td> Json-Ld Input File format </td>
	<td colspan=4>
	<pre>
<CODE>
{
  "@context": {
	"@base": "http://purl.org/dada/schema/0.2/",
	"&lt;prefix&gt;": {
    		"@id": "&lt;URI&gt;"
	},
	"&lt;prefix&gt;": {
    		"@id": "&lt;URI&gt;"
	},
	.
	.
	.
	"&lt;prefix&gt;": {
    		"@id": "&lt;URI&gt;"
	},
  },
  "@graph":[
    {
        "@type": "http://purl.org/dada/schema/0.2#UTF8Region",
        "&lt;property_name&gt;": "value",
        "&lt;property_name&gt;": "value",
        .
        .
        .
        "&lt;property_name&gt;": "value",
    },
    {
        "@type": "http://purl.org/dada/schema/0.2#UTF8Region",
        "&lt;property_name&gt;": "value",
        "&lt;property_name&gt;": "value",
        .
        .
        .
        "&lt;property_name&gt;": "value",
    },
    .
    .
    .
    {
        "@type": "http://purl.org/dada/schema/0.2#UTF8Region",
        "&lt;property_name&gt;": "value",
        "&lt;property_name&gt;": "value",
        .
        .
        .
        "&lt;property_name&gt;": "value",
    }
  ]
}
</CODE>
	</pre>
	</td>
</tr>
<tr>
	<td> Example Response </td>
	<td colspan=4>	
	<pre>
<CODE>
Success:
	{"success":"file &lt;filename&gt; uploaded successfully"}
Failure:
	{"error":"No Item with id '&lt;item_id&gt;' exists."}
	or
	{"error":"Uploaded file is not present or empty."}
	or
	{"error":"File already uploaded."}
	or
	{"error":"Error uploading file &lt;filename&gt;."}
</CODE>
	</pre>
	</td>
</tr>
</table>




## Search

### Search Metadata
<table>
<tr>
<th> Description </th>
<th> URL </th>
<th> Method </th>
<th> Returns </th>
<th> Notes </th>
</tr>

<tr> 
<td> Search Metadata </td>
<td> 
	/catalog/search?metadata=&lt;query&gt;
	<br>Example:<br>
	/catalog/search?metadata=collection_name:cooee 
</td>
<td> Read </td>
<td> List of search results </td>
<td />
</tr>
<tr>
<td> Example Response </td>
<td colspan=4>
<pre>
<CODE>
{
	"num_results":4,
	"items":[
		"https://app.alveo.edu.au/catalog/cooee/2-036",
		"https://app.alveo.edu.au/catalog/cooee/2-037",
		"https://app.alveo.edu.au/catalog/cooee/2-038",
		"https://app.alveo.edu.au/catalog/cooee/2-040"
	]
}
</CODE>
</pre>
</td>
</tr>
</table>


### Query Metadata And Annotations Using SPARQL
<table>
<tr>
<th> Description </th>
<th> URL </th>
<th> Method </th>
<th> Returns </th>
<th> Notes </th>
</tr>


<tr> 
	<td> Query metadata and annotations using sparql</td>
	<td> /sparql/&lt;collection-name&gt;?query=&lt;sparql-query&gt;</td>
	<td> Read </td>
	<td> Json formatted query result </td>
	<td> 
		curl -g -H "X-API-KEY: &lt;API_KEY&gt;" -H "Accept: application/json" "&lt;host&gt;/sparql/&lt;collection-name&gt;?query=&lt;sparql-query&gt;"
		<br>
		Example:<br>
		curl -g -H "X-API-KEY: &lt;API_KEY&gt;" -H "Accept: application/json" "&lt;host&gt;/sparql/cooee?query=select * where {?s &lt;http://purl.org/dc/terms/isPartOf&gt; ?o}"  (e.g.curl -H "X-API-KEY:<APIkey>" -H "Content-Type: application/json" -H "Accept: application/json" https://<serverurl>/licences
	</td>
</tr>
<tr>
	<td> Example Response </td>
	<td colspan=4>	
	<pre>
<CODE>
{
  "head" : {
    "vars" : [ "s", "o" ]
  },
  "results" : {
    "bindings" : [{
      "s" : {
        "type" : "uri",
        "value" : "http://ns.ausnc.org.au/corpora/cooee/items/1-001"
      },
      "o" : {
        "type" : "uri",
        "value" : "http://ns.ausnc.org.au/corpora/cooee"
      }
    }, {
      "s" : {
        "type" : "uri",
        "value" : "http://ns.ausnc.org.au/corpora/cooee/items/1-002"
      },
      "o" : {
        "type" : "uri",
        "value" : "http://ns.ausnc.org.au/corpora/cooee"
      }
    }, {
      "s" : {
        "type" : "uri",
        "value" : "http://ns.ausnc.org.au/corpora/cooee/items/1-003"
      },
      "o" : {
        "type" : "uri",
        "value" : "http://ns.ausnc.org.au/corpora/cooee"
      }
    }]
  }
}
</CODE>
	</pre>
	</td>
</tr>
</table>

## Collection

### Read Collection
<table>
<tr>
<th> Description </th>
<th> URL </th>
<th> Method </th>
<th> Returns </th>
<th> Notes </th>
</tr>

<tr> 
<td> Read collection </td>
<td> /catalog/{collection_name} </td>
<td> Read </td>
<td> Collection url, name and metadata </td>
<td />
</tr>
<tr>
<td> Example Response </td>
<td colspan=4>
<pre>
<CODE>
{
	"collection_url":"https://app.alveo.edu.au/catalog/monash",
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
}
</CODE>
</pre>
</td>
</tr>
</table>

### Create Collection
<table>
<tr>
<th> Description </th>
<th> URL </th>
<th> Method </th>
<th> Returns </th>
</tr>
<tr>
<td>Create collection</td>
<td>/catalog</td>
<td>POST</td>
<td>Result of operation (error/success)</td>
</tr>
<tr>
<td>Notes</td>	
<td colspan="4">
<ol>
<li>Users are only authorised to create a collection if they have the role of 'admin' or 'data owner'.</li>
<li>The collection name can either be supplied as a JSON parameter or as a URL paramaeter. See the example input for examples of this. NOTE: Collection names currently can not contain a colon (<code>:</code>) in the name as it causes an error. This is a known issue which will be dealt  with in subsequent releases.</li>
<li>A licence can be assigned to the collection by passing in the <code>licence_id</code> parameter. As with the collection name, this can either be supplied as a JSON parameter or as a URL parameter.</li>
<li>Collection privacy can be set to change whether or not approval is required for other users to access the collection, by passing in the <code>private</code> parameter set to either <code>true</code> or <code>false</code>. By default this is set to true so that approval is required for collection access. As with the collection name and licence, this can either be supplied as a JSON parameter or as a URL parameter.
<li>This is a POST request that requires a JSON-LD set of collection metadata to be sent with it.
Hence, cannot be replicated through a browser but through curl this can be done with something akin to the following:
<ul>
<li>When creating a collection with the collection name supplied as a URL parameter:
<br><code>curl -X POST -H "X-API-KEY: &lt;key&gt;" -H "Content-Type: application/json" -H "Accept: application/json" -d '{ "collection_metadata": &lt;collection_metadata&gt;}' '&lt;server&gt;/catalog?name=&lt;collection_name&gt;&licence_id=1&private=true'</code>
</li>
<li>When creating a collection with the collection name supplied as a JSON parameter:
<br><code>curl -X POST -H "X-API-KEY: &lt;key&gt;" -H "Content-Type: application/json" -H "Accept: application/json" -d '{ "collection_metadata": &lt;collection_metadata&gt;, "name":"&lt;collection_name&gt;", "licence_id":1, "private":true}' &lt;server&gt;/catalog</code>
</li>
</ul>
</li>
</ol>
</td>
</tr>
<tr>
<td>Example Input</td>
<td colspan=4> 
The following is an example of expected input for &lt;collection_metadata&gt;:
<pre>
<code>
{ 
	"@context": { 
		"dc": "http://purl.org/dc/terms/", 
		"dcterms": "http://purl.org/dc/terms/", 
		"dcmitype": "http://purl.org/dc/dcmitype/" 
	}, 
	"@type": "dcmitype:Collection", 
	"dcterms:creator": "Data Owner", 
	"dcterms:rights": "All rights reserved to Data Owner", "dcterms:subject": "English Language", 
	"dcterms:title": "Test" 
}
</code>
</pre>
</td>
</tr>
<tr>
<td>Example Response</td>
<td colspan=4> 
<br>Success:
<br>{"success":"New collection '&lt;collection_name&gt;' (&lt;collection_URI&gt;) created"}
<br>Failure:
<br>{"error":"Permission Denied: Your role within the system does not have sufficient privileges to be able to create a collection. Please contact an Alveo administrator."}
<br>or
<br>{"error":"name parameter not found"}
<br>or
<br>{"error":"metatdata parameter not found"}
<br>or
<br>{"error":"name and metadata parameters not found"}
<br>or
<br>{"error":"invalid-json"}
<br>or
<br>{"error":"JSON-LD formatted metadata must be sent to the add collection api call as a POST request"}
<br>or
<br>{"error":"Collection '&lcollection_namet&gt;' (&lt;collection_URI&gt;) already exists in the system - skipping"}
</td>
</tr>
</table>

### Edit Collection
<table>
<tr>
<th> Description </th>
<th> URL </th>
<th> Method </th>
<th> Returns </th>
</tr>
<tr>
<td>Edit Collection</td>
<td>/catalog/{collection_name}</td>
<td>PUT</td>
<td>Result of operation (error/success)</td>
</tr>
<tr>
<td>Notes</td>
<td colspan="4">
<ol>
<li>Users are only authorised to edit a collection which they own.</li>
<li>It is not neccessary to include the collection URI within the JSON-LD metadata <code>("@id":"&lt;collection_uri&gt;")</code> as this will be automatically generated.</li>
<li>The "replace" JSON parameter indicates whether or not the provided metadata should replace the existing collection metadata. 
<ul>
<li>If replace is true then the entire existing collection metadata (including the context) will be replaced with the provided metadata.</li>
<li>If replace is false or unsupplied then the collection metadata will be updated with the provided metadata.</li>
</ul>
</li>
<li>This is a PUT request. Hence, cannot be replicated through a browser but through curl this can be done with something akin to the following.
<ul>
<li><code>curl -H "X-API-KEY: &lt;key&gt;"  -H "Content-Type: application/json" -H "Accept: application/json" -X PUT -d '{ "replace": &lt;true/false&gt;, "collection_metadata": &lt;collection_metadata&gt;}' &lt;server&gt;/catalog/&lt;collection_id&gt;</code></li>
</ul>
</li>
</ol>
</td>
</tr>
<tr>
<td>Example Input</td>
<td colspan=4> 
<ul>
<li>When updating collection metadata without specifying the "replace" parameter: <br><code>{ "collection_metadata": {"http://purl.org/dc/elements/1.1/title": "An updated test collection"}}</code></li>
<li>When updating collection metadata by setting the "replace" parameter to false: <br><code>{ "replace": false, "collection_metadata": {"http://purl.org/dc/elements/1.1/title": "An updated test collection"}}</code></li>
<li>When overwriting collection metadata by setting the "replace" parameter to true: <br><code>{ "replace": true, "collection_metadata": {"@context": {"dc": "http://purl.org/dc/elements/1.1/", "dcmitype": "http://purl.org/dc/dcmitype/"}, "@type": "dcmitype:Collection", "dcterms:title": "An updated test collection", "dcterms:subject": "English Language", "dcterms:creator": "Test User"}}</code></li>
</ul>
</td>
</tr>
<tr>
<td>Example Response</td>
<td colspan=4> 
<br>Success:
<br>{"success":"Updated collection &lt;collection_name&gt;"}
<br>Failure:
<br>{"error":"Requested collection not found"}
<br>or
<br>{"error":"User is unauthorised"}
<br>or
<br>{"error":"Invalid metadata"}
</td>
</tr>
</table>

### Add Items With Documents To An Owned Collection
<table>
<tr>
<th> Description </th>
<th> URL </th>
<th> Method </th>
<th> Returns </th>
</tr>
<tr>
<td>Add items with documents to an owned collection</td>
<td>/catalog/{collection_id}</td>
<td>POST</td>
<td>Result of operation (error/success) and a list of item ids for each item added</td>
</tr>
<tr>
	<td>Notes</td>
<td colspan="4">
<ol>
<li>Users are only authorised to add items to a collection which they own.</li>
<li>The expected format for adding a document is to have the metadata for that document nested within the item level metadata. See the example input for examples of this.</li>
<li>When adding items or documents, the "@id" that specifies the URI of the item/document needs to be supplied. This  will then be automatically converted into Alveo catalog URLs. For example:
<br>When adding an item to a test collection if the user specifies <code>"@id":"item1"</code> then the system will convert this into <code>"@id":"http://app.avleo.edu.au/catalog/test/item1"</code>. 
<br>Similarly a document with <code>"@id":"document1"</code> will be converted to <code>"@id":"http://app.alveo.edu.au/catalog/test/item1/document/document1"</code>.
</li>
<li>The document metadata term <code>"dcterms:source":{"@id":"&lt;file_or_http_uri&gt;"}</code> does not need to be provided for any documents whose contents are embedded in the JSON item metadata or are uploaded as part of the HTTP request. This is since the server will assign a location for uploaded documents and modify the aforementioned metadata document source term appropriately. However it is essential to include the document source metadata term for any documents referenced with "file://" or "http://".</li>
<li>For document files referenced with "file://" or "http://" the file basename (filename and extension) must match the <code>"dcterms:identifier"</code> of that document. For example, if referencing the file <code>"dcterms:source":{"@id":"file:///data/directory/example.txt"}</code> then it is expected that the identifier of that document will be <code>"dcterms:identifier":"example.txt"</code></li>
<li>For document files uploaded as part of the HTTP request or whose content is embedded in the JSON item metadata, for that document content or file to be associated with the metadata of a particular document the value given to the <code>"dcterms:identifier"</code> within the document metadata needs to match the name of the uploaded file or identifier of the embedded document content. For example: <br>a document with metadata containing <code>"dcterms:identifier":"sample.txt"</code> will be associated with the uploaded file "sample.txt".</li>
<li>The <code>"dcterms:isPartOf":{"@id":"corpus:&lt;collection_name&gt;"}</code> should not be supplied when adding an item to a collection. This metadata will be automatically generated and will correspond with the collection that the item is being added to. If this metadata is supplied it will be overwritten when the system generates it for the corresponding corpus.</li>
<li>If an item being added contains invalid metadata that item will not be added to the collection but all other valid items shall  continue to be added to the collection. The API response will contain a list of ids of each item that was successfully added to the collection.</li>
<li>
This is a POST request that requires a JSON-LD set of collection metadata to be sent with it.
Hence, cannot be replicated through a browser but through curl this can be done with something akin to the following:
<ul>
<li>If adding an item with document(s) that are referenced (with "file://" or "http://"):
<br><code>curl -X POST -H "X-API-KEY: &lt;key&gt;" -H "Accept: application/json" -H "Content-Type: application/json"  -d '{"items":[{"metadata":{&lt;item_metadata&gt;}'}]}' &lt;server&gt;/catalog/&lt;collection_id&gt;</code>
</li>
<li>If adding an item with document(s) whose contents are embedded in the JSON item metadata:
<br><code>curl -X POST -H "X-API-KEY: &lt;key&gt;" -H "Accept: application/json" -H "Content-Type: application/json"  -d '{"items":[{"documents":[{&lt;document_metadata&gt;}], "metadata":{&lt;item_metadata&gt;}'}]}' &lt;server&gt;/catalog/&lt;collection_id&gt;</code>
</li>
<li>If adding an item with a single document uploaded as part of the HTTP request:
<br><code>curl -X POST -H "X-API-KEY: &lt;key&gt;" -H "Accept: application/json" -F file=@"&lt;file_location&gt;" -F items='[{&lt;item_metadata&gt;}]' &lt;server&gt;/catalog/&lt;collection_id&gt;</code>
</li>
<li>If adding an item with multiple documents uploaded as part of the HTTP request:
<br><code>curl -X POST -H "X-API-KEY: &lt;key&gt;" -H "Accept: application/json" -F file[]=@"&lt;file_1_location&gt;" -F file[]=@"&lt;file_2_location&gt;" -F items='[{&lt;item_metadata&gt;}]' &lt;server&gt;/catalog/&lt;collection_id&gt;</code>
</li>
</ul>
</li>
</ol>
</td>
</tr>
<td>Example Input</td>
<td colspan=4> 
<ul>
<li>If adding an item with document(s) that are referenced (with "file://" or "http://"):
<br><code>-d '{ "items": [ { "metadata": { "@context": { "ausnc": "http://ns.ausnc.org.au/schemas/ausnc_md_model/", "corpus": "http://ns.ausnc.org.au/corpora/", "dc": "http://purl.org/dc/terms/", "dcterms": "http://purl.org/dc/terms/", "foaf": "http://xmlns.com/foaf/0.1/", "hcsvlab": "http://hcsvlab.org/vocabulary/" }, "@graph": [ { "@id": "item1", "@type": "ausnc:AusNCObject", "ausnc:document": [ { "@id": "document1", "@type": "foaf:Document", "dcterms:extent": 1234, "dcterms:identifier": "document1.txt", "dcterms:source": { "@id": "file:///data/test_collections/ausnc/test/document2.txt" }, "dcterms:title": "document1#Text", "dcterms:type": "Text" } ], "dcterms:identifier": "item1", "hcsvlab:indexable_document": { "@id": "document1.txt" }, "hcsvlab:display_document": { "@id": "document1.txt" } } ] } } ] }'</code>
</li>
<li>If adding an item with document(s) whose contents are embedded in the JSON item metadata:
<br><code>-d '{"items": [ { "documents": [ { "identifier": "document1.txt", "content": "This document had its content provided as part of the JSON request." } ], "metadata": { "@context": { "ausnc": "http://ns.ausnc.org.au/schemas/ausnc_md_model/", "corpus": "http://ns.ausnc.org.au/corpora/", "dc": "http://purl.org/dc/terms/", "dcterms": "http://purl.org/dc/terms/", "foaf": "http://xmlns.com/foaf/0.1/", "hcsvlab": "http://hcsvlab.org/vocabulary/" }, "@graph": [ { "@id": "item1", "@type": "ausnc:AusNCObject", "ausnc:document": [ { "@id": "document1.txt", "@type": "foaf:Document", "dcterms:extent": 72636, "dcterms:identifier": "document1.txt", "dcterms:title": "document1#Text", "dcterms:type": "Text" } ], "dcterms:identifier": "item1", "hcsvlab:display_document": { "@id": "document1.txt" }, "hcsvlab:indexable_document": { "@id": "document1.txt" } } ] } } ] }'
</code>
</li>
<li>If adding an item with a single document uploaded as part of the HTTP request:
<br><code>-F file=@"1-001-plain.txt" -F items='[ { "metadata": { "@context": { "ausnc": "http://ns.ausnc.org.au/schemas/ausnc_md_model/", "corpus": "http://ns.ausnc.org.au/corpora/", "dc": "http://purl.org/dc/terms/", "dcterms": "http://purl.org/dc/terms/", "foaf": "http://xmlns.com/foaf/0.1/", "hcsvlab": "http://hcsvlab.org/vocabulary/" }, "@graph": [ { "@id": "item1", "@type": "ausnc:AusNCObject", "ausnc:document": [ { "@id": "1-001-plain.txt", "@type": "foaf:Document", "dcterms:extent": 72636, "dcterms:identifier": "1-001-plain.txt", "dcterms:title": "document1#Text", "dcterms:type": "Text" } ], "dcterms:identifier": "item1", "hcsvlab:display_document": { "@id": "1-001-plain.txt" }, "hcsvlab:indexable_document": { "@id": "1-001-plain.txt" } } ] } } ]'</code>
</li>
</ul>
</td>
</tr>
<tr>
<td>Example Response</td>
<td colspan=4> 
<br>Success:
<br>{"success":["&lt;item_1_id&gt;", "&lt;item_2_id&gt;"]}
<br>or
<br>{"success":["&lt;item_id&gt;"], "failures":["Unknown item contains invalid metadata"]}
<br>Failure:
<br>{"error":"JSON-LD formatted item metadata must be sent with the api request"}
<br>or
<br>{"error":"JSON item metadata is ill-formatted"}
<br>or
<br>{"error":"Requested collection not found"}
<br>or
<br>{"error":"User is unauthorised"}
<br>or
<br>{"error":"The item &lt;item_id&gt; already exists in the collection &lt;collection_name&gt;"}
<br>or
<br>{"error":"identifier missing from document"}
<br>or
<br>{"error":"The identifier "&lt;document_id&gt;" is used for multiple documents"}
<br>or
<br>{"error":"content missing from document &lt;document_id&gt;"}
<br>or
<br>{"error":"The file &lt;filename&gt; has already been uploaded to the collection &lt;collection_name&gt;"}
<br>or
<br>{"error":"Error in file parameter."}
<br>or
<br>{"error":"Uploaded file is not present or empty."}
<br>or
<br>{"error":"No items were added"}
</td>
</tr>
</table>



### Add Document To Owned Item
<table>
<tr>
<th> Description </th>
<th> URL </th>
<th> Method </th>
<th> Returns </th>
</tr>
<tr>
<td>Add document to owned item</td>
<td>/catalog/{collection_id}/{item_id}</td>
<td>POST</td>
<td>Result of operation (error/success)</td>
</tr>
<tr>
<td>Notes</td>
<td colspan="4">
<ol>
<li>Users are only authorised to add documents to existing items in collections which they own.</li>
<li>When adding a document a shortened form of the "@id" which specifies the document URI needs to be supplied. This will be automatically converted into Alveo catalog URLs. For example, if adding a document named "document1.txt" then the shortened document URI <code>"@id":"document1.txt"</code> should be supplied and will be automatically converted to <code>"@id":"http://app.alveo.edu.au/catalog/test/item1/document/document1"</code>.</li>
<li>The identifiers in the supplied document metadata should have matching values, specifically the "@id" and the "dcterms:identifier". This also applies to the document filename if a file is uploaded or a referenced file is used.</li>
<li> When uploading files or including the document content as JSON the document source metadata  <code>"dcterms:source":{"@id":"&lt;file_or_http_uri&gt;"}</code> does not need to be supplied. Instead the system will automatically assign a location for these document files and generate this metadata accordingly. However it is essential to include the document source metadata term for any documents referenced with "file://" or "http://".</li>
<li>This is a POST request that requires a JSON-LD set of document metadata to be sent with it. Hence, cannot be replicated through a browser but through curl this can be done with something akin to the following.
<ul>
<li>If adding a a document referenced (with "file://" or "http://"): <code>curl -X POST -H "X-API-KEY: &lt;key&gt;" -H "Accept: application/json" -H "Content-Type: application/json"  -d '{"metadata":{&lt;document_metadata&gt;}' &lt;server&gt;/catalog/&lt;collection_id&gt;/&lt;item_id&gt;</code></li>
<li>If adding a document whose content is embedded in JSON:
<code>curl -X POST -H "X-API-KEY: &lt;key>" -H "Accept: application/json" -H "Content-Type: application/json"  -d '{"document_content": "&lt;document_content&gt;", "metadata":{&lt;document_metadata&gt;}' &lt;server&gt;/catalog/&lt;collection_id&gt;/&lt;item_id&gt;</code></li>
<li>If adding an item with a single document uploaded as part of the HTTP request:
<code>curl -X POST -H "X-API-KEY: &lt;key&gt;" -H "Accept: application/json" -F file=@"&lt;file_location&gt;" -F metadata='{&lt;document_metadata&gt;}' &lt;server&gt;/catalog/&lt;collection_id&gt;/&lt;item_id&gt;</code></li>
</ul>
</li>
<li>To add document to owned contribution, append parameter <code>contrib_id</code> to indicate the specific contribution</li>
</ol>
</td>
</tr>
<tr>
<td>Example Input</td>
<td colspan=4>
<ul>
<li>If adding a document referenced (with "file://" or "http://"): <code>-d '{ "metadata": { "@context": { "dcterms": "http://purl.org/dc/terms/", "foaf": "http://xmlns.com/foaf/0.1/" }, "@id": "document2.txt", "@type": "foaf:Document", "dcterms:identifier": "document2.txt", "dcterms:title": "document2#Text", "dcterms:type": "Text", "dcterms:source": { "@id": "file:///data/test_collections/ausnc/test/document2.txt" } } }'</code></li>
<li>If adding a document whose contents are embedded in the JSON: <code>-d '{ "document_content": "Hello World!", "metadata": { "@context": { "dcterms": "http://purl.org/dc/terms/", "foaf": "http://xmlns.com/foaf/0.1/" }, "@id": "document2.txt", "@type": "foaf:Document", "dcterms:identifier": "document2.txt", "dcterms:title": "document2#Text", "dcterms:type": "Text" } }'</code></li>
<li>If adding a document uploaded as part of the HTTP request: <code>-F file=@"document2.txt" -F metadata='{ "@context": { "dcterms": "http://purl.org/dc/terms/", "foaf": "http://xmlns.com/foaf/0.1/" }, "@id": "document2.txt", "@type": "foaf:Document", "dcterms:identifier": "document2.txt", "dcterms:title": "document2#Text", "dcterms:type": "Text" }'</code></li>
</ul>
</td>
</tr>
<tr>
<td>Example Response</td>
<td colspan=4>
<br>Success:
<br>{"success":"Added the document &lt;document_filename&gt; to item &lt;item_id&gt; in collection &lt;collection_id&gt;"}
<br>Failure:
<br>{"error":"User is unauthorised"}
<br>or
<br>{"error":"Requested collection not found"}
<br>or
<br>{"error":"Requested item not found"}
<br>or
<br>{"error":"JSON-LD formatted item metadata must be sent with the api request"}
<br>or
<br>{"error":"JSON item metadata is ill-formatted"}
<br>or
<br>{"error":"The document &lt;document_id&gt; already exists in the collection &lt;collection_name&gt;"}
<br>or
<br>{"error":"content missing from document &lt;document_id&gt;"}
<br>or
<br>{"error":"The file &lt;filename&gt; has already been uploaded to the collection &lt;collection_name&gt;"}
<br>or
<br>{"error":"Error in file parameter."}
<br>or
<br>{"error":"Uploaded file is not present or empty."}
</td>
</tr>
</table>

### Delete Document From Owned Item
<table>
<tr>
<th> Description </th>
<th> URL </th>
<th> Method </th>
<th> Returns </th>
</tr>
<tr>
<td>Delete document from owned item</td>
<td>/catalog/{collection_id}/{item_id}/document/{document_filename}</td>
<td>DELETE</td>
<td>Result of operation (error/success)</td>
</tr>
<tr>
<td>Notes</td>	
<td colspan="4">
<ol>
<li>Users are only authorised to delete documents from items in collections which they own.</li>
<li>If the source for the document is a file, e.g. on upload was sourced at: { "dcterms:source": { "@id": "file:///home/devel/&lt;file_name&gt;.txt"}, on API delete document call, the ORIGINAL SOURCE file is deleted.
<li>This is a DELETE request. Hence, cannot be replicated through a browser but through curl this can be done with something akin to the following.
<ul>
<li><code>curl -H "X-API-KEY: &lt;api_key&gt;"  -H "Accept: application/json" -X DELETE &lt;server&gt;/catalog/&lt;collection_id&gt;/&lt;item_id&gt;/document/&lt;document_filename&gt;</code></li>
</ul>
</li>
</ol>	
</td>	
</tr>
<tr>
<td>Example Response</td>
<td colspan=4> 
<br>Success:
<br>{"success":"Deleted the document &lt;document_filename&gt; from item &lt;item_id&gt; in collection &lt;collection_id&gt;"}
<br>Failure:
<br>{"error":"User is unauthorised"}
<br>or
<br>{"error":"Requested collection not found"}
<br>or
<br>{"error":"Requested item not found"}
<br>or
<br>{"error":"Requested document not found"}
</td>
</tr>
</table>

## Annotation Contribution

A **Contribution** is a collection of documents uploaded to Alveo that will be attached to different existing items.  These will generally be annotation files or other files derived from existing documents in the system. 

For example. 

1. Researcher finds ten items in Austalk representing interviews
2. Downloads the audio files for these items
3. Transcribes the interviews using desktop software
4. Would now like to share these transcriptions with other Austalk users

The collection of ten transcription files would become a **Contribution**.  

A Contribution has

-  an automatically generated identifier
-  a unique URL (that includes the identifier)
-  an owner (an Alveo user)
-  associated metadata including at least
	- dcterms:isPartOf (associated collection URL)
	- dcterms:title
	- dcterms:creator (== owner)
	- dcterms:created (the creation date)
	- dcterms:abstract (textual description of the contribution)
-  zero or more documents 

### Create Contribution 

<table>
<tr>
<th> Description </th>
<th> URL </th>
<th> Method </th>
<th> Returns </th>
</tr>
<tr>
<td>Create contribution</td>
<td>/contrib/</td>
<td>POST</td>
<td>Result of operation</td>
</tr>
<tr>
<td>Notes</td>	
<td colspan="4">
create a new contribution, metadata as per collection creation in JSON-LD	
</td>	
</tr>
<tr>
<td>Example Input</td>
<td colspan=4> 
<pre><code>
{
	"contribution_name": "HelloWorld",
	"contribution_collection": "Cooee",
	"contribution_text": "This is contribution description",
	"contribution_abstract": "This is contribution abstract"
}		
</code></pre>
</td>
</tr>
<tr>
<td>Example Response</td>
<td colspan=4> 
<ul>
<li>Success:</li>
redirect (302) to show contribution url 
<li>Failure:</li>
{"error":"{error message}"}
</ul>
</td>
</tr>
</table>

### Read Contribution List

<table>
<tr>
<th> Description </th>
<th> URL </th>
<th> Method </th>
<th> Returns </th>
</tr>
<tr>
<td>Read contribution list</td>
<td>/contrib/</td>
<td>GET</td>
<td>array of contribution id, name and URL</td>
</tr>
<tr>
<td>Notes</td>	
<td colspan="4">
N/A
</td>	
</tr>
<tr>
<td>Example Input</td>
<td colspan=4> 
	N/A
</td>
</tr>
<tr>
<td>Example Response</td>
<td colspan=4> 
<pre><code>
{
	"contributions" : [
		{"id" : 1, "name" : "ace", "url" : "https://app.alveo.edu.au/contrib/1"},
		{"id" : 2, "name" : "cooee", "url" : "https://app.alveo.edu.au/contrib/2"}, 
		{"id" : 3, "name" : "ice", "url": "https://app.alveo.edu.au/contrib/3"}, 
		{"id" : 4, "name" : "austalk", "url": "https://app.alveo.edu.au/contrib/4"}
	]
}	
</code></pre>	
</td>
</tr>
</table>

### Read Single Contribution

<table>
<tr>
<th> Description </th>
<th> URL </th>
<th> Method </th>
<th> Returns </th>
</tr>
<tr>
<td>Read single contribution</td>
<td>/contrib/{contribution_id}/</td>
<td>GET</td>
<td>Return the contribution metadata, including a list of document URLs for documents in the contribution</td>
</tr>
<tr>
<td>Notes</td>	
<td colspan="4">
	N/A
</td>	
</tr>
<tr>
<td>Example Input</td>
<td colspan=4> 
N/A
</td>
</tr>
<tr>
<td>Example Response</td>
<td colspan=4> 
<pre><code>
{
	"url":"https://app.alveo.edu.au/contrib/monash",
	"name":"monash",
	"metadata": {
		"title" : "monash", 
		"collection" : "https://app.alveo.edu.au/catalog/monash", 
		"creator":"Karl LI",
		"created":"2017-11-14 05:41:10 UTC",
		"abstract":"Monash Corpus of Spoken English",
		... (other fields)
	},
	"documents" : [
		{"name" : "1-1.wav", "url": "https://app.alveo.edu.au/catalog/monash/item_1/document/1-1.wav"}, 
		{"name" : "1-2.wav", "url" : "https://app.alveo.edu.au/catalog/monash/item_1/document/1-2.wav"}
	] 
}	
</code></pre>	
</td>
</tr>
</table>

### Update Contribution

<table>
<tr>
<th> Description </th>
<th> URL </th>
<th> Method </th>
<th> Returns </th>
</tr>
<tr>
<td>Update contribution</td>
<td>/contrib/{contribution_id}/</td>
<td>PUT</td>
<td></td>
</tr>
<tr>
<td>Notes</td>	
<td colspan="4">
	update name, abstract and text description. No duplicated contribution name.
</td>	
</tr>
<tr>
<td>Example Input</td>
<td colspan=4> 
<pre><code>
{
	"contribution_name": "HelloWorld",
	"contribution_text": "This is contribution description",
	"contribution_abstract": "This is contribution abstract"
}	
</code></pre>	
</td>
</tr>
<tr>
<td>Example Response</td>
<td colspan=4> 
<ul>
<li>Success:</li>
redirect (302) to show contribution url 
<li>Failure:</li>
{"error":"{error message}"}
</ul>
</td>
</tr>
</table>

### Add Document To Contribution

<table>
<tr>
<th> Description </th>
<th> URL </th>
<th> Method </th>
<th> Returns </th>
</tr>
<tr>
<td>Add document to contribution</td>
<td>/catalog/{collection_id}/{item_id}</td>
<td>POST</td>
<td>Result of operation (error/success)</td>
</tr>
<tr>
<td>Notes</td>	
<td colspan="4">
To add document to owned contribution, append parameter contrib_id to indicate the specific contribution
</td>	
</tr>
<tr>
<td>Example Input</td>
<td colspan=4>
Refer to 'Add document to owned item' 
</td>
</tr>
<tr>
<td>Example Response</td>
<td colspan=4> 
Refer to 'Add document to owned item'	
</td>
</tr>
</table>

### Delete Contribution

<table>
<tr>
<th> Description </th>
<th> URL </th>
<th> Method </th>
<th> Returns </th>
</tr>
<tr>
<td>Delete contribution</td>
<td>/contrib/{contribution_id}/</td>
<td>DELETE</td>
<td>Result of operation (error/success)</td>
</tr>
<tr>
<td>Notes</td>	
<td colspan="4">
delete whole contribution (only for admin and contribution owner), including all contribution documents.	
</td>	
</tr>
<tr>
<td>Example Input</td>
<td colspan=4> 
</td>
</tr>
<tr>
<td>Example Response</td>
<td colspan=4> 
<ul>
<li>Success:</li>
{"success":"Contribution [HelloWorld] has been removed successfully. 5 document(s) removed."}
<li>Failure:</li>
{"error":"{error message}"}	
</ul>
</td>
</tr>
</table>

### Export Contribution

<table>
<tr>
<th> Description </th>
<th> URL </th>
<th> Method </th>
<th> Returns </th>
</tr>
<tr>
<td>Export contribution as zip</td>
<td>/contrib/{contribution_id}.zip</td>
<td>GET</td>
<td>Return all of the contribution documents as a zip file</td>
</tr>
<tr>
<td>Notes</td>	
<td colspan="4">
	N/A
</td>	
</tr>
<tr>
<td>Example Input</td>
<td colspan=4> 
	N/A
</td>
</tr>
<tr>
<td>Example Response</td>
<td colspan=4> 
<ul>
<li>Success:</li>
Response as zip file
<li>Failure:</li>
{"error":"{error message}"}	
</ul>	
</td>
</tr>
</table>


## Speaker

### Read Speaker List

<table>
<tr>
<th> Description </th>
<th> URL </th>
<th> Method </th>
<th> Returns </th>
</tr>
<tr>
<td>Read speaker list</td>
<td>/speakers/{collection_name}</td>
<td>GET</td>
<td>Returns a list of speaker identifiers (URIs) associated with this collection</td>
</tr>
<tr>
<td>Notes</td>	
<td colspan="4">
	N/A
</td>	
</tr>
<tr>
<td>Example Input</td>
<td colspan=4> 
N/A	
</td>
</tr>
<tr>
<td>Example Response</td>
<td colspan=4> 
<pre><code>
{
    "speakers": [
      "http://alveo.local:3000/speakers/may10/111",
      "http://alveo.local:3000/speakers/may10/112",
      "http://alveo.local:3000/speakers/may10/105",
      "http://alveo.local:3000/speakers/may10/109",
      "http://alveo.local:3000/speakers/may10/110"
    ]
}	
</code></pre>	
</td>
</tr>
</table>

### Create Speaker

<table>
<tr>
<th> Description </th>
<th> URL </th>
<th> Method </th>
<th> Returns </th>
</tr>
<tr>
<td>Create speaker</td>
<td>/speakers/{collection_name}</td>
<td>POST</td>
<td>Result of operation (error/success)</td>
</tr>
<tr>
<td>Notes</td>	
<td colspan="4">
	Create a new speaker from a JSON-LD payload
</td>	
</tr>
<tr>
<td>Example Input</td>
<td colspan=4> 
- Minimal example, no extra metadata fields	
<pre><code>
{
  "@context": {
      "dbp": "http://dbpedia.org/ontology/",
      "dcterms": "http://purl.org/dc/terms/",
      "foaf": "http://xmlns.com/foaf/0.1/",
      "geo": "http://www.w3.org/2003/01/geo/wgs84_pos#",
      "iso639": "http://downlode.org/rdf/iso-639/languages#",
      "olac": "http://www.language-archives.org/OLAC/1.1/",
      "rdf": "http://www.w3.org/1999/02/22-rdf-syntax-ns#",
      "rdfs": "http://www.w3.org/2000/01/rdf-schema#",
      "xsd": "http://www.w3.org/2001/XMLSchema#",

      "identifier": "dcterms:identifier",
      "gender": "foaf:gender",
      "age": "foaf:age",
      "birthYear": "dbp:birthYear"
   },

  "identifier": "1_116",
  "birthYear": 1967,
  "gender": "male",
  "age": 45
}	
</code></pre>	

- Minimal example with reference to external contextminimal example with reference to external context
<pre><code>
{
  "@context": "http://app.alveo.edu.au/schema/speaker-context",
  "identifier": "1_116",
  "birthYear": 1967,
  "gender": "male",
  "age": 45,
}	
</code></pre>

- JSON-LD is expanded with the addition of @id and @type properties before it is converted to RDF using a standard JSON-LD to RDF conversion. <code>@id</code> is generated from the identifier field prefixed by the speaker uri for this collection.

<pre><code>
{
	"@context": "http://app.alveo.edu.au/schema/speaker-context",
	"@id": "http://app.alveo.edu.au/speakers/austalk/1_116",
	"@type": "foaf:Person",

	"identifier": "1_116",
	"birthYear": 1967,
	"gender": "male",
	"age": 45
}	
</code></pre>

- Expanded example includes additional metadata properties, new namespaces are added to the context

<pre><code>
{
  "@context": [
      "http://app.alveo.edu.au/schema/speaker-context",
      {
          "austalk": "http://ns.austalk.edu.au/"
      }
  ],

  "identifier": "1_116",
  "birthYear": 1967,
  "gender": "male",
  "age": 45,

  "olac:recorder": "132cc092a74f4eab54e89d899228a128",

  "austalk:ageGroup": "30-49",
  "austalk:birthPlace": {
    "@id": "http://sws.geonames.org/2176632",
    "@type": "geo:Feature",
    "geo:countryCode": "AU",
    "geo:countryName": "Australia",
    "geo:lat": "-33.41665",
    "geo:long": "149.5806",
    "geo:state": "New South Wales",
    "geo:town": "Bathurst"
  },
  "austalk:collection": "austalk",
  "austalk:consent": true,
  "austalk:cultural_heritage": "Australian",
  "austalk:education_level": "TAFE Certificate",
  "austalk:father_accent": "Australian",
  "austalk:father_birthPlace": {
      "@id": "http://sws.geonames.org/2177671",
      "geo:countryCode": "AU",
      "geo:countryName": "Australia",
      "geo:lat": "-30.50828",
      "geo:long": "151.67123",
      "geo:state": "New South Wales",
      "geo:town": "Armidale"
  },
  "austalk:father_cultural_heritage": "Australian",
  "austalk:father_education_level": "school certificate",
  "austalk:father_first_language": "iso639:eng",
  "austalk:father_first_language_name": "English",
  "austalk:father_pob_country": "Australia",
  "austalk:father_pob_state": "NSW",
  "austalk:father_pob_town": "Armidale",
  "austalk:father_professional_category": "manager and admin",
  "austalk:first_language": "iso639:eng",
  "austalk:first_language_name": "English",
  "austalk:has_dentures": false
}	
</code></pre>


</td>
</tr>
<tr>
<td>Example Response</td>
<td colspan=4> 
- success	
<pre><code>
{
  "success": {
	  "id": 111,
	  "message": "speaker created",
	  "URI": "http://alveo.local:3000/speakers/may10/111"
  }
}	
</code></pre>	

- failure

<pre><code>
{
  "errors": {
	  "id": "error",
	  "message": "<error message>"
  }
}	
</code></pre>

</td>
</tr>
</table>

### Read Single Speaker

<table>
<tr>
<th> Description </th>
<th> URL </th>
<th> Method </th>
<th> Returns </th>
</tr>
<tr>
<td>Read single Speaker</td>
<td>/speakers/{collection_name}/{speaker_id}</td>
<td>GET</td>
<td>Returns a JSON-LD description of the speaker</td>
</tr>
<tr>
<td>Notes</td>	
<td colspan="4">
</td>	
</tr>
<tr>
<td>Example Input</td>
<td colspan=4> 
	N/A
</td>
</tr>
<tr>
<td>Example Response</td>
<td colspan=4> 
- Success
<pre><code>
{
	"@context": {
		"dbp": "http://dbpedia.org/ontology/",
		"dc": "http://purl.org/dc/terms/",
		"foaf": "http://xmlns.com/foaf/0.1/",
		"geo": "http://www.w3.org/2003/01/geo/wgs84_pos#",
		"iso639": "http://downlode.org/rdf/iso-639/languages#",
		"olac": "http://www.language-archives.org/OLAC/1.1/",
		"rdf": "http://www.w3.org/1999/02/22-rdf-syntax-ns#",
		"rdfs": "http://www.w3.org/2000/01/rdf-schema#",
		"xsd": "http://www.w3.org/2001/XMLSchema#"
	},
	"@id": "http://alveo.local:3000/speakers/may10/110",
	"@type": "foaf:Person",
	"dbp:birthYear": "1967",
	"dc:identifier": "110",
	"foaf:age": "45",
	"foaf:gender": "male",
	"foaf:name": "Alveo"
}	
</code></pre>	

- Failure
<pre><code>
{
	"errors": {
		"id": "no_speaker",
		"message": "speaker [1101] not found"
	}
}	
</code></pre>
</td>
</tr>
</table>

### Update Speaker

<table>
<tr>
<th> Description </th>
<th> URL </th>
<th> Method </th>
<th> Returns </th>
</tr>
<tr>
<td>Update speaker</td>
<td>/speakers/{collection_name}/{speaker_id}</td>
<td>PUT</td>
<td>Result of operation (error/success)</td>
</tr>
<tr>
<td>Notes</td>	
<td colspan="4">
	Updates the speaker metadata from a JSON-LD payload.
</td>	
</tr>
<tr>
<td>Example Input</td>
<td colspan=4> 
<pre><code>
{
	"@context": {
		"dbp": "http://dbpedia.org/ontology/",
		"dcterms": "http://purl.org/dc/terms/",
		"foaf": "http://xmlns.com/foaf/0.1/",
		"geo": "http://www.w3.org/2003/01/geo/wgs84_pos#",
		"iso639": "http://downlode.org/rdf/iso-639/languages#",
		"olac": "http://www.language-archives.org/OLAC/1.1/",
		"rdf": "http://www.w3.org/1999/02/22-rdf-syntax-ns#",
		"rdfs": "http://www.w3.org/2000/01/rdf-schema#",
		"xsd": "http://www.w3.org/2001/XMLSchema#"
	},
	"dbp:birthYear": 1967,
	"dcterms:identifier": "1_116",
	"foaf:age": 45,
	"foaf:gender": "male"
}	
</code></pre>	
</td>
</tr>
<tr>
<td>Example Response</td>
<td colspan=4> 
- Success:	
<pre><code>
{
	"success": {
		"id": 110,
		"message": "speaker updated",
		"URI": "http://alveo.local:3000/speakers/may10/110"
	}
}	
</code></pre>	

- Failure:
<pre><code>
{
	"errors": {
		"id": "<error type>",
		"message": "<error message>"
	}
}	
</code></pre>
</td>
</tr>
</table>

### Delete Speaker

<table>
<tr>
<th> Description </th>
<th> URL </th>
<th> Method </th>
<th> Returns </th>
</tr>
<tr>
<td>Delete speaker</td>
<td>UR/speakers/{collection_name}/{speaker_id}L</td>
<td>DELETE</td>
<td>Result of operation (error/success)</td>
</tr>
<tr>
<td>Notes</td>	
<td colspan="4">
According to SPARQL1.1 spec, "deleting triples that are not present, or from a graph that is not present will have no effect and will result in success." <a href="https://www.w3.org/TR/sparql11-update/#deleteWhere" target="_blank" >DELETE WHERE</a>.

<code>DELETE</code> will always return "success" unless exception exception.
</td>	
</tr>
<tr>
<td>Example Input</td>
<td colspan=4> 
	N/A
</td>
</tr>
<tr>
<td>Example Response</td>
<td colspan=4> 
<pre><code>
{
	"success": {
		"id": 110,
		"message": "speaker [110] deleted"
	}
}	
</code></pre>	
</td>
</tr>
</table>



## Others

### Read API Version

<table>
<tr>
<th> Description </th>
<th> URL </th>
<th> Method </th>
<th> Returns </th>
<th> Notes </th>
</tr>
<tr> 
<td> Read API version </td>
<td> /version </td>
<td> Read </td>
<td> API version </td>
<td />
</tr>
<tr>
<td> Example Response </td>
<td colspan=4>
<CODE>
	{"API version":"V2.0"} 
</CODE>
</td>
</tr>
</table>

### Read Licences
<table>
<tr>
<th> Description </th>
<th> URL </th>
<th> Method </th>
<th> Returns </th>
<th> Notes </th>
</tr>
<tr>
<td>Read licences</td>
<td>/licences</td>
<td>GET</td>
<td>List of licences in the system, their name and ID</td>
<td />
<tr>
<td> Example Response </td>
<td colspan=4> 
<pre>
<CODE>
[
	{
		"name":"Creative Commons",
		"id": 1
	},
	{
		"name":"AusNC Terms of Use",
		"id": 2
	},
	{
		"name":"Creative Commons V3",
		"id": 3
	}
] 
</CODE>
</pre>
</td>
</tr>
</table>