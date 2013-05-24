---
layout: post
title: "Intents and Intent filters"
date: 2013-05-07 15:26
comments: true
categories: android
---
Intent和Intent过滤器:
===

英文原文：[http://developer.android.com/guide/topics/intents/intents-filters.html](http://developer.android.com/guide/topics/intents/intents-filters.html)  
版本：Android 4.0 r1  
译者署名：Rongqi Fan  
译者链接：
<!--more-->
程序的3个核心组件——Activity、services、广播接收器——是通过intent传递消息的。intent消息对于运行时绑定不同的组件是很方便的，这些组件可以是同一个程序也可以是不同的。一个intent对象，是一个被动的数据结构，它保存了一个操作的抽象描述——或通常是一个广播的实例，一些发生的事情的描述，一个通知。传递intent到不同组件的机制是互不相同的。

* intent对象是传递给Context.startActivity() 或Activity.startActivityForResult() 以启动Activity或是让一个存在的Activity做些事情。（也可以传递给Activity.setResult()来返回Activity的信息，这个函数叫startActivityForResult()。）
* intent对象传递给函数来初始化一个service或是分发一个新的指令给一个正在进行的service。同样，intent传递给来建立一个在调用组件和目标service间的联系。如果一个service没有运行，它可以开始它。
* intent可以传递给任何广播函数（如：Context.sendBroadcast()、Context.sendOrderedBroadcast()、 Context.sendStickyBroadcast()），intent被分派给所有感兴趣的广播接收者。很多广播源在系统内核里。

Android系统会寻找合适的Activity、service或设置广播接收器来响应intent，在需要的时候实例化它们。在消息系统里没有交叠：广播intent仅仅分派给广播接收器，不会分派给Activity或service。一个intent分派给startActivity()仅仅分派给Activity，不会分派给service或广播接收器，等等。

本文档先描述intent对象。然后描述Activity把intent映射到组件的规则——如何解决什么组件如何接收一个intent消息。没有显示指明目标组件的intent，进程通过intent过滤器测试intent对象来决定潜在的接收者。

## intent对象

intent对象是一个信息桶。它包含了接收它的组件感兴趣的信息（如：携带的动作和数据），附加Android系统感兴趣的信息（如：处理intent和启动目标Activity指令的组件的类别）。主要包含如下信息：

### 组件名

处理intent的组件的名字。这个域是一个对象——包含了目标组件的全名（如：）包含在manifest文件里设置的组件驻留的包名（如：）。组件名的包的名不需要和在manifest文件里设置的包名匹配。

组件名是可选的。如果设置了，intent对象分派到目的类的实例。如果不设置，Android使用intent的其它信息来本地化合适的目标——阅读本文后面的 Intent Resolution部分。

组件的名字通过函数setComponent()、setClass()、setClassName()设置，通过函数读取getComponent()。

### 动作
需要执行的动作的名字——或在广播intent里，发生动作并且被报告。intent类定义动作的常量，如下：

<table>
<tr>
   <th>Constant</th>
   <th>Target component</th>
   <th>Action</th>
</tr><tr>
   <td><code>ACTION_CALL</code>
   <td>activity
   <td>Initiate a phone call.
</tr><tr>
   <td><code>ACTION_EDIT</code>
   <td>activity
   <td>Display data for the user to edit.
</tr><tr>
   <td><code>ACTION_MAIN</code>
   <td>activity
   <td>Start up as the initial activity of a task, with no data input and no returned output.
</tr><tr>
   <td><code>ACTION_SYNC</code>
   <td>activity
   <td>Synchronize data on a server with data on the mobile device.
</tr><tr>
   <td><code>ACTION_BATTERY_LOW</code>
   <td>broadcast receiver
   <td>A warning that the battery is low.
</tr><tr>
   <td><code>ACTION_HEADSET_PLUG</code>
   <td>broadcast receiver
   <td>A headset has been plugged into the device, or unplugged from it.
</tr><tr>
   <td><code>ACTION_SCREEN_ON</code>
   <td>broadcast receiver
   <td>The screen has been turned on.
</tr><tr>
   <td><code>ACTION_TIMEZONE_CHANGED</code>
   <td>broadcast receiver
   <td>The setting for the time zone has changed.
</tr>
</table>

阅读intent类的描述，关于为产生动作而预定义的常量。其它动作被定义于Android的API。你可以在自己的程序里定义自己的动作字符串来激活组件。你的这些发明，需要包含程序包作为前缀——例如：com.example.project.SHOW_COLOR。

动作定义了intent是什么样的结构——指定数据和扩展域——如一个函数名决定一个参数集、一个返回值。由于这个原因，使用动作名的好方法是尽可能的明确，它们配对的和intent里的其它域不一样。换句话说，不是孤立的定义动作，而是定义一个协议以便你的组件可以处理intent对象。

intent里的动作是通过 setAction()函数设置，通过getAction()函数读取。

### 数据
数据的URI和MIME类型的数据。不同的动作和不同的数据配对。例如：如果动作域是ACTION_EDIT，数据域需要包含文档的URI以便显示，编辑。如果动作是ACTION_CALL，数据域需要是一个带拨号的号码的tel: URI。相同的，如果动作是ACTION_VIEW数据域是http: URI，接收Activity需要调用并下载、显示URI引用的任何数据。

intent和组件匹配是处理数据的能力，它通常是从附加在URI的信息知道数据类型的（数据的MIME类型）。例如：组件能显示图形数据不能用来播放音频文件。

很多时候，数据类型可以通过URI推断——但是 content:URI特殊，它指示数据数据被本地化于设备并通过一个内容提供者来控制（阅读separate discussion on content providers）。但是，这种类型还是可以显示的在intent里设置。setData() 函数指定数据作为一个URI， setType()指定它为一个MIME类型，setDataAndType()指定它是URI也是MIME类型。 getData()函数读取URI， getType()读取类型。

### 类型
包含附加信息的字符串，信息是需要处理intent的组件的类型。任何类型的数字描述可以放入intent对象。就像对动作一样，intent类定义一些类常量，包括：

** 这里有表格

<table>
<tr>
   <th>Constant</th>
   <th>Meaning</th>
</tr><tr>
   <td><code>CATEGORY_BROWSABLE</code>
   <td>The target activity can be safely invoked by the browser to display data 
       referenced by a link &mdash; for example, an image or an e-mail message.
</tr><tr>
   <td><code>CATEGORY_GADGET</code>
   <td>The activity can be embedded inside of another activity that hosts gadgets.
</tr><tr>
   <td><code>CATEGORY_HOME</code>
   <td>The activity displays the home screen, the first screen the user sees when 
       the device is turned on or when the <em>Home</em> button is pressed.
</tr><tr>
   <td><code>CATEGORY_LAUNCHER</code>
   <td>The activity can be the initial activity of a task and is listed in 
       the top-level application launcher.
</tr><tr>
   <td><code>CATEGORY_PREFERENCE</code>
   <td>The target activity is a preference panel.
</tr>
</table>  

阅读intent类的描述，里面有完整的类别列表。

addCategory() 放置一个intent里的类别，removeCategory()删除之前添加的，getCategories()获取当前所有的类别。

### 扩展

附加键值对信息，这个键值会分派给处理intent的组件。一些动作和特定的数据URI匹配，一些和特定的扩展匹配。例如：一个ACTION_TIMEZONE_CHANGED的intent有一个time-zone扩展域指明新的时区，ACTION_HEADSET_PLUG有一个state扩展的状态域指明耳机插入或拔出，name也有一个域来指明耳机的类型。如果你发明一个动作SHOW_COLOR ，颜色值在扩展键值对里设置。

intent有一系列的put...() 函数来插入各种类型的数据和一系列get...()函数来获取各种类型的数据。对Bundle 对象，这些函数是并行的。事实上，可以使用函数`putExtras()`和函数`getExtras()`来把数据作为Bundle读取、插入。

### 标志

各种排序的标志。指示Android如何启动Activity（例如：Activity属于那个任务）启动后如何处理（例如：是否属于现在Activity 的列表）。这些标志在intent类里定义。

和平台相关的Android系统和程序使用intent来发送系统的广播、激活系统定义的组件。和intent激活系统组件相关的内容，在list of intents 。

## intent解决
intent可以分为两类：

* 通过名字指定目标组件（阅读component name field，文档里有一个值的集合）。其它程序的开发人员不需要知道组件名，显式的intent用于程序内部消息——如：Activity启动一个下属服务或启动一个姊妹Activity。

* 隐式intent没有命名一个目标（组件名是空的），隐式intent通常用来激活其它程序的组件。
Android分派一个显式的intent给指定目标类的实例。除了intent对象，没可以匹配获取intent组件的组件名。

隐式intent需要不同的策略。没有指定目标，Android系统需要查找最适合处理intent的组件（或几个组件）——一个单一的Activity或服务来执行请求的动作或设置广播接收器来响应广播通知。通过把intent对象的内容和intent管理器比较，判断那个组件是潜在的接收者。过滤器提供组件的能力并且划定它可以处理的intent。它开启可以接收隐式intent的组件类型。如果组件没有intent过滤器，它仅仅可以接收显式的intent。含有过滤器的组件既可以接收隐式intent也可以接收显式intent。

intent过滤器里测试intent对象的三个方面：  
* 动作  
* 数据（URI和数据类型）  
* 类别  

通过扩展和标志不可以确定那个组件接收intent。

### intent过滤器
为了通知系统那个组件、Activity，service，广播过滤器可以处理intent，系统可以有多个intent过滤器。每个过滤器描述一个组件的能力，一个不处理的intent集合——仅仅是不处理隐式intent（这些不命名一个目标类）。一个显示intent总是分派给它的目标，不管它包含什么内容；过滤器这个时候不起作用。但是一个隐式intent仅当它可以通过一个组件的过滤器，次才被分派给这个组件。

组件区分每个过滤器的功能，每个展示给用户的界面，例如：Note Pad程序的Activity有两个过滤器——一个启动一个指定的note，用户可以看或编辑，另一个启动一个新建的、空的note，用户可以编辑并且保存。（Note Pad所有的过滤器在Note Pad Example 一节里有描述。）

#### 过滤器和安全性
Intent过滤器没有可靠的安全性。当它打开一个接收处理显式intent的组件，它并没有阻止隐式intent。尽管一些过滤器限制组件可以处理哪些动作和数据，有时额可以吧不同的动作和数据放入一个显式的intent，并把组件作为目标。

Intent过滤器是`IntentFilter`类的实例。然而，Android系统在启动组件前必须知道组件的能力，intent过滤器是在manifest文件`AndroidManifest.xml`里作为`<intent-filter>` 元素建立而不是在java代码里。（有一个特例是：广播接收器的过滤器，它是通过`Context.registerReceiver()`函数动态的注册；它被作为IntentFilter对象创建。）

一个过滤器有动作域、数据域、intent对象类别域。一个显式的intent测试这三个域。并派送给拥有过滤器的组件，必须通过三个测试。如果有一个测试失败，Android系统都不会分派——至少不是过滤器的基础。然而，如果组件有多个intent过滤器，不分派给一个组件也会分派给另一个组件。

以上三个测试的细节如下：

动作测试
manifest文件里的`<intent-filter>`元素作为 `<action>` 子元素列举动作。例如：
```
 <intent-filter . . . >
    <action android:name="com.example.project.SHOW_CURRENT" />
    <action android:name="com.example.project.SHOW_RECENT" />
    <action android:name="com.example.project.SHOW_PENDING" />
    . . .
 </intent-filter>
```


就像例子显示的，当一个intent对象名仅仅是一个单独的动作，一个过滤器列举更多。这个列表不可以为空；管理器至少需要包含一个`<action>`元素，或它将阻止所有的intent。

传递这个测试，在intent里指定的动作必须匹配管理器列表里的动作。如果对象或过滤器不指定动作，结果如下：

如果过滤器列举动作失败，没有和intent匹配的动作，因此所有的intent测试都失败。没有intent可以打通这个管理器。

另一方面，intent对象不知道一个动作，并自动的传递测试——只要过滤器包含至少一个动作。

类型测试

```
<intent-filter> 元素列举类别作为子元素。例如：
 <intent-filter . . . >
    <category android:name="android.intent.category.DEFAULT" />
    <category android:name="android.intent.category.BROWSABLE" />
    . . .
 </intent-filter>
```
注意：常量描述的动作和类别在manifest文件里没有使用。使用的是完整的字符串。例如：上面例子里的字符串android.intent.category.BROWSABLE反映的是本文档之前提到的常量CATEGORY_BROWSABLE 。同样地，字符串android.intent.action.EDIT反映的是常量ACTION_EDIT。

传递一个intent给类别测试，每个intent里的类别需要匹配一个过滤器里的类别。过滤器列举附加的类别，但是它不可以忽略任何intent里的类别。

因此，一个没有类别的intent对象总是可以通过测试，不论过滤器里是什么内容。大多数情况是这样。然而，有一个特例，Android处理传递给startActivity()的显式intent就像它包含了一个类别：android.intent.category.DEFAULT（ CATEGORY_DEFAULT常量）。因此，想接收显式intnet的Activity需要在intent过滤器里包含。（设置了android.intent.action.MAIN和android.intent.category.LAUNCHER的过滤器是特殊情况。它们把Activity标记为任务前运行，并且会在启动画面上显示。它们在类别列表里包含android.intent.category.DEFAULT，但是并不必须。）更多过滤器请阅读 Using intent matching。

数据测试
和动作、类别一样，intent过滤器指定的数据包含一个子元素。这种情况下，子元素可以多次出现，或不出现。例如：

```
 <intent-filter . . . >
    <data android:mimeType="video/mpeg" android:scheme="http" . . . /> 
    <data android:mimeType="audio/mpeg" android:scheme="http" . . . />
    . . .
 </intent-filter>
```
每个<data> 元素可以指定一个URI和一个数据类型（MIME类型）。有各种属性——scheme、host、port、path ——是URI的每个部分：

 `scheme://host:port/path`

例如：下面的URI
 `content://com.example.project:200/folder/subfolder/etc`

方案是`content`，主机是`com.example.project`，端口是`200`，路径是`folder/subfolder/etc`。主机和端口构成URIauthority；如果主机没有指定，端口也忽略了。

每个属性都是可选的，但是它们并不是独立的；一个authority哟啊有意义，一个方案需要被指定。一个路径要有意义，一个方案和authority需要被指定。

当intent对象里的URI和过滤器里指定的URI比较的时候，仅仅比较过滤器里有的部分。例如：如果过滤器仅仅指定方案，匹配方案的URI都符合条件。如果过滤器指定方案、权限没有指定路径，那么忽略路径的情况下，和方案、权限匹配的URI都符合条件。如果过滤器指定方案、权限、路径，仅当方案、权限、路径都匹配的时候才符合。然而，一个路径里包含宽字符仅需要部分匹配。

`<data>`元素的type 属性指定数据的MIME类型。一般在过滤器里的情况比在URI里多。Intent对象和过滤器可以使用*来表示子域——例如：`text/*`或`audio/*`——表示匹配任何子域。

数据测试对比intent对象里的URI和数据类型和过滤器里的。规则如下：

a.仅仅当过滤器不指定任何URI或数据类型的时候，才会把一个没有包含URI或数据类型的intent对象传递给测试。

b.仅仅如它的URI匹配一个过滤器里的URI并且过滤器没有指定类型（这个类型不可以通过URI推断），会把一个包含URI不包含数据类型的intent传递给测试。这种情况仅出现于URI如`mailto:` 和`tel:`这样不引用实际数据的情况。

c.如果过滤器列出相同的数据类型并不指定URI，一个包含数据类型不包含URI的intent会传递给测试。

d．包含URI和数据类型的intent对象（或可以通过URI推断数据类型）如果它的类型和过滤器的类型的列表你的匹配，那么就测试数据类型部分。如果它的URI和过滤器列表的匹配，或包含`content:`、`file:`、没有指定URI，那么就测试URI部分。换句话说，组件可以断定如果过滤器列表仅仅有数据类型那么它支持`content:` 和 `file:`数据。

如果intent通过过滤器传递几个Activity或服务，会询问用户激活那个组件。如果没有发现目标会产生一个异常。

通常的例子
上面最后一条规则，规则（d）显示了我们希望组件可以从一个文件或内容提供者李获取局部数据。因此，它们的过滤器仅需要列出数据类型，不需要显示的指定主题名为 `file:` 或 `content:`。这是特例。 `<data>`元素如下，将告诉Android系统组件可以从内容提供者里获取图片数据并且显示出来：
 `<data android:mimeType="image/*" />`

通常情况，很多有效的数据被内容提供者，指定数据类型但没有指定URI的过滤器免除。

另一个通常的配置项是有方案和数据类型的过滤器。例如：如下一个元素`<data>`告诉Android系统组件可以从网络上获取视频数据并且显示：

 `<data android:scheme="http" android:type="video/*" />`

考虑这样一个例子：一个浏览器应用在用户点击一个网络链接的时候。它会试着显示数据（这个连接是一个HTML页）。如果它不能显示数据，它把方案和数据类型放入一个显式的intent并且用它来启动可以实现这个功能的Activity。如果仍不行，它调用下载管理器来下载数据。并且让内容管理器来控制，因此会请求一个潜在的更大的Activity池（这些管理器仅仅命名了数据类型）。

许多程序在没有引用任何特殊数据的时候可以启动一个新的。Activity可以初始化一个有把`android.intent.action`.MAIN指定为动作的过滤器。如果它们在程序启动时显示，它们也指定类别`android.intent.category.LAUNCHER`：
```
 <intent-filter . . . >
    <action android:name="code android.intent.action.MAIN" />
    <category android:name="code android.intent.category.LAUNCHER" />
 </intent-filter>
```

### 使用intent匹配
通过intent和intent过滤器的匹配，不仅可以发现目标组件，还可以发现设备上的组件集。例如：Android系统启动程序，顶层屏幕显示用户启动的程序，查找intent过滤器指定`android.intent.action.MAIN`动作和`android.intent.category.LAUNCHER`类别（前一节有说明）的Activity。然后显示图标并且标记这些Activity。同样地，它通过查找过滤器里有`android.intent.category.HOME`的Activity来发现home屏幕。

你的程序可以使用相同的思路来匹配。`PackageManager`有一系列`query...()`函数可以返回intent可以访问的所有组件，一系列`resolve...()`函数来决定那个是最适合相应intent的组件。例如：`queryIntentActivities()`返回可以处理intent的Activity，`queryIntentServices()`函数返回服务列表。；它们仅仅列出可以响应的那个。有一个类似的函数，`queryBroadcastReceivers()`，广播接受者使用的。

## 例子：Note Pad
Note Pad是一个简单的允许用户浏览note列表，查看每一项细节，编辑每个note，添加一个新的note的程序。查看manifest文件里intent过滤器的声明。（如果你用离线的SDK，你可以看到这个程序的所有源代码，包括manifest文件，路径是：`<sdk>/samples/NotePad/index.html`。如果你查阅在线文档，源代码在：Tutorials and Sample Code 。）

在它的manifest文件里，Note Pad程序声明了三个Activity，每个都有一个intent过滤器。也声明一个内容提供者来管理note数据。Manifest文件如下：
```
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
          package="com.example.android.notepad">
    <application android:icon="@drawable/app_notes"
                 android:label="@string/app_name" >

        <provider android:name="NotePadProvider"
                  android:authorities="com.google.provider.NotePad" />

        <activity android:name="NotesList" android:label="@string/title_notes_list">
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />
                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
            <intent-filter>
                <action android:name="android.intent.action.VIEW" />
                <action android:name="android.intent.action.EDIT" />
                <action android:name="android.intent.action.PICK" />
                <category android:name="android.intent.category.DEFAULT" />
                <data android:mimeType="vnd.android.cursor.dir/vnd.google.note" />
            </intent-filter>
            <intent-filter>
                <action android:name="android.intent.action.GET_CONTENT" />
                <category android:name="android.intent.category.DEFAULT" />
                <data android:mimeType="vnd.android.cursor.item/vnd.google.note" />
            </intent-filter>
        </activity>
        
        <activity android:name="NoteEditor"
                  android:theme="@android:style/Theme.Light"
                  android:label="@string/title_note" >
            <intent-filter android:label="@string/resolve_edit">
                <action android:name="android.intent.action.VIEW" />
                <action android:name="android.intent.action.EDIT" />
                <action android:name="com.android.notepad.action.EDIT_NOTE" />
                <category android:name="android.intent.category.DEFAULT" />
                <data android:mimeType="vnd.android.cursor.item/vnd.google.note" />
            </intent-filter>
            <intent-filter>
                <action android:name="android.intent.action.INSERT" />
                <category android:name="android.intent.category.DEFAULT" />
                <data android:mimeType="vnd.android.cursor.dir/vnd.google.note" />
            </intent-filter>
        </activity>
        
        <activity android:name="TitleEditor" 
                  android:label="@string/title_edit_title"
                  android:theme="@android:style/Theme.Dialog">
            <intent-filter android:label="@string/resolve_title">
                <action android:name="com.android.notepad.action.EDIT_TITLE" />
                <category android:name="android.intent.category.DEFAULT" />
                <category android:name="android.intent.category.ALTERNATIVE" />
                <category android:name="android.intent.category.SELECTED_ALTERNATIVE" />
                <data android:mimeType="vnd.android.cursor.item/vnd.google.note" />
            </intent-filter>
        </activity>
        
    </application>
</manifest>
```

第一个Activity，NoteList，和其它Activity不同，它操作一个note的目录（note列表）而不是一个单一的note。通常是充当程序的初始化界面。它额可以依据下面三个intent过滤器来做三件事。
```
 <intent-filter>
    <action android:name="android.intent.action.MAIN" />
    <category android:name="android.intent.category.LAUNCHER" />
 </intent-filter>
```
这个过滤器声明指向Note Pad程序的实体。标准动作是一个在intent里不需要任何信息（比如：没有数据）的实体点，类别说的是实体点需要在程序的启动里列出。
```
 <intent-filter>
    <action android:name="android.intent.action.VIEW" />
    <action android:name="android.intent.action.EDIT" />
    <action android:name="android.intent.action.PICK" />
    <category android:name="android.intent.category.DEFAULT" />
    <data android:mimeType="vnd.android.cursor.dir/vnd.google.note" />
 </intent-filter>
```
这个过滤器声明Activity可以对节点做的事情。允许用户浏览或编辑词典（通过`VIEW` 和`EDIT` 动作），或是获取节点（通过`PICK` 动作）。

`<data>` 元素的mimeType属性指定这些动作操作的数据类型。它指示了Activity可以从保存Note Pad数据（`vnd.google.note`）的内容提供者里获取零个或多个Cursor项（`vnd.android.cursor.dir`）。启动Activity的intent需要包含一个 `content:URI`来指定Activity需要打开的扩展数据。

主要这个过滤器里指定了`DEFAULT`类别。因为`Context.startActivity()` 和`Activity.startActivityForResult()`函数处理所有的intent就像它们包含`DEFAULT`类别——仅仅有两个特例：

* 显式知道目标Activity的intent。  
* 包含MAIN 动作和LAUNCHER 类别的intent。

因此，`DEFAULT` 类别需要所有的过滤器——处理有动作`MAIN` 和类别`LAUNCHER` 的。（intent过滤器不需要显示的intent。）

```
 <intent-filter>
    <action android:name="android.intent.action.GET_CONTENT" />
    <category android:name="android.intent.category.DEFAULT" />
    <data android:mimeType="vnd.android.cursor.item/vnd.google.note" />
 </intent-filter>
```
过滤器描述了Activity可以翻译一个用户选中的note，用户没有从指定的目录里查询。
动作和动作是类似的。Activity返回用户选中的note的URI。（调用`startActivityForResult()`启动NoteList Activity，）然而，调用者指定数据类型，而不是用户从目录里获取数据类型。

数据类型，`vnd.android.cursor.item/vnd.google.note`，指明了Activity可以返回的数据类型——一个独立的节点的`URI`。从返回的`URI`，调用者可以从保存Note Pad 数据（`vnd.google.note`）的内容提供者里获取一个项的Cursor（`vnd.android.cursor.item`）

换句话说，之前过滤器里的`PICK` 动作，数据类型暗示了Activity显示给用户的数据类型。对于`GET_CONTENT`过滤器，它指示了Activity可以返回给调用者的数据类型。

给定这些功能，下面的intent将解决NotePad Activity：

动作：`android.intent.action.MAIN`
没有数据指定的情况下启动Activity。

动作：`android.intent.action.MAIN` 
类别：`android.intent.category.LAUNCHER`
没有数据选择域的情况启动Activity。

动作：`android.intent.action.VIEW`
数据：`content://com.google.provider.NotePad/notes`
调用Activity显示`content://com.google.provider.NotePad/notes`下note的列表。用户可以通过列表浏览和获取没一个项的信息。

动作：`android.intent.action.PICK`
数据：`content://com.google.provider.NotePad/notes`
调用Activity来显示`content://com.google.provider.NotePad/notes`下的note的列表。用户可以从列表里选择一note，Activity将返回该项的URI来启动一个NoteList Activity。

动作： `android.intent.action.GET_CONTENT`
数据类型：`vnd.android.cursor.item/vnd.google.note`
调用Activity来提供一个单独的Note Pad数据项。

第二个Activity，NoteEditor，显示一个note给用户，并且允许用户编辑。在intent的过滤器里描述了它可以做的两件事情：

```
 <intent-filter android:label="@string/resolve_edit">
    <action android:name="android.intent.action.VIEW" />
    <action android:name="android.intent.action.EDIT" />
    <action android:name="com.android.notepad.action.EDIT_NOTE" />
    <category android:name="android.intent.category.DEFAULT" />
    <data android:mimeType="vnd.android.cursor.item/vnd.google.note" />
 </intent-filter>
```
Activity的第一个、最基本的功能是允许用户和一个note交互——或者浏览或者编辑。（EDIT_NOTE类对EDIT来说是一个。）intent需要包含和数据的URI并和MIME类型`vnd.android.cursor.item/vnd.google.note`匹配——URI是独立的、指定note的URI。需要是一个通过`PICK` 或`GET_CONTENT`动作从NoteList Activity里返回的URI。过滤器列出了`DEFAULT` 因此Activity可以通过intent来启动，这个intent没有显示的指定NoteEditor类。

```
 <intent-filter>
    <action android:name="android.intent.action.INSERT" />
    <category android:name="android.intent.category.DEFAULT" />
    <data android:mimeType="vnd.android.cursor.dir/vnd.google.note" />
 </intent-filter>
```
Activity的第二个作用是允许用户创建新的note，并插入到已有的note字典里。Intent需要包含和数据的`URI`匹配的MIME类型`vnd.android.cursor.dir/vnd.google.note` ——词典的`URI`需要被替换。

给定这些功能，接下来的intent可以NoteEditor的Activity：

动作：`android.intent.action.VIEW`
数据：`content://com.google.provider.NotePad/notes/ID`
调用Activity显示通过ID指定的note的内容。（关于 `content:URI`指定群成员的细节知识，阅读Content Providers。）

动作：`android.intent.action.EDIT`
数据：`content://com.google.provider.NotePad/notes/ID`
调用Activity来显示通过ID 标识的note的内容。如果用户保存修改，Activity更新在内容提供者里的note的数据。

动作：`android.intent.action.INSERT`
数据：`content://com.google.provider.NotePad/notes`
调用Activity在 `content://com.google.provider.NotePad/notes` 的note列表里创建一个新的、空的note。允许用户编辑它。如果用户保存note，note的`URI`返回给调用者。

最后一个Activity，TitleEditor，允许用户编辑note的标题。通过调用Activity（显式的设置它的intent的组件名）来实现，而不是使用intent过滤器。展示如何对已经存在的数据执行操作：
```
 <intent-filter android:label="@string/resolve_title">
    <action android:name="com.android.notepad.action.EDIT_TITLE" />
    <category android:name="android.intent.category.DEFAULT" />
    <category android:name="android.intent.category.ALTERNATIVE" />
    <category android:name="android.intent.category.SELECTED_ALTERNATIVE" />
    <data android:mimeType="vnd.android.cursor.item/vnd.google.note" />
 </intent-filter>
```
这个intent过滤器可以使用一个用户定义的名为`com.android.notepad.action.EDIT_TITLE`的动作。调用一个指定的note（数据类型是`vnd.android.cursor.item/vnd.google.note`），如之前的`VIEW` 和`EDIT` 动作。然而，Activity显示包含着note数据里的标题，不包含note内容本身。

支持类`DEFAULT` ，标题编辑器支持其它两个标准的类：`ALTERNATIVE`和`SELECTED_ALTERNATIVE`。这些类别暗示了Activity可以在一个菜单选项里显示给用户（比如：`LAUNCHER` 类别暗示了Activity需要在程序启动的时候显示给用户）。注意：过滤器也提供显示的标签（通过：`android:label="@string/resolve_title"`）来控制Activity，这个Activity用来替代显示数据的动作。（更多关于这些类别的信息、建立菜单选项，阅读 `PackageManager.queryIntentActivityOptions()`和`Menu.addIntentOptions()`函数。）

给定这些功能，下面的intent可以解决TitleEditor Activity：

动作： `com.android.notepad.action.EDIT_TITLE`  
数据：`content://com.google.provider.NotePad/notes/ID`  
调用Activity来显示和ID关联的标题，允许用户编辑标题。