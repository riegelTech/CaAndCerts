# Les certificats, comment que ça marche ?

## Problème
- En tant qu'utilisateur, je veux pouvoir avoir confiance en mon correspondant.
Réponse facile : je te donne un certificat dans lequel tu vas déclarer avoir confiance.
Oui mais :
- En tant qu'utilisateur je ne veux pas avoir à approuver chaque certificat individuellement, 
ça va être trop chiant
- En tant qu'admin je ne veux pas avoir à gérer chaque certificat, sa clé privée et tout, c'est trop pénible.

## x509 à la rescousse
Les gars, il suffit de créer des chaînes de confiances, et le tour est joué.

Exemple : 
```
Autorité super balèse
    |_Autorité moins balèse
    |    |_le certificat de mon site
    |    |_le certificat d'un autre site
    |_Une autre autorité pas trop balèse
        |_un autre certificat
        |_ ...
```
Si j'ai confiance en l'Autorité super balèse, alors inutile de redonner ma confiance en chaque certificat, je n'ai qu'à 
remonter la chaîne de confiance de chaque certificat jusqu'à l'autorité super balèse.

**Ok mais comment réaliser une chaîne de trust ?**

## Le principe du chiffrement à clés asymétriques
Une clé privée VS une clé publique.

**la clé privée sert à**
* déchiffrer
* signer

**la clé publique sert à**
* chiffrer
* vérifier une signature

La clé privée ne sort jamais, la clé publique circule librement.
Voici deux scénarios de base à connaître :

### Je veux discuter de manière sécurisée
1- Je donne ma clé publique à mon correspondant
2- Il chiffre le message secret
3- Je le déchiffre avec ma clé privée

### Je veux prouver que je suis l'auteur d'un message
1- Je signe le message avec ma clé privée
2- J'envoie le message et la clé publique à mon correspondant (pas sur le même canal)
3- Il peut vérifier avec la clé publique que seule ma clé privée a pu signer le message

Pour échanger par email c'est parfait, peu coûteux, et ultra sécurisé.

> Mais pour un chiffrement - déchiffrement à la volée c'est naze.
> Solution : négocier une clé symétrique à l'aide de clés asymétriques, puis utiliser cette clé symétrique 
> pour le reste de l'échange.

## Anatomie d'un certificat (PEM)
champ | contenu
----- | -------
Subject | les informations sur l'identité du possesseur
Issuer | le subject de la CA qui a signé le certificat
Public key | la clé publique associée à la clé privée du possesseur
Signature | la signature des informations précédentes par la CA
*D'autres informations importantes aussi qui ne nous intéressent pas : serial, date de validité, etc.*

