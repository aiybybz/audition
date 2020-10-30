Android
	1.自定义View
	2.事件拦截分发
	3.性能优化工具
	5.缓存



## 目录

### Android

[一、Activity](####一、Activity)
  [1.1 Activity (生命周期)](#####1.1. Activity (生命周期)) 
  [1.2 Activity (运行状态)](#####1.2. Activity (运行状态)) 
  [1.3 Activity (生存期)](#####1.3. Activity (生存期))

  [2.1. Stack (任务栈)](#####2.1. Stack (任务栈)) 
  [2.2. TaskAffinity (任务相关性)](#####2.2. TaskAffinity (任务相关性)) 
  [2.3. LaunchMode (启动模式)](#####2.3. LaunchMode (启动模式))

  [3. Activity启动流程](#####3. Activity(启动流程)) !!!!!未完待续

[二、Service](####二、Service)









---

#### 一、Activity

##### 1. 生命周期

##### 1.1. Activity (生命周期)

##### 1.1.1 正常情况

1. onCreate() `第一次创建`
     • 完成初始化操作（加载布局、绑定事件等）

2. onStart()
     • 由不可见变为可见。并未出现在前台

3. onResume()  `运行状态`
     • `Activity` 准备好和用户进行交互时调用
     • `Activity` 位于栈顶
     • 可在此重新初始化在`onPause()`或`onStop()`中释放的资源

4. onPause()  `暂停状态`
     • 系统准备去启动或恢复另一个 `Activity` 时调用
     • 通常在此释放消耗CPU的资源，保存关键数据
     • `onPause() `执行速度要快，会影响新栈顶 `Activity` 的使用
     • `onPause() `执行完，新 `Activity` `onResume()` 才会执行
   
5. onStop()  `停止状态`
      • `Activity` 完全不可见时调用。停止或完全被覆盖，仅后台运行
      • 可以释放资源（不能太耗时）
      • 新启动对话框式的 `Activity`，`onPause()` 会执行，`onStop()`不会执行
    
6. onDestroy() `销毁状态`
     • `Activity` 被销毁之前调用

7. onRestart() `重新启动`
     • `Activity` 由`停止状态`变为`运行状态`之前调用

##### 1.1.2. 重建情况

当内存不足杀死 `Activity` 时为重建情况

1. onSaveInstanceState() `保存实例状态`
   • `Activity` 异常终止时调用
   • 会把需要保存的数据保存到 `Bundle`
2. onRestoreInstanceState() `恢复实例状态`
   • `Activity` 异常终止后重建调用
   • 还原异常终止时保存的 `Bundle`

##### 1.1.3. 避免重建

  • 指定 `Activity configChanges` 属性可以避免重建。
  • 避免重建后会调用 `onConfigurationChanged()`

```undefined
“locale“ 所在地区发生变化，系统语言
“keyboard“ 键盘模式发生变化，例如：用户接入外部键盘输入
“keyboardHidden“ 用户打开手机硬件键盘
“orientation“ 设备旋转，横向显示和竖向显示模式切换
“fontScale“ 全局字体大小缩放发生改变
```

##### 1.1.4. 复用情况

1. onNewIntent()
   • `singleTop`、`singleTask`、`singleInstance` 三种模式重复启动时会调用
   • 通过此方法可取出请求信息

##### 1.2. Activity (运行状态)

1. Running `运行状态`
   • `Activity` 处于栈顶时
   • 系统回收等级最低
2. Paused `暂停状态`
   • `Activity` 不再处于栈顶，但仍可见（失去焦点、非全屏遮挡、透明遮挡）
   • 状态及变量都还存在
   • 内存紧张时，可能被系统回收掉
3. Stopped `停止状态`
   • `Activity` 不再处于栈顶，完全不可见
   • 状态及变量都还存在，但并不安全
   • 内存紧张时，可能被系统回收掉
4. Killed `销毁状态`
   • `Activity` 从栈中移除后变为销毁状态
   • 系统优先回收当前状态的活动

##### 1.3. Activity (生存期)

1. 完整生存期
   •  `Activity` 在 `onCreate() ~ onDestroy()` 的经历
   • 一般情况，在 `onCreate()` 中完成初始化操作，在 `onDestroy()` 中完成释放内存操作
2. 可见生存期
   • `Activity` 在 `onStart() ~ onStop()` 的经历
   • 可通过`onStart()`、`onStop()`，管理用户可见资源
   • 在 `onStart()` 中加载源，在 `onStop()` 中释放资源
3. 前台生存期
   • `Activity` 在 `onResume() ~ onPause()` 的经历
   • `Activity` 处于运行状态，可与用户进行交互

##### 2. 启动模式

##### 2.1. Stack (任务栈)

  • 栈是一种先进先出的数据结构
  • 当启动一个新`Activity`，会入栈并处于栈顶
  • 当销毁一个 `Activity`，会出栈并前一个入栈的`Activity`重新处于栈顶
  • 任务栈分为前台任务栈和后台任务栈，后台任务栈中 `Activity`  位于暂停状态，可通过切换将后台任务栈再次调到前台

##### 2.2. TaskAffinity (任务相关性)

  • 标识 `Activity` 所需的任务栈名
  • 默认情况下，所有 `Activity` 任务栈名为应用包名
  • 可为每个 `Activity` 单独指定 `TaskAffinity`
  • 主要和 `singleTask 启动模式` 或 `allowTaskReparenting(允许任务重排) `属性配对使用

##### 2.3. LaunchMode (启动模式)

一个任务栈中可以有多个实例每个实例也可以属于不同的任务栈
代码中 `Intent.setFlags(int flags)` 设置启动模式

1. standard (标准)

  • 系统默认模式。每次启动 `Activity` 都会重新创建新实例
  • 此模式下 `Activity`运行在启动它的那个 `Activity` 所在栈中
  • 用 `ApplicationContext` 去启动 `standard` 模式的 `Activity` 会报错。因无任务栈

2. singleTop (栈顶复用)

  • 此模式下如果新 `Activity` 已经位于栈顶，则不会被重建
  • `onNewIntent()`会被回调
  • `onCreate`、`onStart` 不会被调用，因为它并没有发生改变
  • 如果新 `Activity` 的实例已存在但不是位于栈顶，那么新`Activity` 仍会重建

3. singleTask (栈内单例)

  • 单实例，只要 `Activity` 在栈中存在，那么多次启动此 `Activity` 都不会重建
  • `onNewIntent()`会被回调
  • 默认具有 `clearTop` 的效果，会清空栈顶其它 `Activity`
  • 当该模式 `A` 请求启动后，会先寻找是否存在 `A` 想要的任务栈。如不存在则创建任务栈并创建 `A` 压入栈中；如存在所需任务栈，则查看 `A` 是否在栈中如不存在则创建 `A` 压入栈中

4. singleInstance (堆内单例)

  • 加强版的 `singleTask`
  • 此种模式的 `Activity` 只能单独地位于一个任务栈中

##### 3. Activity(启动流程) 

!!!!!未完待续



#### 二、Service

##### 1. 生命周期

##### 1.1. Service (生命周期)

<img src="https://upload-images.jianshu.io/upload_images/3087349-f1a9f1307d0b506f.png" alt="生命周期" style="zoom:65%;" />

##### 1.1.1. startService() 启动 `Service`

• `onCreate()` -- `onStartCommand()` -- `onDestroy()` 

##### 1.1.2. bindService() 绑定 `Service`

• `onCreate()` -- `onBind()` -- `onUnbind()` -- `onDestroy()` 

##### 1.2. Service 方法

1. onCreate() `创建服务`
   • 服务创建时调用，创建后不会再调用

2. onStartCommand() `开始服务`
   返回值杀死服务后如何运行
     • START_NOT_STICKY：不重建`Service`(存在未发送的intent除外)
     • START_STICKY：重建`Service`并用`null intent`调用`onStatCommand()`(存在未发送的intent除外) 
       • 适用播放器类服务，不执行命令，但需要一直运行并随时待命
     • START_REDELIVER_INTENT：重建`Service`并用上个`intent`调用`onStartCommand()`
       • 适用需立即恢复工作的`Service`，如下载文件
     • ...

3. onBind() `绑定服务`
     • `IBinder onBind(Intent intent)`
     • 用来`activity`与`Service`进行通信

4. onUnbind() `解绑服务`

5. stopSelf() `停止服务`

6. onDestroy() `销毁服务`

##### 2. 外部调用

1. startService() `启动服务`
   • 调用：`onCreate()`、`onStartCommand()`

2. stopService() `关闭服务`
   • 调用：`onDestory()`
   • 当`Service`为绑定状态需先解绑后再调用`stopService()`，直接调用`stopService()`无法停止
3. bindService() `绑定服务`
   • 调用：`onCreate()`、`onBind()`
   • `boolean bindService(Intent service, ServiceConnection conn, int flags)`
     • Intent：意图
     • ServiceConnection：监听服务状态
       • `onServiceConnected(ComponentName name, IBinder service)` `activity`与`Service`成功绑定
         •  `IBinder`为`Service`的`Binder`实例，通过此实例操作`Service`
         • 运行在主线程
       • `onServiceDisconnected` `activity`与`Service`解绑
         • 运行在主线程
     • 标志位
       • `BIND_AUTO_CREATE`：绑定存在，就自动创建
       • ...

4. unbindService() `解绑服务`
   • 调用：`onUnbind()`、`onDestory()`

##### 2.1. 其他常用方法

1. `startForeground(int id, Notification notification)` 使后台`Service`变为前台`Service`
   • id：通知id
   • notification：通知对象
2. `stopForeground() ` 从前台状态中删除此`Service`

3. 配置
   • `android:enabled=["true" | "false"]`  能否被系统实例化
   • `android:exported=["true" | "false"]` 能否与其它应用交互
   • `android:process=["Sting"]` 是否在单独的进程

##### 3. 标准服务

• `Service` 默认在主线程，如处理耗时逻辑容易ANR ( Application Not Responding)
• 应在`Service`每个具体方法里开启子线程处理耗时逻辑

##### 4. 服务分类

1. 本地 vs 远程
   • 本地服务(LocalService)
     • 运行在主进程（节约资源）
     • 主线程被终止后，`Service`也会终止（限制性大）
     • 在同一进程不需`IPC`和`AIDL`通信
     • 最常用的`Service`类型，如音乐播放
   • 远程服务(RemoteService)
     • 运行在独立进程（消耗资源）
     • `Service`常驻后台，不受其他`Ativity`影响（灵活）
     • 使用`AIDL`进行`IPC`通信复杂
     • 常作为系统级服务
2. 前台 vs 后台
   • 前台服务
     • 通知栏显示通知 (用户可见)
     • `Service` 显示在通知栏，如音乐播放`Service`
     • 终止时，通知栏的通知也会消失
   • 后台服务
     • 处于后台（用户无法看到）
     • 不需要让用户知道，如天气更新、日期同步。
     • 终止时，用户无法知道

3. 不可通信 vs 可通信
   • 不可通信的后台服务
     • 用`startService()`启动，调用者退出后服务仍然存在
     • `Service` 不需与`Activity`通信
   • 可通信的后台`Service`
     • bindService() 启动：
       • 用`bindService()`绑定，调用者退出后`Service`销毁
       • `Service`需与`Activity`通信、需控制`Service`开始时刻
       • 节约系统资源，第一次`BindService()`时才会创建实例并运行，当服务为远程服务(Remote Service)时效果明显
       • `Service`只是公开远程接口，供客户端调用执行
       • `BroadcastReceiver`也可完成需求，但若交互频綮，容易造成性能问题
     • StartService()、bindService()启动：
       • 用`StartService()`、`bindService()`启动，调用者退出后服务销毁
       • `Service`需与`Activity`、不需控制`Service`开始时刻(服务一开始便运行)

##### 5. IntentService

处理异步请求实现多线程，任务在后台按顺序执行，执行后自行关闭。常用做离线下载等。

1. 本质
   `IntentService` = `Handler` + `HandlerThread`
   • 通过`HandlerThread`单独开启1个工作线程：`IntentService`
   • 创建1个内部`Handler`：`ServiceHandler`
   • 绑定`ServiceHandler`与`IntentService`
   • 通过`onStartCommand()`传递服务`intent`到`ServiceHandler`、依次插入`Intent`到工作队列中并逐个发送给`onHandleIntent()`
   • 通过`onHandleIntent()`依次处理所有`Intent`对象所对应的任务
   因此我们通过复写`onHandleIntent()` 并在里面根据`Intent`的不同进行不同线程操作即可
2. 说明
   • 多次启动`IntentService`，每个耗时操作以队列的方式在`IntentService`的`onHandleIntent`回调中依次执行，执行完自动结束
   • 工作任务队列是顺序执行的，若服务停止，则会清除消息队列中的消息，后续的事件不执行
   • 不建议通过`bindService()`启动`IntentService`。在`IntentService`中`onBind()`默认返回`null`。







进程保活
Service的运行线程（生命周期方法全部在主线程）
Service启动方式以及如何停止
ServiceConnection里面的回调方法运行在哪个线程？

Android Service、IntentService，Service和组件间通信



























- IPC：Inter-Process Communication，即跨进程通信。
- AIDL：Android Interface Definition Language，即Android接口定义语言。用于让某个Service与多个应用程序组件之间进行跨进程通信，从而可以实现多个应用程序共享同一个Service的功能。






##### Activity间通信方式

1. 借助Intent携带数据
2. 借助类的静态变量
3. 借助全局变量 Application
4. 借助 Service 服务
5. 借助外部存储来实现通讯

##### Activity上有Dialog的时候按Home键时的生命周期

- `onCreate()` --  `onStart()` --  `onResume()` --点击**Dialog**--  `onPause()` --  `onStop()`

##### Application 和 Activity 的 Context 对象的区别

- 生命周期不同，Application的生命周期即App的生命周期，通常比Activity的生命周期长的多，所以对于生命周期长的对象，一般使用application作为context，避免不恰当的持有Activity造成内存泄漏。

- Application 不能showDialog

- Application startActivity 时，必须new一个Task

- Application在layoutInflate 时，直接使用默认主题，可能与当前主题不一样

##### Application的生命周期

- onCreate() 程序创建的时候执行
- onTerminate() 程序终止的时候执行
- onLowMemory() 低内存的时候执行
- onConfigurationChanged(Configuration newConfig) 配置改变时触发这个方法。
- onTrimMemory(int level) 程序在进行内存清理时执行

##### RecyclerView

1. 基础优化

   - 不要在`onBind`设置监听。因为`mRecyclerPool`取`ViewHolder`时要重新调用`onBind`。`onClickListener`应设置在`ViewHolder`。

   - 不要在`onBindViewHolder`做逻辑判断和计算。

     `bindViewHolder`方法是在UI线程进行的，如果在该方法进行耗时操作，将会影响滑动的流畅性。

   - 设置高度固定

     如果`RecyclerView`固定宽高，只是用于展示固定大小的组件，可设置`recyclerView.setHasFixedSize(true)`这样可避免每次绘制`Item`时，不再重新计算高度。

   - 重写 onScroll

     对于大量图片、复杂布局的`RecyclerView`考虑重写`onScroll`事件，滑动暂停后再加载。

   - 最大程度不使用 notifyDataSetChanged

     对于新增或删除数据通过`DiffUtil`，来进行局部数据刷新，而不是一味的全局刷新数据。

   - 减少每个 ItemView 的层级嵌套，减少过度绘制

   - 刷新闪烁，设置稳定Id

     当调用`notifyDataSetChanged`的时候，`recyclerView`会刷新所有数据，将所有的`viewHolder`都放入到`pool`中。
     但是，如果我们设置了`stable ids`，那么就会不一样了:`viewHolder`被放入了`scrap`中，而不是`pool`中。注意，这里，它的性能提升了很多！

   - 设置数据预取功能

2. 缓存优化
   - 设置`mCachedViews`大小（壁纸库）
   - 设置每个类型储存的容量 `RecycledViewPool`
   - 多个列表公用一个`RecycledViewPool`
3. 其他
   
   - 分页加载

##### 自定义View

![](https://upload-images.jianshu.io/upload_images/3087349-70cb7dd14ce2a2ff.jpg)

##### Apk的大小如何压缩

- 减少res，压缩图文文件
- 减少dex文件大小
- 减少lib文件大小

##### 类加载机制

- Android中常用的两种类加载器，DexClassLoader和PathClassLoader，它们都继承于BaseDexClassLoader，两者区别在于PathClassLoader只能加载内部存储目录的dex/jar/apk文件。DexClassLoader支持加载指定目录(不限于内部)的dex/jar/apk文件

##### 性能优化

1. 布局优化

2. 绘制优化

3. 内存泄漏优化

4. 响应速度优化

5. ListView/RecycleView及Bitmap优化

6. 线程优化

7. 其他性能优化的建议



#### 二、Java

##### 抽象类

- 解决复用问题，适用于is-a的关系。

##### 接口

- 解决抽象问题，适用于has-a的关系。更加形象的叫法，那就是协议（contract）。
- 接口侧重于解耦。

##### 类加载

![Class的生命周期](https://img2018.cnblogs.com/blog/774371/201907/774371-20190711091002721-196644638.png)

#### 四、设计模式

##### 1. MVC

- 解决的问题
  解决**程序模块化**问题，MVC解耦
- 角色划分
  • Model 模型：负责数据的加载和存储
  • View 视图：负责界面的展示
  • Controller 控制器：负责逻辑控制
- 数据流向
  `View` 产生事件，通知 `Controller` 进行逻辑处理，之后通知给 `Model` 更新数据，更新完成后，再将数据通知给 `View` 去更新界面
- 关系图
  ![MVC](https://raw.githubusercontent.com/aiybybz/ImageBad/master/MVC_20200909163051.png)


##### 2. MVP








##### 2. MVP

解决**隔离度不够**，Activity代码**臃肿**
隔离了MVC中的 M 与 V 的直接联系

- MVP

  解决**隔离度不够**，Activity代码**臃肿**

  隔离了MVC中的 M 与 V 的直接联系

  - M：`Model` 数据模型
  - V：`View` 视图层
  - P：Presenter(代替Controller)负责逻辑的处理

  ![MVP](https://upload-images.jianshu.io/upload_images/15226743-947a7c01f8199148.png)

- MVVM

  **更加分离M、V层**，更加释放Activity的压力

  V和M层之间的耦合程度进一步降低
  
  以**数据模型数据双向绑定**为核心
  
  - M：`Model` 数据模型
  - V：`View` 视图层
  - VM：`ViewModel` 绑定器，View的数据模型和Presenter的合体。双向绑定(data-binding)
  
  ![MVVM](https://upload-images.jianshu.io/upload_images/15226743-1b2adc4a66e12c6e.png)



算法

