# Three Node RBAC Cluster

# Installation

- login each instance at a time and copy install.sh file using executable file

### instance.sh

```jsx
#!/bin/bash
echo "server Enter the file path: "
read file_path
echo "server 1 Enter the public host ip: "
read public_ip1
echo "server 2 Enter the public host ip: "
read public_ip2
echo "server 3 Enter the public host ip: "
read public_ip3

#ssh -i $1 azureuser@$2
#$ cat myscript
gnome-terminal --tab -e "ssh -i ${file_path} ubuntu@${public_ip1} -y " --tab -e "ssh -i ${file_path} ubuntu@${public_ip2} -y" --tab -e "ssh -i ${file_path} ubuntu@${public_ip3} -y"

scp -i ${file_path} install.sh ubuntu@${public_ip1}:
scp -i ${file_path} install.sh ubuntu@${public_ip2}:
scp -i ${file_path} install.sh ubuntu@${public_ip3}:
```

- Run the install.sh in each instance

### Install.sh

```jsx
sudo su <<EOF

wget -qO - https://packages.confluent.io/deb/7.4/archive.key | sudo apt-key add -

sudo add-apt-repository "deb [arch=amd64] https://packages.confluent.io/deb/7.4 stable main"

sudo add-apt-repository "deb https://packages.confluent.io/clients/deb $(lsb_release -cs) main"

sudo apt-get update

sudo apt-get install confluent-platform -y 

sudo apt-get install confluent-security -y

apt-get update

apt-get upgrade

apt install openjdk-17-jdk openjdk-17-jre -y 

apt install acl -y

setfacl -R -m u:cp-kafka-connect:rwx /var/log/kafka
EOF
```

- **Commands**
    
    ```jsx
    # For login to all instance
    instance.sh
    # Each instance on inside run following command the file install the confluent platform in each node
    install.sh
    ```
    

## Configurations

### server.properties

