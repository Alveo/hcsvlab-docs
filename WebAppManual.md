## Manual installation of web application components

In case the [Ansible scripts](https://github.com/alveo/alveo-ansible) aren't working for you, you may be able to follow the following manual installation instructions for certain components.

**Install Tomcat**

Tomcat is the serverlet container that runs SOLR and Sesame. Form more information see the [Apache Tomcat](http://tomcat.apache.org/) site.

    $ curl -O http://apache.mirror.serversaustralia.com.au/tomcat/tomcat-6/v6.0.37/bin/apache-tomcat-6.0.37.tar.gz
    $ tar -xvzf apache-tomcat-6.0.37.tar.gz
    $ sudo mv apache-tomcat-6.0.37 /opt
    $ sudo ln -s /opt/apache-tomcat-6.0.37 /opt/tomcat
    $ sudo chown -R devel:devel /opt/tomcat
	
**Install Solr**

Solr is the search engine platform used by the web application, for more information see the [Apache Solr](http://lucene.apache.org/solr/) site.

    $ ansible-playbook -i hosts-alveo-solr site-alveo-solr.yml

Solr is installed during project deployment, but we must make a directory structure for it.

    $ sudo mkdir -p /opt/solr/hcsvlab/solr/hcsvlab-core/conf
    $ sudo mkdir -p /opt/solr/hcsvlab/solr/hcsvlab-AF-core/conf
    $ sudo chown -R devel:devel /opt/solr

