---
title: Android证书应用
urlname: android_security_certificate_apps
date: 2017-02-25
comments: true
#description: 
#tags: [git,hah,hha]
#keywords: git
categories: 11Android_Security
tags:
    - wifi certificate
    - vpn certificate
    - Android
    - 安全
---

Android Settings中主要有凭据存储管理、VPN和WIFI应用等应用到证书的知识。

## 凭据存储(系统TAB/用户TAB)和用户凭据区别

Setting中的显示
1. 受信任凭据-系统TAB
显示/system/etc/security/cacerts/ 下的根Root证书文件，system分区只读，不可更改
2. 受信任凭据-用户TAB 
显示TrustCertificateStore默认路径的CA证书文件，默认路径本来是/data/misc/keychain/cacerts-added，在ActivityThread.main起来的时候改为用户相关/data/misc/user/0/
// Make sure TrustedCertificateStore looks in the right place for CA certificates
final File configDir = Environment.getUserConfigDirectory(UserHandle.myUserId());
TrustedCertificateStore.setDefaultUserDirectory(configDir);

3. 用户凭据
显示/data/misc/keystore/user_0/下，1000开头的所有证书文件。
包含有用户导入的X509和PKCS格式
包含有用户导入的X509证书，作为系统证书的补充
KeyChain API，使得用户可以安全导入自己的证书，包含两种格式X.509(PEM or ASN.1 DER encoded)/PKCS#12，安装过程提示Wlan或VPN/APP两个选项，如下：
1)如果选择WLAN，那么只有Wifi可用此证书，且不会在用户凭据和受信任的凭据里边显示。
2)如果选择VPN/APP，那么第三方应用均可请求使用该证书，且会根据证书类型，选择安全的具体位置，比如CER/DER这种格式会在用户凭据和受信任凭据同时添加显示，PKCS12只会用户凭据里显示。第三方应用可以调用KeyChain接口获取PrivateKey进行签名等操作，还可以使用证书/PublicKey进行加密等操作，但第一次调用接口使用证书需要弹窗验证权限，具体使用依据程序需要，可以用于HTTPS/Email等客户端验证
查看/data/misc/keystore/user_0目录文件如下：
1000_CACERT_123
1000_CACERT_12306
1000_USRPKEY_My+PKey+PChain
1010_CACERT_12306
1010_USRCERT_xxx_WPA_EAPIEEE8021X_TLS_NULL
1010_USRPKEY_xxx_WPA_EAPIEEE8021X_TLS_NULL
1010_USRPKEY_My+PKey+PChain
此目录均为android.security.Keystore接口导入，可加密可不加密存储。
文件可分为三类：
1000  对system可见，上层可以通过KeyChain接口请求用户授权访问
1010  对wifi可见, 其他不可见
other_uid 只对本身应用可见，system等不可见
其中主要又可分为CACERT、USRPKEY、USRCERT这三种类别。


## VPN应用
### VPN的关键技术参数 
VPN产品的关键技术参数包括：支持那些标准（PPTP，L2TP，IPsec，MSLP，SOCK5），支持那些加密算法；最大加密强度是多少；是否支持公钥体系PKI），及IKE密钥管理等。

