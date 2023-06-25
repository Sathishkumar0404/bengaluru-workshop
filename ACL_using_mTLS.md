# Kafka ACL/TLS using mTLS

# **Creating TLS Keys and Certificates**

### Certificate Authority - creation

```
openssl req -new -newkey rsa:4096 -days 365 -x509 -keyout ca.key -out ca.crt -nodes

```

### Adding openssl.cnf file

```
[ req ]
distinguished_name  = req_distinguished_name
policy              = policy_match
x509_extensions     = user_crt
req_extensions      = v3_req
[ req_distinguished_name ]
countryName                     = Country Name (2 letter code)
countryName_default             = IN
countryName_min                 = 2
countryName_max                 = 2
stateOrProvinceName             = State or Province Name (full name)
stateOrProvinceName_default     = UNKNOWN
localityName                    = Locality Name (eg, city)
localityName_default            = UNKNOWN
0.organizationName              = Organization Name (eg, company)
0.organizationName_default      = UNKNOWN
organizationalUnitName          = Organizational Unit Name (eg, section)
organizationalUnitName_default  = UNKNOWN
commonName                      = Common Name
commonName_max                  = 64
emailAddress                    = Email Address
emailAddress_max                = 64
[ user_crt ]
nsCertType              = client, server, email
nsComment               = "OpenSSL Generated Certificate"
subjectKeyIdentifier    = hash
authorityKeyIdentifier  = keyid,issuer
[ v3_req ]
basicConstraints        = CA:FALSE
extendedKeyUsage        = serverAuth, clientAuth
subjectAltName          = @alt_names
[alt_names]
DNS.1 = localhost
DNS.2 = sathish.com
DNS.3 = ec2-44-203-168-246.compute-1.amazonaws.com
DNS.4 = ec2-52-90-93-217.compute-1.amazonaws.com
DNS.5 = ec2-54-175-225-159.compute-1.amazonaws.com
DNS.6 = ec2-52-206-30-51.compute-1.amazonaws.com
DNS.7 = ec2-3-82-24-14.compute-1.amazonaws.com
IP.1 = 127.0.0.1
                                                                                                                                                           6,26          Top

```

### Sever key and Server sign request certification creation

```
openssl req -config  openssl.cnf -newkey rsa:4096 -keyout server.key -out server.csr -nodes

```

### Create signed server certification

```
openssl x509 -req -CA ca.crt -CAkey ca.key -in server.csr -out server.crt -days 365 -CAcreateserial -extensions v3_req -extfile openssl.cnf

```

### Server certification & CA certification adding to text file

```
cat server.crt ca.crt > server.chain.txt

```

### Create pkcs12 format keystore file using preview created text file

```
openssl pkcs12 -export -in server.chain.txt -inkey server.key -out server.keystore.p12 -passout pass:p12pass

```

### Create jks file using the pkcs12 file

```
keytool -v -importkeystore -srckeystore server.keystore.p12 -srcstoretype pkcs12 -srcstorepass p12pass -destkeystore server.keystore.jks -deststoretype jks -deststorepass confluent

```

### Client key and Client sign request certification creation

```
openssl req -config  openssl.cnf -newkey rsa:4096 -keyout client.key -out client.csr -nodes

```

### Create singned client certification

```
openssl x509 -req -CA ca.crt -CAkey ca.key -in client.csr -out client.crt -days 365 -CAcreateserial -extensions v3_req -extfile openssl.cnf/

```

### Client certification & CA certification adding to text file

```
cat client.crt ca.crt > client.chain.txt

```

### Create pkcs12 format keystore file using preview created text file

```
openssl pkcs12 -export -in client.chain.txt -inkey client.key -out client.keystore.p12 -passout pass:p12pass

```

### Create jks file using the pkcs12 file

```
keytool -v -importkeystore -srckeystore client.keystore.p12 -srcstoretype pkcs12 -srcstorepass p12pass -destkeystore client.keystore.jks -deststoretype jks -deststorepass confluent

```

# create Keystore using CA certification

```
keytool -keystore truststore.jks -storepass confluent -storetype jks -alias rootca -import -file ca.crt -no-prompt

```

# Configurations

## Broker ( server.properties )

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
broker.id=0

############################# Socket Server Settings #############################

# The address the socket server listens on. If not configured, the host name will be equal to the value of
# java.net.InetAddress.getCanonicalHostName(), with PLAINTEXT listener name, and port 9092.
#   FORMAT:
#     listeners = listener_name://host_name:port
#   EXAMPLE:
#     listeners = PLAINTEXT://your.host.name:9092
listeners=INTERNAL://:9092,BROKER://:9091,EXTERNAL://:9093

