---
layout: post
title: "JNI实战"
date: 2014-01-17 20:50
comments: true
categories: 
---
# 引子
对于刚入门的Android开发者来说，JNI是较难掌握的。JNI有一层神秘面纱，让人敬而远之。需要同时掌握C/C++语言与Android应用开发技术也提高了进入门槛。
让我们揭开面纱，由浅入深地学习JNI技术。


# JNI是什么
JNI是Java Native Interface的缩写，直译成中文就是Java本地接口。JNI并不是Android特有的技术。在Android之前就在Java中得到广泛应用。Java能做到平台无关是因为运行在虚拟机上，而不同虚拟机在各自平台的实现，大多采用C/C++语言。通过JNI技术，Java代码可以调用C/C++的函数，反过来，C/C++也可以调用Java层的函数。JNI是沟通Java世界与Native世界的桥梁。
<!--more-->

# JNI与Android
JNI在Android开发中占有重要地位。
大家都知道Android系统的四层结构。

1.  最底层是Linux内核层。
2.  第二层Native层包括了运行时库和其他动态库，比如OpenGL ES，WebKit库等，都是用C或C++写的。
3.  再上一层系统框架层是使用Java写的，它与Native层的交互，就大量使用了JNI技术，比如文件操作和socket操作等。
4.  在最上层应用层，NDK开发组件使得开发者可以使用C++来编写核心算法，提高性能和代码复用性。应用需要使用到Android平台特性的地方，通过JNI来与Java层沟通，调用Android系统库提供的、Java语言编写的API，比如播放音乐视频，获取用户地理位置信息或者发送短信。


# 我们用JNI来做什么
目前，市面上大部分手机游戏由于性能和跨平台的考虑，核心部分使用C/C++语言编写。而且，开发过程中或多或少会使用到开源库或游戏引擎，包括渲染引擎，物理引擎和音乐音效引擎等，它们大都使用C/C++编写。在制作Android平台的版本时，前端与手机特性相关的功能使用Java语音编写。两部分交互时需要使用JNI技术。

# JNI开发流程
这里介绍我们平时常用的，在C++部分调用Java方法的场景。假设我们的游戏需要获取用户当前的地理位置信息，用来匹配附近玩家。获取地理位置信息需要从Android的`LocationManager`中获取，匹配工作在C++代码中完成。

直接上代码

*MainActivity.java*

	package com.samxu.hello_jni;
	public class MainActivity extends Activity {
		public LocationUtil locationUtil;
		public LocationManager mLocationManager;
		public Button button1;
		@Override
		protected void onCreate(Bundle savedInstanceState) {
			super.onCreate(savedInstanceState);
			setContentView(R.layout.activity_main);
			// Get a reference to the LocationManager object.
			mLocationManager = (LocationManager) getSystemService(Context.LOCATION_SERVICE);
			locationUtil = new LocationUtil();
			locationUtil.mLocationManager = mLocationManager;

			//调用Native方法stringFromJNI()
			Log.v("hello_jni", stringFromJNI());

			button1 = (Button) findViewById(R.id.button1);
			button1.setOnClickListener(new OnClickListener() {
				@Override
				public void onClick(View v) {
					//调用Native方法callSetupFromJNI()
					Log.v("hello_jni", callSetupFromJNI());
				}
			 });
		}
		public native String stringFromJNI();
		public native String callSetupFromJNI();

		static {
			System.loadLibrary("hellojni"); //加载JNI库
		}
	}

*LocationUtil.java*

	package com.samhxu.hello_jni;

	public class LocationUtil {
		public LocationManager mLocationManager;
		private Location mLocation;

		public void setup() {
			Log.v("hello_jni", "initial setup");
			//init location
			mLocation = requestUpdatesFromProvider(LocationManager.GPS_PROVIDER);
		}
		public String getLocation() {
			return mLocation.getLatitude() + "," + mLocation.getLongitude();
		}
		private final LocationListener listener = new LocationListener() {
			@Override
			public void onLocationChanged(Location location) {
				// A new location update is received.  Do something useful with it.  Update the UI with 
				// the location update.
				mLocation = location;
			}
			//.......other override methods omitted
		};

		private Location requestUpdatesFromProvider(final String provider) {
			Location location = null;
			if (mLocationManager.isProviderEnabled(provider)) {
				mLocationManager.requestLocationUpdates(provider, TEN_SECONDS, TEN_METERS, listener);
				location = mLocationManager.getLastKnownLocation(provider);
			} else {
				//Toast.makeText(this, errorResId, Toast.LENGTH_LONG).show();
			}
			return location;
		}
	}

