# Alfresco Search Services using Replication with mTLS

Deployment template based in official [Docker Composition](https://github.com/Alfresco/acs-community-deployment/tree/master/docker-compose) using SOLR Replication with two nodes in Master / Slave configuration with mTLS communication mode.

This configuration provides a dedicated node for *tracking* named `solr6master` and a dedicated node for *searching* name `solr6slave`. Despite this feature is available for both Enterprise and Community versions, documentation is available at [SOLR Replication](https://docs.alfresco.com/search-enterprise/concepts/solr-replication.html).

SOLR Replication with Master / Slave configuration is recommended to fulfil some of following requirements:

* Splits read and write load and operations in SOLR Slaves and SOLR Master
* Load distribution for search queries
* High availability for searching

You should review volumes, configuration, modules & tuning parameters before using this composition in **Production** environments.

## ACS Versions

* ACS 7.3.0
* Search Services 2.0.5

## Project structure

```
.
├── .env
├── alfresco
│   └── Dockerfile
├── config
│   └── nginx.conf
├── docker-compose.yml
├── keystores
│   ├── alfresco
│   │   ├── keystore
│   │   ├── ssl.keystore
│   │   └── ssl.truststore
│   └── solr
│       ├── ssl-repo-client.keystore
│       └── ssl-repo-client.truststore
└── search
    └── Dockerfile
```

* `.env` includes Docker environment variables to set Docker Image release numbers
* `alfresco` folder includes configuration for Alfresco Dockerfile to add mTLS support
* `config` folder includes configuration for HTTP Proxy based in NGINX
* `docker-compose.yml` is a Docker Compose template to use ACS Community with SOLR Replication and mTLS communication mode
* `keystores` folder includes keystores and truststores for mTLS communication mode between alfresco and solr
* `search` folder includes a customised Dockerfile to configure SOLR Docker Image in master and slave mode

## Configuration details

Since the Docker Compose template is providing all the settings to run this configuration, following instructions can be applied to an on premise deployment to provide a dedicated node for *tracking* named `solrmaster` and a dedicated node for *searching* name `solrslave`.

**solr6master**

Add following properties to `solrcore.properties` file:

```
enable.master=true
enable.slave=false
```

Add following lines to `solrconfig.xml` file:

```xml
<requestHandler name="/replication" class="solr.ReplicationHandler" > 
    <lst name="master">
      <str name="replicateAfter">commit</str>
      <str name="replicateAfter">startup</str>
      <str name="confFiles">schema.xml,stopwords.txt</str>
    </lst>
</requestHandler>
```

**solr6slave**

Add following properties to `solrcore.properties` file:

```
enable.master=false
enable.slave=true
```

Add following lines to `solrconfig.xml` file:

```xml
 <requestHandler name="/replication" class="solr.ReplicationHandler" > 
    <lst name="slave">
      <str name="masterUrl">https://solr6master:8983/solr/alfresco</str>
      <str name="pollInterval">00:00:60</str>
    </lst>
 </requestHandler>
```

In addition, mTLS client settings must be added to `solr.in.sh` (Linux) or `solr.in.cmd` (Windows) files:

```
# Slave needs to connect to Master using client mTLS settings
SOLR_SSL_CLIENT_KEY_STORE: "/opt/alfresco-search-services/keystore/ssl-repo-client.keystore"
SOLR_SSL_CLIENT_KEY_STORE_PASSWORD: "keystore"
SOLR_SSL_CLIENT_KEY_STORE_TYPE: "JCEKS"
SOLR_SSL_CLIENT_TRUST_STORE: "/opt/alfresco-search-services/keystore/ssl-repo-client.truststore"
SOLR_SSL_CLIENT_TRUST_STORE_PASSWORD: "truststore"
SOLR_SSL_CLIENT_TRUST_STORE_TYPE: "JCEKS" 
```

# How to use this composition

## Start Docker

Start docker and check the ports are correctly bound.

```bash
$ docker-compose up
```

### Viewing System Logs

You can view the system logs by issuing the following.

```bash
$ docker-compose logs -f
```

Logs for every service are also available at `logs` folder.

## Access

Use the following username/password combination to login.

 - User: admin
 - Password: admin

Alfresco and related web applications can be accessed from the below URIs when the servers have started.

```
http://localhost/alfresco      - Alfresco Repository
http://localhost/share         - Alfresco Share
```

## Contributions

Thanks to [Francesco Papini](https://www.linkedin.com/in/francesco-papini) for helping to build this template.