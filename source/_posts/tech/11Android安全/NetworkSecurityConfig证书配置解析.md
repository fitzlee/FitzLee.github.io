---
title: NetworkSecurityConfig证书配置
urlname: android_security_NetworkSecurityConfig_settings
date: 2017-03-11
comments: true
#description: 
#tags: [git,hah,hha]
#keywords: git
categories: 11Android_Security
tags:
    - NetworkSecurityConfig
    - 证书配置
    - Android
    - 安全
---

SecurityConfig证书配置 NetworkSecurityConfig
##	SecurityConfig
此API是24版本的新特性，面向之前的版本并不适用，请适用自验证方式进行自定义验证
## 添加application安全配置文件
    <application android:networkSecurityConfig="@xml/network_security_config" />

## 配置network_security_config.xml
                 标签和继承关系
```
<network-security-config>
  <base-config   />      <!--0 或 1 个-->
  <domain-config>     <!--任意数量-->
		<domain />      <!--1 个或多个 -->
  <trust-anchors  />    <!--0 或 1 个-->
  <pin-set />   <!--0 或 1 个-->
  <domain-config />   <!--任意数量的已嵌套-->
  </domain-config>    
  <debug-overrides />    <!--0 或 1 个 -->
</network-security-config>
```
## 匹配策略及标签解析
先进行域名匹配，匹配到domain-config(domain-config必须包含至少一个domain标签)，则执行相关的trust-anchors和pin-set策略(覆盖上层策略)，如果没有找到trust-anchors等策略就向上一级找，找到终止，使用找到的trust-anchors策略。 查找策略，内部的domain-config   =>  domain-config => base-config => platform默认方式   (target sdk > 23 system;  23 or lower: system or user).
如果域名没有匹配到domain-config那么使用上一级策略。

<domain-config />
是按照 domain 元素的定义连接到特定目的地的配置。 仅对domain元素匹配到的域名生效。如果目的地不在 domain-config 涵盖范围内的所有连接所使用的默认配置。
如果未在特定条目中设置值，将使用通用条目中的值。例如，未在 domain-config 中设置的值将从父级 domain-config（如果已嵌套）或者 base-config（如果未嵌套）中获取。未在 base-config 中设置的值将使用平台默认值。如果找到执行相关trust-anchors和pin-set策略。

includeSubdomains
```<domain includeSubdomains=["true" | "false"]>example.com</domain>```
如果为 "true"，此域规则将与域及所有子域（包括子域的子域）匹配。否则，该规则仅适用于精确匹配项。

expiration="2018-01-01"
```<pin-set expiration="2018-01-01">```
设置pin策略失效时间，失效后将不再进行该pin策略的检查。
这有助于防止尚未更新的应用出现连接性问题。不过，设置固定的到期时间可能会绕过证书固定。
设置的日期一定要保证这个时间之前证书CA和公钥不会发生变化，如果发生变化那么将认证失败。
假如设置的日期早于CA有效期，那么这个日期之后pin策略将失效。

digest="SHA-256"
```
<pin digest="SHA-256">7HIpactkIAq2Y49orFOOQKurWxmmSFZhBCoQYcRhJ3Y=</pin>
```
PEM Base64编码，使用如下命令获取Base64加密的Sha256的公钥，可以用server本身证书或中间CA或根CA，一个验证通过即可
openssl x509 -in GeoTrust_G3.cer -pubkey -noout | openssl pkey -pubin -outform der | openssl dgst -sha256 -binary | openssl enc -base64

overridePins=["true" | "false"]
```
<certificates src=["system" | "user" | "raw resource"]    overridePins=["true" | "false"] />
```
针对certificates标签生效，指定此来源的 CA 是否绕过证书固定。如果为 "true"，则不对此来源的 CA 签署的证书链执行证书固定。这对于调试 CA 或测试对应用的安全流量进行中间人攻击 (MiTM) 非常有用。
默认值为 "false"，除非在 debug-overrides 元素中另外指定（在这种情况下，默认值为 "true"）

cleartextTrafficPermitted="false"
```<domain-config cleartextTrafficPermitted="false">```
例如，应用可能需要确保与 secure.example.com 的所有连接始终通过 HTTPS 完成，以防止来自恶意网络的敏感流量。
res/xml/network_security_config.xml：
```
<?xml version="1.0" encoding="utf-8"?>
<network-security-config>
    <domain-config cleartextTrafficPermitted="false">
        <domain includeSubdomains="true">secure.example.com</domain>
    </domain-config>
</network-security-config>
```
更多请参考Google专业测试用例，详细可参考如下APK： 
frameworks/base/tests/ NetworkSecurityConfigTest测试APK
