---
layout: default
title: Hashicorp vault integration with Kubernetes
parent: Linux
nav_order: 2
---

## How to setup Hashicorp Vault in Kubernetes

```shell
# Deploy Hashicorp Vault
helm repo add hashicorp https://helm.releases.hashicorp.com
helm repo update
helm upgrade --install vault hashicorp/vault --version 0.22.0 --values hashicorp-vault/values.yaml
kubectl exec vault-0 -- vault operator init -key-shares=1 -key-threshold=1 -format=json > cluster-keys.json
jq -r ".unseal_keys_b64[]" cluster-keys.json
VAULT_UNSEAL_KEY=$(jq -r ".unseal_keys_b64[]" cluster-keys.json)
kubectl exec vault-0 -- vault operator unseal $VAULT_UNSEAL_KEY
kubectl exec -ti vault-1 -- vault operator raft join http://vault-0.vault-internal:8200
kubectl exec vault-1 -- vault operator unseal $VAULT_UNSEAL_KEY
kubectl exec -ti vault-2 -- vault operator raft join http://vault-0.vault-internal:8200
kubectl exec vault-2 -- vault operator unseal $VAULT_UNSEAL_KEY

#after vault server pods are unsealed, they report `1/1`
# To avoid the manual step of unsealing, Vault supports automatic unsealing via cloud based key management services like Azure Key Vault, Amazon KMS, AliCloud KMS, and Google Cloud KMS which enables us to let trusted cloud providers take care of the unsealing process
```

```shell
# login using root token
kubectl exec -ti vault-0 -- vault login

# ssh to vault-0 and set a Secret
kubectl exec --stdin=true --tty=true vault-0 -- /bin/sh
$ vault login
Token (will be hidden): (Enter root token)

acharolia@ankitcharolia:~/$ kubectl exec -ti vault-0 -- vault operator raft list-peers
Node                                    Address                        State       Voter
----                                    -------                        -----       -----
ca72acd0-b009-4c3c-6ebf-4c8d76bed6e1    vault-0.vault-internal:8201    leader      true
a7f8b6ab-f77d-aeaa-66a6-8a97642fdade    vault-1.vault-internal:8201    follower    true
e09d734d-e949-a7e9-05ce-9ccae62b3cc2    vault-2.vault-internal:8201    follower    true
```


## Enable kubernetes authentication
```shell
# ssh to vault-0 and set a Secret
kubectl exec --stdin=true --tty=true vault-0 -- /bin/sh

$ vault auth enable kubernetes

$ vault policy write readonly -<<EOF     
path "*"                                                  
{  capabilities = ["read","create","delete","update","list"]                
}                         
EOF

# Configure the kubernetes authentication method
$ vault write auth/kubernetes/config token_reviewer_jwt="$(cat /var/run/secrets/kubernetes.io/serviceaccount/token)" kubernetes_host=https://${KUBERNETES_PORT_443_TCP_ADDR}:443 kubernetes_ca_cert=@/var/run/secrets/kubernetes.io/serviceaccount/ca.crt
Success! Data written to: auth/kubernetes/config

# create a role for kubernetes and attach this policy. In the below mentioned role all the service accounts in any namespaces are allowed to authenticate. We can also limit authentication to specific service accounts and namespaces as well.
$ vault write auth/kubernetes/role/k8s-role bound_service_account_names=* bound_service_account_namespaces=* policies=readonly ttl=1h
Success! Data written to: auth/kubernetes/role/k8s-role
```

## validate kubernetes authentication

```shell
# ssh to vault-0 and set a Secret
kubectl exec --stdin=true --tty=true vault-0 -- /bin/sh

/ $ vault write auth/kubernetes/login role=k8s-role jwt="$(cat /var/run/secrets/kubernetes.io/serviceaccount/token)"
Key                                       Value
---                                       -----
token                                     hvs.CAESIHo21bGw0IsDjZ_H5nrxk_lidxc0-uVBPyg5fW27rSwsGh4KHGh2cy41aG1CYTJsNU02NnpudU5iOEZrTTNkM00
token_accessor                            Y5HK3CgT42BOFct806JukHd6
token_duration                            1h
token_renewable                           true
token_policies                            ["default" "readonly"]
identity_policies                         []
policies                                  ["default" "readonly"]
token_meta_service_account_name           vault
token_meta_service_account_namespace      default
token_meta_service_account_secret_name    n/a
token_meta_service_account_uid            e5ef6dce-0345-47b4-9a40-b095465aa1df
token_meta_role                           k8s-role
```

##  Set secrets in vault
```shell
$ vault secrets enable -path=secret kv-v2 (Enable kv-v2 secret engine)
# set secrets into vault
vault kv put secret/redis redis_password=secretpassword
vault kv put secret/grafana grafana_password=secretpassword
```