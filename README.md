Self-signed certificate generation file
----

This section is dedicated to the generation of the self-signed certificates. The variables $CA_DN & $NODE_DN used correspond to the ones defined in the previous guides.

#### Generate the certificates using the [generate_crt.sh](https://github.com/eostis-sarl/wpsolr-generate-self-signed-certificates/tree/e36b46d1f84a224485f94976365c42002903f04e) script :

    ./generate_crt.sh --cadn "$CA_DN" --nodedn "$NODE_DN" --admindn "$ADMIN_DN"

To check all the options use :

    ./generate_crt.sh --help
or

    ./generate_crt.sh -h

If you already have generated a CA certificate and want to reuse it, you can do the following :

    ./generate_crt.sh --cacert path/to/ca.pem --cakey path/to/ca-key.pem --nodedn "$NODE_DN"

This will generate the node certificate and key signed by the CA certificate.



#### or if you want to do it manually, follow the steps below.

    mkdir opensearch_certs
    cd opensearch_certs

Generate the Root CA :

    openssl genrsa -out ca-key.pem 2048
    openssl req -new -x509 -sha256 -key ca-key.pem -subj "$CA_DN" -out ca.pem -days 730

Generate the Admin certificate : 

    openssl genrsa -out admin-key-temp.pem 2048
    openssl pkcs8 -inform PEM -outform PEM -in admin-key-temp.pem -topk8 -nocrypt -v1 PBE-SHA1-3DES -out admin-key.pem
    openssl req -new -key admin-key.pem -subj "/C=CA/ST=ONTARIO/L=TORONTO/O=ORG/OU=UNIT/CN=admin" -out admin.csr
    openssl x509 -req -in admin.csr -CA ca.pem -CAkey ca-key.pem -CAcreateserial -sha256 -out admin.pem -days 730

This certificate is used for administrative tasks. It's typically assigned to individuals or systems that need to perform administrative actions on the cluster, such as managing settings, adding or removing nodes, and configuring security settings. The admin certificate grants full control over the cluster.

Generate the node certificate :

    openssl genrsa -out "$NODE_CN"-key-temp.pem 2048
    openssl pkcs8 -inform PEM -outform PEM -in "$NODE_CN"-key-temp.pem -topk8 -nocrypt -v1 PBE-SHA1-3DES -out "$NODE_CN"-key.pem
    openssl req -new -key "$NODE_CN"-key.pem -subj "$NODE_DN" -out "$NODE_CN".csr
    echo "subjectAltName=DNS:"$NODE_CN", DNS:localhost" > "$NODE_CN".ext
    openssl x509 -req -in "$NODE_CN".csr -CA ca.pem -CAkey ca-key.pem -CAcreateserial -sha256 -out "$NODE_CN".pem -days 730 -extfile "$NODE_CN".ext