Pre-release Test Procedure

<!-- MarkdownTOC -->

1. [Purpose](#purpose)
1. [Simulate production in staging](#simulate-production-in-staging)
  1. [environment preparation](#environment-preparation)
  1. [system testing](#system-testing)
1. [Deploy release candidate](#deploy-release-candidate)
  1. [environment preparation](#environment-preparation-1)
  1. [system testing](#system-testing-1)
1. [Rollback testing](#rollback-testing)
  1. [environment preparation](#environment-preparation-2)
  1. [system testing](#system-testing-2)

<!-- /MarkdownTOC -->


<a name="purpose"></a>
# Purpose

**pre-release regression testing**

Run a series of testing which verifies that v3.4-rc2 performs correctly after it was changed or interfaced with other component. 

Changes may include new features, enhancements, patches, configuration changes, etc.

**rollback testing**

If during production release, something wrong happen then we need to roll-back to the original branch (v3.3.1).

<a name="simulate-production-in-staging"></a>
# Simulate production in staging

This test is on staging server, so first we need to prepare the environment to simulate production server.

<a name="environment-preparation"></a>
## environment preparation
  **switch to production branch (v3.3.1)**

  - downgrade DB 

 Check DB migrate status to make sure current version to match '20160114051837'

  */home/devel/hcsvlab-web/current is the working directory*

```
[devel@alveo-staging current]$ bundle exec rake db:migrate:status

database: hcsvlab

 Status   Migration ID    Migration Name
--------------------------------------------------
   up     20130402224811  Devise create users
   up     20130402224815  Create roles
   up     20130403032728  Create searches
   up     20130403032729  Create bookmarks
   up     20130403032730  Remove editable fields from bookmarks
   up     20130403032731  Add user types to bookmarks searches
   up     20130606013944  Create item lists
   up     20130618072806  Add authentication token to users
   up     20130819004059  Create user licence agreements
   up     20131021005313  Create user licence request
   up     20131104014327  Fix licence agreement column names
   up     20131106011128  Add user session
   up     20131107002123  Create item metadata field name mappings
   up     20140120011702  Create user annotations
   up     20140302222240  Create items in item lists
   up     20140313030321  Add user searches
   up     20140317042527  Add user api calls
   up     20140317231930  Add shared to item lists
   up     20140318230337  Add aaf field to users
   up     20140902040237  Create licences
   up     20141013050709  Create collection and list
   up     20141020015616  Create items and documents
   up     20141022045851  Change type to private in licences
   up     20141028113041  Add indexed at to item
   up     20141029032823  Add owner id to user licence requests
   up     20141030052534  Create handle index for items
   up     20141031001607  Rename item to handle in item list
   up     20141113045807  Add json metadata to item
   up     20141127010040  Create document audits
   up     20141202235844  Change user licence agreement to support collection lists
   up     20141204234643  Add index on document filename
   up     20151111011322  Create languages
   up     20160114051837  Add index to documents item
   up     20161122000349  Create doorkeeper tables
   up     20161222221407  Create collection properties
   up     20170111033406  Remove rdf file path from collection
   up     20170202101231  Create attachments

[devel@alveo-staging current]$
```
  
So migrate down to related version.

```
[devel@alveo-staging current]$ bundle exec rake db:migrate:down VERSION=20160114051837
==  AddIndexToDocumentsItemId: reverting ======================================
-- remove_index("documents", {:name=>"index_documents_item_id"})
   -> 0.0118s
==  AddIndexToDocumentsItemId: reverted (0.0118s) =============================

```

  - hcsvlab

  `$ cap staging deploy `

  Select 'v3.3.1' to deploy.
 
Run bunle install to ensure all gem are installed.

```
[devel@alveo-staging current]$ bundle install
Fetching gem metadata from https://rubygems.org/.............
Fetching version metadata from https://rubygems.org/...
Fetching dependency metadata from https://rubygems.org/..
...
Bundle complete! 92 Gemfile dependencies, 158 gems now installed.
Gems in the groups development and test were not installed.
Bundled gems are installed into /home/devel/hcsvlab-web/shared/bundle.
```

Restart rails.

```
[devel@alveo-staging current]$ touch tmp/restart.txt
```

Check staging log.

```
[devel@alveo-staging current]$ tail -f log/staging.log
```

Make sure v3.3.1 is running.

```
[devel@alveo-staging current]$ curl https://staging.alveo.edu.au/version.json
{"API version":"v3.3.1"}
```

<a name="system-testing"></a>
## system testing
  **smoke testing**

  - login as data_owner@alveo.edu.au
  - create collection
  - ingest collection data
  - update collection
    - basic metadata
    - additional metadata
    - abstract
    - description
  
<a name="deploy-release-candidate"></a>
# Deploy release candidate 

<a name="environment-preparation-1"></a>
## environment preparation
  **switch to release candidate (v3.4-rc3)**

  - hcsvlab

  `$ cap staging deploy`

  Select "v3.4-rc3" to deploy.

  - data (DB/solr/sesame)

```
[devel@alveo-staging current]$ bundle exec rake db:migrate
==  CreateDoorkeeperTables: migrating =========================================
-- create_table(:oauth_applications)
NOTICE:  CREATE TABLE will create implicit sequence "oauth_applications_id_seq" for serial column "oauth_applications.id"
NOTICE:  CREATE TABLE / PRIMARY KEY will create implicit index "oauth_applications_pkey" for table "oauth_applications"
   -> 0.0292s
-- add_index(:oauth_applications, :uid, {:unique=>true})
   -> 0.0047s
-- create_table(:oauth_access_grants)
NOTICE:  CREATE TABLE will create implicit sequence "oauth_access_grants_id_seq" for serial column "oauth_access_grants.id"
NOTICE:  CREATE TABLE / PRIMARY KEY will create implicit index "oauth_access_grants_pkey" for table "oauth_access_grants"
   -> 0.0065s
-- add_index(:oauth_access_grants, :token, {:unique=>true})
   -> 0.0030s
-- create_table(:oauth_access_tokens)
NOTICE:  CREATE TABLE will create implicit sequence "oauth_access_tokens_id_seq" for serial column "oauth_access_tokens.id"
NOTICE:  CREATE TABLE / PRIMARY KEY will create implicit index "oauth_access_tokens_pkey" for table "oauth_access_tokens"
   -> 0.0068s
-- add_index(:oauth_access_tokens, :token, {:unique=>true})
   -> 0.0030s
-- add_index(:oauth_access_tokens, :resource_owner_id)
   -> 0.0033s
-- add_index(:oauth_access_tokens, :refresh_token, {:unique=>true})
   -> 0.0032s
==  CreateDoorkeeperTables: migrated (0.0601s) ================================

==  CreateCollectionProperties: migrating =====================================
-- create_table(:collection_properties)
NOTICE:  CREATE TABLE will create implicit sequence "collection_properties_id_seq" for serial column "collection_properties.id"
NOTICE:  CREATE TABLE / PRIMARY KEY will create implicit index "collection_properties_pkey" for table "collection_properties"
   -> 0.0056s
==  CreateCollectionProperties: migrated (0.0057s) ============================

==  RemoveRdfFilePathFromCollection: migrating ================================
-- remove_column(:collections, :rdf_file_path)
   -> 0.0025s
==  RemoveRdfFilePathFromCollection: migrated (0.0025s) =======================

==  CreateAttachments: migrating ==============================================
-- create_table(:attachments)
NOTICE:  CREATE TABLE will create implicit sequence "attachments_id_seq" for serial column "attachments.id"
NOTICE:  CREATE TABLE / PRIMARY KEY will create implicit index "attachments_pkey" for table "attachments"
   -> 0.0055s
-- add_index(:attachments, :collection_id)
   -> 0.0030s
==  CreateAttachments: migrated (0.0085s) =====================================
```
  
Migrate collection metadata

```
[devel@alveo-staging current]$ bundle exec rake collection:migrate_metadata
```

  Run ruby script to ingest testing data

```
[devel@alveo-staging ~]$ cd ~/alveo-workers/
```

```
[devel@alveo-staging alveo-workers]$ ruby script/clear_data.rb ace
```

*Remember to remove all log file in alveo-workers directory before ingest*

```
[devel@alveo-staging alveo-workers]$ rm *.log
```

```
[devel@alveo-staging alveo-workers]$ ruby script/ingest_austalk.rb cooee /mnt/volume/cooee_json/`
``` 

  *Make sure the directory /mnt/volume/ has cooee_json* 


<a name="system-testing-1"></a>
## system testing

  **regression testing**

  Use `cucumber` to run regression testing.

  ```
  cucumber --tags ~@ingest_qa_collections
  ```

  *exclude @ingest_qa_collections because ingest collection procedure changed, but relevant testcase doesn't update yet*

  **API testing**

  - ingest collection data
  - speaker

  *TBC*


<a name="rollback-testing"></a>
# Rollback testing

<a name="environment-preparation-2"></a>
## environment preparation

The same as above `environment preparation` procedure.

**update DB**  

Since this is a "downgrade" operation, table `collections` would recover its original column `rdf_file_path`. So we need to update this column and provide data accordingly.

```
hcsvlab=# select id, name, rdf_file_path from collections order by rdf_file_path;
 id |        name         |                     rdf_file_path
----+---------------------+-------------------------------------------------------
 31 | a                   | /data/collections/a.n3
 30 | democollection_name | /data/collections/democollection_name.n3
 32 | karl1               | /data/collections/karl1.n3
 28 | llc                 | /data/collections/llc.n3
 29 | mava                | /data/collections/mava.n3
 25 | mbep                | /data/collections/mbep.n3
 26 | pixar               | /data/collections/pixar.n3
 27 | rirusyd             | /data/collections/rirusyd.n3
 33 | testcollection      | /data/collections/testcollection.n3
 10 | ace                 | /mnt/volume/alveo-production-data/ace.n3
 24 | art                 | /mnt/volume/alveo-production-data/art.n3
 20 | austalk             | /mnt/volume/alveo-production-data/austalk.n3
 11 | austlit             | /mnt/volume/alveo-production-data/austlit.n3
 23 | avozes              | /mnt/volume/alveo-production-data/avozes.n3
 22 | braidedchannels     | /mnt/volume/alveo-production-data/braidedchannels.n3
 12 | cooee               | /mnt/volume/alveo-production-data/cooee.n3
 13 | gcsause             | /mnt/volume/alveo-production-data/gcsause.n3
 14 | ice                 | /mnt/volume/alveo-production-data/ice.n3
 15 | mitcheldelbridge    | /mnt/volume/alveo-production-data/mitcheldelbridge.n3
 16 | monash              | /mnt/volume/alveo-production-data/monash.n3
 18 | test1               | /mnt/volume/alveo-production-data/test1.n3
 19 | test2               | /mnt/volume/alveo-production-data/test2.n3
 17 | trove               | /mnt/volume/alveo-production-data/trove.n3
(23 rows)
```   

**update rdf_file_path**

```
update collections set rdf_file_path='/data/collections/a.n3' where name='a';
update collections set rdf_file_path='/data/collections/democollection_name.n3' where name='democollection_name';
update collections set rdf_file_path='/data/collections/karl1.n3' where name='karl1';
update collections set rdf_file_path='/data/collections/llc.n3' where name='llc';
update collections set rdf_file_path='/data/collections/mava.n3' where name='mava';
update collections set rdf_file_path='/data/collections/mbep.n3' where name='mbep';
update collections set rdf_file_path='/data/collections/pixar.n3' where name='pixar';
update collections set rdf_file_path='/data/collections/rirusyd.n3' where name='rirusyd';
update collections set rdf_file_path='/data/collections/testcollection.n3' where name='testcollection';
update collections set rdf_file_path='/mnt/volume/alveo-production-data/ace.n3' where name='ace';
update collections set rdf_file_path='/mnt/volume/alveo-production-data/art.n3' where name='art';
update collections set rdf_file_path='/mnt/volume/alveo-production-data/austalk.n3' where name='austalk';
update collections set rdf_file_path='/mnt/volume/alveo-production-data/austlit.n3' where name='austlit';
update collections set rdf_file_path='/mnt/volume/alveo-production-data/avozes.n3' where name='avozes';
update collections set rdf_file_path='/mnt/volume/alveo-production-data/braidedchannels.n3' where name='braidedchannels';
update collections set rdf_file_path='/mnt/volume/alveo-production-data/cooee.n3' where name='cooee';
update collections set rdf_file_path='/mnt/volume/alveo-production-data/gcsause.n3' where name='gcsause';
update collections set rdf_file_path='/mnt/volume/alveo-production-data/ice.n3' where name='ice';
update collections set rdf_file_path='/mnt/volume/alveo-production-data/mitcheldelbridge.n3' where name='mitcheldelbridge';
update collections set rdf_file_path='/mnt/volume/alveo-production-data/monash.n3' where name='monash';
update collections set rdf_file_path='/mnt/volume/alveo-production-data/test1.n3' where name='test1';
update collections set rdf_file_path='/mnt/volume/alveo-production-data/test2.n3' where name='test2';
update collections set rdf_file_path='/mnt/volume/alveo-production-data/trove.n3' where name='trove';
```

<a name="system-testing-2"></a>
## system testing

After rollback to previous version, we need to run some test to ensure the recovered system works.

** smoke testing **   

  - login as data_owner@alveo.edu.au
  - create collection
  - ingest collection data
  - update collection
    - basic metadata
    - additional metadata
    - abstract
    - description