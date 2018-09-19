title: Android-jni
date: 2016/2/29 15:39:53  
categories: Android
---
# Android-jni #

JNI( Java Native Interface)
> - JNI 是一个协议,通过这个协议用来沟通java代码和外部的本地代码(c/c++).
> - 通过这个协议,java代码就可以调用外部的c/c++代码，外部的c/c++代码也可以调用java代码。

> - 在Android架构中，Android的底层是linux kernel，而Android的Framewok classes的运行也依赖于，系统库。
> - linux kernel和系统库都是用c或c++编写的。C/C++都是可以直接操作硬件的语言。 因此若想通过java代码操作硬件，
> - 那么java调C/C++是不可避免的。

学习jni应先了解几件事：

1）交叉编译
> - 简单的说，就是在一个平台下，编译出另一个平台能够执行的二进制的代码。
> - 为什么要这么做呢？普通的android的开发，我们写的java类是运行在JVM中的，并不需要交叉编译。
> - 而对于jni的开发， 我们编写的是C/C++代码。显然他们不可能在JVM中运行。C/C++是可以直接操作硬件的语言，所以他们可以直接由
> 硬件支持而执行，一般android手机的硬件平台都有arm、x86、mips。
> - 对于交叉编译，比如，我们在windows下进行C/C++代码的编译，而这些代码的最终执行平台显然不是windows，而是在手机上执行。

2）NDK（native development kit）
> NDK是常见的交叉编译工具

3）eclipse CDT
> eclipse插件， 使用这个插件，我们可以在eclipse下进行jni的开发。

4）cygwin
> 一个模拟器，让我们可以在window是运行linux指令。
> NDK开发大都涉及到C/C++在GCC环境下编译、运行，所以在Windows环境下，需要用Cygwin模拟Linux编译环境。

**For Windows, Cygwin 1.7 or higher is required. The NDK will not work with Cygwin 1.5 installations.**


## NDK ##
先来看一下NDK的目录中一些重要的文件。（以ndk-r9d为例）

*  docs:帮助文档
*  build/tools：linux的批处理文件
*  platforms：不同平台编译c代码需要使用的头文件和类库
   "jni.h"这个头文件就在这里。  
		这个头文件中定义了java类型与C/C++类型的关系。
		还有其他一些与jni开发相当重要的内容
*  prebuilt：预编译使用的二进制可执行文件
*  sample：jni的使用例子
*  source：ndk的源码
*  toolchains
   进行交叉编译所使用的各种工具链
*  ndk-build.cmd:编译打包c代码的一个指令


具备了C/C++运行的基本环境，那么java与C++是怎么样实现互相调用的呢？？

## jni简单入门 ##
以在eclipse下进行jni开发为例，基本步骤为：

1）在项目根目录下创建jni文件夹
>     该文件夹下存放我们编写的C代码	

2）在java代码中，创建一个本地方法，例如 helloFromC
> - public native String helloFromC();
> - java代码调的肯定是java方法。 他不可能那么生猛的去调C方法，而本地方法，就是java调C的桥梁

3) 编写C代码（需要引入jni.h头文件）

	若想实现jni，那么我们的C代码必须遵循一定的格式，根据本地方法名，我们的C方法应这样写
	返回类型： Java类型   （jni头文件中的）
	方法名（应叫函数名）：   Java_本地方法类的全路径_本地方法名
	函数参数：类型应该为java类型， 且 JNIEnv* env, jobject obj这两个参数必须存在。
	例如 helloFromC()   对应的C函数名应为：
		jstring Java_com_suixin_helloworld1_MainActivity_helloFromC(JNIEnv* env, jobject obj)

4) 把返回类型，由C类型转换为java类型

	例如：
		char* cstr = "hello from c";  //C字符串
		jstring jstr = (*env)->NewStringUTF(env, cstr);  //jni.h头文件
		return jstr；

5) 在jni文件中创建Android.mk文件（与我们编写的C代码位于同一路径）

	内容为： 
		LOCAL_PATH := $(call my-dir)
	    include $(CLEAR_VARS)
		#编译生成的文件的类库叫什么名字
	    LOCAL_MODULE    := hello
	    #要编译的c文件
	    LOCAL_SRC_FILES := Hello.c
	    include $(BUILD_SHARED_LIBRARY)
	这个文件主要用来指示ndk去按其定义内容生成相应的C链接库，即 libXXX.so文件

