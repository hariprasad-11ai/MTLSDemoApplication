# Spring boot MTLS Demo

## Notes
- This is a simple Spring Boot application that demonstrates how to enable Mutual TLS (mTLS) in a Spring Boot application.
- The application is configured to run on port 8443.
- The application is configured to require client authentication.
- localhost:8443/api/test is the endpoint that requires client authentication.
- The endpoint also requires basic authentication. user and password are the username and password respectively.
- Without the client certificate, the application will return an error.

## Curl Commands
- The following curl command will return an error because the client certificate is not provided.
```shell
curl -k https://localhost:8443/api/test
```
- The following curl command will return a success message because the client certificate is provided.
- ### Run this curl command in the src/main/resources directory or use path to the client certificate and key.
```shell
curl -v -k --cert client.crt --key client.key -u user:password https://localhost:8443/api/test
```

# Run the following commands in the src/main/resources directory or use path to the client certificate and key.
## Generate CA Private Key
```shell
openssl genrsa -out ca.key 2048
```

## Generate CA Certificate
```shell
openssl req -x509 -new -nodes -key ca.key -sha256 -days 365 -out ca.crt
```

## Generate Server Private Key
```shell
openssl genrsa -out server.key 2048
```

## Generate Server Certificate Signing Request
```shell
openssl req -new -key server.key -out server.csr
```

## Generate Server Certificate
```shell
openssl x509 -req -in server.csr -CA ca.crt -CAkey ca.key -CAcreateserial -out server.crt -days 365 -sha256
```

## Convert Server Certificate to PKCS12 format
```shell
openssl pkcs12 -export -in server.crt -inkey server.key -out server.p12 -name server-cert -CAfile ca.crt -caname root
```

## Convert PKCS12 to JKS
```shell
keytool -importkeystore -deststorepass password -destkeypass password -destkeystore server-keystore.jks -srckeystore server.p12 -srcstoretype PKCS12 -alias server-cert
```

## Import CA Certificate to JKS
```shell
keytool -import -trustcacerts -alias root -file ca.crt -keystore server-truststore.jks -storepass password
```

## Generate Client Private Key
```shell
openssl genrsa -out client.key 2048
```

## Generate Client Certificate Signing Request
```shell
openssl req -new -key client.key -out client.csr
```
