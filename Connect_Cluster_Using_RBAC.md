# Connect Cluster Using RBAC

**Prerequisites:**

- The user completing this procedure must have permission to create role bindings for the Connect service principal, on the Connect cluster, and on the Kafka cluster.
- An installed RBAC-enabled Confluent Platform environment with an installed and running Kafka cluster.
- The Confluent CLI installed.
- Authorization to create and modify principals for the organization.

********File permission Change command********

```bash
setfacl -R -m u:<user>:rwx <dir path>
```

## Configurations

### connect-distributed.properties

```bash
##
# Licensed to the Apache Software Foundation (ASF) under one or more
# contributor license agreements.  See the NOTICE file distributed with
# this work for additional information regarding copyright ownership.
# The ASF licenses this file to You under the Apache License, Version 2.0
# (the "License"); you may not use this file except in compliance with
# the License.  You may obtain a copy of the License at
#
#    http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.
##

# This file contains some of the configurations for the Kafka Connect distributed worker. This file is intended
# to be used with the examples, and some settings may differ from those used in a production system, especially
# the `bootstrap.servers` and those specifying replication factors.

# A list of host/port pairs to use for establishing the initial connection to the Kafka cluster.
bootstrap.servers=ec2-34-201-67-56.compute-1.amazonaws.com:9093
security.protocol=SASL_SSL
ssl.truststore.location=/etc/kafka/truststore.jks
ssl.truststore.password=confluent
ssl.endpoint.identification.algorithm=

# unique name for the cluster, used in forming the Connect cluster group. Note that this must not conflict with consumer group IDs
group.id=sathish-connect-cluster

# The converters specify the format of data in Kafka and how to translate it into Connect data. Every Connect user will
# need to configure these based on the format they want their data in when loaded from or stored into Kafka
key.converter=org.apache.kafka.connect.json.JsonConverter
value.converter=org.apache.kafka.connect.json.JsonConverter
# Converter-specific settings can be passed in by prefixing the Converter's setting with the converter we want to apply
# it to
key.converter.schemas.enable=false
value.converter.schemas.enable=false

# Topic to use for storing offsets. This topic should have many partitions and be replicated and compacted.
# Kafka Connect will attempt to create the topic automatically when needed, but you can always manually create
# the topic before starting Kafka Connect if a specific topic configuration is needed.
# Most users will want to use the built-in default replication factor of 3 or in some cases even specify a larger value.
# Since this means there must be at least as many brokers as the maximum replication factor used, we'd like to be able
# to run this example on a single-broker cluster and so here we instead set the replication factor to 1.
offset.storage.topic=sathish-connect-offsets
offset.storage.replication.factor=3
offset.storage.partitions=5

# Topic to use for storing connector and task configurations; note that this should be a single partition, highly replicated,
# and compacted topic. Kafka Connect will attempt to create the topic automatically when needed, but you can always manually create
# the topic before starting Kafka Connect if a specific topic configuration is needed.
# Most users will want to use the built-in default replication factor of 3 or in some cases even specify a larger value.
# Since this means there must be at least as many brokers as the maximum replication factor used, we'd like to be able
# to run this example on a single-broker cluster and so here we instead set the replication factor to 1.
config.storage.topic=sathish-connect-configs
config.storage.replication.factor=3

# Topic to use for storing statuses. This topic can have multiple partitions and should be replicated and compacted.
# Kafka Connect will attempt to create the topic automatically when needed, but you can always manually create
# the topic before starting Kafka Connect if a specific topic configuration is needed.
# Most users will want to use the built-in default replication factor of 3 or in some cases even specify a larger value.
# Since this means there must be at least as many brokers as the maximum replication factor used, we'd like to be able
# to run this example on a single-broker cluster and so here we instead set the replication factor to 1.
status.storage.topic=sathish-connect-status
status.storage.replication.factor=3
status.storage.partitions=5

# Flush much faster than normal, which is useful for testing/debugging
offset.flush.interval.ms=10000

# List of comma-separated URIs the REST API will listen on. The supported protocols are HTTP and HTTPS.
# Specify hostname as 0.0.0.0 to bind to all interfaces.
# Leave hostname empty to bind to default interface.
# Examples of legal listener lists: HTTP://myhost:8083,HTTPS://myhost:8084"
#listeners=HTTP://:8083
# Or SASL_SSL if using SSL
sasl.mechanism=PLAIN
sasl.jaas.config=org.apache.kafka.common.security.plain.PlainLoginModule required \
  username="connectAdmin" \
  password="connectAdmin" ;

producer.bootstrap.servers=ec2-34-201-67-56.compute-1.amazonaws.com:9093
producer.security.protocol=SASL_SSL
producer.ssl.truststore.location=/etc/kafka/truststore.jks
producer.ssl.truststore.password=confluent
producer.ssl.keystore.location=/etc/kafka/producer.keystore.jks
producer.ssl.keystore.password=confluent
producer.ssl.key.password=p12pass
producer.ssl.endpoint.identification.algorithm=
producer.sasl.mechanism=PLAIN
producer.sasl.jaas.config=org.apache.kafka.common.security.plain.PlainLoginModule required \
  username="connectAdmin" \
  password="connectAdmin" ;

consumer.bootstrap.servers=ec2-34-201-67-56.compute-1.amazonaws.com:9093
consumer.security.protocol=SASL_SSL
consumer.ssl.truststore.location=/etc/kafka/truststore.jks
consumer.ssl.truststore.password=confluent
consumer.ssl.keystore.location=/etc/kafka/producer.keystore.jks
consumer.ssl.keystore.password=confluent
consumer.ssl.key.password=p12pass
consumer.ssl.endpoint.identification.algorithm=
consumer.sasl.mechanism=PLAIN
consumer.sasl.jaas.config=org.apache.kafka.common.security.plain.PlainLoginModule required \
  username="connectAdmin" \
  password="connectAdmin" ;

# The Hostname & Port that will be given out to other workers to connect to i.e. URLs that are routable from other servers.
# If not set, it uses the value for "listeners" if configured.
rest.advertised.host.name=206.189.136.144
rest.advertised.port=8083
rest.advertiser.listener=http

#metadata
confluent.metadata.basic.auth.user.info=connectAdmin:connectAdmin
confluent.metadata.bootstrap.server.urls=https://ec2-34-201-67-56.compute-1.amazonaws.com:8090
confluent.metadata.http.auth.credentials.provider=BASIC
confluent.metadata.ssl.endpoint.identification.algorithm=
confluent.metadata.ssl.truststore.location=/etc/kafka/truststore.jks
confluent.metadata.ssl.truststore.password=confluent

connector.client.config.override.policy=All

confluent.monitoring.interceptor.topic=_confluent-monitoring

# Set to a list of filesystem paths separated by commas (,) to enable class loading isolation for plugins
# (connectors, converters, transformations). The list should consist of top level directories that include 
# any combination of: 
# a) directories immediately containing jars with plugins and their dependencies
# b) uber-jars with plugins and their dependencies
# c) directories immediately containing the package directory structure of classes of plugins and their dependencies
# Examples: 
# plugin.path=/usr/local/share/java,/usr/local/share/kafka/plugins,/opt/connectors,
plugin.path=/usr/share/java
```

