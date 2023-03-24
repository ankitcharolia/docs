---
layout: default
title: External Key Material for AWS KMS
parent: AWS
nav_order: 2
---


### A Script to Use External Key Material for AWS KMS
```shell
#!/bin/bash

# Reference: https://docs.aws.amazon.com/kms/latest/developerguide/importing-keys-create-cmk.html
# Example Reference: https://gist.github.com/madhusudangohil/a6cc80e1be9074ff1e7f59e88ace3a79

# Export your AWS Profile credentials: source exportAWSProfile.sh

# Downloading the public key and import token (AWS KMS API)
export KEY=`aws kms get-parameters-for-import \
    --key-id 077036ed-fcc2-4ea7-8509-088aca92c99c \
    --wrapping-algorithm RSAES_OAEP_SHA_256 \
    --wrapping-key-spec RSA_2048 --query '{Key:PublicKey,Token:ImportToken}' --output text`

echo $KEY | awk '{print $1}' > PublicKey.b64
echo $KEY | awk '{print $2}' > ImportToken.b64

# To base64 decode the public key and import token
openssl enc -d -base64 -A -in PublicKey.b64 -out PublicKey.bin
openssl enc -d -base64 -A -in ImportToken.b64 -out ImportToken.bin

# To use OpenSSL to generate binary key material and encrypt it for import into AWS KMS
openssl rand -out PlaintextKeyMaterial.bin 32

# Encrypt the Key material with PublicKey.bin
openssl pkeyutl \
    -in PlaintextKeyMaterial.bin \
    -out EncryptedKeyMaterial.bin \
    -inkey PublicKey.bin \
    -keyform DER -pubin \
    -encrypt \
    -pkeyopt rsa_padding_mode:oaep \
    -pkeyopt rsa_oaep_md:sha256

# Import key material (AWS KMS API)
aws kms import-key-material \
    --key-id 077036ed-fcc2-4ea7-8509-088aca92c99c --encrypted-key-material fileb://EncryptedKeyMaterial.bin \
    --import-token fileb://ImportToken.bin \
    --expiration-model KEY_MATERIAL_DOES_NOT_EXPIRE
```

**NOTE:** Kindly change `--key-id` with your own key ID.