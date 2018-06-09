---
title: Android查缺补漏-流行框架
urlname: android_review_frame_library
date: 2017-10-16
comments: true
#description: 
#tags: [git,hah,hha]
#keywords: git
categories: 21Android_review
tags:
    - Android
    - 查缺补漏
    - 流程框架
---



## Andriod流行框架速查
https://www.ctolib.com/cheatsheets-Android-ch.html

## 推荐 Team 可使用如下优秀轮子：
	• Retrofit
	• RxAndroid
	• OkHttp
	• Glide/Fresco
	• Gson/Fastjson
	• EventBus/AndroidEventBus
	• GreenDao
	• Dagger2（选用）
	• Tinker（选用）
来自 <https://github.com/Blankj/AndroidStandardDevelop> 

## 缓存方案


## 服务重试机制
超时,重试,熔断,限流 n^2重试 1,2,4,6,8… 不断超时重试。
http://wtoutiao.com/p/534r4HP.html
http://www.sczyh30.com/posts/Microservice/circuit-breaker-pattern/
http://www.cnblogs.com/yangecnu/p/Introduce-Circuit-Breaker-Pattern.html

## 图解Android应用的后台任务和提醒
https://blog.csdn.net/qq284565035/article/details/51705341
简单来说就是：
- 小于60s直接用handler
- AlarmManager 休眠唤醒 精准唤醒 重复唤醒
- JobScheduler

## 网络请求处理心路历程
http://www.jianshu.com/p/3141d4e46240

## StateMachine机制
http://blog.csdn.net/maybe_windleave/article/details/9881991
http://blog.csdn.net/lilian0118/article/details/21974229

## 上传100M大文件
如果有个100M大的文件，需要上传至服务器中，而服务器form表单最大只能上传2M，可以用什么方法。
上传文件用的都是POST方式，POST方式对大小没什么限制。
回到题目，可以说假设每次真的只能上传2M，那么可能我们只能把文件截断，然后分别上传了。
* 上传大文件，可以分块上传，与文件分块下载方法基本类似，使用RandomAcessFile来进行划分文件组装文件即可。使用Post方法上传文件。基本过程如下：
https://blog.csdn.net/leiyaqiang/article/details/68491521

```
    //普通字符串数据  
    private void writeStringParams() throws Exception {  
        Set<String> keySet = textParams.keySet();  
        for (Iterator<String> it = keySet.iterator(); it.hasNext();) {  
            String name = it.next();  
            String value = textParams.get(name);  
            ds.writeBytes("--" + boundary + "\r\n");  
            ds.writeBytes("Content-Disposition: form-data; name=\"" + name  
                    + "\"\r\n");  
            ds.writeBytes("\r\n");  
            ds.writeBytes(encode(value) + "\r\n");  
        }  
    }  
    //文件数据  
    private void writeFileParams() throws Exception {  
        Set<String> keySet = fileparams.keySet();  
        for (Iterator<String> it = keySet.iterator(); it.hasNext();) {  
            String name = it.next();  
            File value = fileparams.get(name);  
            ds.writeBytes("--" + boundary + "\r\n");  
            ds.writeBytes("Content-Disposition: form-data; name=\"" + name  
                    + "\"; filename=\"" + encode(value.getName()) + "\"\r\n");  
            ds.writeBytes("Content-Type: " + getContentType(value) + "\r\n");  
            ds.writeBytes("\r\n");  
            ds.write(getBytes(value));  
            ds.writeBytes("\r\n");  
        }  
    }  
```


