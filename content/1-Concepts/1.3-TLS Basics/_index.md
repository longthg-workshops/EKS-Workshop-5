---
title: "TLS basics"
weight: 3
chapter: false
pre: "<b> 1.3 </b>"
---

### TLS Certificate

A certificate is used to guarantee trust between 2 parties during a transaction.

**_Example:_** when a user tries to access web server, tls certificates ensure that the communication between them is encrypted.

  ![EKS](/images/0002/0001.png?featherlight=false&width=90pc)
  
  
### Symmetric Encryption
It is a secure way of encryption, but it uses the same key to encrypt and decrypt the data and the key has to be exchanged between the sender and the receiver, there is a risk of a hacker gaining access to the key and decrypting the data.

![EKS](/images/0002/0002.png?featherlight=false&width=90pc)
  
### Asymmetric Encryption
Instead of using single key to encrypt and decrypt data, asymmetric encryption uses a pair of keys, a private key and a public key.

![EKS](/images/0002/0003.png?featherlight=false&width=90pc)

![EKS](/images/0002/0004.png?featherlight=false&width=90pc)

![EKS](/images/0002/0005.png?featherlight=false&width=90pc)

![EKS](/images/0002/0006.png?featherlight=false&width=90pc)
  

#### How do you look at a certificate and verify if it is legit?

- Who signed and issued the certificate ?

- If you generate the certificate then you will have it sign it by yourself; that is known as self-signed certificate.

![EKS](/images/0002/0007.png?featherlight=false&width=90pc)

#### How do you generate legitimate certificate? How do you get your certificates singed by someone with authority?
- That's where **`Certificate Authority (CA)`** comes in for you. Some of the popular ones are Symantec, DigiCert, Comodo, GlobalSign etc.

![EKS](/images/0002/0008.png?featherlight=false&width=90pc)

![EKS](/images/0002/0009.png?featherlight=false&width=90pc)

![EKS](/images/0002/00010.png?featherlight=false&width=90pc)

### Public Key Infrastructure
   
![EKS](/images/0002/00011.png?featherlight=false&width=90pc)
   
### Certificates naming convention

![EKS](/images/0002/00012.png?featherlight=false&width=90pc)