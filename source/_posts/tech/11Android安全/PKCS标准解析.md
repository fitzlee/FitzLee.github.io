---
title: PKCS标准解析
urlname: android_security_pkcs_standard
date: 2017-04-10
comments: true
#description: 
#tags: [git,hah,hha]
#keywords: git
categories: 11Android_Security
tags:
    - PKCS标准解析
    - pkcs
    - 安全
---

## PKCS标准  Public-Key Cryptography Standards
RSA主导标准，RSA信息安全公司旗下的RSA实验室为了发扬公开密钥技术的使用，1990年开始便发展了一系列的公开密钥密码编译标准。只不过，虽然该标准具有相当大的象征性，也被信息界的产业所认同；但是，若RSA公司认为有必要，这些标准的内容仍然可能会更动。所幸，这些变动并不大；此外，这几年RSA公司也与其他组织（比较知名的有IETF、PKIX）将标准的制定通过standards track程序来达成。

1)	PKCS#1：RSA加密标准。PKCS#1定义了RSA公钥函数的基本格式标准，特别是数字签名。它定义了数字签名如何计算，包括待签名数据和签名本身的格式；它也定义了PSA公/私钥的语法    
2)	PKCS#2：涉及了RSA的消息摘要加密，这已被并入PKCS#1中。
3)	PKCS#3：Diffie-Hellman密钥协议标准。PKCS#3描述了一种实现Diffie- Hellman密钥协议的方法。
4)	PKCS#4：最初是规定RSA密钥语法的，现已经被包含进PKCS#1中。
5)	PKCS#5：基于口令的加密标准。PKCS#5描述了使用由口令生成的密钥来加密8位位组串并产生一个加密的8位位组串的方法。PKCS#5可以用于加密私钥，以便于密钥的安全传输（这在PKCS#8中描述）。
6)	PKCS#6：扩展证书语法标准。PKCS#6定义了提供附加实体信息的X.509证书属性扩展的语法（当PKCS#6第一次发布时，X.509还不支持扩展。这些扩展因此被包括在X.509中）。
7)	PKCS#7：密码消息语法标准。PKCS#7为使用密码算法的数据规定了通用语法，比如数字签名和数字信封。PKCS#7提供了许多格式选项，包括未加密或签名的格式化消息、已封装（加密）消息、已签名消息和既经过签名又经过加密的消息。
8)	PKCS#8：私钥信息语法标准。PKCS#8定义了私钥信息语法和加密私钥语法，其中私钥加密使用了PKCS#5标准。
9)	PKCS#9：可选属性类型。PKCS#9定义了PKCS#6扩展证书、PKCS#7数字签名消息、PKCS#8私钥信息和PKCS#10证书签名请求中要用到的可选属性类型。已定义的证书属性包括E-mail地址、无格式姓名、内容类型、消息摘要、签名时间、签名副本（counter signature）、质询口令字和扩展证书属性。
10)	PKCS#10：证书请求语法标准。PKCS#10定义了证书请求的语法。证书请求包含了一个唯一识别名、公钥和可选的一组属性，它们一起被请求证书的实体签名（证书管理协议中的PKIX证书请求消息就是一个PKCS#10）。
11)	PKCS#11：密码令牌接口标准。PKCS#11或“Cryptoki”为拥有密码信息（如加密密钥和证书）和执行密码学函数的单用户设备定义了一个应用程序接口（API）。智能卡就是实现Cryptoki的典型设备。注意：Cryptoki定义了密码函数接口，但并未指明设备具体如何实现这些函数。而且Cryptoki只说明了密码接口，并未定义对设备来说可能有用的其他接口，如访问设备的文件系统接口。
12)	PKCS#12：个人信息交换语法标准。PKCS#12定义了个人身份信息（包括私钥、证书、各种秘密和扩展字段）的格式。PKCS#12有助于传输证书及对应的私钥，于是用户可以在不同设备间移动他们的个人身份信息。
13)	PDCS#13：椭圆曲线密码标准。PKCS#13标准当前正在完善之中。它包括椭圆曲线参数的生成和验证、密钥生成和验证、数字签名和公钥加密，还有密钥协定，以及参数、密钥和方案标识的ASN.1语法。
14)	PKCS#14：伪随机数产生标准。PKCS#14标准当前正在完善之中。为什么随机数生成也需要建立自己的标准呢？PKI中用到的许多基本的密码学函数，如密钥生成和Diffie-Hellman共享密钥协商，都需要使用随机数。然而，如果“随机数”不是随机的，而是取自一个可预测的取值集合，那么密码学函数就不再是绝对安全了，因为它的取值被限于一个缩小了的值域中。因此，安全伪随机数的生成对于PKI的安全极为关键。
15)	PKCS#15：密码令牌信息语法标准。PKCS#15通过定义令牌上存储的密码对象的通用格式来增进密码令牌的互操作性。在实现PKCS#15的设备上存储的数据对于使用该设备的所有应用程序来说都是一样的，尽管实际上在内部实现时可能所用的格式不同。PKCS#15的实现扮演了翻译家的角色，它在卡的内部格式与应用程序支持的数据格式间进行转换。


