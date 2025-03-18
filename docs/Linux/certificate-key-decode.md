---
layout: default
title: Decode certificate key and verify
parent: Linux
nav_order: 5
---

### How to decode certificate key and verify

**NOTE:** PEM on its own is not a certificate, it's just a way of encoding data. X.509 certificates are one type of data that is commonly encoded using PEM.

```shell
# decode the certificcate private key
openssl rsa -in encoded_private_key.pem -out myserver_decoded.key
```

#### To verify that an RSA private key matches the RSA public key in a certificate you need to 

*  verify the consistency of the private key 
```shell
# To verify the consistency of the RSA private key and to view its modulus:
openssl rsa -modulus -noout -in myserver_decoded.key | openssl md5
openssl rsa -check -noout -in myserver_decoded.key
RSA Key is ok
If it doesn't say 'RSA key ok', it isn't OK!"
```

* compare the modulus of the public key in the certificate against the modulus of the private key.
```shell
# To view the modulus of the RSA public key in a certificate:
openssl x509 -modulus -noout -in myserver.crt/pem | openssl md5
```

**NOTE:** If the first commands shows any errors, or if the modulus of the public key in the certificate and the modulus of the private key do not exactly match, then you're not using the correct private key. 



### Reference

* [Verify certificate private key](https://www.ssl247.com/knowledge-base/detail/how-do-i-verify-that-a-private-key-matches-a-certificate-openssl-1527076112539/ka03l0000015hscaay/)
