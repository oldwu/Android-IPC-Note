# Android中的IPC方式

## Bundle
* 传输的数据必须要能够序列化

## 文件
* A进程把数据写入文件， B进程通过读取这个文件来获得数据
* SharedPerferences是特例，SharePreferences在底层上采用xml文件来存储键值对， 系统对它的读写有一定的缓存策略，在内存中会有一份SharedPreferences文件的缓存，高并发读写访问有很大几率丢失数据。

## Messenager

Messenger是以串行方式处理客户端发来的消息

### 服务器进程
1. 在服务端创建一个Service来处理客户端的连接请求
2. 创建一个handler并通过他来创建一个Messenger对象
3. 在service的onBind中返回Messenger的Binder


###客户端进程
1. 使用ServiceConnection绑定Service
2. 绑定成功之后使用服务端返回IBInder对象创建一个Messenger
3. 通过这个Messenger向服务端发送消息，消息类型为Messenger对象
4. 如果需要服务端能够回复，则需要在客户端创建一个handler，并创建一个Messenger，并把这个Messenger对象通过Message的replyTo参数传递给服务端

## AIDL

### 服务端
1. 首先创建一个Service用来监听客户端的连接请求
2. 然后创建一个AIDL文件，暴露给客户端的接口在这个AIDL文件中的声明
3. 在Service中实现这个AIDL接口

### 客户端
1. 绑定服务端的Service
2. 绑定成功之后将服务端返回的Binder对象转成AIDL接口所属的类型
3. 调用AIDL中的方法

### AIDL支持的数据类型
* 基本数据类型
* String和CharSequence
* ArrayList，且当中每个元素都要被AIDL支持
* HashMap，且当中每个元素都要被AIDL支持
* Paracelabel
* AIDL

除此之外的，其他类型的参数必须加上方向:
* in 表示输入型参数
* out 表示输出型参数
* inout 表示输入输出型参数

AIDL接口中只支持方法，不支持声明静态常量
*需要引入的外部类需要明确import包名，哪怕AIDL与该类处于同一个包下*

AIDL所生成Java文件：
* 继承自IInterface接口
* 当中声明了所包含的方法的id
* 声明了一个内部类的Stub，该类相当于一个Binder，new Stub()可以创建一个Binder
* Stub内部有一个代理类Proxy，提供了一个逻辑，当客户端和服务端在同一个进程时，不走Stub中的transact，否则则走transact  

详细的方法含义：
* asInterface   用于将服务端的Binder对象转换成客户端所需的AIDL接口类型的对象。 若客户端和服务端位于同一个进程，则方法返回的是服务端的Stub对象本身，否则返回的是系统封装后的Stub.proxy对象
* asBinder      用于返回当前Binder对象
* onTransact    此方法运行于Binder线程池中，客户端发起跨进程请求时，远程请求会通过系统底层封装之后交由此方法来处理  


RemoteCallBackList是系统专门用于删除挂进程listern的接口，是一个泛型，但无法像操作List一样去操作它，必须要使用beginBroadcast和finishBroadcast的配对方式，样例代码如下：

```java

private RemoteCallbackList<IOnNewBookArrived> mListenerList = new RemoteCallbackList<IOnNewBookArrivedListener>();
//注册listener
mListenerList.register(listener);

//注销listener
mListenerList.unregister(listener);

//获取RemoteCallbackList中的元素个数
final int N = mListenerList.beginBroadcast();
for (int i = 0; i < N; i++){
    IOnNewBookArrivedListener l = mListenerList.getBroadcastItem(i);
}
mListenerList.finishBroadcast();
```
