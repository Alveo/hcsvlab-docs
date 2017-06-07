Pre-release Test Procedure

# Purpose

**pre-release regression testing**

Run a series of testing which verifies that v3.4-rc2 performs correctly after it was changed or interfaced with other component. 

Changes may include new features, enhancements, patches, configuration changes, etc.

**rollback testing**

If during production release, something wrong happen then we need to roll-back to the original branch (v3.3.1).

# Simulate production in staging

This test is on staging server, so first we need to prepare the environment to simulate production server.

## environment preparation
  **switch to production branch (v3.3.1)**

  - hcsvlab

  `$ cap deploy staging `

  *TBC - Karl*

  - data (DB/solr/sesame)

  Migrate production data.

  *Question: export & import production data or directly ingest data to simulate production?*

  *TBC - Andy & Michael* 

## system testing
  **smoke testing**

  *TBC - Karl*
  
# Deploy release candidate 

## environment preparation
  **switch to release candidate (v3.4-rc2)**

  - hcsvlab

  `$ cap deploy staging`

  *TBC - Karl*

  - data (DB/solr/sesame)
  
  Run ruby script to ingest testing data

  `$ ruby script/clear_data.rb ace`

  `$ ruby script/ingest_austalk.rb ace /mnt/volume/ace_json/`
  
  *Make sure the directory /mnt/volume/ has ace_json cooee_json* 

  *TBC - Andy & Michael*  

## system testing

  **regression testing**

  *TBC - Karl*  

  **API testing**

  *TBC - Michael*

  **Performance testing**

  *TBC - Andy*

# Rollback testing

## environment preparation

  Rollback to previous version (v3.3.1):

  - hcsvlab

  `$ cap deploy staging`

  *TBC - Karl*

  - data
    - DB (prd: 20160114051837  Add index to documents item)

    `$ bundle exec rake db:migrate:down VERSION=20160114051837`

    *TBC - Karl*

    - slor

    - sesame

    *Question: do we need to remove testing data from solr & sesame?*

## system testing

After rollback to previous version, we need to run some test to ensure the recovered system works.

** smoke testing **   

*TBC - Karl* 