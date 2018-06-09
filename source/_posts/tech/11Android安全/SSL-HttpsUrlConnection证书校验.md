---
title: ssl证书校验机制
urlname: android_security_ssl_verify
date: 2017-04-15
comments: true
#description: 
#tags: [git,hah,hha]
#keywords: git
categories: 11Android_Security
tags:
    - ssl
    - verify
    - 证书
    - 安全
---


2.3	SSL证书校验机制 Https
2.3.1	简单解释
典型的场景是现在我们常用的 HTTPS 机制。HTTPS 实际上是利用了 Transport Layer
Security/Secure Socket Layer（TLS/SSL）来实现可靠的传输。TLS 为 SSL 的升级版本，目前广泛应用的为 TLS 1.0，对应到 SSL 3.1 版本。
建立安全连接的具体步骤如下：
•	客户端浏览器发送信息到服务器，包括随机数 R1，支持的加密算法类型、协议版本、压缩算法等。注意该过程为明文。
•	服务端返回信息，包括随机数 R2、选定加密算法类型、协议版本，以及服务器证书。注意该过程为明文。
•	浏览器检查带有该网站公钥的证书。该证书需要由第三方CA 来签发，浏览器和操作系统会预置权威CA 的根证书。如果证书被篡改作假（中间人攻击），很容易通过 CA 的证书验证出来。
•	如果证书没问题，则用证书中公钥加密随机数 R3，发送给服务器。此时，只有客户端和服务器都拥有 R1、R2 和 R3 信息，基于 R1、R2 和 R3，生成对称的会话密钥（如 AES算法）。后续通信都通过对称加密进行保护。
 https://www.cnblogs.com/Anker/p/6082966.html
2.3.2	SSL具体协议分析
协议具体内容解释如下：
 
1. client_hello
（1）支持的协议版本，比如TLS 1.0
（2）支持的加密算法(Cipher Specs)
（3）客户端生成的随机数1(Challenge)，稍后用于生成"对话密钥"。

2. server_hello
（1） 确认使用的协议版本
（2） 服务器生成的随机数2，稍后用于生成"对话密钥"
（3） session id
（4） 确认使用的加密算法
Certificate 服务器证书
server_key_exchange
如果是DH算法，这里发送服务器使用的DH参数。RSA算法不需要这一步。
certificate_request 要求客户端提供证书，包括
（1）客户端可以提供的证书类型
（2）服务器接受的证书distinguished name列表，可以是root CA或者subordinate CA。如果服务器配置了
trust keystore, 这里会列出所有在trust keystore中的证书的distinguished name。
server_hello_done
server hello结束

3. certificate
客户端证书
client_key_exchange包含premaster secret。客户端生成第三个随机数。如果是采用RSA算法，会生成一个48字节随机数，然后用server的公钥加密之后再放入报文中；如果是DH算法，这里发送的就是客户端的DH参数，之后服务器和客户端根据DH算法，各自计算出相同的premaster secret。
certificate_verify
发送使用客户端证书给到这一步为止收到和发送的所有握手消息签名结果。
change_cipher_spec
客户端通知服务器开始使用加密方式发送报文。客户端使用上面的3个随机数client random, server random, premaster secret, 计算出48字节的master secret, 这个就是对称加密算法的密钥。
finished
客户端发送第一个加密报文。使用HMAC算法计算收到和发送的所有握手消息的摘要，然后通过RFC5246中定义的一个伪函数PRF计算出结果，加密后发送。

4.服务器发送给客户端
服务器端发送change_cipher_spec和finished消息。到这里握手结束。

双向认证协议抓包数据如下：
 
2.3.3	基本校验机制
 
2.3.4	参考wireshark抓包分析和中间人攻击应用
[具体协议wireshark原理]   http://blog.csdn.net/fw0124/article/details/40983787 
[中间人攻击-看证书重要性(Man-in-the-middle attack)] http://www.freebuf.com/sectool/48016.html

http://www.freebuf.com/sectool/48016.html
https://g2ex.github.io/2015/05/18/Man-in-the-Middle-Attack/
http://sec.chinabyte.com/221/13821221.shtml
https://wizardforcel.gitbooks.io/daxueba-kali-linux-tutorial/content/58.html



3.2	SSL/HttpsUrlConnection流程与证书校验
3.2.1	一个完整的应用

// Load CAs from an InputStream
// (could be from a resource or ByteArrayInputStream or ...)
CertificateFactory cf = CertificateFactory.getInstance("X.509");
// From https://www.washington.edu/itconnect/security/ca/load-der.crt
InputStream caInput = new BufferedInputStream(new FileInputStream("load-der.crt"));
Certificate ca;
try {
    ca = cf.generateCertificate(caInput);
    System.out.println("ca=" + ((X509Certificate) ca).getSubjectDN());
} finally {
    caInput.close();
}

