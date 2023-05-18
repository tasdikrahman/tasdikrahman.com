---
layout: post
title: "Renewing your root CA in a way such that the certificates signed by it are still valid when you replace the root CA"
description: "Renewing your root CA in a way such that the certificates signed by it are still valid when you replace the root CA"
tags: [ssl]
comments: true
share: true
cover_image: ''
---

# Context

If you have a root CA which you used to sign certificates, and if the root certificate is about to expire, the certificates signed by the root CA will also become invalid after the root CA expires even if the certificates signed by it haven't expired. As every certificate in the chain must remain valid for your certificate to be valid.

Also for example the kube-apiserver when it comes up, it `--client-ca-file` while it comes up, where you can pass the root CA.

## Going about this

Now if you end up replacing this cert, you would have to replace all the certs with certs signed by the new root CA in the clients to allow un-interrupted functioning.

The other option is to append the new root CA inside the same file, this will allow the certs signed by both the new root CA and the old root CA to work, allowing time to update the certs for the clients. Why append in the same file? [RFC 1422](https://datatracker.ietf.org/doc/html/rfc1422) mentions the same that a pem file can have multiple certificates

This will also allow no changes for any configuration changes being passed for `--client-ca-file` for example for the kube-apiserver

## Generating the root CA again

In order for things to work as expected, we would need to generate the root CA with the same serial and v3 extensions, this will allow the certs signed by the older root CA to verify against the root CA

the CSR generated from the old private key and the old root CA

```sh
CACRT=existing-ca.pem
CAKEY=existing-ca-key.pem

NEWCA=renewed-ca.pem

# using the same serial as this is how the client certs signed by the original CA will be respecting the new CA
serial=`openssl x509 -in $CACRT -serial -noout | cut -f2 -d=`

# generate the csr from the old private key and the existing root CA
openssl x509 -x509toreq -in $CACRT -signkey $CAKEY -out $NEWCA.csr

# v3extensions from the last pem file to be used for the new root CA and using the same subject identifier as the original pem file
cat renewed-ca.pem.conf
[ v3_ca ]
basicConstraints= critical, CA:TRUE
keyUsage= critical, Certificate Sign, CRL Sign
subjectKeyIdentifier= <the-serial>

openssl x509 -req -days 3650 -in $NEWCA.csr -set_serial 0x$serial -singkey $CAKEY -out $NEWCA.crt -extfile ./$NEWCA.conf -extensions v3ca
```

And that's pretty much, the `$NEWCA.crt` can be used now

## References

- [https://stackoverflow.com/questions/9234796/does-the-expiration-status-of-an-issuers-certificate-affect-a-subjects-expirat](https://stackoverflow.com/questions/9234796/does-the-expiration-status-of-an-issuers-certificate-affect-a-subjects-expirat)
- [https://stackoverflow.com/questions/40061263/what-is-ca-certificate-and-why-do-we-need-it](https://stackoverflow.com/questions/40061263/what-is-ca-certificate-and-why-do-we-need-it)
- [https://security.stackexchange.com/questions/20803/how-does-ssl-tls-work](https://security.stackexchange.com/questions/20803/how-does-ssl-tls-work)
- [https://serverfault.com/questions/9708/what-is-a-pem-file-and-how-does-it-differ-from-other-openssl-generated-key-file](https://serverfault.com/questions/9708/what-is-a-pem-file-and-how-does-it-differ-from-other-openssl-generated-key-file)
