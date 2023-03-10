ARG SEARCH_TAG
FROM alfresco/alfresco-search-services:${SEARCH_TAG}

# COMMON
ARG ALFRESCO_HOSTNAME
ARG SOLR_HOSTNAME
ENV ALFRESCO_HOSTNAME $ALFRESCO_HOSTNAME
ENV SOLR_HOSTNAME $SOLR_HOSTNAME

# Configure Alfresco Service Name
RUN sed -i '/^bash.*/i sed -i "'"s/alfresco.host=localhost/alfresco.host=${ALFRESCO_HOSTNAME}/g"'" ${DIST_DIR}/solrhome/templates/rerank/conf/solrcore.properties\n' \
    ${DIST_DIR}/solr/bin/search_config_setup.sh && \
    sed -i '/^bash.*/i sed -i "'"s/solr.host=localhost/solr.host=${SOLR_HOSTNAME}/g"'" ${DIST_DIR}/solrhome/conf/shared.properties\n' \
    ${DIST_DIR}/solr/bin/search_config_setup.sh

# COMMS
ARG ALFRESCO_COMMS
ENV ALFRESCO_COMMS $ALFRESCO_COMMS

# Configure SOLR cores to run in HTTPs mode from template
RUN if [ "$ALFRESCO_COMMS" == "https" ] ; then \
    sed -i '/^bash.*/i sed -i "'"s/alfresco.secureComms=none/alfresco.secureComms=https/g"'" ${DIST_DIR}/solrhome/templates/rerank/conf/solrcore.properties\n' \
    ${DIST_DIR}/solr/bin/search_config_setup.sh; \
elif [ "$ALFRESCO_COMMS" == "secret" ] ; then \
    sed -i '/^bash.*/i sed -i "'"s/alfresco.secureComms=https/alfresco.secureComms=secret/g"'" ${DIST_DIR}/solrhome/templates/rerank/conf/solrcore.properties\n' \
    ${DIST_DIR}/solr/bin/search_config_setup.sh; \
else \
    sed -i '/^bash.*/i sed -i "'"s/alfresco.secureComms=https/alfresco.secureComms=none/g"'" ${DIST_DIR}/solrhome/templates/rerank/conf/solrcore.properties\n' \
    ${DIST_DIR}/solr/bin/search_config_setup.sh; \
fi

# SSL
ARG TRUSTSTORE_TYPE
ENV TRUSTSTORE_TYPE $TRUSTSTORE_TYPE
ARG KEYSTORE_TYPE
ENV KEYSTORE_TYPE $KEYSTORE_TYPE

# Set mTLS properties
RUN if [ "$ALFRESCO_COMMS" == "https" ] ; then \
  sed -i '/^bash.*/i \
  sed -i "'"s/alfresco.encryption.ssl.keystore.location=.*/alfresco.encryption.ssl.keystore.location=\\\/opt\\\/alfresco-search-services\\\/keystore\\\/ssl-repo-client.keystore/g"'" ${DIST_DIR}/solrhome/templates/rerank/conf/solrcore.properties && \
  sed -i "'"s/alfresco.encryption.ssl.keystore.passwordFileLocation=.*/alfresco.encryption.ssl.keystore.passwordFileLocation=/g"'" ${DIST_DIR}/solrhome/templates/rerank/conf/solrcore.properties && \
  sed -i "'"s/alfresco.encryption.ssl.keystore.type=.*/alfresco.encryption.ssl.keystore.type=${KEYSTORE_TYPE}/g"'" ${DIST_DIR}/solrhome/templates/rerank/conf/solrcore.properties && \
  sed -i "'"s/alfresco.encryption.ssl.truststore.location=.*/alfresco.encryption.ssl.truststore.location=\\\/opt\\\/alfresco-search-services\\\/keystore\\\/ssl-repo-client.truststore/g"'" ${DIST_DIR}/solrhome/templates/rerank/conf/solrcore.properties && \
  sed -i "'"s/alfresco.encryption.ssl.truststore.passwordFileLocation=.*/alfresco.encryption.ssl.truststore.passwordFileLocation=/g"'" ${DIST_DIR}/solrhome/templates/rerank/conf/solrcore.properties && \
  sed -i "'"s/alfresco.encryption.ssl.truststore.type=.*/alfresco.encryption.ssl.truststore.type=${TRUSTSTORE_TYPE}/g"'" ${DIST_DIR}/solrhome/templates/rerank/conf/solrcore.properties' \
  ${DIST_DIR}/solr/bin/search_config_setup.sh; \
fi

# Configuration arguments passed from Docker Compose service definition
ARG SOLR_HOSTNAME
ARG ENABLE_MASTER
ARG ENABLE_SLAVE
ARG MASTER_HOST
ENV SOLR_HOSTNAME $SOLR_HOSTNAME
ENV ENABLE_MASTER $ENABLE_MASTER
ENV ENABLE_SLAVE $ENABLE_SLAVE
ENV MASTER_HOST $MASTER_HOST

# Set Master / Slave configuration for this Node
RUN sed -i '/^bash.*/i echo "\nenable.master=${ENABLE_MASTER}\nenable.slave=${ENABLE_SLAVE}" >> ${DIST_DIR}/solrhome/templates/rerank/conf/solrcore.properties\n' \
    ${DIST_DIR}/solr/bin/search_config_setup.sh

# Apply configuration for Master SOLR Node
RUN if [ "$ENABLE_MASTER" = "true" ] ; then \
sed -i "/^bash.*/i sed -i '/^\\\\\s*<requestHandler name=\"\\\\/replication\".*/a \
    <lst name=\"master\">\
      <str name=\"replicateAfter\">commit</str>\
      <str name=\"replicateAfter\">startup</str>\
      <str name=\"confFiles\">schema.xml,stopwords.txt</str>\
    </lst>' ${DIST_DIR}/solrhome/templates/rerank/conf/solrconfig.xml\n" ${DIST_DIR}/solr/bin/search_config_setup.sh; \
# Apply configuration for Slave SOLR Node
else \
sed -i "/^bash.*/i sed -i '/^\\\\\s*<requestHandler name=\"\\\\/replication\".*/a \
    <lst name=\"slave\">\
      <str name=\"masterUrl\">https://${MASTER_HOST}:8983/solr/alfresco</str>\
      <str name=\"pollInterval\">00:00:60</str>\
    </lst>' ${DIST_DIR}/solrhome/templates/rerank/conf/solrconfig.xml\n" ${DIST_DIR}/solr/bin/search_config_setup.sh; \
fi