# Listener name, hostname and port the broker will advertise to clients.
# If not set, it uses the value for "listeners".
advertised.listeners=INTERNAL://localhost:9092,BROKER://localhost:9091,EXTERNAL://sathish.com:9093

# Maps listener names to security protocols, the default is for them to be the same. See the config documentation for more details
#listener.security.protocol.map=PLAINTEXT:PLAINTEXT,SSL:SSL,SASL_PLAINTEXT:SASL_PLAINTEXT,SASL_SSL:SASL_SSL
listener.security.protocol.map=INTERNAL:PLAINTEXT,BROKER:PLAINTEXT,EXTERNAL:SSL
#security.inter.broker.protocol=PLAINTEXT
#inter.broker.listener.name=BROKER
inter.broker.listener.name=BROKER
listener.name.external.ssl.client.auth=required
listener.name.external.ssl.truststore.location=/etc/kafka/mtls/truststore.jks
listener.name.external.ssl.truststore.password=confluent
listener.name.external.ssl.keystore.location=/etc/kafka/mtls/server.keystore.jks
listener.name.external.ssl.keystore.password=confluent
listener.name.external.ssl.key.password=p12pass

#ACL config
authorizer.class.name=kafka.security.authorizer.AclAuthorizer
super.users=User:admin;User:client;User:ANONYMOUS
listener.name.external.ssl.principal.mapping.rules=RULE:.*CN=([a-zA-Z0-9.]*),.*$/$1/ , DEFAULT

# Listener name, hostname and port the broker will advertise to clients.
# Maps listener names to security protocols, the default is for them to be the same. See the config documentation for more details
#listener.security.protocol.map=PLAINTEXT:PLAINTEXT,SSL:SSL,SASL_PLAINTEXT:SASL_PLAINTEXT,SASL_SSL:SASL_SSL

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
log.dirs=/tmp/kafka-logs

# The default number of log partitions per topic. More partitions allow greater
# parallelism for consumption, but this will also result in more files across
# the brokers.
num.partitions=1

# The number of threads per data directory to be used for log recovery at startup and flushing at shutdown.
# This value is recommended to be increased for installations with data dirs located in RAID array.
num.recovery.threads.per.data.dir=1

############################# Internal Topic Settings  #############################
# The replication factor for the group metadata internal topics "__consumer_offsets" and "__transaction_state"
# For anything other than development testing, a value greater than 1 is recommended to ensure availability such as 3.
confluent.tier.metadata.replication.factor=1
confluent.license.topic.replication.factor=1
confluent.cluster.link.metadata.topic.replication.factor=1
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
zookeeper.connect=localhost:2181

# Timeout in ms for connecting to zookeeper
zookeeper.connection.timeout.ms=18000

############################# Group Coordinator Settings #############################

# The following configuration specifies the time, in milliseconds, that the GroupCoordinator will delay the initial consumer rebalance.
# The rebalance will be further delayed by the value of group.initial.rebalance.delay.ms as new members join the group, up to a maximum of max.poll.interval.ms.
# The default value for this is 3 seconds.
# We override this to 0 here as it makes for a better out-of-the-box experience for development and testing.
# However, in production environments the default value of 3 seconds is more suitable as this will help to avoid unnecessary, and potentially expensive, rebalances during application startup.
group.initial.rebalance.delay.ms=0
```

## Zookeeper ( zookeeper.properties )

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
#zookeeper.log.file=/var/log/kafka/zookeeper.log
# the directory where the snapshot is stored.
dataDir=/var/lib/zookeeper
# the port at which the clients will connect
clientPort=2181
# disable the per-ip limit on the number of connections since this is a non-production config
maxClientCnxns=0
# Disable the adminserver by default to avoid port conflicts.
# Set the port to something non-conflicting if choosing to enable this
admin.enableServer=false
# admin.serverPort=8080

# Enable audit logging. See log4j.properties for audit logger configuration
audit.enable=true
```

## Client ( client.properties )

```bash
security.protocol=SSL
ssl.truststore.location=/home/sathishkumar/bengaluru-workshop/mtls/truststore.jks
ssl.truststore.password=confluent
ssl.keystore.location=/home/sathishkumar/bengaluru-workshop/mtls/client.keystore.jks
ssl.keystore.password=confluent
ssl.key.password=p12pass
```

# Commands

- Server Start Command

```bash
sudo systemctl start confluent-zookeeper
sudo systemctl start confluent-server
```

- List all topics

```bash
kafka-topics --bootstrap-server sathish.com:9093 --command-config client.properties --list
```

- Create user & group

```bash
kafka-acls --bootstrap-server sathish.com:9093 --command-config client.properties --add --allow-principal user:sathish --allow-host sathish.com --group group1 --operation read
```