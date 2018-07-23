**Android Studio用户指南：[https://developer.android.com/studio/run/index.html](https://developer.android.com/studio/run/index.html)**

#简介#
Instant Run，是android studio2.0新增的一个运行机制，在你编码开发、测试或debug的时候，它都能显著减少你对当前应用的构建和部署的时间。
当我们第一次点击run、debug按钮的时候，它运行时间和我们往常一样。但是接下去的时间里，你每次修改代码后点击run、debug按钮，对应的改变将迅速的部署到你正在运行的程序上，传说速度快到你都来不及把注意力集中到手机屏幕上，它就已经做好相应的更改。

#一个典型的构建周期流程图#
![](http://upload-images.jianshu.io/upload_images/1313748-36d4c846f79411a8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
> 构建->部署->安装->app登录->activity创建

**instant run的目标：尽可能多的剔除不必要的步骤，然后提升必要步骤的速度。**

在实践中，这意味着：

- 只对代码改变部分做构建和部署
- 不重新安装应用
- 不重启应用
- 不重启activity

**热拔插，温拔插，冷拔插**
![](http://upload-images.jianshu.io/upload_images/1313748-ca1496925395d633.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
> instant run = 增量构建 + 热 或 温 或 冷拔插

- **热插拔**：代码改变被应用、投射到APP上，不需要重启应用，不需要重建当前activity。
	- 场景：适用于多数的简单改变（包括一些方法实现的修改，或者变量值修改）

- **温插拔：** activity需要被重启才能看到所需更改。
	- 场景：典型的情况是代码修改涉及到了资源文件，即resources。

- **冷插拔：**app需要被重启（但是仍然不需要重新安装）
	- 场景：任何涉及结构性变化的，比如：修改了继承规则、修改了方法签名等。

#Instant Run运行原理#
![](http://upload-images.jianshu.io/upload_images/1313748-64826a1e2847e169.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
> Manifest整合，然后跟res、dex.file一起被合并到APK

**首次运行Instant Run，Gradle执行的操作**
![](http://upload-images.jianshu.io/upload_images/1313748-b77963070354a0b7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
> APK生成流程

在有Instant Run的环境下：一个新的App Server类会被注入到App中，与Bytecode instrumentation协同监控代码的变化。
同时会有一个新的Application类，它注入了一个自定义类加载器（Class Loader）,同时该Application类会启动我们所需的新注入的App Server。于是，Manifest会被修改来确保我们的应用能使用这个新的Application类。（这里不必担心自己继承定义了Application类，Instant Run添加的这个新Application类会代理我们自定义的Application类）
至此，Instant Run已经可以跑起来了，在我们使用的时候，它会通过决策，合理运用冷温热拔插来协助我们大量地缩短构建程序的时间。

> 在Instant Run运行之前，Android Studio会检查是否能连接到App Server中。并且确保这个App Server是Android Studio所需要的。这同样能确保该应用正处在前台，因为目前是不支持多台设备多程序同时执行。

#热拔插#
![](http://upload-images.jianshu.io/upload_images/1313748-e7c0b89defecdc1e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
> App Server

Android Studio monitors： 运行着Gradle任务来生成增量.dex文件（这个dex文件是对应着开发中的修改类） Android Studio会提取这些.dex文件发送到App Server，然后部署到App。
因为原来版本的类都装载在运行中的程序了，Gradle会翻译，更新好这些.dex文件，发送到App Server的时候，交给自定义的类加载器来加载.dex文件。看看下面原理图：

![](http://upload-images.jianshu.io/upload_images/1313748-932358d7cce43515.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
> App Server

App Server会不断监听是否需要重写类文件，如果需要，任务会被立马执行。新的更改便能立即被响应。我们可以通过打断点调试来发现它确实是这么做，操作如下：
![](http://upload-images.jianshu.io/upload_images/1313748-34eb7c2ae543c5f7.gif?imageMogr2/auto-orient/strip)

#温拔插#
温拔插需要重启Activity，因为资源文件是在Activity创建时加载，所以必须重启Activity来重载资源文件。
目前来说，任何资源文件的修改都会导致重新打包再发送到APP。但是，google的开发团队正在致力于开发一个增量包，这个增量包只会包装修改过的资源文件并能部署到当前APP上。
> **注意：**温拔插涉及到的资源文件修改，在manifest上是无效的（这里的无效是指不会启动Instant Run），因为，manifest的值是在APK安装的时候被读取，所以想要manifest下资源的修改生效，还需要触发一个完整的应用构建和部署。总结起来：如果你修改了manifest相关的资源文件，还是需要面临和以前一样的龟速构建。

#冷拔插#
应用部署的时候，会把工程拆分成十个部分，每部分都拥有自己的.dex文件，然后所有的类会根据包名被分配给相应的.dex文件。当冷拔插开启时，修改过的类所对应的.dex文件，会重组生成新的.dex文件，然后再部署到设备上。
之所以能这么做，是依赖于Android的ART模式，它能允许加载多个.dex文件。ART模式在android4.4（API-19）中加入，但是Dalvik依然是首选，到了android5.0（API-21），ART模式才成为系统默认首选，所以Instant Run只能运行在API-21及其以上版本，至于低版本的话，会重新构建整个应用（下文会提及低版本解决思路）

#Instant Run是不能回退的#
代码更改可以通过热拔插快速部署，但是热拔插会影响应用的初始化，所以我们不得不通过重启应用来响应这些修改。

![](http://upload-images.jianshu.io/upload_images/1313748-db151fbe8ad34afe.gif?imageMogr2/auto-orient/strip)
> 展示增量构建，重启APP，然后回退（这里回退不了，数据还是4）