PKCS #1
RSA Cryptography Standard[1]
See RFC 3447. Defines the mathematical properties and format of RSA public and private keys (ASN.1-encoded in clear-text), and the basic algorithms and encoding/padding schemes for performing RSA encryption, decryption, and producing and verifying signatures.

PKCS #2	-	Withdrawn	No longer active as of 2010. Covered RSA encryption of message digests; subsequently merged into PKCS #1.

PKCS #3	1.4	Diffie–Hellman Key Agreement Standard[2]
A cryptographic protocol that allows two parties that have no prior knowledge of each other to jointly establish a shared secret key over an insecure communications channel.

PKCS #4	-	Withdrawn	No longer active as of 2010. Covered RSA key syntax; subsequently merged into PKCS #1.
PKCS #5	2.0	Password-based Encryption Standard[3]
See RFC 2898 and PBKDF2.

PKCS #6	1.5	Extended-Certificate Syntax Standard[4]
Defines extensions to the old v1 X.509 certificate specification. Obsoleted by v3 of the same.

PKCS #7	1.5	Cryptographic Message Syntax Standard[5]
See RFC 2315. Used to sign and/or encrypt messages under a PKI. Used also for certificate dissemination (for instance as a response to a PKCS #10 message). Formed the basis for S/MIME, which is as of 2010 based on RFC 5652, an updated Cryptographic Message Syntax Standard (CMS). Often used for single sign-on.

PKCS #8
1.2	Private-Key Information Syntax Standard[6]
See RFC 5958. Used to carry private certificate keypairs (encrypted or unencrypted).
PKCS #9	2.0	Selected Attribute Types[7]
See RFC 2985. Defines selected attribute types for use in PKCS #6 extended certificates, PKCS #7 digitally signed messages, PKCS #8 private-key information, and PKCS #10 certificate-signing requests.
PKCS #10	1.7	Certification Request Standard[8]
See RFC 2986. Format of messages sent to a certification authority to request certification of a public key. See certificate signing request.

PKCS #11
2.40	Cryptographic Token Interface[9]
Also known as "Cryptoki". An API defining a generic interface to cryptographic tokens (see also hardware security module). Often used in single sign-on, public-key cryptography and disk encryption[10] systems. RSA Security has turned over further development of the PKCS #11 standard to the OASIS PKCS 11 Technical Committee.

PKCS #12
1.1	Personal Information Exchange Syntax Standard[11]
See RFC 7292. Defines a file format commonly used to store private keys with accompanying public key certificates, protected with a password-based symmetric key. PFX is a predecessor to PKCS #12.
This container format can contain multiple embedded objects, such as multiple certificates. Usually protected/encrypted with a password. Usable as a format for the Java key store and to establish client authentication certificates in Mozilla Firefox. Usable by Apache Tomcat.

PKCS #13	–	Elliptic Curve Cryptography Standard
(Apparently abandoned, only reference is a proposal from 1998.)[12]

PKCS #14	–	Pseudo-random Number Generation
(Apparently abandoned, no documents exist.)

PKCS #15	1.1	Cryptographic Token Information Format Standard[13]
Defines a standard allowing users of cryptographic tokens to identify themselves to applications, independent of the application's Cryptoki implementation (PKCS #11) or other API. RSA has relinquished IC-card-related parts of this standard to ISO/IEC 7816-15.[14]


参考：
https://apj.emc.com/emc-plus/rsa-labs/standards-initiatives/public-key-cryptography-standards.htm
https://en.wikipedia.org/wiki/PKCS 
https://zh.wikipedia.org/wiki/%E5%85%AC%E9%92%A5%E5%AF%86%E7%A0%81%E5%AD%A6%E6%A0%87%E5%87%86 

