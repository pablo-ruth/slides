# VAULT



## What is Vault

* A tool for managing secrets
* A product from Hashicorp
* Written in Go
* Single binary
* Current version 0.10.1 (26/04/2018)
* Actively developed
* +9000 stars on Github


## Why Vault

* Secret sprawl
* Decentralized keys
* Limited visibility
* No "break glass" procedure


## Vault goals

* Single source of secrets
* Full auditing
* Programmatic access
* Modern datacenter friendly (cloud)


## Key features

* Secure Secret Storage
* Dynamic Secrets
* Leasing, Renewal and Revocation
* Auditing
* Rich ACLs
* Multiple Client Authentication Methods


## Secure storage

* Data is encrypted in transit and at rest
* 256bit AES in GCM mode
* TLS 1.2 for clients


## Libraries

* Official
  * Go, Ruby
* Community
  * Java, NodeJS, PHP, Python...


## Integrations

* Ansible
* Terraform
* Consul-template
* Confd
* AWS 



## Architecture
![vault_arch](img/vault-arch.png)


## 4 types of backend


## Storage backend

> A storage backend represents the location for the durable storage of Vault's information.


## Storage backend

* Filesystem
* Consul (HA and supported)
* S3
* Mysql
* DynamoDB (HA)


## Auth backend

> Auth backends are the components in Vault that perform authentication and are responsible for assigning identity and a set of policies to a user.


## Auth backend

* Generic
  * token
* User oriented
  * User/Pass, LDAP, Github
* Machine oriented
  * AppRole, AWS EC2, Kubernetes


## Secret backend

> Secret backends are the components in Vault which store and generate secrets. They behave very similarly to a virtual filesystem.


## Secret backend

* Key value (generic)
* Database (Mysql, MongoDB, Cassandra, ...)
* AWS IAM
* PKI


## Audit backend

> Audit backends are the components in Vault that keep a detailed log of all requests and response to Vault.


## Audit backend

* File
* Syslog



## Threat model


### In scope

* Eavesdropping on any Vault communication (client, storage backend)
* Tampering with data at rest or in transit
* Access to data or controls without authentication or authorization
* Access to data or controls without accountability
* Availability of secret material in the face of failure


### Out scope

* Protecting against arbitrary control of the storage backend
* Protecting against the leakage of the existence of secret material
* Protecting against memory analysis of a running Vault



## Initialization
![vault-init](img/vault-init.png)


## Initialization

* Uses Shamir's secret sharing algorithm
* Split the master key into 5 shares
* Threshold of 3 to reconstruct the master key
* Master key is used to protect the encryption key


## Initialization

```
$ vault init
```
```
Unseal Key 1: 6kGTk+gIEz8PyNGsx/TWSm/8yKQdn/qcmsJG1SO4RM8B
Unseal Key 2: 3F2MaXjCrDfdr6RhKQgY5DcFxZOatHDFXq5XS49rcHkC
Unseal Key 3: IAo452DivgYFxHb6JyLIfhNL1Z/si1M46py7rCbMTgkD
Unseal Key 4: PTknIi15Ugtx/jZsz1eY1GNrFlLmOwE9wqSzRqtzrbAE
Unseal Key 5: wW6TrDVZQDqpleT3wX1ITkclBl6QBCLAdpZfoQLUk8AF
Initial Root Token: 16e8ea9e-ebac-53ad-4043-22a5e9c610d2
```


## Initialization

We can use PGP to protect unseal keys

```
$ vault init -key-shares=3 -key-threshold=2 \
             -pgp-keys="jeff.asc,vishal.asc,seth.asc"
```



## Unseal

> Unsealing is the process of constructing the master key necessary to read the decryption key to decrypt the data, allowing access to the Vault.


## Unseal

* Vault starts in a sealed state
* Almost no operations are possible when sealed
* We need to unseal with threshold number of keys


## Unseal

```
$ vault operator unseal
Key (will be hidden): 
```
```
Sealed: true
Key Shares: 5
Key Threshold: 3
Unseal Progress: 1
Unseal Nonce: 04130de4-08d5-8dc0-d0c9-684c2cb3cc18
```
```
$ vault operator unseal
Key (will be hidden): 
```
```
Sealed: false
Key Shares: 5
Key Threshold: 3
Unseal Progress: 0
```