### VPN分类
IPSEC/Xauth	 
Background	PPTP 是一个基于 PPP 的很基本的协议。PPTP 是微软 Windows 平台第一个支持的 VPN 协议。PPTP 标准并没有实际描述加密和授权特性，并且依赖于 PPP 协议的隧道来实现安全功能。	L2TP 是一个在 IETF RFC 3193 中被正式标准化的高级协议。推荐在需要安全加密的地方用来替代 PPTP。		OpenVPN 是一个高级的开源 VPN 解决方案，由 “OpenVPN technologies” 支持，并且已经成为开源网络领域里的事实标准。OpenVPN 使用成熟的 SSL/TLS 加密协议。
Data Encryption	PPP 负载是使用微软点对点协议（Microsoft’s Point-to-Point Encryption protocol，MPPE）加密。MPPE 实现了 RSA RC4 加密算法，并使用最长 128 位密钥。	L2TP 负载使用标准的 IPSec 协议加密。在 RFC 4835 中指定了使用 3DES 或 AES 加密算法作为保密方式。		OpenVPN 使用 OpenSSL 库来提供加密。OpenSSL 支持好几种不同的加密算法，如：3DES，AES，RC5 等。
Setup / Configuration	Windows 所有版本和大多数其他操作系统包括移动平台都内建了对 PPTP 的支持。PPTP 只需要一个用户名和密码，以及一个服务器地址，所以安装和配置相当简单。	从 2000/XP 起的所有 Windows 平台和 Mac OS X 10.3+ 都内建了 L2TP/IPSec 的支持。大多数现代的移动平台比如 iPhone 和 Android 也有内建的客户端。		OpenVPN 不包含在任何操作系统中，需要安装客户端软件，但安装也是相当简单，基本上 5 分钟可以完成。
Speed	由于使用 128 位密钥，加密开销相比 OpenVPN 使用 256位密钥要小，所以速度感觉稍快一点，但这个差异微不足道。	L2TP/IPSec 将数据封装两次，所以相比其他竞争者效率稍低，速度也慢一些。		当使用默认的 UDP 模式，OpenVPN 的表现是最佳的。
Ports	PPTP 使用 TCP 1723 端口和 GRE（协议 47）。通过限制 GRE 协议，PPTP 可以轻易地被封锁。	L2TP/IPSec 使用 UDP 500 端口用来初始化密钥交换，使用协议 50 用来传输 IPSec 加密的数据（ ESP ），使用 UDP 1701 端口用来初始化 L2TP 的配置，还使用 UDP 4500 端口来穿过 NAT。L2TP/IPSec 相比 OpenVPN 容易封锁，因为它依赖于固定的协议和端口。		OpenVPN 可以很容易的配置为使用任何端口运行，也可以使用 UDP 或 TCP 协议。为了顺利穿越限制性的防火墙，可以将 OpenVPN 配置成使用 TCP 443 端口，因为这样就无法和标准的 HTTPS 无法区分，从而极难被封锁。
Stability / Compatibility	PPTP 不如 OpenVPN 可靠，也不能像 OpenVPN 那样在不稳定网络中快速恢复。另外还有部分同 GRE 协议和一些路由器的兼容性问题。	L2TP/IPSec 比 OpenVPN 更复杂，为了使在 NAT 路由器下的设备可靠地使用，配置可以会更加困难。但是，只要服务器和客户端都支持 NAT 穿越，那么就没什么问题了。		无论是无线网络、蜂窝网络，还是丢包和拥塞经常发生的不可靠网络，OpenVPN 都非常稳定、快速。对于那些相当不可以的连接，OpenVPN 有一个 TCP 模式可以使用，但是要牺牲一点速度，因为将 TCP 封装在 TCP 时效率不高。
Security weaknesses	微软实现的 PPTP 有一个严重的安全问题（serious security vulnerabilities）。对于词典攻击来说 MSCHAP-v2 是很脆弱的，并且 RC4 算法也会遭到“位翻转攻击（ bit-flipping attack ）”。如果保密是重要的，微软也强烈建议升级到 IPSec。	IPSec 没有明显的漏洞，当和安全加密算法如 AES 一起使用时，被认为是很安全的。		OpenVPN 也没有明显漏洞，当和安全加密算法如 AES 一起使用时，也被认为是相当安全的。
Client compatibility	•	Windows
•	Mac OS X
•	Linux
•	Apple iOS
•	Android
•	DD-WRT	Windows
Android	•		•	Windows
•	Mac OS X
•	Linux
Conclusion	由于主要的安全漏洞，除了兼容性以外没有好的理由选择使用 PPTP。如果你的设备既不支持 L2TP/IPSec 又不支持 OpenVPN，那么 PPTP 是一个合理的选择。如果关心快速安装和简易配置，那么 L2TP/IPSec 值得考虑。	L2TP/IPSec 是优秀的，但相比 OpenVPN 的高效和杰出的稳定性要落后一点。如果你使用运行 iOS 或 Android 的移动设备，那么这就是最佳的选择，因为 OpenVPN 目前还不支持这些平台。另外，如果需要快速安装，L2TP/IPSec 也是一个较佳的选择。		对于所有的 Windows, Mac OS X 以及 Linux 桌面用户来说，OpenVPN 是最好的选择。OpenVPN 速度快，并且安全可信。但劣势是缺乏对移动设备的支持，另外还需要安装第三方客户端。


