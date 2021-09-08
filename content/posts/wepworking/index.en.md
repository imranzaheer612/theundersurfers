---
title: "WEP"
date: 2021-08-11T03:27:44+05:00
lastmod: 2021-10-01T17:55:28+08:00
description: "How WEP security works."
resources:
- name: "featured-image"
  src: "featured-image.png"

tags: ["wireless", "rc4", "wep working", "Wired Equivalent Privacy"]
categories: ["security", "wireless"]
theme: "full"
---


WEP(Wired Equivalent Privacy) is the wireless security protocol using RC4 algorithm.

<!--more-->


Wired Equivalent Privacy (WEP) is a security protocol and encryption algorithm that secures wireless and Wi-Fi networks. This is not preferred today because of its vulnerabilities thats why its also called **Worst Ever Privacy**

## WEP

Important components of WEP are

* Initialization Vector (IV)
* KEY (Password)
* RC4
* Key Stream

So to encrypt each packet WEP uses the RC4 algorithm which generates a keystream that is used to encrypt the package.

**Keystream XOR "plain data to encrypt" = CipherText**

Remember the keystream is generated using RC4 algorithm.

### RC4 

RC4 uses **Initialization Vector (IV) + KEY(Password)** to make a keystream which gonna encrypt the package. The reason IV is used here because if RC4 uses the Key only then all the packets gonna have the same keystream which is not good.

So in order to generate a different keystream for each packet **IV** is introduced.

{{< figure src="https://upload.wikimedia.org/wikipedia/commons/thumb/4/44/Wep-crypt-alt.svg/305px-Wep-crypt-alt.svg.png" title="WEP Working" >}}



###	Initializing Vector
So Initializing vector nothing but a **random 24bit number** produced so that every packet has unique keystream to decrypt it.

## Device Side

1. **IV + KEY (Password) --> RC4 --> keystream**
    *	IV + KEY is also known as **seed** (64/128 bit)
    *	Seed is converted to keystream using **RC4 algorithm**

1. **Keystream xor “data to send to the router” = Cipher Text**
    *	Data is encrypted using simple **XOR** function 

    {{< figure src="wep keystream.png" title="Using Keystream to generate CypherText" >}}


1.	Packet is sent to the router and now the packet contains two components
	  * **IV (initialization vector)**
	  * **Cypher Text**
    
{{< figure src="packet wep.png" title="Packet" >}}

**IV** is added to the packet because AP (Access Point) only have the pre shared KEY (password). AP doesn't know which random number (IV) is used with the KEY to generate the Keystream.

## Access Point (AP) Side

Ap cannot simply decrypt the package using the **KEY only** because the packet’s encryption key (keystream) is generated using a random number (IV)

1. So router gets this random number (IV) from the packet, generate the keystream (encryption key) using the same RC4 algorithm <br>
```
IV (Obtained from the packet) + KEY(Password) --> RC4 --> Keystream
```
<br>
  Now AP can use this key stream to decrypt data

2. **Decryption:** 
   ```javascript
   Keystream . XOR . "Cypher Text" = "Plain Text"
   ```
{{< figure src="wep decrypt.jpg" title="Decrypting Packet" >}}