exemple avec un vrai certificat :
```
$ openssl x509 -noout -text -in myCertificate.pem
Certificate:
    Data:
        Version: 1 (0x0)
        Serial Number: 4098 (0x1002)
    Signature Algorithm: sha256WithRSAEncryption
        Issuer: C=FR, ST=France, O=YMCA, OU=YMCA certifications department, CN=YMCA intermediate CA cert
        Validity
            Not Before: Sep 19 12:07:57 2017 GMT
            Not After : Sep 19 12:07:57 2018 GMT
        Subject: C=FR, ST=France, O=sns02 YMCA group, OU=sns02 crew, CN=sns02
        Subject Public Key Info:
            Public Key Algorithm: rsaEncryption
                Public-Key: (2048 bit)
                Modulus:
                    00:b5:70:71:27:ea:bc:92:f1:90:25:52:da:e6:76:
                    8e:a9:54:a2:b2:04:2f:8b:8b:9b:88:76:5d:a3:5e:
                    d1:7e:0c:00:61:2f:0a:da:65:c3:6c:05:d8:84:cc:
                    5c:f3:ff:d2:2e:d2:da:59:9d:5b:6a:d7:90:d0:35:
                    d0:5a:6c:d4:7d:dc:3a:ba:41:96:87:9a:14:c9:35:
                    1e:65:cd:47:12:c7:ea:ab:c6:11:4d:2b:23:56:77:
                    bb:50:8a:3e:5a:7e:cd:d0:b5:cf:7d:d5:dc:9b:1b:
                    b2:d8:91:f8:74:32:0f:18:90:c9:b2:bd:1c:29:1e:
                    ba:c5:b8:54:53:d6:47:a0:52:9a:f9:d2:dd:e9:f1:
                    9e:88:ce:01:4e:6c:f1:90:0d:5f:db:1d:06:12:20:
                    aa:ff:1d:55:26:d0:fc:f2:f3:e5:45:71:13:86:96:
                    b0:aa:55:1a:7d:ab:40:df:2a:25:47:9f:46:4a:27:
                    fb:34:d3:25:6d:16:4a:d3:11:07:09:17:fc:8a:cd:
                    29:1c:9a:b8:50:f2:42:ad:c6:1e:1c:f8:14:ef:35:
                    26:d1:ef:cb:e6:5b:50:32:fd:7e:e4:51:a4:1e:04:
                    d9:39:f6:81:bb:f6:f6:97:09:92:4a:e1:5d:7d:32:
                    2e:fe:e4:c4:c9:af:2e:fe:49:b7:a4:9d:3a:67:14:
                    7a:a7
                Exponent: 65537 (0x10001)
    Signature Algorithm: sha256WithRSAEncryption
         9f:06:e5:33:21:a7:e3:a2:c9:4b:5e:b9:97:fe:8e:68:ed:b4:
         7f:86:9d:6b:01:dd:1d:c7:ec:c1:10:a5:f5:e1:0d:7f:87:59:
         25:53:f8:0c:2c:33:8d:24:f4:3f:8f:39:a6:8a:d9:b9:5a:21:
         c9:66:50:3c:18:5c:7a:68:8a:11:56:54:f1:a0:17:2d:5d:59:
         8a:5f:45:87:e3:c2:6a:27:af:37:7d:90:54:34:dd:08:81:62:
         00:7b:3f:19:21:8d:ba:2c:98:37:68:4f:c1:a6:fa:ed:bc:b6:
         07:f5:bf:94:6f:7e:f3:70:8a:27:ff:fb:15:6d:00:ef:af:0d:
         58:7e:1b:43:4c:42:bb:2b:8b:e5:97:b4:be:b2:79:c6:a5:3b:
         6c:c1:dd:7c:a7:68:31:82:24:6b:81:b1:d9:01:c6:db:64:d3:
         e1:1b:f1:57:5a:cf:61:2f:c8:f9:b7:78:80:00:03:33:b9:75:
         0b:9d:24:fe:e8:1b:f3:ec:09:fb:11:dc:f1:41:84:61:5b:86:
         c2:01:a5:87:7b:fd:e6:d5:72:72:fe:c7:62:23:81:ad:e2:29:
         74:a1:c0:f3:90:19:5d:39:70:f2:08:97:57:24:b2:92:06:e7:
         6a:bd:d6:86:e0:b8:4c:91:a6:07:ca:85:ca:37:ff:76:9f:df:
         9c:b3:68:72:10:91:b1:78:fb:99:35:09:5b:6b:27:c3:a4:94:
         55:d6:a8:24:44:26:7e:0e:2f:12:75:dc:d4:2a:2a:dc:f6:e0:
         ea:34:f8:bb:71:36:4c:36:79:d4:77:3a:c2:d6:48:11:8b:ce:
         8f:60:f4:31:19:2c:3c:d2:49:ab:52:8b:cc:98:87:49:6a:f9:
         17:84:2a:96:9d:b3:ce:a3:6f:00:8c:08:f0:b5:f9:18:cc:27:
         09:74:87:29:17:39:4a:0f:c8:d4:d9:68:e3:31:8f:17:5a:4c:
         e0:f2:c4:61:11:04:23:6f:da:6c:3d:22:9d:04:6b:d9:38:7c:
         5d:73:67:f2:68:76:5a:b0:96:74:0b:e5:16:29:d4:bf:3e:33:
         da:39:db:cc:d0:87:16:18:ef:c1:06:9d:a7:79:dd:51:ec:19:
         1c:60:67:18:7e:28:f5:ad:62:ea:20:e3:33:0a:86:71:81:b6:
         2b:64:a9:90:89:75:10:5f:52:77:53:f5:a7:4b:93:5a:62:a0:
         f1:13:21:66:8b:80:df:82:5b:af:90:38:3b:d4:0d:0a:fe:f8:
         40:da:f0:49:0a:2c:31:11:bd:be:b2:59:30:15:84:f3:98:cd:
         96:3b:cd:9c:dd:fc:95:c4:4e:20:a0:38:6a:08:b4:70:77:ae:
         74:df:21:c6:25:66:70:8f
```