```bash
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

#
# This configuration file is intended for use in ZK-based mode, where Apache ZooKeeper is required.
# See kafka.server.KafkaConfig for additional details and defaults
#

############################# Server Basics #############################

# The id of the broker. This must be set to a unique integer for each broker.
broker.id=3

############################# Socket Server Settings #############################

# The address the socket server listens on. If not configured, the host name will be equal to the value of
# java.net.InetAddress.getCanonicalHostName(), with PLAINTEXT listener name, and port 9092.
#   FORMAT:
#     listeners = listener_name://host_name:port
#   EXAMPLE:
#     listeners = PLAINTEXT://your.host.name:9092
#listeners=PLAINTEXT://:9092
listeners=INTERNAL://:9091,EXTERNAL://:9092,CLIENT://:9093
advertised.listeners=INTERNAL://44.212.75.101:9091,EXTERNAL://44.212.75.101:9092,CLIENT://44.212.75.101:9093

# Listener name, hostname and port the broker will advertise to clients.
# If not set, it uses the value for "listeners".
#advertised.listeners=PLAINTEXT://your.host.name:9092

inter.broker.listener.name=INTERNAL
# Maps listener names to security protocols, the default is for them to be the same. See the config documentation for more details
listener.security.protocol.map=INTERNAL:PLAINTEXT,EXTERNAL:PLAINTEXT,CLIENT:SASL_PLAINTEXT
# List of enabled mechanisms, can be more than one
sasl.enabled.mechanisms=OAUTHBEARER

# Specify one of of the SASL mechanisms
#sasl.mechanism.inter.broker.protocol=PLAIN

#ssl.client.auth=required

#ssl.truststore.location=/var/private/ssl/kafka.server.truststore.jks
#ssl.truststore.password=test1234
#ssl.keystore.location=/var/private/ssl/kafka.server.keystore.jks
#ssl.keystore.password=test1234
#ssl.key.password=test1234

#sasl
listener.name.client.sasl.enabled.mechanisms=OAUTHBEARER
#listener.name.client.sasl.enabled.mechanism=PLAIN
listener.name.client.oauthbearer.sasl.login.callback.handler.class=io.confluent.kafka.server.plugins.auth.token.TokenBearerServerLoginCallbackHandler
listener.name.client.oauthbearer.sasl.server.callback.handler.class=io.confluent.kafka.server.plugins.auth.token.TokenBearerValidatorCallbackHandler

listener.name.client.oauthbearer.sasl.jaas.config=org.apache.kafka.common.security.oauthbearer.OAuthBearerLoginModule required \
              publicKeyPath="/home/ubuntu/public.pem" ;
super.users=User:admin,User:mds,User:ANONYMOUS
# MDS Server Settings
confluent.metadata.topic.replication.factor=1

# MDS Token Service Settings
confluent.metadata.server.token.key.path=/home/ubuntu/private.pem

# Configure the RBAC Metadata Service authorizer
authorizer.class.name=io.confluent.kafka.security.authorizer.ConfluentServerAuthorizer
confluent.authorizer.access.rule.providers=CONFLUENT,ZK_ACL

# Bind Metadata Service HTTP service to port 8090
#confluent.metadata.server.listeners=http://44.212.75.101:8090
confluent.metadata.server.listeners=http://0.0.0.0:8090
# Configure HTTP service advertised hostname. Set this to http://127.0.0.1:8090 if running locally.
confluent.metadata.server.advertised.listeners=http://44.212.75.101:8090
# Supported authentication methods
confluent.metadata.server.authentication.method=BEARER
confluent.metadata.server.user.store=FILE
confluent.metadata.server.user.store.file.path=/tmp/login.properties
allow.everyone.if.no.acl.found=true

############################# Identity Provider Settings (LDAP) #############################
# Search groups for group-based authorization.
#ldap.group.name.attribute=memberUid
#ldap.group.object.class=group
#ldap.group.member.attribute=cn
#ldap.group.object.class=posixGroup      
#ldap.group.member.attribute.pattern=cn=(.*),ou=users,dc=confluentdemo,dc=io
#ldap.group.search.base=ou=groups,dc=confluentdemo,dc=io
#Limit the scope of searches to subtrees off of base
#ldap.user.search.scope=2
#Enable filters to limit search to only those groups needed
#ldap.group.search.filter=(|(CN=<specific group>)(CN=<specific group>))

# Kafka authenticates to the directory service with the bind user.
#ldap.java.naming.provider.url=http://ec2-54-236-7-37.compute-1.amazonaws.com:389
#ldap.java.naming.security.authentication=simple
#ldap.java.naming.security.credentials=admin
#ldap.java.naming.security.principal=cn=admin,dc=confluentdemo,dc=io

# Locate users. Make sure that these attributes and object classes match what is in your directory service.
#ldap.user.name.attribute=uid
#ldap.user.object.class=inetOrgPerson
#ldap.user.search.base=ou=users,dc=confluentdemo,dc=io

# The number of threads that the server uses for receiving requests from the network and sending responses to the network
num.network.threads=3

# The number of threads that the server uses for processing requests, which may include disk I/O
num.io.threads=8

# The send buffer (SO_SNDBUF) used by the socket server
socket.send.buffer.bytes=102400

# The receive buffer (SO_RCVBUF) used by the socket server
socket.receive.buffer.bytes=102400

# The maximum size of a request that the socket server will accept (protection against OOM)
socket.request.max.bytes=104857600

############################# Log Basics #############################

# A comma separated list of directories under which to store log files
log.dirs=/var/lib/kafka

# The default number of log partitions per topic. More partitions allow greater
# parallelism for consumption, but this will also result in more files across
# the brokers.
num.partitions=3

# The number of threads per data directory to be used for log recovery at startup and flushing at shutdown.
# This value is recommended to be increased for installations with data dirs located in RAID array.
num.recovery.threads.per.data.dir=3

############################# Internal Topic Settings  #############################
# The replication factor for the group metadata internal topics "__consumer_offsets" and "__transaction_state"
# For anything other than development testing, a value greater than 1 is recommended to ensure availability such as 3.
offsets.topic.replication.factor=1
transaction.state.log.replication.factor=1
transaction.state.log.min.isr=1

############################# Log Flush Policy #############################

# Messages are immediately written to the filesystem but by default we only fsync() to sync
# the OS cache lazily. The following configurations control the flush of data to disk.
# There are a few important trade-offs here:
#    1. Durability: Unflushed data may be lost if you are not using replication.
#    2. Latency: Very large flush intervals may lead to latency spikes when the flush does occur as there will be a lot of data to flush.
#    3. Throughput: The flush is generally the most expensive operation, and a small flush interval may lead to excessive seeks.
# The settings below allow one to configure the flush policy to flush data after a period of time or
# every N messages (or both). This can be done globally and overridden on a per-topic basis.

# The number of messages to accept before forcing a flush of data to disk
#log.flush.interval.messages=10000

# The maximum amount of time a message can sit in a log before we force a flush
#log.flush.interval.ms=1000

############################# Log Retention Policy #############################

# The following configurations control the disposal of log segments. The policy can
# be set to delete segments after a period of time, or after a given size has accumulated.
# A segment will be deleted whenever *either* of these criteria are met. Deletion always happens
# from the end of the log.

# The minimum age of a log file to be eligible for deletion due to age
log.retention.hours=168

# A size-based retention policy for logs. Segments are pruned from the log unless the remaining
# segments drop below log.retention.bytes. Functions independently of log.retention.hours.
#log.retention.bytes=1073741824

# The maximum size of a log segment file. When this size is reached a new log segment will be created.
#log.segment.bytes=1073741824

# The interval at which log segments are checked to see if they can be deleted according
# to the retention policies
log.retention.check.interval.ms=300000

############################# Zookeeper #############################

# Zookeeper connection string (see zookeeper docs for details).
# This is a comma separated host:port pairs, each corresponding to a zk
# server. e.g. "127.0.0.1:3000,127.0.0.1:3001,127.0.0.1:3002".
# You can also append an optional chroot string to the urls to specify the
# root directory for all kafka znodes.
zookeeper.connect=44.212.75.101:2181

# Timeout in ms for connecting to zookeeper
zookeeper.connection.timeout.ms=18000

##################### Confluent Metrics Reporter #######################
# Confluent Control Center and Confluent Auto Data Balancer integration
#
# Uncomment the following lines to publish monitoring data for
# Confluent Control Center and Confluent Auto Data Balancer
# If you are using a dedicated metrics cluster, also adjust the settings
# to point to your metrics kakfa cluster.
#metric.reporters=io.confluent.metrics.reporter.ConfluentMetricsReporter
#confluent.metrics.reporter.bootstrap.servers=localhost:9092
#
# Uncomment the following line if the metrics cluster has a single broker
#confluent.metrics.reporter.topic.replicas=1

############################# Group Coordinator Settings #############################

# The following configuration specifies the time, in milliseconds, that the GroupCoordinator will delay the initial consumer rebalance.
# The rebalance will be further delayed by the value of group.initial.rebalance.delay.ms as new members join the group, up to a maximum of max.poll.interval.ms.
# The default value for this is 3 seconds.
# We override this to 0 here as it makes for a better out-of-the-box experience for development and testing.
# However, in production environments the default value of 3 seconds is more suitable as this will help to avoid unnecessary, and potentially expensive, rebalances during application startup.
group.initial.rebalance.delay.ms=0

############################# Confluent Authorizer Settings  #############################

# Uncomment to enable Confluent Authorizer with support for ACLs, LDAP groups and RBAC
#authorizer.class.name=io.confluent.kafka.security.authorizer.ConfluentServerAuthorizer
# Semi-colon separated list of super users in the format <principalType>:<principalName>
#super.users=
# Specify a valid Confluent license. By default free-tier license will be used
#confluent.license=
# Replication factor for the topic used for licensing. Default is 3.
confluent.license.topic.replication.factor=1

# Uncomment the following lines and specify values where required to enable CONFLUENT provider for RBAC and centralized ACLs
# Enable CONFLUENT provider 
#confluent.authorizer.access.rule.providers=ZK_ACL,CONFLUENT
# Bootstrap servers for RBAC metadata. Must be provided if this broker is not in the metadata cluster
#confluent.metadata.bootstrap.servers=PLAINTEXT://127.0.0.1:9092
# Replication factor for the metadata topic used for authorization. Default is 3.
confluent.metadata.topic.replication.factor=1

# Replication factor for the topic used for audit logs. Default is 3.
confluent.security.event.logger.exporter.kafka.topic.replicas=1

# Listeners for metadata server
#confluent.metadata.server.listeners=http://0.0.0.0:8090
# Advertised listeners for metadata server
#confluent.metadata.server.advertised.listeners=http://127.0.0.1:8090

############################# Confluent Data Balancer Settings  #############################

# The Confluent Data Balancer is used to measure the load across the Kafka cluster and move data
# around as necessary. Comment out this line to disable the Data Balancer.
confluent.balancer.enable=true

# By default, the Data Balancer will only move data when an empty broker (one with no partitions on it)
# is added to the cluster or a broker failure is detected. Comment out this line to allow the Data
# Balancer to balance load across the cluster whenever an imbalance is detected.
#confluent.balancer.heal.uneven.load.trigger=ANY_UNEVEN_LOAD

# The default time to declare a broker permanently failed is 1 hour (3600000 ms).
# Uncomment this line to turn off broker failure detection, or adjust the threshold
# to change the duration before a broker is declared failed.
#confluent.balancer.heal.broker.failure.threshold.ms=-1

# Edit and uncomment the following line to limit the network bandwidth used by data balancing operations.
# This value is in bytes/sec/broker. The default is 10MB/sec.
#confluent.balancer.throttle.bytes.per.second=10485760

# Capacity Limits -- when set to positive values, the Data Balancer will attempt to keep
# resource usage per-broker below these limits.
# Edit and uncomment this line to limit the maximum number of replicas per broker. Default is unlimited.
#confluent.balancer.max.replicas=10000

# Edit and uncomment this line to limit what fraction of the log disk (0-1.0) is used before rebalancing.
# The default (below) is 85% of the log disk.
#confluent.balancer.disk.max.load=0.85

# Edit and uncomment these lines to define a maximum network capacity per broker, in bytes per
# second. The Data Balancer will attempt to ensure that brokers are using less than this amount
# of network bandwidth when rebalancing.
# Here, 10MB/s. The default is unlimited capacity.
#confluent.balancer.network.in.max.bytes.per.second=10485760
#confluent.balancer.network.out.max.bytes.per.second=10485760

# Edit and uncomment this line to identify specific topics that should not be moved by the data balancer.
# Removal operations always move topics regardless of this setting.
#confluent.balancer.exclude.topic.names=

# Edit and uncomment this line to identify topic prefixes that should not be moved by the data balancer.
# (For example, a "confluent.balancer" prefix will match all of "confluent.balancer.a", "confluent.balancer.b",
# "confluent.balancer.c", and so on.)
# Removal operations always move topics regardless of this setting.
#confluent.balancer.exclude.topic.prefixes=

# The replication factor for the topics the Data Balancer uses to store internal state.
# For anything other than development testing, a value greater than 1 is recommended to ensure availability.
# The default value is 3.
confluent.balancer.topic.replication.factor=1

################################## Confluent Telemetry Settings  ##################################

# To start using Telemetry, first generate a Confluent Cloud API key/secret. This can be done with
# instructions at https://docs.confluent.io/current/cloud/using/api-keys.html. Note that you should
# be using the '--resource cloud' flag.
#
# After generating an API key/secret, to enable Telemetry uncomment the lines below and paste
# in your API key/secret.
#
#confluent.telemetry.enabled=true
#confluent.telemetry.api.key=<CLOUD_API_KEY>
#confluent.telemetry.api.secret=<CCLOUD_API_SECRET>
```

### zookeeper.properties

```bash
tickTime=2000
dataDir=/var/lib/zookeeper/
clientPort=2181
initLimit=15
syncLimit=10
server.1=172.31.88.208:2888:3888
server.2=172.31.81.48:2888:3888
server.3=172.31.94.82:2888:3888
autopurge.snapRetainCount=3
autopurge.purgeInterval=24
```

## Command

- **Zookeeper Start command**
    
    ```
    sudo sytemctl start confluent-zookeeper
    
    ```
    
- **Server Start command**
    
    ```
    sudo sytemctl start confluent-server
    
    ```
    
- **Server Status command**
    
    ```
    # zookeeper status
    sudo sytemctl status confluent-zookeeper
    
    # server status
    sudo systemctl staus confluent-server
    ```
    
- **Logs directory**
    
    ```
    # Server
    /var/log/kafka/server.log
    
    #zookeeper 
    /var/lip/zookeeper/zookeeper.log
    
    #authorizer
    /var/log/kafka/authorizer-gz.log
    ```
    
- **Log reading command**
    
    ```
    # track writting log 
    tail -f <log dir>
    
    # read last 200 line
    tail -n 200 <log dir>
    ```