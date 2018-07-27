---
title: Android binder原理
urlname: android_app_binder_sumup
#date: 2017-04-10
comments: true
#description: 
#tags: [git,hah,hha]
#keywords: git
categories: 10Android_App
tags:
    - Android
    - binder
---

## Binder实现原理
**用法总结：** 
http://gityuan.com/2015/11/22/binder-use/
http://gityuan.com/2015/11/23/binder-aidl/
http://qiangbo.space/2017-01-15/AndroidAnatomy_Binder_Driver/
主要思路是:
- aidl两端：
服务端stub端，实现aidl接口，通过onTransact接收数据。继承binder，实现aidl接口，是一个binder对象。注册到ServiceManager中，或者通过serviceConnected获取。

客户端proxy端，实现aidl接口，通过mRemote.transact传送数据。通过ServiceManager获取binder对象，然后asInterface活的aidl接口，即可通过Proxy transact调用相关方法。
- asInterface:
new com.yuanhh.appbinderdemo.IRemoteService.Stub.Proxy(obj);

![image.png](https://upload-images.jianshu.io/upload_images/11010834-2d27b1713cde4eee.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![image.png](https://upload-images.jianshu.io/upload_images/11010834-af22fcd4e3bf86b4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
```
package com.yuanhh.appbinderdemo;
public interface IRemoteService extends android.os.IInterface {
    /** * Local-side IPC implementation stub class. */
    public static abstract class Stub extends android.os.Binder implements com.yuanhh.appbinderdemo.IRemoteService {
        private static final java.lang.String DESCRIPTOR = "com.yuanhh.appbinderdemo.IRemoteService";

        /** * Stub构造函数 */
        public Stub() {
            this.attachInterface(this, DESCRIPTOR);
        }

        /** * 将IBinder 转换为IRemoteService interface */
        public static com.yuanhh.appbinderdemo.IRemoteService asInterface(android.os.IBinder obj) {
            if ((obj == null)) {
                return null;
            }
            android.os.IInterface iin = obj.queryLocalInterface(DESCRIPTOR);
            if (((iin != null) && (iin instanceof com.yuanhh.appbinderdemo.IRemoteService))) {                 return ((com.yuanhh.appbinderdemo.IRemoteService) iin);
            }
            return new com.yuanhh.appbinderdemo.IRemoteService.Stub.Proxy(obj);
        }

        @Override
        public android.os.IBinder asBinder() {
            return this;
        }

        @Override
        public boolean onTransact(int code, android.os.Parcel data, android.os.Parcel reply, int flags) throws android.os.RemoteException {
            switch (code) {
                case INTERFACE_TRANSACTION: {
                    reply.writeString(DESCRIPTOR);
                    return true;
                }
                case TRANSACTION_getPid: {
                    data.enforceInterface(DESCRIPTOR);
                    int _result = this.getPid();
                    reply.writeNoException();
                    reply.writeInt(_result);
                    return true;
                }
                case TRANSACTION_getMyData: {
                    data.enforceInterface(DESCRIPTOR);
                    com.yuanhh.appbinderdemo.MyData _result = this.getMyData();
                    reply.writeNoException();
                    if ((_result != null)) {                         reply.writeInt(1);
                        _result.writeToParcel(reply, android.os.Parcelable.PARCELABLE_WRITE_RETURN_VALUE);
                    } else {
                        reply.writeInt(0);
                    }
                    return true;
                }
            }
            return super.onTransact(code, data, reply, flags);
        }

        private static class Proxy implements com.yuanhh.appbinderdemo.IRemoteService {
            private android.os.IBinder mRemote;

            /** * Proxy构造函数 */
            Proxy(android.os.IBinder remote) {
                mRemote = remote;
            }

            @Override
            public android.os.IBinder asBinder() {
                return mRemote;
            }

            public java.lang.String getInterfaceDescriptor() {
                return DESCRIPTOR;
            }

            @Override
            public int getPid() throws android.os.RemoteException {
                android.os.Parcel _data = android.os.Parcel.obtain();
                android.os.Parcel _reply = android.os.Parcel.obtain();
                int _result;
                try {
                    _data.writeInterfaceToken(DESCRIPTOR);
                    mRemote.transact(Stub.TRANSACTION_getPid, _data, _reply, 0);
                    _reply.readException();
                    _result = _reply.readInt();
                } finally {
                    _reply.recycle();
                    _data.recycle();
                }
                return _result;
            }

            @Override
            public com.yuanhh.appbinderdemo.MyData getMyData() throws android.os.RemoteException {
                android.os.Parcel _data = android.os.Parcel.obtain();
                android.os.Parcel _reply = android.os.Parcel.obtain();
                com.yuanhh.appbinderdemo.MyData _result;
                try {
                    _data.writeInterfaceToken(DESCRIPTOR);
                    mRemote.transact(Stub.TRANSACTION_getMyData, _data, _reply, 0);
                    _reply.readException();
                    if ((0 != _reply.readInt())) {                         _result = com.yuanhh.appbinderdemo.MyData.CREATOR.createFromParcel(_reply);
                    } else {
                        _result = null;
                    }
                } finally {
                    _reply.recycle();
                    _data.recycle();
                }
                return _result;
            }
        }

        static final int TRANSACTION_getPid = (android.os.IBinder.FIRST_CALL_TRANSACTION + 0);
        static final int TRANSACTION_getMyData = (android.os.IBinder.FIRST_CALL_TRANSACTION + 1);
    }

    public int getPid() throws android.os.RemoteException;

    public com.yuanhh.appbinderdemo.MyData getMyData() throws android.os.RemoteException;
}
```

**binder die处理思路**
比较简单之间直接在客户端注册相应callback即可。
IBinder.DeathRecipient mDeathRecipient = new IBinder.DeathRecipient()
https://blog.csdn.net/lea_fy/article/details/52987004
具体机制可以参考：
http://gityuan.com/2016/10/03/binder_linktodeath/