*hello_jni.c*

	#include <string.h>
	#include <jni.h>
	
	jmethodID g_setupID = 0;
	jfieldID g_locationUtilID = 0;
	jobject g_locationUtil = 0;
	jmethodID g_getLocaitonID = 0;
	
	jstring Java_com_samhxu_hello_1jni_MainActivity_stringFromJNI( JNIEnv* env, jobject thiz ){
		jclass clazz = (*env)->FindClass(env, "com/samhxu/hello_jni/LocationUtil");
		//jobject locationUtil = getInstance(env, clazz);
		g_setupID = (*env)->GetMethodID(env, clazz, "setup", "()V");
		g_getLocaitonID = (*env)->GetMethodID(env,clazz, "getLocation", "()Ljava/lang/String;");
		jclass mainClazz = (*env)->FindClass(env, "com/samhxu/hello_jni/MainActivity");
		g_locationUtilID = (*env)->GetFieldID(env, mainClazz, "locationUtil", "Lcom/samhxu/hello_jni/LocationUtil;");
		jobject _locationUtil = (*env)->GetObjectField(env,thiz, g_locationUtilID);
		g_locationUtil = (*env)->NewGlobalRef(env, _locationUtil);

		(*env)->CallVoidMethod(env, g_locationUtil, g_setupID );

		return (*env)->NewStringUTF(env, "Hello from JNI !");
	}

	jstring Java_com_samhxu_hello_1jni_MainActivity_callSetupFromJNI( JNIEnv* env, jobject thiz ){
		jstring location = (jstring) (*env)->CallObjectMethod(env, g_locationUtil, g_getLocaitonID, NULL);
		return location;
	}

*Android.mk*

	#LOCAL_PATH := $(call my-dir)
	include $(CLEAR_VARS)
	LOCAL_MODULE    := hellojni
	LOCAL_SRC_FILES := src/hello_jni.c
	include $(BUILD_SHARED_LIBRARY) 

# 代码分析
项目怎样搭建起来，网上有很多资料，就不再赘述。我们只分析上述代码的要点。

Java代码中，`System.loadLibrary("hellojni");`加载了JNI库。System.loadLibrary("libname")用来加载Android.mk中定义的Module库名。`public native String stringFromJNI();`声明了native函数，native关键字表示函数由JNI层来实现。
C代码中，实现了两个native函数。对于函数命名规则，我们采用了较为简单的静态注册法，即Java_PackageName_ClassName_MethodName,这样Java就能找到对应的native方法，并建立对应关系。在两个JNI方法中，参数列表都包含了JNIEnv*和jobject，返回值都为jstring。JNIEnv是JNI环境指针，后面会有详细表述。jobject是Java层调用方的对象类型，thiz表示由哪个对象来调用该函数。返回值类型jstring表示在Java层是String类型。数据类型的对应关系，可见下一小节。


# JNI数据类型
Java中的数据类型和JNI的数据类型存在一种对应关系。JNI的数据类型在jni.h中可以找到定义。
下面的表可以概括：
<table>
<tr>
	<th>Java类型</th><th>本地类型</th><th>JNI中定义的别名</th>
</tr>
<tr>
	<td>int</td><td>long</td><td>jint</td>
</tr>
<tr>
	<td>long</td><td>_int64</td><td>jlong</td>
</tr>
<tr>
	<td>byte</td><td>signed char</td><td>jbyte</td>
</tr>
<tr>
	<td>boolean</td><td>unsigned char</td><td>jboolean</td>
</tr>
<tr>
	<td>char</td><td>unsigned short</td><td>jchar</td>
</tr>
<tr>
	<td>short</td><td>short</td><td>jshort</td>
</tr>
<tr>
	<td>float</td><td>float</td><td>jfloat</td>
</tr>
<tr>
	<td>double</td><td>double</td><td>jdouble</td>
