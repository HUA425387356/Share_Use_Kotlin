# Binder机制之AIDL
### 基本概念

- IPC ：`Inter-Process Communication`
进程间的通信或跨进程通信。简单点理解，一个应用可以存在多个进程，但需要数据交换就必须用IPC；或者是二个应用之间的数据交换。
- Binder：Binder是Android的一个类，它实现了IBinder接口。从IPC角度来说，Binder是Android中的一种跨进程通信方式。通过这个Binder对象，客户端就可以获取服务端提供的服务或数据，这里的服务包括普通服务和基于AIDL的服务。
- AIDL `Android Interface Definition language`
它是一种Android内部进程通信接口的描述语言，是Binder机制的一种实现。

### AIDL的使用
![](https://img-blog.csdn.net/20160819221518358?)

#### 服务端
创建一个服务端工程，在工程中点击右键`New->AIDL->AIDL File`，默认直接点确定，这时会在工程中出现一个aidl文件：
![](https://img-blog.csdn.net/20160819221944798?)     
   
我们打开这个aidl文件，我们创建一个我们需要测试的方法：
![](https://img-blog.csdn.net/20160819222358019?)    

由于Android Studio是要`手动编译`才能生成对应AIDL的java文件，既然`aidl文件是个接口`，那就`必须`存在着实现这个接口的类，点击编译，系统自动生成一个java类，该java类的代码就是整个Binder机制的原理所在（会在下面第二步骤介绍原理）：
![](https://img-blog.csdn.net/20160819223250927?) 
![](https://img-blog.csdn.net/20160819223516180?)    

既然是个`服务端`，那么我们就要开始`写服务`了，创建一个类，继承Service：
```Java
import android.app.Service;
import android.content.Intent;
import android.os.IBinder;
import android.os.RemoteException;
import android.support.annotation.Nullable;
import android.util.Log;
 
import com.handsome.boke.IMyAidlInterface;
 
public class MyService extends Service {
    @Nullable
    @Override
    public IBinder onBind(Intent intent) {
        return myS;
    }
 
    private IBinder myS = new IMyAidlInterface.Stub() {
 
        @Override
        public void basicTypes(int anInt, long aLong, boolean aBoolean, float aFloat, double aDouble, String aString) throws RemoteException {
 
        }
 
        @Override
        public int add(int num1, int num2) throws RemoteException {
            Log.i("Hensen", "从客户端发来的AIDL请求:num1->" + num1 + "::num2->" + num2);
            return num1 + num2;
        }
    };
}
```  

既然是个服务，就必须在manifests文件中配置：
```Java
<!--exported:允许外界访问该服务，AIDL必备条件-->
        <service
            android:name=".Aidl.MyService"
            android:exported="true"/>
```   

到现在服务端写好了，开启模拟器`启动这个程序`，记得在代码中开启服务：
```Java
public class LoginActivity extends AppCompatActivity {
 
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_login);
 
        startService(new Intent(this, MyService.class));
    }
}
```    

#### 客户端   
在工程中点击右键`New->Module`，按默认确定，finish：
![](https://img-blog.csdn.net/20160819224539622?)    

关键的一步来了，`复制服务端的aidl整个文件夹`（包括里面的包、aidl文件、完整无缺）`粘贴到客户端对应放aidl的地方`：
![](https://img-blog.csdn.net/20160819224915562?)    

不要忘了，客户端还要 `手动编译`：
![](https://img-blog.csdn.net/20160819223250927?)    

好了我们来写客户端的代码（我们在MainActivity中放一个”AIDL“的按钮，先绑定服务，然后点击按钮调用）：
```Java
public class MainActivity extends AppCompatActivity {
 
    IMyAidlInterface iMyAidlInterface;
 
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
 
        //绑定服务
        Intent intent = new Intent();
        intent.setComponent(new ComponentName("com.handsome.boke", "com.handsome.boke.Aidl.MyService"));
        bindService(intent, conn, BIND_AUTO_CREATE);
    }
 
    /**
     * 点击“AIDL”按钮事件
     *
     * @param view
     */
    public void add(View view) {
        try {
            int res = iMyAidlInterface.add(1, 2);
            Log.i("Hensen", "从服务端调用成功的结果：" + res);
        } catch (RemoteException e) {
            e.printStackTrace();
        }
    }
 
    /**
     * 服务回调方法
     */
    private ServiceConnection conn = new ServiceConnection() {
        @Override
        public void onServiceConnected(ComponentName name, IBinder service) {
            iMyAidlInterface = IMyAidlInterface.Stub.asInterface(service);
        }
 
        @Override
        public void onServiceDisconnected(ComponentName name) {
            iMyAidlInterface = null;
        }
    };
 
    @Override
    protected void onDestroy() {
        super.onDestroy();
        //解绑服务，回收资源
        unbindService(conn);
    }
}
```    
测试结果（先开启服务端，开启服务后，接着开启客户端，绑定远程服务）：
```Java
08-19 10:59:34.548 6311-6328/com.handsome.boke I/Hensen: 从客户端发来的AIDL请求:num1->1::num2->2

08-19 10:59:34.550 7052-7052/com.handsome.app2 I/Hensen: 从服务端调用成功的结果：3
```   

### AIDL的Binder机制原理分析
![](https://img-blog.csdn.net/20160819231408390?)    
分析原理：
```Java
/*
 * This file is auto-generated.  DO NOT MODIFY.
 * Original file: D:\\workspace5\\Boke\\app\\src\\main\\aidl\\com\\handsome\\boke\\IMyAidlInterface.aidl
 */
package com.handsome.boke;
// Declare any non-default types here with import statements
 
public interface IMyAidlInterface extends android.os.IInterface {
    /**
     * Local-side IPC implementation stub class.
     */
    public static abstract class Stub extends android.os.Binder implements com.handsome.boke.IMyAidlInterface {
        private static final java.lang.String DESCRIPTOR = "com.handsome.boke.IMyAidlInterface";
 
        /**
         * Construct the stub at attach it to the interface.
         */
        public Stub() {
            this.attachInterface(this, DESCRIPTOR);
        }
 
        /**
         * Cast an IBinder object into an com.handsome.boke.IMyAidlInterface interface,
         * generating a proxy if needed.
         */
        public static com.handsome.boke.IMyAidlInterface asInterface(android.os.IBinder obj) {
            if ((obj == null)) {
                return null;
            }
            android.os.IInterface iin = obj.queryLocalInterface(DESCRIPTOR);
            if (((iin != null) && (iin instanceof com.handsome.boke.IMyAidlInterface))) {
                return ((com.handsome.boke.IMyAidlInterface) iin);
            }
            return new com.handsome.boke.IMyAidlInterface.Stub.Proxy(obj);
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
                case TRANSACTION_basicTypes: {
                    data.enforceInterface(DESCRIPTOR);
                    int _arg0;
                    _arg0 = data.readInt();
                    long _arg1;
                    _arg1 = data.readLong();
                    boolean _arg2;
                    _arg2 = (0 != data.readInt());
                    float _arg3;
                    _arg3 = data.readFloat();
                    double _arg4;
                    _arg4 = data.readDouble();
                    java.lang.String _arg5;
                    _arg5 = data.readString();
                    this.basicTypes(_arg0, _arg1, _arg2, _arg3, _arg4, _arg5);
                    reply.writeNoException();
                    return true;
                }
                case TRANSACTION_add: {
                    data.enforceInterface(DESCRIPTOR);
                    int _arg0;
                    _arg0 = data.readInt();
                    int _arg1;
                    _arg1 = data.readInt();
                    int _result = this.add(_arg0, _arg1);
                    reply.writeNoException();
                    reply.writeInt(_result);
                    return true;
                }
            }
            return super.onTransact(code, data, reply, flags);
        }
 
        private static class Proxy implements com.handsome.boke.IMyAidlInterface {
            private android.os.IBinder mRemote;
 
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
 
            /**
             * Demonstrates some basic types that you can use as parameters
             * and return values in AIDL.
             */
            @Override
            public void basicTypes(int anInt, long aLong, boolean aBoolean, float aFloat, double aDouble, java.lang.String aString) throws android.os.RemoteException {
                android.os.Parcel _data = android.os.Parcel.obtain();
                android.os.Parcel _reply = android.os.Parcel.obtain();
                try {
                    _data.writeInterfaceToken(DESCRIPTOR);
                    _data.writeInt(anInt);
                    _data.writeLong(aLong);
                    _data.writeInt(((aBoolean) ? (1) : (0)));
                    _data.writeFloat(aFloat);
                    _data.writeDouble(aDouble);
                    _data.writeString(aString);
                    mRemote.transact(Stub.TRANSACTION_basicTypes, _data, _reply, 0);
                    _reply.readException();
                } finally {
                    _reply.recycle();
                    _data.recycle();
                }
            }
 
            @Override
            public int add(int num1, int num2) throws android.os.RemoteException {
                android.os.Parcel _data = android.os.Parcel.obtain();
                android.os.Parcel _reply = android.os.Parcel.obtain();
                int _result;
                try {
                    _data.writeInterfaceToken(DESCRIPTOR);
                    _data.writeInt(num1);
                    _data.writeInt(num2);
                    mRemote.transact(Stub.TRANSACTION_add, _data, _reply, 0);
                    _reply.readException();
                    _result = _reply.readInt();
                } finally {
                    _reply.recycle();
                    _data.recycle();
                }
                return _result;
            }
        }
 
        static final int TRANSACTION_basicTypes = (android.os.IBinder.FIRST_CALL_TRANSACTION + 0);
        static final int TRANSACTION_add = (android.os.IBinder.FIRST_CALL_TRANSACTION + 1);
    }
 
    /**
     * Demonstrates some basic types that you can use as parameters
     * and return values in AIDL.
     */
    public void basicTypes(int anInt, long aLong, boolean aBoolean, float aFloat, double aDouble, java.lang.String aString) throws android.os.RemoteException;
 
    public int add(int num1, int num2) throws android.os.RemoteException;
}
```    
我们来分析一下这个类

首先本身继承Iinterface，所以他也是个接口，接口中必须有方法，`代码定位到结尾`有2个方法。
```Java
public void basicTypes(int anInt, long aLong, boolean aBoolean, float aFloat, double aDouble, java.lang.String aString) throws android.os.RemoteException;
    public int add(int num1, int num2) throws android.os.RemoteException;
```   
这两个方法就是basicTypes和add，就是我们服务端的2个方法。
接着发现该接口中有1个内部类Stub，继承自本身（IMyAidlInterface）接口，`代码定位到Stub`类。
这个Stub有个构造方法、asInterface、asBinder、onTransact（先不介绍）。
接着发现该内部类Stub还有一个内部类，`代码定位到Proxy（我们把它称为代理）`类，也是继承自本身（IMyAidlInterface）接口，所以`实现`该接口的两个方法。   
```Java
/**
             * Demonstrates some basic types that you can use as parameters
             * and return values in AIDL.
             */
            @Override
            public void basicTypes(int anInt, long aLong, boolean aBoolean, float aFloat, double aDouble, java.lang.String aString) throws android.os.RemoteException {
                android.os.Parcel _data = android.os.Parcel.obtain();
                android.os.Parcel _reply = android.os.Parcel.obtain();
                try {
                    _data.writeInterfaceToken(DESCRIPTOR);
                    _data.writeInt(anInt);
                    _data.writeLong(aLong);
                    _data.writeInt(((aBoolean) ? (1) : (0)));
                    _data.writeFloat(aFloat);
                    _data.writeDouble(aDouble);
                    _data.writeString(aString);
                    mRemote.transact(Stub.TRANSACTION_basicTypes, _data, _reply, 0);
                    _reply.readException();
                } finally {
                    _reply.recycle();
                    _data.recycle();
                }
            }
 
            @Override
            public int add(int num1, int num2) throws android.os.RemoteException {
                android.os.Parcel _data = android.os.Parcel.obtain();
                android.os.Parcel _reply = android.os.Parcel.obtain();
                int _result;
                try {
                    _data.writeInterfaceToken(DESCRIPTOR);
                    _data.writeInt(num1);
                    _data.writeInt(num2);
                    mRemote.transact(Stub.TRANSACTION_add, _data, _reply, 0);
                    _reply.readException();
                    _result = _reply.readInt();
                } finally {
                    _reply.recycle();
                    _data.recycle();
                }
                return _result;
            }
        }
```   
在这个类里面我们会发现有2个标识：用来区分两个方法，到底你远程请求哪个方法的唯一标识，`代码定位到代理类的结尾`。
```Java
 static final int TRANSACTION_basicTypes = (android.os.IBinder.FIRST_CALL_TRANSACTION + 0);
        static final int TRANSACTION_add = (android.os.IBinder.FIRST_CALL_TRANSACTION + 1);
```   
回过头来，还记得我们客户端做了什么吗？   
答案：绑定一个服务，在回调方法获取一个接口（iMyAidlInterface），它是`直接静态使用IMyAidlInterface里面的静态类Stub的asInterface的方法`：（好了我们去跟踪到Stub类asInterface这个方法）
```Java
 /**
     * 服务回调方法
     */
    private ServiceConnection conn = new ServiceConnection() {
        @Override
        public void onServiceConnected(ComponentName name, IBinder service) {
            iMyAidlInterface = IMyAidlInterface.Stub.asInterface(service);
        }
 
        @Override
        public void onServiceDisconnected(ComponentName name) {
            iMyAidlInterface = null;
        }
    };
```   
代码定位到Stub类`asInterface方法`：
```Java
public static com.handsome.boke.IMyAidlInterface asInterface(android.os.IBinder obj) {
            if ((obj == null)) {
                return null;
            }
            android.os.IInterface iin = obj.queryLocalInterface(DESCRIPTOR);
            if (((iin != null) && (iin instanceof com.handsome.boke.IMyAidlInterface))) {
                return ((com.handsome.boke.IMyAidlInterface) iin);
            }
            return new com.handsome.boke.IMyAidlInterface.Stub.Proxy(obj);
        }
```    
前面只是做一些判断、`看一下最后一句话`：我们将传过来的obj还是传给了它的`代理类`来处理，`返回的是代理类的对象`
```Java
return new com.handsome.boke.IMyAidlInterface.Stub.Proxy(obj);
```   
所以在客户端的iMyAidlInterface = ……，则是拿到它的`代理类`，好了，这个时候就看客户端调用代理类干嘛了：
```Java
int res = iMyAidlInterface.add(1, 2);
```  
他调用了代理类的add方法，`代码定位到 代理类的add方法`：
```Java
@Override
            public int add(int num1, int num2) throws android.os.RemoteException {
                android.os.Parcel _data = android.os.Parcel.obtain();
                android.os.Parcel _reply = android.os.Parcel.obtain();
                int _result;
                try {
                    _data.writeInterfaceToken(DESCRIPTOR);
                    _data.writeInt(num1);
                    _data.writeInt(num2);
                    mRemote.transact(Stub.TRANSACTION_add, _data, _reply, 0);
                    _reply.readException();
                    _result = _reply.readInt();
                } finally {
                    _reply.recycle();
                    _data.recycle();
                }
                return _result;
            }
```    
你会发现，它把数据写进了_data里面，最后调用`transact方法`，传入_data数据，唯一标识Stub.TRANSACTION_add。
```Java
mRemote.transact(Stub.TRANSACTION_add, _data, _reply, 0);
```   
然后这个transact方法就是通过底层了，通过底层结束后，这些参数送到哪了？答案：底层会走到stub类中的`onTransact方法，通过判断唯一标识，确定方法`：
```Java
case TRANSACTION_add: {
                    data.enforceInterface(DESCRIPTOR);
                    int _arg0;
                    _arg0 = data.readInt();
                    int _arg1;
                    _arg1 = data.readInt();
                    int _result = this.add(_arg0, _arg1);
                    reply.writeNoException();
                    reply.writeInt(_result);
                    return true;
                }
```   
在这个地方将传过来的参数解包，readInt方法。然后调用`this.add方法`，this指的就是服务端，调用`服务端的add的方法`：
```Java
int _result = this.add(_arg0, _arg1);
```   

将得到的结果，写入`reply`：
```Java
reply.writeNoException();
reply.writeInt(_result);
```   
最后一句话，最后返回系统的ontransact方法，传入结果reply：
```Java
return super.onTransact(code, data, reply, flags);
```    
所以我们在上面获得的结果就是reply（答案：3）：
```Java
int res = iMyAidlInterface.add(1, 2);
```    
最后总结下整个过程:
![](https://img-blog.csdn.net/20160820000454677?) 






