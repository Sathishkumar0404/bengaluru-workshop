# Confluent Kafka Certification Creation

### Certificate Authority - creation

```bash
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

```bash
openssl req -config  openssl.cnf -newkey rsa:4096 -keyout server.key -out server.csr -nodes
```

### Create signed server certification

```bash
openssl x509 -req -CA ca.crt -CAkey ca.key -in server.csr -out server.crt -days 365 -CAcreateserial -extensions v3_req -extfile openssl.cnf
```

### Server certification & CA certification adding to text file

```bash
cat server.crt ca.crt > server.chain.txt
```

### Create pkcs12 format keystore file using preview created text file

```bash
openssl pkcs12 -export -in server.chain.txt -inkey server.key -out server.keystore.p12 -passout pass:p12pass
```

### Create jks file using the pkcs12 file

```bash
keytool -v -importkeystore -srckeystore server.keystore.p12 -srcstoretype pkcs12 -srcstorepass p12pass -destkeystore server.keystore.jks -deststoretype jks -deststorepass confluent
```

### Client key and Client sign request certification creation

```
openssl req -config  openssl.cnf -newkey rsa:4096 -keyout client.key -out client.csr -nodes
```

### Create singned client certification

```bash
openssl x509 -req -CA ca.crt -CAkey ca.key -in client.csr -out client.crt -days 365 -CAcreateserial -extensions v3_req -extfile openssl.cnf/
```

### Client certification & CA certification adding to text file

```bash
cat client.crt ca.crt > client.chain.txt
```

### Create pkcs12 format keystore file using preview created text file

```bash
openssl pkcs12 -export -in client.chain.txt -inkey client.key -out client.keystore.p12 -passout pass:p12pass
```

### Create jks file using the pkcs12 file

```bash
keytool -v -importkeystore -srckeystore client.keystore.p12 -srcstoretype pkcs12 -srcstorepass p12pass -destkeystore client.keystore.jks -deststoretype jks -deststorepass confluent
```

# create Keystore using CA certification

```bash
keytool -keystore truststore.jks -storepass confluent -storetype jks -alias rootca -import -file ca.crt -no-prompt
```