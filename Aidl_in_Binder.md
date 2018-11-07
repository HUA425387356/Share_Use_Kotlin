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
   