// Create a KeyStore containing our trusted CAs
String keyStoreType = KeyStore.getDefaultType();
KeyStore keyStore = KeyStore.getInstance(keyStoreType);
keyStore.load(null, null);
keyStore.setCertificateEntry("ca", ca);

// Create a TrustManager that trusts the CAs in our KeyStore
String tmfAlgorithm = TrustManagerFactory.getDefaultAlgorithm();
TrustManagerFactory tmf = TrustManagerFactory.getInstance(tmfAlgorithm);
tmf.init(keyStore);

// Create an SSLContext that uses our TrustManager
SSLContext context = SSLContext.getInstance("TLS");
context.init(null, tmf.getTrustManagers(), null);

// Tell the URLConnection to use a SocketFactory from our SSLContext
URL url = new URL("https://certs.cac.washington.edu/CAtest/");
HttpsURLConnection urlConnection =
    (HttpsURLConnection)url.openConnection();
urlConnection.setSSLSocketFactory(context.getSocketFactory());
InputStream in = urlConnection.getInputStream();
copyInputStreamToOutputStream(in, System.out);



3.2.2	JSSE相关类简要说明
1.	SSL/TLS的Java实现—JSSE 
请参考： http://blog.csdn.net/fw0124/article/details/40889167  
http://www.aneasystone.com/archives/2016/04/java-and-https.html 
写的太好抄一遍来
使用SSL/TLS协议通信，客户端和服务器端都可能要设置用于证实自己身份的安全证书，并且还要设置信任对方的哪些安全证书。
按照理论上，一共需要准备四个文件，两个keystore文件和两个truststore文件。
通信双方分别拥有一个keystore和一个truststore，keystore用于存放自己的密钥和公钥，truststore用于存放所有需要信任方的公钥。
当然为了方便可以直接使用keystore替代truststore（免去证书导来导去），因为对方的keystore包含了自己需要的信任公钥。
在用JSSE实现SSL通信过程中主要会遇到以下类和接口，由于过程中涉及到加解密、密钥生成等运算的框架和实现，所以也会间接用到JCE包的一些类。如图为JSSE接
口的主要类图：
 
①  通信核心类——SSLSocket和SSLServerSocket。对于使用过socket进行通信开发的朋友比较好理解，它们对应的就是Socket与ServerSocket，只是表示实现了SSL协议的Socket和ServerSocket，同时它们也是Socket与ServerSocket的子类。SSLSocket负责的事情包括设置加密套件、管理SSL会话、处理握手结束时间、设置客户端模式或服务器模式。
②  客户端与服务器端Socket工厂——SSLSocketFactory和SSLServerSocketFactory。在设计模式中工厂模式是专门用于生产出需要的实例，这里也是把SSLSocket、SSLServerSocket对象创建的工作交给这两个工厂类。
③  SSL会话——SSLSession。安全通信握手过程需要一个会话，为了提高通信的效率，SSL协议允许多个SSLSocket共享同一个SSL会话，在同一个会话中，只有第一个打开的SSLSocket需要进行SSL握手，负责生成密钥及交换密钥，其余SSLSocket都共享密钥信息。
④  SSL上下文——SSLContext。它是对整个SSL/TLS协议的封装，表示了安全套接字协议的实现。主要负责设置安全通信过程中的各种信息，例如跟证书相关的信息。并且负责构建SSLSocketFactory、SSLServerSocketFactory和SSLEngine等工厂类。
⑤  SSL非阻塞引擎——SSLEngine。假如你要进行NIO通信，那么将使用这个类，它让通过过程支持非阻塞的安全通信。
⑥  密钥管理器——KeyManager。此接口负责选择用于证实自己身份的安全证书，发给通信另一方。KeyManager对象由KeyManagerFactory工厂类生成。
⑦  信任管理器——TrustManager。此接口负责判断决定是否信任对方的安全证书，TrustManager对象由TrustManagerFactory工厂类生成。
2.	SSL握手简单
https协议利用SSL协议验证通信双方（客户端、服务器端）的身份、协商加密算法与密钥。Android中HttpsConnection类实现了https协议。SSLSocket相关类实现了SSL协议。HttpsConnection只需通过SocketFactory以及Socket提供的抽象接口就可以启动SSL握手过程。
主要由SSLSocket、OpenSSLSocketImpl、SSLParameters、SSLSession、OpenSSLSocketFactoryImpl、OpenSSLSessionImpl、SSLSessionContext实现SSL协议。HttpsConnection使用系统提供的默认工厂SSLSocketFactoryImpl创建OpenSSLSocketImpl。OpenSSLSocketImpl通过JNI调用openSSL库提供的api进行握手。注意握手信息是在SocketImpl对象上传输的。SSLSession类封装了SSL会话信息，它存储了SSL握手的结果。最后SSLSessionContext相当与SSLSession的缓存池。这些类之间的关系如下图所示：图3 SSLSocket类图
 