</tr>
<tr>
	<td>Object</td><td>_jobject*</td><td>jobject</td>
</tr>
</table> 
<<图>>



# JNIEnv介绍
JNIEnv可以说是JNI中的主角。JNIEnv是与线程相关的代表JNI环境的结构体。每个线程都有一份JNIEnv。在使用时，应稍加注意线程一致性。本文不详细展开。
JNIEnv提供了一系列JNI系统函数，可以用来调用Java函数或者操作jobject对象等。操作jobject对象，实质上就是操作该对象的成员变量和成员方法。
JNIEnv获取对象的成员变量和成员方法的两个函数：
`jfieldID GetFieldID(jclass clazz, const char* name, const char* sig);`
`jmethodID GetMethodID(jclass clazz, const char* name, const char* sig);`
有了成员变量的句柄，我们就可以对成员变量进行get和set操作。
`NativeType Get<type>Field(JNIEnv* env, jobject obj, jfieldID fieldID);`
`void Set<type>Field(JNIEnv* env, jobject obj, jfieldID fieldID, NativeType value);`
有了成员方法的句柄，我们可以进行函数调用。
`NativeType Call<type>Method(JNIEnv* env, jobject obj, jmethodID methodID, … );`
对于静态方法，则使用`CallStatic<type>Method`系列方法。
`NativeType CallStatic<type>Method(JNIEnv* env, jobject obj, jmethodID methodID, … );`

# jstring使用
`stringFromJNI`方法的最后一行，`return (*env)->NewStringUTF(env, "Hello from JNI")`用来返回一个C的字符串。
JNI中的`jstring`对应Java的`String`。而在C/C++中的`string`是另一种类型。两者的构成方式和属性完全不同。JNIEnv的`NewStringUTF`方法会把native的一个UTF-8字符串转换而得到一个`jstring`对象。这样就可以被Java层识别了。


# JNI类型签名
看到前面的`GetMethodID`的最后一个参数，表示方法的参数和返回值。一定觉得这种表现形式奇怪吧。其实是根据转换规则来构成的。

基础类型的JNI描述符
<table>
<tr>
	<th>JNI字段描述符</th><th>java编程语言</th>
</tr>
<tr>
	<td>Z</td><td>boolean</td>
</tr>
<tr>
	<td>B</td><td>byte</td>
</tr>
<tr>
	<td>C</td><td>char</td>
</tr>
<tr>
	<td>S</td><td>short</td>
</tr>
<tr>
	<td>I</td><td>int</td>
</tr>
<tr>
	<td>J</td><td>long</td>
</tr>
<tr>
	<td>F</td><td>float</td>
</tr>
<tr>
	<td>D</td><td>double</td>
</tr>
</table>


一个引用类型的签名，例如`java.lang.String`，由`L`字母开头，并且以分号结束，包名”java.lang.String“中的”.“被”/“替换。所以`java.lang.String`被表示为：`”Ljava/lang/String;"`
下表提供了一个关于如何格式话方法描述符的完整的描述：

<table>
<tr>
	<th>方法描述符</th><th>java语言类型</th>
</tr>
<tr>
	<td><code>"()Ljava/lang/String"</code></td><td>String f();</td>
 </tr>
<tr>
	<td><code>"(ILjava/lang/Class)J"</code></td><td>long f(int i, Class c);</td>
</tr>
<tr>
	<td><code>"([B)v"</code></td><td>void f(byte[ ] bytes);</td>
</tr>
</table>


括号中的表示参数列表，括号后跟着返回值类型。


# 引用与资源回收
我们知道Java中创建的对象会由垃圾回收器来回收和释放对象。
·jobject g_locationUtil·是一个引用对象，在声明的scope结束时会被释放。`stringFromJNI`方法中`g_locationUtil = (*env)->NewGlobalRef(env, _locationUtil)`就是来解决这个问题的。这样，在`callSetupFromJni`方法中使用`g_locationUtil`的时候，就不会有问题。

# 小结
本文结合Android开发中使用JNI的实际问题，主要介绍了JNI的相关内容。想要全面了解JNI的知识，还需要认真阅读JDK文档中关于< < Java Native Interface Specification > >部分和Android开发文档中NDK开发部分，并结合大量实际使用。