参考资料：
https://liweitianux.wordpress.com/2011/04/30/pptp-l2tp-openvpn/ 
http://www.cnblogs.com/haitao-fan/archive/2012/05/30/2526142.html 
https://linux.cn/article-3407-1.html 

### 主要概述

**PPTP**
(Point-to-Point Tunneling Protocol (PPTP))
PPTP使用传输控制协议（TCP）创建控制通道来发送控制命令，以及利用通用路由封装（GRE）通道来封装点对点协议（PPP）数据包以发送数据
微软点对点协议（PPP）协议堆栈中，提供了各种标准的身份验证与加密机制来支持PPTP.
可以搭配PAP、CHAP、MS-CHAP v1/v2或EAP-TLS来进行身份验证。也可以搭配微软点对点加密（MPPE）或IPSec的加密机制来提高安全性

**L2TP(Layer Two Tunneling Protocol)**
IPSec PSK、IPSec RSA
整个L2TP数据包，包括有效附载（payload）及标头（header），皆是用用户数据报协议（UDP）来发送。
L2TP本身并不提供加密和认证，但常用IPsec来确保L2TP的安全及完整性，两种协议的组合一般被称为L2TP/IPsec
PPTP和 L2TP都是OSI第二层的VPN，也是较早期的VPN协议，在IPsec出现前是最主要的VPN类型，今天使用仍然相当广泛，典型地是使用两台托管的Windows 2000服务器作为VPN网关。前者是微软在1996年制定，后者则由CISCO汇同微软在PPTP和L2F的基础上制定。第二层协议对ＰＰＰ协议本身并没有做任何修改，只是将用户的 PPP帧基于GRE封装成IP报文。 
PPTP和L2TF均具有简单易行的优点，但是它们的可扩展性都不好。更重要的是，它们都没有提供内在的安全机制，它们不能支持企业和企业的外部客户以及供应商之间会话的保密性需求，因此它们不支持用来连接企业内部网和企业的外部客户及供应商的企业外部网Extranet的概念。Extranet需要对隧道进行加密并需要相应的密钥管理机制。PPTP和L2TP限制同时最多只能连接255个用户。端点用户需要在连接前手工建立加密信道。认证和加密受到限制，没有强加密和认证支持。安全程度差，是PPTP/L2TF简易型VPN最大的弱点。 
PPTP和L2TP最适合用于客户端远程访问虚拟专用网，作为安全要求高的企业信息，使用PPTP/L2TP与明文传送的差别不大。PPTP/L2TP不适合于向Ipv6的转移。

**IPSEC**  
Xauth PSK、Xauth RSA、Hybrid RSA
IPsec协议工作在OSI模型的第三层，使其在单独使用时适于保护基于TCP或UDP的协议（如安全套接子层（SSL）就不能保护UDP层的通信流）。这就意味着，与传输层或更高层的协议相比，IPsec协议必须处理可靠性和分片的问题，这同时也增加了它的复杂性和处理开销。相对而言，SSL/TLS依靠更高层的TCP（OSI的第四层）来管理可靠性和分片。
IPSec是IETF(Internet Engineer Task Force)完善的安全标准，它把几种安全技术结合在一起形成一个较为完整的体系，通过对数据加密、认证、完整性检查来保证数据传输的可靠性、私有性和保密性。IPSec由IP认证头AH(Authentication Header)、IP安全载荷封载ESP(Encapsulated Security Payload)和密钥管理协议组成。是目前支持最广泛的VPN协议。 
IPSec协议是一个范围广泛、开放的虚拟专用网安全协议。IPSec适应向IP v6迁移，它提供所有在网络层上的数据保护，提供透明的安全通信。IPSec用密码技术从三个方面来保证数据的安全。即： 
认证：用于对主机和端点进行身份鉴别。 
完整性检查：用于保证数据在通过网络传输时没有被修改。 
加密：加密IP地址和数据以保证私有性。 
IPsec是完全意义上的VPN，能直接与PKI、CA设备密切协同完成认证功能。IPSec的缺占是需要固定范围的IP地址，因此在动态分配IP地址时不太适合于IPSec。（图腾VPN是目前不多的部分支持动态IP的VPN网关技术）；除了TCP/IP协议外，IPSec不支持其他协议。除了包过滤之外，它没有指定其他访问控制方法。另外，微软在windows中没有集成对IPSec的支持，因此，windows客户端需要专门的软件或硬件的支持。 
IPSec最适合可信的网关到网关之间的虚拟专用网，即企业广域网(Extranet)的构建。
SSTP
SSL/TLS