## Command

- **Server Start command**
    
    ```bash
    sudo sytemctl start confluent-kafka-connect
    ```
    
- **Server Status command**
    
    ```bash
    sudo sytemctl status confluent-kafka-connect
    ```
    

## Confluent CLI

- **********Login**********
    
    ```bash
    confluent login --url <cluster metadata url>:<port> --ca-cert-path certs/ca.crt
    ```
    
- ************************************Iam rbac role list************************************
    
    ```bash
    confluent iam rbac role list -o "json"
    ```
    
- **********Get Cluster metadata**********
    
    ```bash
    curl https://<mds public ip>:8090/kafka/v3/clusters --cacert ca.crt --cert kafka.crt --key client.key --user superUser:superUser
    ```
    
- **********RBAC  Role binding to connect offset topic**********
    
    ```bash
    confluent iam rbac role-binding create --kafka-cluster <cluster ID> --role ResourceOwner --principal connectAdmin --resource Topic:sathish-connect-offsets
    ```
    
- **********RBAC  Role binding to connect config topic**********
    
    ```bash
    confluent iam rbac role-binding create --kafka-cluster <cluster ID> --role ResourceOwner --principal connectAdmin --resource Topic:sathish-connect-configs
    ```
    
- **********RBAC  Role binding to connect status topic**********
    
    ```bash
    confluent iam rbac role-binding create --kafka-cluster <cluster ID> --role ResourceOwner --principal connectAdmin --resource Topic:sathish-connect-status
    ```
    
- **********RBAC  Role binding to connect group id**********
    
    ```bash
    confluent iam rbac role-binding create --kafka-cluster <cluster ID> --role ResourceOwner --principal connectAdmin --resource Group:sathish-connect-cluster
    ```