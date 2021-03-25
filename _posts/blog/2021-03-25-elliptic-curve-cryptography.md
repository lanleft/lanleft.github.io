---
layout: post
title: "Elliptic Curve Cryptography"
modified: 2021-03-25 13:56:39 +0700
category: [blog]
tags: [ecc, ecdh, ecdsa]
image:
  feature: 
  credit: 
  creditlink: 
comments: 
share: 
---

Today, Elliptic Curve Cryptography (ECC) appears in [TSL](https://tools.ietf.org/html/rfc4492), [SSL](https://tools.ietf.org/html/rfc5656), [PGP](https://tools.ietf.org/html/rfc6637), and many other things including [Bitcoin](https://en.bitcoin.it/wiki/Elliptic_Curve_Digital_Signature_Algorithm) and [Blockchain](https://arxiv.org/ftp/arxiv/papers/1808/1808.02988.pdf). 

## **The Goal**

In this post, I explain how I apply ECC algorithm on secure transmission channel from echo server to client. I used *Elliptic Curve Diffie-Hellman* (**ECDH**) key exchange to generate keys for *Advanced Encryption Standard* (**AES**). That key used to encrypt the data exchanged between the client and the server. 

In addition,  use of  *Elliptic Curve Digital Signature Algorithm* (**ECDSA**) as a authentication mechanism.

## **Elliptic Curve**

In mathematics, elliptic curve is described in detail [here](https://mathworld.wolfram.com/EllipticCurve.html). But in cryptography, an elliptic curve have the form known as:

$$
y^2 = x^3 + ax + b
$$
satisfied:
$$
4a^3 + 27b^2 \ne 0
$$
This is required to exclude [singular curves](https://en.wikipedia.org/wiki/Singularity_(mathematics)).





### **Generate keys with ECDH**





### **Encryption with AES_ECB**





### **Signing with ECDSA**



### **Sources**

[Echo Server](https://github.com/lanleft/Project-II)















