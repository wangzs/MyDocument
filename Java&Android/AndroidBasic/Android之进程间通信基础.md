# Android进程间通信
## 1. [AIDL][1]
#### 1.1 基本通信
提供功能的app（或进程）属于服务端，调用另一个应用的功能的app（或进程）属于客户端。
建立客户端和服务端通信的步骤如下：
* **服务端**先创建aidl（android studio中新建有aidl文件创建选项），android studio创建完后，会在main目录下创建aidl的目录，并包含形如com/xxx/yyy/aidl的子目录，目录中包含了刚刚创建了的test.aidl文件。Build之后就会生产相应的java文件
	```aidl
	interface IAidlInterface {
		/**
		 * Demonstrates some basic types that you can use as parameters
		 * and return values in AIDL.
		 */
		void basicTypes(int anInt, long aLong, boolean aBoolean, float aFloat,
				double aDouble, String aString);

		int getPid();
	}

	```
* **服务端**创建供其他进程调用的相关功能的service子类
	```java
	// 会在build/generated/source/aidl目录下生产aidl的java接口
	import com.xxx.yyy.aidl.IAidlInterface;

	public class AidlService extends Service {
		private final IAidlInterface.Stub mBinder = new IAidlInterface.Stub() {
			@Override
			public void basicTypes(int anInt, long aLong,
				boolean aBoolean, float aFloat, double aDouble,
				String aString) throws RemoteException { }

			// aidl文件中自定义的一个接口
			@Override
			public int getPid() throws RemoteException {
				return android.os.Process.myPid();
			}
		};
		@Override
		public IBinder onBind(Intent intent) {
			return mBinder;
		}
	}
	```
* **服务端**的AndroidManifest.xml文件中配置该service，其中action字段名称用于**客户端**使用该服务时用
	```xml
	<service android:name=".service.AidlService">
		<intent-filter >
			<action android:name="com.test.aidl" />
			<category android:name="android.intent.category.DEFAULT" />
		</intent-filter>
	</service>
	```
* **客户端**需要将**服务端**创建的aidl文件（包含main下aidl/的所有子目录）拷贝到**客户端**；使用**服务端**提供的服务前，需要进行相应的binding，示例代码如下：
	```java
	private IAidlInterface mAidl;
    private ServiceConnection connection = new ServiceConnection() {
        @Override
        public void onServiceConnected(ComponentName name, IBinder service) {
            LogUtils.d("onServiceConnected");
			// 连接上service后，就可以通过aidl对象来调用服务器app提供的相关功能了
            mAidl = IAidlInterface.Stub.asInterface(service);
        }

        @Override
        public void onServiceDisconnected(ComponentName name) {
            LogUtils.d("onServiceDisconnected");
        }
    };

	// 像使用普通service一样，需要绑定服务端app的service
	Intent intent = new Intent("com.test.aidl");
	bindService(intent, connection, Context.BIND_AUTO_CREATE);
	// 解绑
	unbindService(connection);
	```

#### 1.2 添加回调函数
client向server注册回调函数，供client的非主动调用的情况下，server通知client。
* server所在进程需要添加回调的接口
	```java
	// aidl 接口
	interface IAidlListener {
		void handleX();
		void handleY();
	}

	// 同时需要在上面描述的基本aidl的接口中添加设置client需要的监听
	interface IAidlInterface {
		// client端调用server端的功能
		void normalFunc();

		// client端设置回调，server端因某事件触发回调
		void setAidlListener(IAidlListener listener);
	}

	// JAVA代码部分
	public class AidlService extends Service {
		private IAidlListener mListener;
		private final IAidlInterface.Stub mBinder = new IAidlInterface.Stub() {
			@Override
			public void normalFunc() throws RemoteException {
				Logger.d("====> [server] normalFunc");	// Logger的使用需要init
			}
			@Override
			public void setAidlListener(IAidlListener listener) throws RemoteException {
				mListener = listener;
				// 设置listener之后2秒，server端触发监听(根据实际逻辑触发)
				Observable.timer(2, TimeUnit.SECONDS)
						.subscribe(new Subscriber<Long>() {
							@Override
							public void onCompleted() {
								Logger.d("====> [server] onCompleted");
							}
							@Override
							public void onError(Throwable e) {
								Logger.d("====> [server] onError");
							}
							@Override
							public void onNext(Long aLong) {
								Logger.d("====> [server] onNext");
								try {
									mListener.handleX();
								} catch (RemoteException e) {
									e.printStackTrace();
								}
								// mListener.handleY();
							}
						});
			}
		};

		@Nullable
		@Override
		public IBinder onBind(Intent intent) {
			Logger.d("====> [server] onBind server service");
			return mBinder;
		}
	}

	// AndroidManifest.xml中添加
	 <service android:name=".AidlService">
		<intent-filter >
			<action android:name="com.wangzs.android.server.aidl" />
			<category android:name="android.intent.category.DEFAULT" />
		</intent-filter>
	</service>
	```
* client端调用
	```java
	// 首先需要将server端创建的两个aidl文件（包含所在目录/main下的aidl目录的所有内容）
	// java代码调用server端的service
	Intent intent = new Intent("com.wangzs.android.server.aidl");
	bindService(intent, mConnection, Context.BIND_AUTO_CREATE);
	// 解绑定
	unbindService(mConnection);

	// 调用server的功能
	try {
		mAidl.normalFunc();
	} catch (RemoteException e) {
		e.printStackTrace();
	}
	// 设置回调
	try {
		mAidl.setAidlListener(new IAidlListener.Stub() {
			@Override
			public void handleX() throws RemoteException {
				Logger.d("====> [client] server trigger client handleX");
			}

			@Override
			public void handleY() throws RemoteException {
				Logger.d("====> [client] server trigger client handleY");
			}
		});
	} catch (RemoteException e) {
		e.printStackTrace();
	}
	```
[AIDL一个简单例子][2]



## 2. IPC的机制
* GNU/Linux:
	> * 信号Signals: Oldest IPC method. A process can send signals to processes with the
same uid and gid or in the same process group
 * 管道Pipe: Pipes are unidirectional bytestreams that connect the standard output
from one process with the standard input of another process.
 * Socket: A socket is an endpoint of bidirectional communication. Two processes
can communicate with bytestreams by opening the same socket.
 * 信号量Semaphore: A semaphore is a shared variable that can be read and written by
many processes
 * 消息队列Message queue: Processes can write a message to a message queue that is readable for other Processes
 * 共享内存: A location in system memory mapped into virtual address
spaces of two processes, that each process can fully access

* Android：
	> * Binder： 轻量级RPC（Remote Procedure Communication）机制

## 3. Binder理解












[1]:http://developer.android.com/guide/components/aidl.html
[2]:https://github.com/wangzs/android-example/tree/master/aidl
