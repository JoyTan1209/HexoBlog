---
title: "Android IPC机制"
date: 2016-04-07 10:38:40
tags: 
	- Android
	- 移动开发
---
**Android IPC机制**

在任何一个操作系统中都需要IPC机制的存在，例如：Linux中通过共享内存、命名通道等来实现进程间的通信。然而在Android系统中是什么来实现呢？答案是Binder。当然，不仅使用了Binder机制来实现了IPC,还使用了Socket实现不同终端之间的通信。

### IPC介绍

1、Serializable接口是Java中为对象提供标准的序列化和反序列化操作的接口，而Parcelable接口是Android提供的序列化方式的接口。

2、serialVersionUId是一串long型数字，主要是用来辅助序列化和反序列化的，原则上序列化后的数据中的serialVersionUId只有和当前类的serialVersionUId相同才能够正常地被反序列化。serialVersionUId的详细工作机制：序列化的时候系统会把当前类的serialVersionUId写入序列化的文件中，当反序列化的时候系统会去检测文件中的serialVersionUId，看它是否和当前类的serialVersionUId一致，如果一致就说明序列化的类的版本和当前类的版本是相同的，这个时候可以成功反序列化；否则说明版本不一致无法正常反序列化。一般来说，我们应该手动指定serialVersionUId的值：
	1.静态成员变量属于类不属于对象，所以不参与序列化过程；
	2.声明为transient的成员变量不参与序列化过程。
	3.Parcelable接口内部包装了可序列化的数据，可以在Binder中自由传输，Parcelable主要用在内存序列化上，可以直接序列化的有Intent、Bundle、Bitmap以及List和Map等等

### 实现IPC的方式

1、 Bundle：Bundle实现了Parcelable接口，Activity、Service和Receiver都支持在Intent中传递Bundle数据。

2、 Messenger：Messenger是一种轻量级的IPC方案，其底层是用AIDL实现的。Messenger是以串行的方式处理请求的，服务端只能一个个处理，不能并发执行。

3、 AIDL：第一步创建一个Service和一个AIDL接口，第二步创建一个类继承自AIDL接口中的Stub类并实现Stub类中的抽象方法，然后在Service的onBind方法中返回这个类的对象，再在客户端绑定服务端Service，建立连接后就可以访问远程服务端的方法。

	1.AIDL支持的数据类型：基本数据类型、String和CharSequence、ArrayList、HashMap、Parcelable以及AIDL；
	2.某些类即使和AIDL文件在同一个包中也要显式import进来；
	3.AIDL中除了基本数据类，其他类型的参数都要标上方向：in、out或者inout；
	4.AIDL接口中支持方法，不支持声明静态变量；
	5.为了方便AIDL的开发，建议把所有和AIDL相关的类和文件全部放入同一个包中，这样做的好处是，当客户端是另一个应用的时候，可以直接把整个包复制到客户端工程中。
	6.RemoteCallbackList是系统提供来删除跨进程Listener的接口。是一个泛型，可以管理任何的ALDL接口。

 4、ContentProvider：

	1.ContentProvider主要以表格的形式来组织数据，并且可以包含多个表；
	2.ContentProvider还支持文件数据，比如图片、视频等，系统提供的MediaStore就是文件类型的ContentProvider；
	3.ContentProvider对底层的数据存储方式没有任何要求，可以是SQLite、文件，甚至是内存中的一个对象都行；
	4.要观察ContentProvider中的数据变化情况，可以通过ContentResolver的registerContentObserver方法来注册观察者；

 5、Socket：Socket是网络通信中“套接字”的概念，分为流式套接字和用户数据包套接字两种，分别对应网络的传输控制层的TCP和UDP协议。 

 6、文件共享：这种方式简单，适合在对数据同步要求不高的进程之间进行通信，并且要妥善处理并发读写的问题。

 ### Binder连接池
	
如果项目规模较大，创建过多的Service是不合理的，因为service是系统资源，过多的service会使得应用看起来很重，所以最好的做法是将所有的AIDL放在同一个Service中去管理。其工作机制是：每一个业务模块创建自己的AIDL接口并实现此接口，此时不同业务模块之间不能有耦合，所有实现细节我们要单独开来，然后向服务端提供自己的唯一标识和其对应的Binder对象；对于服务端来说，只需要一个Service，服务端提供一个queryBinder接口，这个接口能够根据业务模块的特征来返回相应的Binder对象给它们，不同的业务模块拿到所需的Binder对象后就可以进行远程方法调用了。
Binder连接池的主要作用就是将每个业务模块的Binder请求统一转发到远程Service去执行，从而避免了重复创建Service的过程。