**VPN配置文件**
由系统创建并由KeyStore进行加密存储，如下：
配置文件解密后
 包含：名称/Type/IP/用户名/密码////用户证书/CA证书/ 等内容
Type: 1 = L2TP/IPSEC
Type: 0= PPTP
Type: 4= IPSec Xauth

**证书使用：**
At runtime, the framework retrieves the client certifcate (w in
Listing 9-8) and the CA certifcate (u in Listing 9-8) from the system keystore and passes them to racoon via the control socket.
运行时，framework按照配置文件将证书从系统keystore中抽取出来，通过socket传递给racoon守护进程。

The private key blob is never directly passed to the racoon daemon, only its alias (vpnclient). 私钥文件不传递给racoon，而是用到的时候直接由keystore进行加解密操作，从而保证私钥安全性。

PPTP和L2TP守护进程：
PPTP和L2TP隧道实现，包含PPP守护进程mtpd、和PPPoPNS和PPPoLAC内核驱动。
IPSec实现，包含内置IPSec内核支持和racoon IKE守护进程。
Init.rc中可以看到两个守护进程的声明：
 
系统设置启动VPN后，通过套接字发送控制命令，IP地址网络掩码等，两个守护进程负责根据参数创建隧道，完成接口配置，写入并处理/data/misc/vpn/state文件，调用系统ConnectivityService配置路由服务器等，然后发送给netd(root权限运行)，开始修改内核包过滤和路由表。
 
**VPN对Android SDK开放的API**
VPNService和VPNService.Builder
这种方法最好的实现是OpenVPN的实现，参考OpenVPN

参考：
https://developer.android.com/reference/android/net/VpnService.html 
https://developer.android.com/reference/android/net/VpnService.Builder.html 



## WLan应用
### WiFi分类
WiFi无线电分类：
Wi-Fi可分为五代。[6]由于ISM频段中的2.4GHz频段被广泛使用，例如微波炉、蓝牙，它们会干扰WiFi，令速度减慢，5GHz干扰则较小。双频路由器可同时使用2.4GHz和5GHz，设备则只能使用某一个频段。
第一代802.11，1997年制定，只使用2.4GHz，最快2Mbit/s
第二代802.11b，1999年制定，只使用2.4GHz，最快11Mbit/s，正逐渐淘汰
第三代802.11g/a，2007收录，分别使用2.4GHz和5GHz，最快54Mbit/s
            802.11i，安全和鉴权（Authentication）方面的补充。
第四代802.11n，2008以后，可使用2.4GHz或5GHz，20和40MHz信道宽度下最快72和150Mbit/s
第五代802.11ac，2011年后，只使用5GHz

### Wifi认证分类：
WEP	口令即可	
WPA/WPA2   PSK	口令即可	
802.1 EAP	PEAP
TLS 
TTLS
PWD
SIM
AKA	PEAP认证，通过SSL连接，只认证服务端
TLS双向认证，需要签发X.509客户端证书私钥，可以撤销禁止网络访问
TTLS不需要认证客户端
PWD预共享秘钥
WAPI PSK	口令即可	
WAPI CERT	AS证书/用户证书	

其中WiFi Protected Access (WPA) 和 WiFi Protected Access II(WPA) 是主流协议，均支持PSK预共享密钥模式，也成为Personal个人模式。
PSK模式缺点：需要预共享口令，用户增加无法扩展，需要撤销某设备的访问，必须要修改共享的口令凭据，还要同步修改其他设备的配置，不能区分用户和设备，难以计费和进行灵活的访问控制。