类之间的调用顺序如下： 图4 SSLSocket 时序图
3.2.3	Android HttpsUrlConnection流程图解
主要流程如下：

URL. openConnection
-> streamHandler.openConnection    ->   setupStreamHandler
-> HttpsHandler -> OkHttpClient  open ->  HttpsURLConnectionImpl
-> DelegatingHttpsURLConnection

TrustManagerFactory (TrustManagerFactoryImpl engineGetTrustManagers) -> init (AndroidCAStore default) -> getTrustManagers -> TrustManagerImpl

TrustManagerImpl     checkTrusted
TrustManagerImpl    checkTrustedRecursive
TrustManagerImplTest  testLearnIntermediate
 
TrustManagerImpl     findAllTrustAnchorsByIssuerAndSignature
TrustManagerImpl     findAllIssuers
 
 
TrustedCertificateStore  findCert
TrustedCertificateStore  hash
 
NativeCrypto.X509_NAME_hash_old

SSLContext init  (null, tmf.getTrustManagers(), null);
getInstance -> SSLContextSpi -> OpenSSLContextImpl -> DefaultSSLContextImpl
-> OpenSSLSocketFactoryImpl -> OpenSSLSocketImpl  verifyCertificateChain
-> SSLParametersImpl setCertificateValidation
-> Platform -> TrustManagerImpl 接上


 
这里需要额外说明的是：在执行nativeconnect函数时，已经在openSSL库中执行了握手过程。但openSSL库中不会进行任何验证，因为验证模式被设置为SSL_VERIFY_NONE。因此，Android中身份验证是由checkServerTrusted函数实现的。 
其中TrustManagerImpl     checkTrusted是验证的主要流程
checkServerTrusted –> checkTrusted -> checkTrustedRecursive

1. If the current certificate in the chain is self-signed verify the chain as is.
2. Try building a chain via any trust anchors that issued the current certificate.
3. If we were unable to find additional trusted issuers, verify the current chain.
4. Use the certificates provided by the peer to grow the chain.
5. Finally try the cached intermediates to handle server that failed to send them.
6. We were unable to build a valid chain, throw the last error encountered.
7. If no errors were encountered above then verifyChain was never called because it was


Android系统使用openSSL库来交换证书、协商加密算法，但认证服务器段身份并非由openSSL库完成。Android系统将自己的证书、私钥存储在密钥仓库中。.在进行openSSL握手之前，将私钥以及证书放入SSL_CTX变量中。握手结束后，Android系统从SSLSession中取出服务器的证书，利用可信密钥仓库来检验该证书是否可信。相关类以及类之间的关系如下图所示：图 5 cetificate 类图
 

3.2.4	证书校验
证书校验算法简单说明如下：参考《公钥基础设施(pki)_实现和管理电子安全.pdf》一书
证书验证过程主要包含：证书路径生成和证书路径验证两个步骤，其中具体的验证流程参考代码，还有一些很复杂的交叉证书验证的流程，请参考PKI相关的书籍。
https://en.wikipedia.org/wiki/X.509 
https://en.wikipedia.org/wiki/Certification_path_validation_algorithm 
http://www.oasis-pki.org/pdfs/Understanding_Path_construction-DS2.pdf 
https://www.ietf.org/rfc/rfc5280.txt 
http://itmyhome.com/java-api/java/security/cert/CertPathBuilder.html 
http://www.beansoft.biz/weblogic/docs92/security/certpath.html 
https://www.entrust.com/wp-content/uploads/2013/05/pathvalidation_wp.pdf 
3.2.5	SSL证书使用的建议
参考Wiki案例：Android HttpsSSL证书安全与预制浅析.doc


## 中间证书
如果安全套接字层 (SSL) 证书是在本地生成，那么必须从认证中心 (CA) 获取根 (root.cer) 证书以及任何中间 (intermediate.cer) 证书，然后将这些证书添加到 HTTP Server 密钥库。安装根证书和中间证书会创建信任链，从而使客户机和服务器之间的 SSL 握手能够正常工作。

中间证书有两种方式验证：
•	原理上浏览器预置或缓存这种方法也存在
•	客户端自动解析下载(大多数客户端并不支持解析下载，会直接返回错误)
•	配置服务器直接把中间证书传给客户端 (一般采用这种方式，配置服务器即可)
服务器配置方式参考： https://whatsmychaincert.com/ 
如Apache配置方式：
SSLEngine on
SSLCertificateKeyFile /path/to/example.com.key
SSLCertificateFile /path/to/example.com.crt
SSLCertificateChainFile /path/to/example.com.chain.crt

参考：
https://www.myssl.cn/tools/downloadchain.html 
https://www.myssl.cn/home/article42.html   
http://professor.blog.51cto.com/996189/1596985 
