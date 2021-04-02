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

Today, *Elliptic Curve Cryptography* (**ECC**) appears in [TSL](https://tools.ietf.org/html/rfc4492), [SSL](https://tools.ietf.org/html/rfc5656), [PGP](https://tools.ietf.org/html/rfc6637), and many other things including [Bitcoin](https://en.bitcoin.it/wiki/Elliptic_Curve_Digital_Signature_Algorithm) and [Blockchain](https://arxiv.org/ftp/arxiv/papers/1808/1808.02988.pdf). 

## **The Goal**

In this post, I explain how I apply ECC algorithm on secure transmission channel from echo server to client. I used *Elliptic Curve Diffie-Hellman* (**ECDH**) key exchange to generate keys for *Advanced Encryption Standard* (**AES**). That key used to encrypt the data exchanged between the client and the server.

In addition,  use of  *Elliptic Curve Digital Signature Algorithm* (**ECDSA**) as a authentication mechanism.

### **ECC Over Finite Fields GF(p)**

In mathematics, elliptic curve is described in detail [here](https://mathworld.wolfram.com/EllipticCurve.html). But in cryptography, an elliptic curve have the form known as:

$$
y^2 = x^3 + ax + b\, (\,mod \,\, p)
$$


satisfied

$$
\\4a^3 + 27b^2 \ne 0.
$$

This is required to exclude [singular curves](https://en.wikipedia.org/wiki/Singularity_(mathematics)).


Example: [link](https://www.desmos.com/calculator)

$$
\\ y^2 = x^3 - x + 4
$$

![image01](/images/1_blog_ecc/image01.png)

### **Group operator on ECC**

For an elliptic curve, I have  a set of points that belong to the curve denoted by a set *E(a, b)* along with a special point at infinity, denoted by *O*.  *E(a, b)* is a abelian group under a special addtion operator, denoted by *+*

### **Algebraic addition**

### *P and Q*

If I want to add a point *P* to another point *Q*, I take the following steps:

1. Draw a straight line joining points *P* and *Q*
2. Find the intersection of the joining line and the elliptic curve to get a third point R
3. Reflect the point R along the *x*-axis. It give a point denoted by *R'*.
4. Denoted: *P + Q = R'*

$$
y^2 = x^3 - 5x + 9
$$



![image02](/images/1_blog_ecc/image02.png)



The straight line passing through points *P* and *Q* is
$$
\\ y = \alpha x + \beta
$$

with

$$
\\ \alpha = \frac{y_Q - y_P}{x_Q - x_P}
$$


In this image, there are three intersection points between the straight line and the elliptic curve, satisfied equation under

$$
\\(\alpha x + \beta)^2 = x^3 + ax + b
$$

Two of these roots are *x*-coordinates of points *P* and *Q*, the third root (of point R) can be found using property of a monic polynomial.

$$
\\x^3 - \alpha ^ 2 x^2 + (a - 2 \alpha \beta)x + (b - \beta ^ 2) = 0
$$


By the [Viet theorem](https://www.emathhelp.net/notes/algebra-1/quadratic-equations/viet-theorem/)

$$
\\x_P + x_Q + x_R = \alpha ^ 2
$$


from which I can find the third root

$$
\\x_R = \alpha ^ 2 - x_P - x_Q
$$


In addition, *R* is a straight line passing through *P* and *Q*. So, I can express *y*-coordinate of point *R* as:

$$
\\y_R = \alpha x + \beta
$$

$$
\\y_R = \alpha x_R + (y_P - \alpha x_P) = \alpha (x_R - x_P) + y_P
$$


Result of  addition of two points is the reflection of point *R* along *x*-axis

$$
\\x_{P+Q} = \alpha ^2 - x_P - x_Q
$$

$$
\\y_{P+Q} = -\alpha (x_{P+Q} - x_P) - y_P
$$



### *P and P*

ECC is based on adding a point to ifself *k* times in order to obtain another point

*k x P*.  Adding point to ifself is called point doubling and expressed as *P + P = 2P*

I can compute *2P* using the following steps:

1. Draw a tangent line at *P*
2. Find the intersection of the tangent line with the elliptic curve to obtain point *R*
3. Reflect the point of intersection along the *x*-axis


$$
y^2 = x^3 - 5x + 9
$$


![image03](/images/1_blog_ecc/image03.png)



Point doubling can also be easily expressed algebraically. Similarly to point addition, I calculate the slope. I obtain the slope of a single point *P* at *(x, y)* by:


$$
2y\frac{dy}{dx} = 3x^2 + a
$$


and expressing the slope as:


$$
\alpha^2 = \frac{3x_P^2 + a}{2y_P}
$$


Similar to the above, I have the result:


$$
x_{2P} = \alpha ^2 - 2x_P
$$

$$
y_{2P} = -\alpha (x_{2P} - x_P) - y_P
$$


After that, I illustrate the *Point addition* algorithm on python as follows:

```python
from collections import namedtuple
Point = namedtuple("Point", "x y")

def point_addition(P, Q):    
    if P != Q:
        lamda = (Q.y - P.y)*inverse_mod(Q.x - P.x, p)
    else:
        lamda = (3* P.x^2 + A)*inverse_mod(2*P.y, p)
    Rx = (lamda^2 - Q.x - P.x) % p
    Ry = (lamda*(P.x - Rx)- P.y) % p
    return Point(Rx, Ry)
```

### **Scalar Multiplication**

Scalar multiplication can represented as repeated point addition. 

For example, I want to calculate *3P*. The multiplication can be represented as a series of addition:


$$
3P = P + P + P 
$$


To calculate *3P*, I divide it by 2 steps. 

- Addition two point the similarities:


$$
P + P
$$

- Addition two point are different:


$$
2P + P
$$


Using point addition technique described in *Algebraic addition* to obtain result.

Scalar multiplication algorithm is implemented on python like this code:

```python
from collections import namedtuple
Point = namedtuple("Point", "x y")

def scalar_multiplication(Q, n):
    R = Point(0, 0)
    while n > 0:
        if n % 2 == 1: 
            R = point_addition(R, Q)
        Q = point_addition(Q, Q)
        n = n // 2
    return R
```

### **Generate keys with Elliptic Curve Diffie-Hellman**

Elliptic Curve Diffie-Hellman (*ECDH*) secret key exchange scheme can be used for establishing a shared secret key between two parties. 

Alice and Bob want to create a secure communication channel. They choose ECC parameters *p, a, b* for an Elliptic Curve and base point *G*

After that, following like this image:

![image08](/images/1_blog_ecc/image08.png)

### **Encryption with AES**

After obtain the key by ECDH,I used it to encrypt the data exchanged on the communication channel.

I use AES algorithm, mode ECB. 

You can learn more about this algorithm [here](https://en.wikipedia.org/wiki/Advanced_Encryption_Standard)

### **Signing with ECDSA**



### **Sources**

[Echo Server](https://github.com/lanleft/Project-II)