>- 学过硬件编程的同学，Android.mk 是不是类似于makefile文件了？？？ 
>- libXXX.so是动态C链接库文件，java代码在调用C时，调用的就是其中的内容。（这个链接库文件出来后，.c文件就不需要了）
	
6）在jni文件夹下执行ndk-build.cmd指令

	ndk-build.cmd	 即编译我们C代码生成，*.so文件


>- 在这里编译出来的*.so文件只能在arm架构上使用（eclipse默认）
>- 若想编译出在x86平台上使用的*.so文件， 则应在jni文件夹下新建Application.mk文件
>- 并添加  APP_ABI := armeabi armeabi-v7a x86   就ok了， 这时候就会生成x86、armv7、armeabi的链接库了
> - 还可以  APP_ABI := all  

7）运行工程。 

	java代码中加载so类库，调用本地方法  
	**前面干了这么多事， 其实咱就是为了生成.so类库**

通过以上步骤，就可以简单实现java调用c代码。

###jni开发常见错误
> -  findLibrary returned null
> 	- CPU平台不匹配
> 	- 加载类库时，写错类库名字

> - 本地方法找不到
> 	- 忘记加载类库
> 	- c代码中方法名写错了 

## 使用eclipse提供好的NDK开发环境 ##
> 在eclipse下开发，只要添加了NDK开发环境，其实并不需要向上面那么麻烦的一步一步的去自己操作。

步骤：

1. 配置NDK路径。（类似与配置SDK路径）
1. 右击Android项目, Android tools -> add Native support 
1.   直接写上我们最终想产生的.so文件的名字。 eclipse会自动产生 jni文件夹， 对应的c文件和Android.mk文件
1. 接下来我们所要做的就是在产生的C文件中，写C代码就ok了。

> 在编写C代码时，由于我们没有关联源文件（可以这样理解）， 所以eclipse是无法自动提示的。为了简化开发
> 我们还应关联C/C++的源文件
> -右击项目->propertites->C/C++ Gernel-Path and Symbols->include  （我们把ndk下对应平台的include目录文件夹选定）
> 接下来， 编写C代码就方便多了

### javah指令 ###
> 在进行jni开发时，可能会发现在根据java方法写对应c函数名时，是一件十分蛋疼的事。
> 这是我们可以借助javah指令，来帮我们根据java方法名生成c方法名。

步骤：

- 在项目的src路径下，Shift+右击， 打开命令窗口。
- javah 包名.类名    （相对于src目录）   （注意， 这要求jdk1.7以上，  1.7以下要在bin/classes目录下执行这个命令）
- 这样就会生成一个xxx.h头文件， 里面就有自动生成的c函数名

## java类型与C类型的关键点 ##
	java中，我们平常编写代码都是对象类型，一般我们持有的都是java引用类型变量。
	熟悉C的同学可能会感觉到java引用类型变量就类似c的指针。
	所以我们在将java类型（java引用类型变量）传递给C时，C接收它是可以的。因为C需要的就是指针。
	其实在jni.h头文件中也可以看出：
				typedef    void*           jobject;
	是不是！！！  ->java中是一切对象上帝的Object， 在C中被定义为 void*指针 （void*指针在C中也是类似Object的存在）。

## 在C中调java ##

> java调用C方法是通过本地方法实现的，  而C调java方法，则类似于java的反射。

> 既然类似反射， 那么就先把一个范例代码 贴出来：

	jclass clazz = (*env)->FindClass(env, "com/suixin/ccalljava/MainActivity");  //jclass -> void* 
	jmethodID methodID = (*env)->GetMethodID(env, clazz, "show", "(Ljava/lang/String;)V");   
	(*env)->CallVoidMethod(env, obj, methodID, (*env)->NewStringUTF(env, "反射吗？？？？"));

是不是和java的反射一模一样呢？
- 先拿到类对象
- 再根据方法名和类对象得到方法
- 调用方法。


	细节：   "(Ljava/lang/String;)V" 是我们要调用的java方法的签名
		这个签名怎么的到呢? 我相信你拍脑袋是拍不出来的。  这里我们可以使用 javap指令
		这里与javah不同  ->  在bin/classes/ 下 -> Shift+右击， 打开命令窗口 -> javah -s  包名.类名






























