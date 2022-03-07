<!-- *********************************************************************** -->
<!--                                                                         -->
<!--                                +###****.                                -->
<!--                                =***@@@+                                 -->
<!--            *%*   -%%:  -*%%%#     :@@@=:   #%##%%#=-*%%%*:              -->
<!--            %@%   =@@: #@@*=+*.    -==*@@+  @@@*=+@@@%+=#@@=             -->
<!--            %@%   -@@:-@@-     .==.    *@@. %@#   =@@:   %@#             -->
<!--            +@@+-=%@@..%@@+--+..%@@+--*@@*  @@#   +@@:   @@#             -->
<!--             -*%@@#+.   =#%@@%.  -*%@@%*-   %@*   =@@:   %@*             -->
<!--                                                                         -->
<!-- README.md                                                               -->
<!--                                                                         -->
<!-- By: aperez-b <100429952@alumnos.uc3m.es>                                -->
<!--                                                                         -->
<!-- Created: 2022/03/07 09:50:11 by aperez-b                                -->
<!-- Updated: 2022/03/07 10:22:40 by aperez-b                                -->
<!--                                                                         -->
<!-- *********************************************************************** -->

# OpenSSL 2022 - Data Protection & Cybersecurity

## Introduction

The objective of this practice is to understand the concepts underlying a public key infrastructure
based on the hierarchical trust model.
Specifically, the objectives are the following:

- Understand required steps in order to create a mini PKI
- Understand required steps in order to an Authority issuing a certificate
- Understand what the certificate role is regarding the signing and verifying of documents

To achieve these objectives each student (or student group) becomes a ROOT CERTIFICATION AUTHORITY (equal to the role of “Fábrica Nacional de Moneda y Timbre” in the real world). Such Authority (AC1), according to organizational reasons (for example, to have a local office in all different districts), has some SUBORDINATE CERTIFICATION AUTHORITIES (AC2, AC3,…ACn). Moreover, these last Authorities are in charge of issuing public key certificates to people (A, B, C…).

The group of all the Authorities composes a Public Key Infrastructure (PKI).

## Configuration of AC1 (root AC)

We will apply the following commands: ``ca``, ``req``, ``x509`` and ``verify``.

The ``ca`` command is a minimal Certification Authority application. It can be used to sign certificate requests in a variety of forms. Its characteristics can be established by means of the file ``openssl.cnf``. Moreover, we must prepare the working environment as a specific directory structure.

The ``req`` command primarily creates and processes certificate requests. It can additionally create self signed certificates for use as root CAs for example.

Command ``x509`` can be used to display certificate information, convert certificates to various forms, and sign certificate requests.
Verify command verifies certificate chains.

1. Generate the directory structure necessary for AC1 and initialize the files ``serial`` and ``index.txt``.

```shell
cd AC1
mkdir requests crls newcerts private
echo '01' > serial
touch index.txt
cd ..
```

2. Generate the RSA key-pair and the self signed certificate for AC1. Analyze the output.

```shell
cd AC1
openssl req -x509 -newkey rsa:2048 -days 360 -out ac1cert.pem -outform PEM -config openssl_AC1.cnf
cd ..
```

The command requests a passphrase to generate AC1 private key. Remember it when
you want to use this key.

```shell
cd AC1
openssl x509 -in ac1cert.pem -text -noout
cd ..
```

## Configuration of AC2 (subordinate AC)

3. Generate the directory structure necessary for AC2 and initialize the files ``serial`` and ``index.txt``.

```shell
cd AC2
mkdir requests crls newcerts private
echo '01' > serial
touch index.txt
cd ..
```

4. Generate the RSA key-pair for AC2 and the certificate request which will be sent to AC1 and 'send' it to AC1. Analyze the results.

```shell
cd AC2
openssl req -newkey rsa:2048 -days 360 -out ac2req.pem -outform PEM -config openssl_AC2.cnf
cd ..
```

The command requests a passphrase to generate AC2 private key. Remember it when you want to use this key.

```shell
cd AC2
openssl req -in ac2req.pem -text -noout
cp ac2req.pem ../AC1/requests
cd ..
```

## Generation of AC2’s certificate by AC1

5. Verify the request **sent** by AC2.

```shell
cd AC1
openssl req -in ./requests/ac2req.pem -text -noout
cd ..
```

6. Generate the corresponding certificate for AC2 and **send** it back to AC2. Rename ``01.pem`` into ``ac2cert.pem``, because AC2 has this name in its configuration file.

```shell
cd AC1
openssl ca -in ./requests/ac2req.pem -notext -extensions v3_subca -config openssl_AC1.cnf
cd ..
```

AC1 needs its private key to generate AC2 certificate. AC1 passphrase is requested

```shell
cd AC1
cp ./newcerts/01.pem ../AC2/ac2cert.pem
cd ..
```

## Generation of keys for entity A as well as its corresponding certificate request

7. For entity A, generate an RSA key-pair as well as a certificate request and **send** it to AC2 (when generating the certificate requests, fill in ALL the requested fields and indicate ``ES`` as country, ``MADRID`` as province, “UC3M” as organization, and any common name (eg, one NIA) and the email is your student email.

```shell
cd A
openssl req -newkey rsa:1024 -days 360 -sha1 -keyout Akey.pem -out Areq.pem
cd ..
```

The command requests a passphrase to generate A private key. Remember it when you want to use this key.

```shell
cd A
openssl req -in Areq.pem -text -noout
cp Areq.pem ../AC2requests/
cd ..
```

## Generation of A certificate by AC2

8. Verify the request “sent” by A.

```shell
cd AC2
openssl req -in ./requests/Areq.pem -text -noout
cd ..
```

9. Generate certificate for A and “send” it back to this entity.

```shell
cd AC2
openssl ca -in ./requests/Areq.pem -notext -config ./openssl_AC2.cnf
cd ..
```

AC2 needs its private key to generate A certificate. AC2 passphrase is request

```shell
cd AC2
cp ./newcerts/01.pem ../A/Acert.pem
cd ..
```

10. Analyze changes in AC2 directory and check the resulting certificate:

```shell
cd A
openssl x509 -in Acert.pem -text -noout
cd ..
```
