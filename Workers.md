## Alveo Workers installation

[Alveo Workers](https://github.com/Alveo/alveo-workers) are the primary method of data ingest into Alveo.

The workers can be run on any machine, but they must have network connectivity to your Solr, Sesame, Postgres and RabbitMQ instance.

### Check out the workers code

`git clone https://github.com/Alveo/alveo-workers.git`

### Configure for your network

Review the settings in [config.yml](https://github.com/Alveo/alveo-workers/blob/master/config.yml) and adjust as necessary. Pay particular attention to the following:

* Solr URL, e.g. `http://localhost:8080/solr/hcsvlab`
* RabbitMQ host and port, e.g. `localhost` and `5673` respectively
* Sesame URL, e.g. `http://localhost:8081/openrdf-sesame/`.
  * _Be sure to specify the correct Tomcat port if you ended up installing two Tomcats on the same box for Solr and Sesame!_
* PostgreSQL host, username, password and database.

### Running workers

TODO