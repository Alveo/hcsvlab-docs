## Alveo Web Application installation

### Background

These instructions describe the process of installing and configuring the Alveo (was HCSvLab) web application and its components on a CentOS machine.

### Assumptions

* You have a fresh CentOS 6 machine. 
* You have a non-root user account on the machine with sudo privileges called 'devel'. It is helpful for devel to be able to sudo without a password (e.g. `usermod -a -G wheel devel && echo "%wheel ALL=(ALL) NOPASSWD: ALL" > /etc/sudoers.d/devel-sudo`) both for this guide and future Capistrano deployments.
* You are logged in as this user and are in the home directory.
* You have a GitHub account and have set up ssh keys (see https://help.github.com/articles/generating-ssh-keys)
* You have a local installation of Ansible

### Set up Ansible

Check out a local copy of the Ansible scripts which will assist in automating the build.

    $ git clone https://github.com/Alveo/alveo-ansible.git
    $ cd alveo-ansible

Now edit the various inventory files (`aepm/hosts-*`) according to your target hosts. Note: you can list the same machine under multiple roles, for example if you're building everything onto a single server.

For the simplicity of the inventory file, we recommend you set up your local `~/.ssh/config` with each hosts' connectivity details like so:

    Host my-alveo-host
    HostName 192.0.2.1
    User root
    IdentityFile ~/.ssh/alveo.pem

You can then use the host's alias in the inventory file, for example:

    [alveo-pg]
    my-alveo-host

To test connectivity via Ansible, try running a simple remote command:

    $ ansible all -i hosts-alveo-pg -m shell -b -a "uptime"

### Run the Playbooks

Assuming we have a working Ansible setup, we can now install services semi-automatically using the included playbooks.

See also the [manual instructions](WebAppManual.md) for some components.

#### Solr

Optional: set Apache Tomcat credentials in the `secrets.yml` file under the `alveo-solr` Ansible role.

    $ ansible-playbook -i hosts-alveo-solr site-alveo-solr.yml

If installation is successful, you should be able to browse to Tomcat on port 8080 of your server. Click on the 'Manager' link and log in using the credentials found in `/usr/share/tomcat6/conf/tomcat-users.xml`.

You should see a `/solr` application listed with an `hcsvlab-core` instance loaded.

#### Sesame

**Note:** running the Solr and Sesame playbooks to the same target currently results in two versions of Tomcat being installed (version 6 in `/usr/share/tomcat` and version 8 in `/opt/tomcat`). If you're planning on doing this, you will need to edit one instance's `conf/server.xml` after installation and de-conflict its three critical network ports (by default 8005, 8080 8010). For example, change them to 8006, 8081 and 8011 respectively and restart Tomcat. If you're installing Solr and Sesame on different hosts, this is not required.

Optional: set Apache Tomcat credentials in the `secrets.yml` file under the `alveo-sesame` Ansible role.

    $ ansible-playbook -i hosts-alveo-sesame site-alveo-sesame.yml

If installation is successful, you should be able to browse to Tomcat on port 8080 (or 8081, see above) of your server. Click on the 'Manager' link and log in using the credentials found in `/opt/tomcat/conf/tomcat-users.xml`.

You should see `/openrdf-sesame` and `/openrdf-workbench` applications listed.

#### Postgresql

TODO - please follow the [manual instructions](Postgres.md)

#### RabbitMQ

Optional: set Apache Tomcat credentials in the `secrets.yml` file under the `alveo-rabbitmq` Ansible role.

On the server

    $ cd /home/devel
    $ git clone https://github.com/Alveo/alveo-workers.git

From your local ansible install

    $ ansible-playbook -i hosts-alveo-rabbitmq site-alveo-rabbitmq.yml

If installation is successful, you should see RabbitMQ in the process list and listening on port 5672 (see `netstat -ltnp`). You should also be able to log into the management web interface on port 15672.

#### Galaxy

TODO

### Continue the setup

**Correct directory permissions**

    $ sudo chmod og+rx /home/devel

**Turn off SELinux**

    $ sudo setenforce 0
    
    $ sudo vi /etc/selinux/config
    ...
    SELINUX=disabled  # Change SELINUX value to disabled

**Install EPEL**

These are additional repositories that are not enabled by default but contain some of the packages required by the HCSvLab web application

    $ curl -O http://dl.fedoraproject.org/pub/epel/6/x86_64/epel-release-6-8.noarch.rpm
    $ curl -O  http://rpms.famillecollet.com/enterprise/remi-release-6.rpm
    $ sudo rpm -Uvh remi-release-6*.rpm epel-release-6*.rpm

**Install git and other useful packages**

    $ sudo yum install gcc gcc-c++ patch readline readline-devel zlib zlib-devel libyaml-devel libffi-devel openssl openssl-devel make bzip2 autoconf automake libtool bison httpd httpd-devel apr-devel apr-util-devel mod_xsendfile ntp postgresql postgresql-libs postgresql-server postgresql-devel postfix curl curl-devel openssl openssl-devel tzdata libxml2 libxml2-devel libxslt libxslt-devel sqlite-devel git system-config-firewall-tui ImageMagick ImageMagick-devel mod_ssl wget

**Make directories to rotate Apache log files**

    $ sudo mkdir /var/log/httpd/old
    $ sudo chmod 700 /var/log/httpd/old

**Setup Postgres**

See [Postgres setup](Postgres.md)

**Install RVM and Ruby**

On the web server, depending on what playbooks you've run, [RVM](http://rvm.io) may already be installed. 

If so:

    $ sudo chown -cR devel: ~/.rvm
    $ rvm install ruby-2.1.4
    $ source /home/devel/.rvm/scripts/rvm
    $ rvm gemset create hcsvlab

If not:

    $ \curl -#L https://get.rvm.io | bash -s stable --ruby=2.1.4
    $ source /home/devel/.rvm/scripts/rvm
    $ rvm gemset create hcsvlab

**Install Passenger**

Passenger is an Apache2 module that serves Ruby on Rails applications. For more information see [Phusion Passenger](http://www.modrails.com/)

Install

    $ rvm use ruby-2.1.4
    $ gem install rubygems-update
    $ update_rubygems # for rack backwards compatibility
    $ gem install rack -v=1.6.4
    $ gem install passenger -v=5.0.10 --conservative
    $ passenger-install-apache2-module
    
Configure

> Note: ensure your `ServerName` and any file paths (including Passenger versions) are valid for your environment.
	
    $ sudo vi /etc/httpd/conf.d/hcsvlab.conf
    ...
    LoadModule ssl_module modules/mod_ssl.so
    Listen 443
    
    <VirtualHost *:80>
            ServerName alveo.example.com
            Redirect permanent / https://alveo.example.com/
    </VirtualHost>
    
    <VirtualHost *:443>
        ServerName alveo.example.com
        DocumentRoot /home/devel/hcsvlab-web/current/public

        LoadModule passenger_module /home/devel/.rvm/gems/ruby-2.1.4/gems/passenger-5.0.10/buildout/apache2/mod_passenger.so
        <IfModule mod_passenger.c>
            PassengerRoot /home/devel/.rvm/gems/ruby-2.1.4/gems/passenger-5.0.10
            PassengerDefaultRuby /home/devel/.rvm/gems/ruby-2.1.4/wrappers/ruby
        </IfModule>

        RailsEnv qa2

        SSLEngine on
        SSLCertificateFile /etc/httpd/ssl/ca.crt
        SSLCertificateKeyFile /etc/httpd/ssl/ca.key
    
        # Uploads of up to 100MB permitted
        LimitRequestBody 100000000
        <Directory /home/devel/hcsvlab-web/current/public>
                AllowOverride all
                Options -MultiViews
                
                # Allow CORS (javascript cross-site) requests
            	Header always set Access-Control-Allow-Origin "*"
            	Header always set Access-Control-Max-Age "1000"
            	Header always set Access-Control-Allow-Headers "X-Requested-With, Content-Type, Origin, Authorization, Accept, Client-Security-Token, Accept-Encoding, X-API-KEY"
            	Header always set Access-Control-Allow-Methods "POST, GET, OPTIONS, DELETE, PUT"

            	# Make sure OPTIONS response returns 200
            	RewriteEngine On
            	RewriteCond %{REQUEST_METHOD} OPTIONS
            	RewriteRule ^(.*)$ $1 [R=200,L]
        </Directory>
    </VirtualHost>
    ...

Remove old ssl configuration

    $ sudo rm /etc/httpd/conf.d/ssl.conf
    
Be sure to place proper SSL cerificates into /etc/httpd/ssl

**Open Ports for the Web Services**

Edit `iptables` to open up the web ports, optionally including 8080 for Tomcat (see below):

    $ sudo vi /etc/sysconfig/iptables
	
Add the lines:

    -A INPUT -m state --state NEW -m tcp -p tcp --dport 80 -j ACCEPT
    -A INPUT -m state --state NEW -m tcp -p tcp --dport 8080 -j ACCEPT
    -A INPUT -m state --state NEW -m tcp -p tcp --dport 443 -j ACCEPT

right after this line:

    -A INPUT -m state --state NEW -m tcp -p tcp --dport 22 -j ACCEPT
    
Restart the service:

    $ sudo service iptables restart
	
Restart Apache -- won't work without Passenger installed

    $ sudo chkconfig --level 345 httpd on
    $ sudo service httpd restart

**Make a directory for the app (Capistrano needs this)**

    $ mkdir -p ~/hcsvlab-web/releases

**Install Java**

    $ yum install jdk1.8.0_74 # or similar

Alternatively, download the [Oracle JDK](http://www.oracle.com/technetwork/java/javase/downloads/index.html). At the time of writing, the version was 8u121. This requires clicking a license agreement, so you will have to download it to your local machine, then scp it to your target machine.

For example:

    $ scp jdk-6u45-linux-x64-rpm.bin devel@example.com:
    $ ssh devel@example.com
    ...
    $ sudo rpm -i jdk-8u74-linux-x64.rpm

    $ sudo alternatives --install /usr/bin/java java /usr/java/jdk1.8.0_74/jre/bin/java 20000
    $ sudo alternatives --install /usr/bin/javac javac /usr/java/jdk1.8.0_74/bin/javac 20000
    $ sudo alternatives --install /usr/bin/jar jar /usr/java/jdk1.8.0_74/bin/jar 20000
    $ sudo alternatives --config java
    $ sudo alternatives --config javac
    $ sudo alternatives --config jar
	
**Install ActiveMQ**

ActiveMQ is the messaging component used by the system. For more informaiton see the [Apache ActiveMQ](http://activemq.apache.org/) site.

    $ wget http://archive.apache.org/dist/activemq/apache-activemq/5.8.0/apache-activemq-5.8.0-bin.tar.gz
    $ tar -xvzf apache-activemq-5.8.0-bin.tar.gz
    $ sudo mv apache-activemq-5.8.0 /opt
    $ sudo ln -s /opt/apache-activemq-5.8.0 /opt/activemq
    $ sudo chown -R devel:devel /opt/activemq
	
### Set the Environment Variables

**On the Server**

Edit `.bashrc` and add the following:

    export RAILS_ENV=YOUR_ENV # e.g. staging, production
    export ACTIVEMQ_HOME=/opt/activemq
    export CATALINA_HOME=/opt/tomcat
    export SOLR_HOME=/opt/solr

The following configuration files are in the Rails project config.

    hcsvlab / activemq_conf / activemq.xml      ---->  $ACTIVEMQ_HOME/conf/

            / tomcat_conf / setenv.sh           ---->  $CATALINA_HOME/bin/

            / solr_conf / hcsvlab               ---->  $SOLR_HOME/
                        / hcsvlab-solr.xml      ---->  $CATALINA_HOME/conf/Catalina/localhost/solr.xml
  
They need to be copied to the specified locations. This is performed by Capistrano as part of the `full_redeploy` task.

### Deployment

Deployment is done from another machine to the target server setup in the steps above. The deployment machine can be your local desktop or any other machine.

**Install RVM on Deployment Machine**

    $ \curl -#L https://get.rvm.io | bash -s stable --autolibs=3 --ruby=2.1.4
    $ source /home/devel/.rvm/scripts/rvm

**Download the WebApp**

On your deployment machine (which is probably your local machine, but can be another machine) clone the web app:

    $ git clone git@github.com:Alveo/hcsvlab.git
    
**Create a Gemset and Download Gems**

    $ cd projects/hcsvlab
    $ rvm use ruby-2.1.4@hcsvlab --create
    $ gem install bundler
    $ bundle

**Edit Configuration to Point to Target Server**

Configure the following files to reference your server. The files should match your Rails environment name (e.g. staging, production).

    config/deploy/production.rb
      - role :web
      - role :app
      - role :db
    config/environments/production.rb
      - config.action_mailer.default_url_options
      - config.galaxy_url
      
Configure the following files if necessary:

    config/broker.yml
    config/database.yml
    config/hcsvlab-web_config.yml
    config/linguistics.yml
    config/solr.yml

When you run `bundle exec cap YOUR_ENV deploy:setup` in the next step, the files listed above will be copied from your deployment machine to a shared directory on the server at /home/devel/hcsvlab-web/shared/files in the same folder structure.

If you update the configuration files on the server, you will have to restart the server for any changes to take effect.

**Deploy**

    $ bundle exec cap YOUR_ENV deploy:setup
    $ bundle exec cap YOUR_ENV deploy:full_redeploy
    $ bundle exec cap YOUR_ENV deploy:create_solr_core # may not be required if you ran the solr ansible playbook
    $ bundle exec cap YOUR_ENV deploy:configure_activemq
    $ bundle exec cap YOUR_ENV deploy:start_activemq
    $ bundle exec cap YOUR_ENV deploy:start_services
    $ bundle exec cap YOUR_ENV deploy:migrate
    $ bundle exec cap YOUR_ENV deploy:refresh_db

If you ever need to redeploy, make sure you run the following command to stop the services (ActiveMQ, Tomcat, messaging pollers) first: 

    $ bundle exec cap YOUR_ENV deploy:stop_services

### Verifying the deployment, aka running the "Smoke Test" - This must be conducted for each release to Production

There are several steps to this, but together they exercise all key parts of the deployment and, if passed, give a good degree of confidence that the deployment has worked.

**System Check script**

There is a script that is deployed with the web application that can be used to verify that the various processed of the deployment are running and configured to the correct port. To run the script, from the web application's directory type
    
    $ script/system_check.sh
    
The output should look like:

    Checking HCS vLab environment

    Rails env= YOUR_ENV
    Java Container url= http://localhost:8080/
    Web App url= http://localhost:80/
    Free disk space= 19G
    
    Checking ActiveMQ...
    + ActiveMQ is listening on port 8161 (status= 200)
    + ActiveMQ is listening on port 61616
    + ActiveMQ is listening on port 61613
    
    Checking the Java Container...
    + The Java Container is listening on port 8080 (status= 302)
    + It looks like Solr is available (status= 200)
    
    Checking A13g pollers...
    + It looks like the A13g pollers are running (processes= 3)
    
    Checking the web app...
    + The Web App is listening on port 80 (status= 200)

**Version Number**

Verify that the version number you see in the bottom right-hand corner of the webapp is the version you expect.

**E-mail Notifications**

This test will verify that the system can send e-mails as required. It requires you to have access to the e-mail inbox of a user who is registered in the system.

If you are logged in to the webapp, then log out. Navigate to the login page and click on the "Forgot your Password?" link, which will take you to the "Forgot Your Password?" page. Once there, fill in the e-mail address of the user and click the "Send me reset password instructions" button, which will send an e-mail to the user. Check that this e-mail is received.

**Sample Ingest**

See the section "Ingesting Data", below, for a description of what ingesting is in the context of HCSvLab. Here we are concerned with verifying that the process will work, and thus that the system can accept data properly.

First identify a small data collection which can be processed quickly. These instructions assume that this is in directory /data/qa/small_collection. The sample data collection must also come with a description of the collection, which in this case would be small_collection.n3 in the directory /data/qa. These steps cover ingesting not only this sample data but also the sample data licences which come with the application. See the section "Ingesting Data" for important information about setting the data owner for a collection.

Ingest the licences with the command:

    $ rake fedora:ingest_licences

The output should look like:

    Licence 'AusNC Terms of Use' = hcsvlab:1
    Licence 'AusTalk Terms of Use' = hcsvlab:2
    Licence 'AVOZES Non-commercial (Academic) Licence' = hcsvlab:3
    Licence 'Creative Commons v3.0 BY-NC-ND' = hcsvlab:4
    Licence 'Creative Commons v3.0 BY-NC-SA' = hcsvlab:5
    Licence 'Creative Commons v3.0 BY-NC' = hcsvlab:6
    Licence 'Creative Commons v3.0 BY-ND' = hcsvlab:7
    Licence 'Creative Commons v3.0 BY-SA' = hcsvlab:8
    Licence 'Creative Commons v3.0 BY' = hcsvlab:9
    Licence 'PARADISEC Conditions of Access' = hcsvlab:10
    Shutdown completed cleanly

Then, ingest the sample corpus:

    $ rake fedora:ingest corpus=/data/qa/small_collection

Examine the console output and the log file (log/ingest_small_collection.log) to verify that all is well.

**Setting Licence Terms**

Log in to the webapp as the data-owner of the corpus just ingested; click on the username at the top right of the webapp and in the drop-down which appears, click on "Admin". When the Admin page loads, click on "Manage Licences" in the list of tasks at the left of the page. The resulting page should show, among other things, a table titled "Group Collections into Collection Lists" which will have a line for the collection just ingested. In the "Licence" column of the table, click the drop-down and select a licence to assign to the collection. The table will update to show the licence just assigned.

Log out and then log in as another user. You should arrive at the webapp's home page, which should show no Items but should show a message directing you to the Licence Agreements page. Click on the provided Licence Agreements link. The resultant page should show a table titled "Review and Acceptance of Licence Terms" which will have a line for the collection to which a licence has just been assigned. Click on the button in the "Actions" column of the collection's table row and Accept the licence on the dialogue which pops up. The table should now show that the user has agreed to the licence terms for the collection.

Now go to the Discover page by clicking on the "Discover" link in the gold banner of the webapp. You should now see a list of search facets at the left of the page. Confirm that the collection is searchable by: clicking on "Collection" to expand that facet and then clicking on the name of the test collection. The page should show a table of the Items in the collection.

**Sample Item List & Galaxy check**

*Test 1.* From the Discover tab, do a search for "Test". Select "Add all to List" -> "Create New List" -> & enter Item list name: "Search Test all" & select "Create List". User should land on the Item list page with "Search Test all" highlighted & ~209 Items in list. Select a different Item List (any). Whilst timing the response, select the previous Item list "Search Test all". List should load in LESS THAN 1 SECOND. 

*Test 2.* From there, click on "Item List Actions" and select "Use in Galaxy". Verify that the application opens a new tab to the Galaxy application production site at http://galaxy.alveo.edu.au/ with the previous Item Lists' url in the "Item List URL" field, your user's API Key auto filled in the "API Key" field, and "Test Search All (DD/MM/20YY HH:MM:SS am/pm)" in the "Supply a name..." field. Select "Execute". History Entries should show as green.

*Test 3.* From the Discover tab, filter by Language (ISO 639-3 Code) = "Eng". Select "Add all to List" -> "Create New List" -> & enter Item list name: "English all" & select "Create List". User should land on the Item list page with "English all" highlighted & ~12910 Items in list. Select a different Item List (any). Whilst timing the response, select the previous Item list "Search Test all". List should load in LESS THAN 4 SECONDS (Benchmark from Production 22.1.2016).

**EOPAS Connection** - (NOTE: NOT IN PRODUCTION ALVEO AT 20.1.2016)

This test is only possible if suitable data has been ingested. There are suitable Items in the qa_eopas collection.

Find an item which has a single video or audio file and an accompanying transcript and go to its Item details, the "View in Eopas" button should be available. Click on this button and verify that you are navigated to the Eopas page with the file opened in Eopas.

**Working With Data**

Using the "Type" facet on the "Discover" page, locate any Item with a text document. Click through to the Item's details page and verify clicking on the text document's name in the "Documents" table at the bottom of the page shows the required document. Do this for an audio and a video document.

**Tidying Up**

If the data used for testing should be deleted rather than left in the running system, then remove it with the command:

    $ rake fedora:clear_corpus corpus=small_collection

(substituting the actual corpus name should it not be "small_collection")

### Ingesting Data

Ingesting is the task of adding data to the system, which involves several processes such as: verifying and storing metadata and files, and creating indices for search.

Corpora are prepared for ingest using [RoboChef](https://github.com/IntersectAustralia/hcsvlab_robochef), which takes heterogeneous metadata and normalises it, sets the URLs for the documents, and describes them in RDF. Preprepared data from another source can be ingested, but the URLs will all point to the host that was configured when it was RoboCheffed. The system requires that each collection has a metadata file which contains a Notation3 RDF description of the collection. This file should be named <collection-name>.n3 and reside one folder up from the collection's metadata files. The contents of the.n3 file for the AVOZES collection are:

    @prefix avozes: <http://ns.ausnc.org.au/corpus/AVOZES/AVOZES> .
    @prefix dcmitype: <http://purl.org/dc/dcmitype/> .
    @prefix dc: <http://purl.org/dc/elements/1.1/> .
    @prefix dcterms: <http://purl.org/dc/terms/> .
    @prefix cld: <http://purl.org/cld/terms/> .
    @prefix marcrel: <http://www.loc.gov/loc.terms/relators/> .

    avozes:
        a dcmitype:Collection ;
        dc:title "The Audio-Video Australian English Speech Data Corpus" ;
        dcterms:alternative "AVOZES" ;
        dcterms:abstract "AVOZES is an audio-video (or auditory-visual) speech data corpus for Australian English. The AVOZES data corpus was designed and recorded with two major goals in mind. Firstly, a new framework for the design of comprehensive, well-structured, multiple-use AV speech data corpora was proposed and followed in the production of the AVOZES data corpus. Secondly, the first publicly available, comprehensive AV speech data corpus for Australian English (AuE) was produced. In addition, it is the first AV speech data corpus to use a stereo vision system." ;
    .

The collection description must also specify which system user is the owner of the data. The data owner of a collection is the only person who can set the licence under which the collection's data can be accessed. Until the data owner sets the licence for a collection, data in that collection will not be visible to other users of the system.  If no data owner is set, the data will not be visible to any user in the system. To set the data owner for a collection, the .n3 file should specify the property http://id.loc.gov/vocabulary/relators/rpy, and should give the login-name (e-mail address) of the user you wish to be the data owner. This user must have a role in the system of either data-owner or hcsvlab-admin.

The command to ingest a corpus is `rake fedora:ingest`, which must be run from the web application's directory, and the path to the directory where the corpus and data files are located. For example, if you wanted to ingest the ace corpus, and it was stored in `/data/processed/ausnc/ace` then the command would be:

    
    $ rake fedora:ingest corpus=/data/processed/ausnc/ace