**Si on veut embarquer embarquer en plus du reste, la clé privée du détenteur du certificat
(pour un échange chiffré par exemple), on utilise le format p12**

## Séquence de génération d'une CA puis d'un certificat

**Création de l'autorité racine (autorité super balèse)**
1. **Création d'une clé privée**
2. Création d'un certificat (auto-signé pour le coup)

**Création d'une sous-autorité (CA intermédiaire) (autorité moins balèse)**
1. **Création d'une clé privée**
2. Création d'une requête de signature (CSR), un certificat auto-signé en fait
3. Création d'un certificat par signature de la CSR par la CA parente

**Création d'un certificat client (le certificat de mon site)**
1. **Création d'une clé privée**
2. Création d'une requête de signature (CSR), un certificat auto-signé en fait
3. Création d'un certificat par signature de la CSR par la CA parente

*La création d'un certificat procède de la même manipulation que la création d'une sous-autorité*

> En l'état, quand j'ai le certificat client, je sais qu'il a été signé par la CA intermédiaire
> J'ai besoin du certificat de cette CA intermédiaire pour vérifier la signature du certificat
> Je ne sais rien de la parenté de cette CA intermédiaire

## Dernier maillon : la chaîne de certificats

Pour vérifier toutes les signatures d'un coup et remonter jusqu'à la CA racine, 
rien de plus simple : au lieu de passer le certificat de la CA intermédiaire, 
on passe une concaténation des certificats de la CA racine et intermédiaire.

