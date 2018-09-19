title: Android项目-访问隐藏方法
date: 2016/3/1 8:42:11   
categories: Android
---

# Android项目-访问隐藏方法 #

## 实现电话的挂断 endCall ##

代码如下：

	/**
	 * 挂断电话
	 */
	public void endCall() {
		try {
			Class clazz = getClassLoader().loadClass("android.os.ServiceManager");
			Method method = clazz.getDeclaredMethod("getService", String.class);
			IBinder iBinder = (IBinder) method.invoke(null, TELEPHONY_SERVICE);
			ITelephony itelephony = ITelephony.Stub.asInterface(iBinder);
			itelephony.endCall();
			//开通呼叫转移 
		} catch (Exception e) {
			e.printStackTrace();
		}
	}


为何会这样写呢？

	首先，我们可以猜出，这个endCall()方法肯定在TelephoneManager中。但是这个对象我们是直接new不出来的
		一般都是这样得到：  tm = (TelephonyManager) getSystemService(TELEPHONY_SERVICE);
	看TelephonyManager的源码可以知道， 这个对象是单例的。
	    private static TelephonyManager sInstance = new TelephonyManager();
	查看TelephonyManager的构造方法有：
	            sRegistry = ITelephonyRegistry.Stub.asInterface(ServiceManager.getService(
	                    "telephony.registry"));
	因此猜测，我们可以利用ServiceManager.getService()来得到我们想到的东西
	2）查看ServiceManager.getService()的源码
	     public static IBinder getService(String name) {
	        try {
	            IBinder service = sCache.get(name);
	            if (service != null) {
	                return service;
	            } else {
	                return getIServiceManager().getService(name);
	            }
	        } catch (RemoteException e) {
	            Log.e(TAG, "error in getService", e);
	        }
	        return null;
	    }
	
	即返回的是一个IBinder对象，那么肯定有和电话相关的aidl，
	即ITelephony.aidl
	
	3）
	找到ITelephony.aidl文件， 可以看到其中有我们需要的endCall方法，
	因此讲这个文件拷贝到我们的工程中（注意包名要与它的相同）
	4）在eclipse下会自动产生ITelephony.java 文件
	当然这个文件中，有我们需要的endCall()方法的实现，
	5）因此我们就可以成功调用endCall()方法了
	
	
	对于以上分析，其实我不懂的地方还有很多。。。。。。。。
	> 为什么	IBinder iBinder = (IBinder) method.invoke(null, TELEPHONY_SERVICE);返回的这个
	> IBinder对象， 我们就一定知道是 ITelephony这个aidl的实现