## Seal

* Throw away the master key
* Only requires a single operator with root privileges
* Data can be locked quickly to try to minimize damage


## Seal

```
$ vault operator seal
```
```
Vault is now sealed.
```



## Basic CLI


## Env

Set Vault address for CLI
```
export VAULT_ADDR="http://vault-server:8200"
```


## Get vault status

```
$ vault status
```
```
Sealed: false
Key Shares: 5
Key Threshold: 3
Unseal Progress: 0
Unseal Nonce: 
Version: 0.10.0
Cluster Name: vault-cluster-c209a2b2
Cluster ID: 95fd6576-9e55-52b9-5c6f-017f57fbbeab

High-Availability Enabled: false
```


## List mounted secret backends
```
$ vault mounts 
```
```
Path        Type       Default TTL  Max TTL  Force No Cache  Replication Behavior  Description
cubbyhole/  cubbyhole  n/a          n/a      false           local                 per-token private secret storage
secret/     generic    system       system   false           replicated            generic secret storage
sys/        system     n/a          n/a      false           replicated            system endpoints used for control, policy and debugging
```


## Generic secret backend

* store arbitrary secrets
* mounted by default at *secret/*
* arborescence like a virtual filesystem
* CRUD operations
* Writing to a key will replace the old value


## Write data
On CLI
```
$ vault write secret/password value=itsasecret
```

From file
```
$ vault write secret/password @data.json
```


## List keys
```
$ vault list secret
```
```
Keys
----
password
```


## Read data
Get data + metadata
```
$ vault read secret/password
```
```
Key                 Value
---                 -----
refresh_interval    768h0m0s
value               itsasecret
```
Get only selected key
```
$ vault read -field=value secret/password
```
```
itsasecret
```


## Delete data
```
$ vault delete secret/password
```
```
Success! Deleted 'secret/password' if it existed.
```


## Mount secret backend
If path is not specified, default to backend name
```
$ vault secrets enable -path=/anothergenericbackend generic
```
```
Successfully mounted 'generic' at '/anothergenericbackend'!
```



## Authentication


## Token

* Core of client authentication
* Enabled by default at */auth/token*
* Every authentication method generate a token
* Token stored in *~/.vault-token* after auth


## Token

```
$ vault login
Token (will be hidden):
```
```
Successfully authenticated! You are now logged in.
token: 16e8ea9e-ebac-53ad-4043-22a5e9c610d2
token_duration: 0
token_policies: [root]
```


## Root tokens

* Special root token mapped to root policy
* Root token can do anything in Vault
* Only used for just enough initial setup
* Generate new root token with quorum unseal keys


## LDAP

* user-oriented
* using an existing LDAP credentials
* mapping of groups and users to policies


## AppRoles

* machine-oriented
* set of Vault policies and login constraints
* RoleID
* SecretID
* CIDR list


## AWS EC2

* EC2 instances have access to metadata
* AWS also provides PKCS#7 signature
* AWS public keys used to verify the signature
* verifies the running status of the instance
* Trust On First Use (TOFU) with instance ID
* bind tags and/or AMI id to Vault policies



## Audit

* File or syslog
* Log every single action in Vault

```
$ vault audit enable file file_path=/var/log/vault_audit.log
```



## Authorization


## ACL

* ACLs are applied to paths
  * secret/*
  * secret/app
* Deny by default
* Most specific ACL is used


## Capabilities

* Fine-grained control over operations
  * create
  * read
  * update
  * delete
  * list
  * sudo
  * deny


## Policy

Group of ACLs to applied to users

```
path "secret/*" {
  capabilities = ["read", "create", "update", "delete", "list"]
}

path "secret/app/db" {
  capabilities = ["read"]
}

path "auth/token/lookup-self" {
  capabilities = ["read"]
}
```


## Policy

Import a policy in Vault

```
$ vault policy write secret secret.hcl
```

Apply a policy to LDAP group
```
$ vault write auth/ldap/groups/myldapgroup policies=secret
```



## Demo

![vault-init](img/vault-demo.png)