# Commandes utiles
Copieusement inspiré de [jamielinux.com](https://jamielinux.com/docs/openssl-certificate-authority/index.html)

## Création de l'autorité racine
Préparation des dossiers
```shell
$ mkdir /root/ca && cd /root/ca
#first create configuration file for openssl
$ touch openssl.cnf
```
Contenu (lire et modifier selon vos exigences): 
```
# OpenSSL root CA configuration file.
# Copy to `/root/ca/openssl.cnf`.

[ ca ]
# `man ca`
default_ca = CA_default

[ CA_default ]
# Directory and file locations.
dir               = /root/ca
certs             = $dir/certs
crl_dir           = $dir/crl
new_certs_dir     = $dir/newcerts
database          = $dir/index.txt
serial            = $dir/serial
RANDFILE          = $dir/private/.rand

# The root key and root certificate.
private_key       = $dir/private/ca.key.pem
certificate       = $dir/certs/ca.cert.pem

# For certificate revocation lists.
crlnumber         = $dir/crlnumber
crl               = $dir/crl/ca.crl.pem
crl_extensions    = crl_ext
default_crl_days  = 30

# SHA-1 is deprecated, so use SHA-2 instead.
default_md        = sha256

name_opt          = ca_default
cert_opt          = ca_default
default_days      = 375
preserve          = no
policy            = policy_strict

[ policy_strict ]
# The root CA should only sign intermediate certificates that match.
# See the POLICY FORMAT section of `man ca`.
countryName             = match
stateOrProvinceName     = match
organizationName        = match
organizationalUnitName  = optional
commonName              = supplied
emailAddress            = optional

[ policy_loose ]
# Allow the intermediate CA to sign a more diverse range of certificates.
# See the POLICY FORMAT section of the `ca` man page.
countryName             = optional
stateOrProvinceName     = optional
localityName            = optional
organizationName        = optional
organizationalUnitName  = optional
commonName              = supplied
emailAddress            = optional

[ req ]
# Options for the `req` tool (`man req`).
default_bits        = 2048
distinguished_name  = req_distinguished_name
string_mask         = utf8only

# SHA-1 is deprecated, so use SHA-2 instead.
default_md          = sha256

# Extension to add when the -x509 option is used.
x509_extensions     = v3_ca

[ req_distinguished_name ]
# See <https://en.wikipedia.org/wiki/Certificate_signing_request>.
countryName                     = Country Name (2 letter code)
stateOrProvinceName             = State or Province Name
localityName                    = Locality Name
0.organizationName              = Organization Name
organizationalUnitName          = Organizational Unit Name
commonName                      = Common Name
emailAddress                    = Email Address

# Optionally, specify some defaults.
countryName_default             = FR
stateOrProvinceName_default     = France
localityName_default            =
0.organizationName_default      = Company Ltd
organizationalUnitName_default  =
emailAddress_default            =

[ v3_ca ]
# Extensions for a typical CA (`man x509v3_config`).
subjectKeyIdentifier = hash
authorityKeyIdentifier = keyid:always,issuer
basicConstraints = critical, CA:true
keyUsage = critical, digitalSignature, cRLSign, keyCertSign

[ v3_intermediate_ca ]
# Extensions for a typical intermediate CA (`man x509v3_config`).
subjectKeyIdentifier = hash
authorityKeyIdentifier = keyid:always,issuer
basicConstraints = critical, CA:true, pathlen:0
keyUsage = critical, digitalSignature, cRLSign, keyCertSign

[ usr_cert ]
# Extensions for client certificates (`man x509v3_config`).
basicConstraints = CA:FALSE
nsCertType = client, email
nsComment = "OpenSSL Generated Client Certificate"
subjectKeyIdentifier = hash
authorityKeyIdentifier = keyid,issuer
keyUsage = critical, nonRepudiation, digitalSignature, keyEncipherment
extendedKeyUsage = clientAuth, emailProtection

[ server_cert ]
# Extensions for server certificates (`man x509v3_config`).
basicConstraints = CA:FALSE
nsCertType = server
nsComment = "OpenSSL Generated Server Certificate"
subjectKeyIdentifier = hash
authorityKeyIdentifier = keyid,issuer:always
keyUsage = critical, digitalSignature, keyEncipherment
extendedKeyUsage = serverAuth

[ crl_ext ]
# Extension for CRLs (`man x509v3_config`).
authorityKeyIdentifier=keyid:always

[ ocsp ]
# Extension for OCSP signing certificates (`man ocsp`).
basicConstraints = CA:FALSE
subjectKeyIdentifier = hash
authorityKeyIdentifier = keyid,issuer
keyUsage = critical, digitalSignature
extendedKeyUsage = critical, OCSPSigning
```

```shell
$ mkdir certs crl newcerts private
$ chmod 700 private
$ touch index.txt
$ echo 1000 > serial
```
**Création de la clé racine** à ne jamais transmettre, à garder précieusement, à protéger
```shell
$ openssl genrsa -aes256 -out private/ca.key.pem 4096
$ chmod 400 private/ca.key.pem
```
**Création du certificat racine** à transmettre le plus largement possible
```shell
$ openssl req -config openssl.cnf \
    -key private/ca.key.pem \
    -new -x509 -days 7300 -sha256 -extensions v3_ca \
    -out certs/ca.cert.pem
$ chmod 444 certs/ca.cert.pem
```
Vérification éventuelle
```shell
openssl x509 -noout -text -in certs/ca.cert.pem
```

## Création d'une sous-autorité
Préparation des dossiers
```shell
$ mkdir /root/ca/intermediate && cd /root/ca/intermediate
$ mkdir certs crl csr newcerts private
$ chmod 700 private
$ touch index.txt
$ echo 1000 > serial
# prepare also Certificate Revocation List
$ echo 1000 > /root/ca/intermediate/crlnumber
# prepare configuration file
$ touch opensl.cnf
```
Contenu du fichier de conf pour la CA intermédiaire
```
# OpenSSL intermediate CA configuration file.
# Copy to `/root/ca/intermediate/openssl.cnf`.

[ ca ]
# `man ca`
default_ca = CA_default

[ CA_default ]
# Directory and file locations.
dir               = /root/ca/intermediate
certs             = $dir/certs
crl_dir           = $dir/crl
new_certs_dir     = $dir/newcerts
database          = $dir/index.txt
serial            = $dir/serial
RANDFILE          = $dir/private/.rand

# The root key and root certificate.
private_key       = $dir/private/intermediate.key.pem
certificate       = $dir/certs/intermediate.cert.pem

# For certificate revocation lists.
crlnumber         = $dir/crlnumber
crl               = $dir/crl/intermediate.crl.pem
crl_extensions    = crl_ext
default_crl_days  = 30

# SHA-1 is deprecated, so use SHA-2 instead.
default_md        = sha256

name_opt          = ca_default
cert_opt          = ca_default
default_days      = 375
preserve          = no
policy            = policy_loose

[ policy_strict ]
# The root CA should only sign intermediate certificates that match.
# See the POLICY FORMAT section of `man ca`.
countryName             = match
stateOrProvinceName     = match
organizationName        = match
organizationalUnitName  = optional
commonName              = supplied
emailAddress            = optional

[ policy_loose ]
# Allow the intermediate CA to sign a more diverse range of certificates.
# See the POLICY FORMAT section of the `ca` man page.
countryName             = optional
stateOrProvinceName     = optional
localityName            = optional
organizationName        = optional
organizationalUnitName  = optional
commonName              = supplied
emailAddress            = optional

[ req ]
# Options for the `req` tool (`man req`).
default_bits        = 2048
distinguished_name  = req_distinguished_name
string_mask         = utf8only

# SHA-1 is deprecated, so use SHA-2 instead.
default_md          = sha256

# Extension to add when the -x509 option is used.
x509_extensions     = v3_ca

[ req_distinguished_name ]
# See <https://en.wikipedia.org/wiki/Certificate_signing_request>.
countryName                     = Country Name (2 letter code)
stateOrProvinceName             = State or Province Name
localityName                    = Locality Name
0.organizationName              = Organization Name
organizationalUnitName          = Organizational Unit Name
commonName                      = Common Name
emailAddress                    = Email Address

# Optionally, specify some defaults.
countryName_default             = GB
stateOrProvinceName_default     = England
localityName_default            =
0.organizationName_default      = Alice Ltd
organizationalUnitName_default  =
emailAddress_default            =

[ v3_ca ]
# Extensions for a typical CA (`man x509v3_config`).
subjectKeyIdentifier = hash
authorityKeyIdentifier = keyid:always,issuer
basicConstraints = critical, CA:true
keyUsage = critical, digitalSignature, cRLSign, keyCertSign

[ v3_intermediate_ca ]
# Extensions for a typical intermediate CA (`man x509v3_config`).
subjectKeyIdentifier = hash
authorityKeyIdentifier = keyid:always,issuer
basicConstraints = critical, CA:true, pathlen:0
keyUsage = critical, digitalSignature, cRLSign, keyCertSign

[ usr_cert ]
# Extensions for client certificates (`man x509v3_config`).
basicConstraints = CA:FALSE
nsCertType = client, email
nsComment = "OpenSSL Generated Client Certificate"
subjectKeyIdentifier = hash
authorityKeyIdentifier = keyid,issuer
keyUsage = critical, nonRepudiation, digitalSignature, keyEncipherment
extendedKeyUsage = clientAuth, emailProtection

[ server_cert ]
# Extensions for server certificates (`man x509v3_config`).
basicConstraints = CA:FALSE
nsCertType = server
nsComment = "OpenSSL Generated Server Certificate"
subjectKeyIdentifier = hash
authorityKeyIdentifier = keyid,issuer:always
keyUsage = critical, digitalSignature, keyEncipherment
extendedKeyUsage = serverAuth

[ crl_ext ]
# Extension for CRLs (`man x509v3_config`).
authorityKeyIdentifier=keyid:always

[ ocsp ]
# Extension for OCSP signing certificates (`man ocsp`).
basicConstraints = CA:FALSE
subjectKeyIdentifier = hash
authorityKeyIdentifier = keyid,issuer
keyUsage = critical, digitalSignature
extendedKeyUsage = critical, OCSPSigning
```
**Création de la clé**
```shell
$ openssl genrsa -aes256 \
    -out private/intermediate.key.pem 4096
```
**Création du certificat de requête de signature**
```shell
$ openssl req -config openssl.cnf -new -sha256 \
    -key private/intermediate.key.pem \
    -out csr/intermediate.csr.pem
```
**Création du certificat signé**
```shell
$ openssl ca -config ../openssl.cnf -extensions v3_intermediate_ca \
    -days 3650 -notext -md sha256 \
    -in csr/intermediate.csr.pem \
    -out certs/intermediate.cert.pem
$ chmod 444 certs/intermediate.cert.pem
```
Vérification éventuelle
```shell
# check certificate
$ openssl x509 -noout -text \
    -in intermediate/certs/intermediate.cert.pem
# check chain of trust
$ openssl verify -CAfile ../certs/ca.cert.pem \
    certs/intermediate.cert.pem
```
**Création de la chaîne de certificats**
```shell
$ cat certs/intermediate.cert.pem \
    ../certs/ca.cert.pem > certs/ca-chain.cert.pem
$ chmod 444 certs/ca-chain.cert.pem
```

## Création d'un certificat client
**Création d'une clé privée**
```shell
$ mkdir -P /root/certificates/www.example.com && cd /root/certificates/www.example.com
$ mkdir private csr certs
$ openssl genrsa -aes256 \
    -out private/www.example.com.key.pem 2048
$ chmod 400 private/www.example.com.key.pem
```
**Création du certificat de requête de signature**
```shell
$ openssl req -config /root/ca/intermediate/openssl.cnf \
    -key private/www.example.com.key.pem \
    -new -sha256 -out csr/www.example.com.csr.pem
```
**Création du certificat signé par la CA intermédiaire**
```shell
$ openssl ca -config /root/ca/intermediate/openssl.cnf \
    -extensions server_cert -days 375 -notext -md sha256 \
    -in csr/www.example.com.csr.pem \
    -out certs/www.example.com.cert.pem
$ chmod 444 intermediate/certs/www.example.com.cert.pem
```
*Ici on voit l'utilité du fichier de configuration : 
pas besoin de renseigner où se trouve la clé de la CA intermédiaire*

Finalement vérification du certificat
```shell
$ openssl x509 -noout -text \
    -in certs/www.example.com.cert.pem
```

Ce qui doit être transmis pour que le client ait confiance et que le chiffrement - déchiffrement soit possible :
* ca-chain.cert.pem (vérification de la chîne de trust jusqu'à la CA root)
* www.example.com.key.pem (déchiffrer)
* www.example.com.cert.pem (chiffrer - vérifier la signature de la CA parente)

## Un conteneur qui embarque tout
Le souci avec le format pem c'est que la clé et les certificats des CAs sont à part, ce qui peut être pénible à gérer, notamment à transmettre.

Pour créer un format p12 :
```shell
$ openssl pkcs12 -export -out certs/www.example.com.p12 -inkey private/www.example.com.key.pem -in www.example.com.cert.pem -certfile /root/ca/intermediate/certs/ca-chain.cert.pem
```
*Comme il contient une clé privée il doit être protégé avec un mot de passe*
*Le p12 n'est pas un certificat, c'est un conteneur qui contient -entre autres- un certificat*

# Glossaire
**Certificat**

C'est le fichier contenant : 
* une clé publique 
* des informations relatives aux parties prenantes 
* une signature émise par une autorité


**CRL**

Certificate Revocation List : c'est une liste de certificats révoqués, 
liste signée par l'autorité de certification qui avait émis les certificats révoqués.


**PKI**

Public Key Infrastructure : c'est l'infrastructure qui gère les certificats et leurs détenteurs. En gros c'est 
l'infrastructure de l'entreprise chargée de gérer des certificats.
