---
layout: default
title: Self-Signed Certificate Generator Script
parent: Linux
nav_order: 1
---

### How to Create Self-Signed Certificate
```shell
#!/bin/bash

# Reference: https://www.baeldung.com/openssl-self-signed-cert
# Reference: https://devopscube.com/create-self-signed-certificates-openssl/

# create private key and Certificate Signing Request (CSR) using one command (optional)
# openssl req -newkey rsa:2048 -keyout test-dev.key -out test-dev.csr

# create rootCA key and certificates
# openssl req -x509 -sha256 -days 3650 -newkey rsa:2048 -keyout rootCA.key -out rootCA.crt

# Create private key with self-signed certificate using single command
# openssl req -newkey rsa:2048 -keyout test-dev.key -x509 -days 3650 -out test-dev.crt

# Verify certificate
# openssl x509 -text -noout -in server.crt

# Verify Key
# openssl rsa -in server.key -check

# Verify CSR
# openssl req -text -noout -verify -in server.csr

# These two commands print out md5 checksums of the certificate and key; the checksums can be compared to verify that the certificate and key match.
# openssl x509 -noout -modulus -in server.crt| openssl md5
# openssl rsa -noout -modulus -in server.key| openssl md5

# USAGE: bash self-signed-certificate.sh default.svc.cluster.local
# To generate wildcard certificate, bash self-signed-certificate.sh wildcard.test.dev (Change CN= *.${DOMAIN)} and DNS.1/2)
#! /bin/bash

if [ "$#" -ne 1 ]
then
  echo "Error: No domain name argument provided"
  echo "Usage: Provide a domain name as an argument"
  exit 1
fi

DOMAIN=$1

# create a folder for wildcard certificate
CERT_DIR=certificates/${DOMAIN}

mkdir -p ${CERT_DIR}

# Create root CA & Private key

openssl req -x509 \
            -sha256 -days 3560 \
            -nodes \
            -newkey rsa:2048 \
            -subj "/CN=*.${DOMAIN}/C=DE/L=Hamburg" \
            -keyout ${CERT_DIR}/rootCA.key -out ${CERT_DIR}/rootCA.crt

# Generate Private key

openssl genrsa -out ${CERT_DIR}/wildcard-${DOMAIN}.key 2048

# Create csr conf

cat > ${CERT_DIR}/csr.conf <<EOF
[ req ]
default_bits = 2048
prompt = no
default_md = sha256
req_extensions = req_ext
distinguished_name = dn
[ dn ]
C = DE
ST = HH
L = Hamburg
O = Covaxin
OU = C24
CN = *.${DOMAIN}
[ req_ext ]
subjectAltName = @alt_names
[ alt_names ]
DNS.1 = *.${DOMAIN}
DNS.2 = www.*.${DOMAIN}
EOF

# create CSR request using private key

openssl req -new -key ${CERT_DIR}/wildcard-${DOMAIN}.key -out ${CERT_DIR}/wildcard-${DOMAIN}.csr -config ${CERT_DIR}/csr.conf

# Create a external config file for the certificate

cat > ${CERT_DIR}/cert.conf <<EOF
authorityKeyIdentifier=keyid,issuer
basicConstraints=CA:FALSE
keyUsage = digitalSignature, nonRepudiation, keyEncipherment, dataEncipherment
subjectAltName = @alt_names
[alt_names]
DNS.1 = *.${DOMAIN}
EOF

# Create SSl Certificate with self signed rootCA

openssl x509 -req \
    -in ${CERT_DIR}/wildcard-${DOMAIN}.csr \
    -CA ${CERT_DIR}/rootCA.crt -CAkey ${CERT_DIR}/rootCA.key \
    -CAcreateserial -out ${CERT_DIR}/wildcard-${DOMAIN}.crt \
    -days 3650 \
    -sha256 -extfile ${CERT_DIR}/cert.conf
```


**NOTE:** Execute the script: `./self-signed-certificate.sh test.com`