WPA和WPA2进行扩展EAP(Extensible Authentication Protocol)，如图其中一种需要证书认证的方式：
 
Supplicant:　无线设备手机等
Authenticator: 网络的网关，AP无线接入点。负责验证客户端身份
Authentication：Radius服务器，验证客户端凭据并根据预先预置的访问策略决定是否授予访问权限 


EAP是一种支持不同认证类型的认证框架。再认证开始之前，客户端和认证服务器需要协商一种同时支持的认证协议 TLS/TTLS/PWD等



### WIFI框架
内核层：WLAN适配内核驱动模块 ko
原声守护进程：wpa_supplicant,1010 与WPA认证器进行密钥协商，并且控制与WIFI驱动关联
硬件抽象层HAL：libhardware_legacy原声库，通过套接字发送命令给wpa_supplicant
系统服务：WifiService和公开接口WifiManager (WifiStateMachine)
系统UI:系统设置进行控制、SystemUI连接状态栏
WiFi上层接口配置：http://www.cnblogs.com/wotakuc/archive/2013/03/22/2975886.html

### WiFI配置文件
/data/misc/wifi/wpa_supplicant.conf
0660 文件所有者system 文件组wifi
只有系统应用和wifi守护进程可以对文件进行读取和修改，第三方不可以读取和修改。

包含
SSID标识
Key_mgmt网络认证协议
Psk 预共享密钥
 
另外有
Identity身份
Passwork密码
ca_cert证书，守护进程调用IKeyStoreService原生服务远程接口，根据URI获取证书
Phase2阶段二认证协议


另外还有
Client_cert 用户证书
Engine_id 引擎ID
Keyid 用户密钥ID Alias别名
 
配置文件中所有需要预共享口令密码的都是以明文进行存储，但用户私钥用keystore进行存储于硬件或软实现加密存储。

其中证书文件从Settings里边导入，如果选择Wifi形式导入，那么密钥所有者就是wifi用户1010，而且只有以wifi用户运行的系统组件，才可以访问到该密钥。 

参考资料：
https://zh.wikipedia.org/wiki/IEEE_802.11 
http://www.howtogeek.com/167783/htg-explains-the-difference-between-wep-wpa-and-wpa2-wireless-encryption-and-why-it-matters/ 
http://m.8p8p.net/ztnews/03xrzw212ooxtwpzuz1x2z1x.html 
http://bbs.ruijie.com.cn/thread-2559-1-1.html 



## 设备管理DPM (Android for work)
设备管理跟证书相关的接口
Public interface for managing policies enforced on a device. Most clients of this class must be registered with the system as a device administrator. Additionally, a device administrator may be registered as either a profile or device owner. A given method is accessible to all device administrators unless the documentation for that method specifies that it is restricted to either device or profile owners. Any application calling an api may only pass as an argument a device administrator component it owns. Otherwise, a SecurityException will be thrown.

**installCaCert**
boolean installCaCert (ComponentName admin, 
                byte[] certBuffer)
 Installs the given certificate as a user CA. 可以使用system|signature签名级别的android.permission.MANGE_CA_CETIFICATES权限来安装证书，安装后位于/data/misc/user/0/cacerts-added
SecurityException
if admin is not null and not a device or profile owner.

**installKeyPair**
boolean installKeyPair (ComponentName admin, 
                PrivateKey privKey, 
                Certificate[] certs, 
                String alias, 
                boolean requestAccess) 

Called by a device or profile owner, or delegated certificate installer, to install a certificate chain and corresponding private key for the leaf certificate. All apps within the profile will be able to access the certificate chain and use the private key, given direct user approval.
The caller of this API may grant itself access to the certificate and private key immediately, without user approval. It is a best practice not to request this unless strictly necessary since it opens up additional security vulnerabilities.
SecurityException
if admin is not null and not a device or profile owner.


参考资料：
https://developer.android.com/reference/android/app/admin/DevicePolicyManager.html 
https://developer.android.com/guide/topics/admin/device-admin.html 
https://developer.android.com/reference/android/app/admin/DeviceAdminReceiver.html 
https://www.ncsc.gov.uk/guidance/android-end-user-device-security 

## 证书存储 KeyStore/Keymaster
