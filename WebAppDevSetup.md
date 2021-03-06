<!-- MarkdownTOC -->

1. [Developer Setup - Virtual Environment](#developer-setup---virtual-environment)
1. [Developer Setup - Web App](#developer-setup---web-app)
    1. [Install PhantomJS](#install-phantomjs)
    1. [Install Postgres](#install-postgres)
        1. [Create Database User](#create-database-user)
    1. [HCSvLab Development Environment Setup](#hcsvlab-development-environment-setup)
        1. [JAVA](#java)
        1. [Ruby](#ruby)
        1. [ImageMagick \(required for the rmagick gem\)](#imagemagick-required-for-the-rmagick-gem)
        1. [ActiveMQ](#activemq)
        1. [Triplestores setup](#triplestores-setup)
        1. [HCSvLab startup](#hcsvlab-startup)
        1. [Data Directory](#data-directory)
        1. [Setup database for running tests](#setup-database-for-running-tests)
    1. [Run Tests](#run-tests)
    1. [Setup https for AAF](#setup-https-for-aaf)
    1. [Ingest an AusNC Corpus into Fedora \(deprecated\)](#ingest-an-ausnc-corpus-into-fedora-deprecated)
    1. [Ingest a single AusNC Corpus Item into Fedora \(deprecated\)](#ingest-a-single-ausnc-corpus-item-into-fedora-deprecated)
    1. [Clear all corpora from Fedora \(deprecated\)](#clear-all-corpora-from-fedora-deprecated)

<!-- /MarkdownTOC -->


<a name="developer-setup---virtual-environment"></a>
<a id="developer-setup---virtual-environment"></a>
# Developer Setup - Virtual Environment

The HCSvLab web app developer installation instructions assume an OSX or Ubuntu environment. To set up an Ubuntu virtual machine run through the following steps. If planning to use an OSX environment ignore this section and continue at the Developer Setup - Web App section.

**Install Virtualbox**

Virtualbox will be used to set up virtual machines on your computer. It can be found at https://www.virtualbox.org/
A user manual containing installation instructions can be found at https://www.virtualbox.org/manual/UserManual.html however installation should be as simple as downloading and clicking the installer.

**Obtain an Ubuntu image**

An Ubuntu image will need to be provided to virtual box for it to be able to install Ubuntu onto a virtual machine. Either the desktop version http://www.ubuntu.com/download/desktop or server version http://www.ubuntu.com/download/server can be used.

**Create a new Virtual Machine**

Once the Ubuntu image is downloaded open Virtualbox and click to create a new vm. Give it some appropriate name such as HCSvLab, set its type to "Linux" and version to "Ubuntu (64-bit)". Allocate the VM at least 2048 mb of RAM, and use the default options to create a dynamically allocated physical hard disk.

Configure the settings of the VM within Virtualbox to have two network adapters. The first should be a bridged adapter and the second a Host-only adapter. This will allow the vm to access the internet and allow ssh'ing into the vm from the host machine.

To make development easier it is handy to clone the HCSvLab repository onto the host machine (desktop) and set up a shared drive to the cloned repo. This allows code changes to be made to the cloned repo within the desktop and still have the changes take effect within the vm. When creating the shared drive ensure to enable the settings "Make permanent" and "Auto mount".

Once the virtual machine has been created start it up and select the downloaded Ubuntu image file when prompted for a virtual disk. Run through the Ubuntu setup, making sure to give the user a memorable name and password ("devel" is normally used as it aligns with our server environments). Once setup is complete and you have logged in install guest additions onto the vm by clicking "Devices > Insert Guest Additions CD Image" within the Virtualbox menu toolbar and follow the instructions within the VM. Once guest additions are installed add your Ubuntu user to the vboxsf group so that they can access the shared folder:

    $ sudo usermod -aG vboxsf $(whoami)
    
After logging out and in again you should be able to access the shared folder at "/media/sf_<shared folder name>". Setup of the Ubuntu virtual machine should now be complete. 

<a name="developer-setup---web-app"></a>
<a id="developer-setup---web-app"></a>
# Developer Setup - Web App
<a name="install-phantomjs"></a>
<a id="install-phantomjs"></a>
## Install PhantomJS
For more info, visit: http://phantomjs.org/download.html

**In Ubuntu**

    $ cd /usr/local/share/
    $ sudo wget https://phantomjs.googlecode.com/files/phantomjs-1.9.1-linux-x86_64.tar.bz2
    $ sudo tar jxvf phantomjs-1.9.1-linux-x86_64.tar.bz2
    $ sudo ln -s /usr/local/share/phantomjs-1.9.1-linux-x86_64/ /usr/local/share/phantomjs
    $ sudo ln -s /usr/local/share/phantomjs/bin/phantomjs /usr/local/bin/phantomjs

**In Mac OS X**

    brew update && brew install phantomjs

Unfortunately, installing with MacPorts is not recommended.

<a name="install-postgres"></a>
<a id="install-postgres"></a>
## Install Postgres
**In Ubuntu**

    $ sudo apt-get install postgresql libpq-dev # install postgresql and libpq-dev packages
    $ sudo -u postgres psql template1 # login to template1 database
    # \password postgres # change password for user postgres
    # \q # exit

**In Mac OS X**

The easiest way to set up Postgres on Mac OS X is to download from http://postgresapp.com/ and follow the "Installing Postgres.app instructions at http://postgresapp.com/documentation/install.html.

**After installation**

1. Run `echo 'export PATH=$PATH:/Applications/Postgres.app/Contents/Versions/latest/bin' >> ~/.bash_profile` in the terminal. This will give you access to the command line tools.
2. Open Postgres.app from Applications. You should see an elephant icon on the menu bar.
3. Left click on the elephant icon and go the Preferences
4. Uncheck "Show Welcome Window..."
5. Check "Start Postgres automatically after login"

<a name="create-database-user"></a>
<a id="create-database-user"></a>
###Create Database User
**In Ubuntu**

    $ sudo -u postgres createuser -P # create a new user called hcsvlab (enable create databases) with password hcsvlab

If you already have a "postgres" user you may get the following error `$ createuser: creation of new role failed: ERROR:  role "postgres" already exists`. This means that a new user with the name hcsvlab won't be created which can cause the database rake in later steps to result in an error. To resolve this you need to open psql as the postgres user and add a new role with the name "hcsvlab" and the relevant permissions.

    $ sudo su - postgres # log into postgres user
    $ psql # open psql console
    # CREATE ROLE hcsvlab WITH LOGIN CREATEDB;
    # \du
    # \q
    $ exit

**In Mac OS X**

    $ createuser -sP hcsvlab # create superuser and prompt for password

<a name="hcsvlab-development-environment-setup"></a>
<a id="hcsvlab-development-environment-setup"></a>
##HCSvLab Development Environment Setup

<a id="java"></a>
### JAVA

Use brew to install java8 in Mac OS X

    $ brew update
    $ brew tap caskroom/versions
    $ brew cask install java8

<a id="ruby"></a>
### Ruby

If Ruby Version Manager (rvm) is not already installed on the development environment please follow the <a href="https://rvm.io/rvm/install" target="_blank">official instructions</a> to install it. 

**In Ubuntu and Mac OS X**

    $ rvm install ruby-2.1.4
    $ rvm use ruby-2.1.4@hcsvlab \--create
    $ git clone git@github.com:Alveo/hcsvlab.git
    $ gem install bundler
    $ gem install therubyracer -v '0.12.1'
    $ gem install libv8 -v '3.16.14.7' -- --with-system-v8
    $ bundle install 

<a name="imagemagick-required-for-the-rmagick-gem"></a>
<a id="imagemagick-required-for-the-rmagick-gem"></a>
### ImageMagick (required for the rmagick gem)
**In Ubuntu**

    $ sudo apt-get install imagemagick

**In Mac OS X**

    $ brew install imagemagick

<a name="activemq"></a>
<a id="activemq"></a>
### ActiveMQ
**In Ubuntu and Mac OS X**

    $ curl http://archive.apache.org/dist/activemq/apache-activemq/5.8.0/apache-activemq-5.8.0-bin.tar.gz | tar xvz
 
***set configuration, using provided file from HCSvLab project***

    $ cd apache-activemq-5.8.0
    $ cp <hcsvlab folder>/activemq_conf/activemq.xml conf/activemq.xml

If running `$ bin/activemq start` in Ubuntu results in an error message `ERROR: Configuration variable JAVA_HOME or JAVACMD is not defined correctly.`, then it is likely that no Java Runtime Environment is installed. If `$ java -version` doesn't list an installed java version, then install the latest version of the JRE using `$ sudo apt-get install default-jre`.

** Make sure ActiveMQ is running while HCSvLab is up. **

<a name="triplestores-setup"></a>
<a id="triplestores-setup"></a>
###Triplestores setup

**development env**

- `<hcsvlab folder>/config/sesame.yml`

```css
development:
    url: http://localhost:8983/openrdf-sesame
```

- `<hcsvlab folder>/config/solr.yml`
```css
development:
    url: http://localhost:8983/solr/development
```
<a name="hcsvlab-startup"></a>
<a id="hcsvlab-startup"></a>
### HCSvLab startup
**For Ubuntu and Mac OS X**

- For first time run to initialise
```
$ cd ~/hcsvlab
$ git submodule init
$ git submodule update
$ rake db:create db:migrate db:seed db:populate
$ RAILS_ENV=test rake db:migrate
$ rake jetty:reset_all 
```

- For subsequent run
```
$ export OBJC_DISABLE_INITIALIZE_FORK_SAFETY=YES
$ RAILS_ENV=development
$ apache-activemq-5.8.0/bin/activemq stop 
$ apache-activemq-5.8.0/bin/activemq start 
$ rake jetty:start a13g:start_pollers
$ rails s
```

If you use macOs High Sierra you need to check this to make sure solr worker works:

<a href="https://blog.phusion.nl/2017/10/13/why-ruby-app-servers-break-on-macos-high-sierra-and-what-can-be-done-about-it/" target="_blank">Why Ruby app servers break on macOS High Sierra and what can be done about it</a>


As mentioned in the calls above, `$ rake jetty:reset_all` should be run the first time when setting up HCSVLAB. It can also be used to reset Solr and Sesame when required.

To check that the required processes are running, the script `/hcsvlab/script/system_check.sh` can be run.


<a name="data-directory"></a>
<a id="data-directory"></a>
### Data Directory
**For Ubuntu and Mac OS X**

    $ mkdir -p /data/contributed_annotations
    $ mkdir -p /data/contrib
    $ mkdir -p /data/collections
    $ mkdir -p /data/hcsvlab_download
    $ chown -R <user>:<user> /data

<a name="setup-database-for-running-tests"></a>
<a id="setup-database-for-running-tests"></a>
###Setup database for running tests
**For Ubuntu and Mac OS X**

    $ RAILS_ENV=development bundle exec rake db:create db:migrate db:test:prepare

<a name="run-tests"></a>
<a id="run-tests"></a>
##Run Tests
**For Ubuntu and Mac OS X**

    $ cd ~/hcsvlab
    $ rspec spec
    $ cucumber features

Make sure all necessary backend services (ActiveMQ/start_pollers) are up during test, otherwise some tests would fail.   

<a name="setup-https-for-aaf"></a>
<a id="setup-https-for-aaf"></a>
##Setup https for AAF
**For Mac OS X**

For testing purpose, please register your application [here](https://rapid.test.aaf.edu.au/registration), it will ask you to login through aaf. Please select idptest.intersect.org.au

Once you are logged in, please fill in the details

<table>
<tr>
<th>Field</th>
<th>Value</th>
</tr>
<tr>
<td>Organisation</td>
<td>Intersect_test</td>
</tr>
<tr>
<td>Name</td>
<td>Your application name</td>
</tr>
<tr>
<td>URL</td>
<td>Fake application URL e.g. https://hcsvlab.intersect.org.au</td>
</tr>
<tr>
<td>Callback URL</td>
<td>The same URL but append /users/aaf_sign_in to the URL. e.g. https://hcsvlab.intersect.org.au/users/aaf_sign_in</td>
</tr>
<tr>
<td>Secret</td>
<td>Please generate by following command:
    <code>tr -dc '[[:alnum:][:punct:]]' &lt; /dev/urandom | head -c32 ;echo </code>
</td>
</tr>
</table>

Please remember the secret token. Once you submit this, you will get a url. Please also keep the url to be used in your configuration.

**Configure apache2**

This has been tested on OSX 10.9.
Add a new line to the /etc/hosts

    127.0.0.1       hcsvlab.intersect.org.au

Please create the certificate for the ssl, answer all questions

    sudo mkdir -p /etc/ssl/private
    sudo mkdir /etc/ssl/certs
    sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout /etc/ssl/private/hcsvlab.key -out /etc/ssl/certs/hcsvlab.crt

Create a new file in /etc/apache2/ naming it hcsvlab.conf and put the content:

    <VirtualHost *:80>
        ServerName hcsvlab.intersect.org.au
        Redirect permanent / https://hcsvlab.intersect.org.au/
    </VirtualHost>
    <VirtualHost _default_:443>
        ServerName hcsvlab.intersect.org.au
        SSLEngine on
        ErrorLog /var/log/apache2/ssl_error_log
        CustomLog /var/log/apache2/ssl_access_log combined
        CustomLog /var/log/apache2/ssl_request_log \
              "%t %h %{SSL_PROTOCOL}x %{SSL_CIPHER}x 636f4ebbf2a79889b02804364ae277836ade24bfquot;%r636f4ebbf2a79889b02804364ae277836ade24bfquot; %b"
        LogLevel warn
        SetEnvIf User-Agent ".*MSIE.*" \
             nokeepalive ssl-unclean-shutdown \
             downgrade-1.0 force-response-1.0
        SSLProtocol all -SSLv2
        SSLCipherSuite ALL:!ADH:!EXPORT56:RC4+RSA:+HIGH:+MEDIUM:+LOW:+SSLv2:+EXP:+eNULL
        SSLCertificateFile /etc/ssl/certs/hcsvlab.crt
        SSLCertificateKeyFile /etc/ssl/private/hcsvlab.key

        ProxyPreserveHost On
        ProxyPass / http://localhost:3000/
        ProxyPassReverse / http://localhost:3000/
        ProxyRequests On
        <Proxy *>
          Order deny,allow
          Allow from all
        </Proxy>
    </VirtualHost>

Add the following lines to `/etc/apache2/httpd.conf`

    Listen 443
    ServerName hcsvlab.intersect.org.au
    Include /etc/apache2/hcsvlab.conf

Then restart apace

    sudo apachectl -k restart

<a name="ingest-an-ausnc-corpus-into-fedora-deprecated"></a>
<a id="ingest-an-ausnc-corpus-into-fedora-deprecated"></a>
##Ingest an AusNC Corpus into Fedora (deprecated)
For example data, see Download the Corpus in Installing AusNC.

After that, refer to https://github.com/IntersectAustralia/hcsvlab_robochef#installation for more instructions.

    $ rake fedora:ingest corpus=/path/to/corpus/directory

<a name="ingest-a-single-ausnc-corpus-item-into-fedora-deprecated"></a>
<a id="ingest-a-single-ausnc-corpus-item-into-fedora-deprecated"></a>
##Ingest a single AusNC Corpus Item into Fedora (deprecated)

    $ rake fedora:ingest_one /path/to/corpus/rdf/file

<a name="clear-all-corpora-from-fedora-deprecated"></a>
<a id="clear-all-corpora-from-fedora-deprecated"></a>
##Clear all corpora from Fedora (deprecated)

    $ rake fedora:clear
