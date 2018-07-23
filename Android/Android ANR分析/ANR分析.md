#概述#
ANR，是“Application Not Responding”的缩写，即“应用程序无响应”。
与Java Crash或者Native Crash不同，ANR并不会导致程序崩溃，如果用户愿意等待，大多数ANR在一段时间后都是可以恢复的。但对于用户而言，打开一个窗口就要黑屏8秒，或者按下一个按钮后10秒程序没有任何响应显然是不可接受的。

为了便于开发者Debug自己程序中响应迟缓的部分，Android提供了ANR机制。ActivityManagerService(简称 AMS)和 WindowManagerService(简称 WMS)会监测应用程序的响应时间，如果应用程序主线程(即 UI 线程)在超时时间内对输入事件没有处理完毕，或者对特定操作没有执行完毕，就会出现 ANR。

#ANR发生的原因#
一般地，ANR的产生需要同时满足三个条件：

- **主线程**：只有应用程序进程的主线程响应超时才会产生ANR。因为只有主线程也就是UI线程需要与用户进行交互，子线程的阻塞或者缓慢只要不影响主线程就不会引发ANR
- **超时时间**：不同类型ANR的超时时间不同，只要主线程在这个时间上限内没有响应就会ANR
- **输入事件、特定操作**:输入事件是指按键、触屏等设备输入事件，特定操作是指BroadcastReceiver和Service的生命周期中的各个函数，产生ANR的场景不同，报出ANR的原因也会不同

如何理解“产生 ANR 的场景不同，报出ANR的原因也会不同”呢？

假设应用程序主线程被阻塞

- 如果用户点击屏幕，稍后会报出“用户输入事件处理超时”ANR；
- 如果来了需要处理的广播，会导致“广播处理超时”；
- 如果用户切换窗口，则可能导致“窗口获取焦点超时”。
 
  同一个阻塞的位置和原因，在不同情况下报出的ANR类型和现象可能是不同的。这就需要在分析过程中透过现象看本质，找到不同Bug共同的原因，从而准确、快速地处理。

#ANR的类型#
##用户输入事件处理超时##
当应用程序的窗口处于活动状态并且能够接收输入事件(例如按键事件、触摸事件等)时，系统底层上报的事件就会被InputDispatcher分发给该应用程序。对大多数窗口而言“处于活动状态”可以理解为“获得焦点”，但是一些具有FLAG_NOT_FOCUSABLE属性的窗口，如Popup窗口，不能获得焦点不能接收按键事件只能接收触摸事件，使得这两个概念不能完全等价。

应用程序的主线程通过InputChannel读取输入事件并交给界面视图处理，界面视图是一个树状结构，DecorView是视图树的根，事件从树根开始一层一层向端点(例如一个 Button)传递。开发者通常需要注册监听器来接收并处理事件，或者创建自定义的视图控件来处理事件。

InputDispatcher运行在system_server进程的一个子线程中，每当接收到一个新的输入事件，InputDispatcher就会检测前一个已经发给应用程序的输入时间是否已经处理完毕，如果超时，会通过一系列的回调通知WMS的notifyANR函数报告ANR发生，

需要注意的是，产生这种ANR的前提是要有输入事件，如果没有输入事件，即使主线程阻塞了也不会报告ANR。从设计的角度看，此时系统会推测用户没有关注手机，寄希望于一段时间后阻塞会自行消失，因此会暂时“隐瞒不报”。从实现的角度看，InputDispatcher没有分发事件给应用程序，当然也不会检测处理超时和报告ANR了。

此类ANR发生时的提示语是：**Reason: Input dispatching timed out (Waiting because the focused window has not finished processing the input events that were previously delivered to it.)**需要注意区分同为Input dispatching timed out大类的窗口获取焦点超时，这两类超时括号内的提示语是不同的。

此类ANR的超时时间在ActivityManagerService.java中定义，默认为5秒。如果有需要可以修改代码将小内存设备上的超时时间改为8秒。另一个常见的修改是在手机启动后的4分钟内将超时时间暂时提高到15秒，因为开机后MediaServer扫描媒体数据库会消耗大量CPU，这样修改有助以提高Monkey测试时的首错时间。

##窗口获取焦点超时##
窗口获取焦点超时是用户输入事件处理超时的一种子类型，它们都由InputDispatcher向AMS上报。当应用程序的窗口处于“活动状态”并且能够接收输入事件时，系统底层上报的事件就会被InputDispatcher分发给该应用程序。如果由于某种原因，窗口迟迟不能达到“活动状态”，不能接收输入事件，此时InputDispatcher就会报出“窗口获取焦点超时”。

此类ANR发生时的提示语是：**Reason: Input dispatching timed out (Waiting because no window has focus but there is a focused application that may eventually add a window when it finishes starting up.)**需要注意区分同为Input dispatching timed out大类的用户输入事件处理超时，这两类超时括号内的提示语是不同的。

为了研究窗口为什么会获取焦点超时，我们需要简单了解在窗口切换过程中焦点应用和焦点窗口的切换逻辑。假设当前正处于应用A中，将要启动应用B。启动过程中焦点应用和焦点窗口转换如下：

-  流程开始，焦点应用是A，焦点窗口是A（的某一个窗口）
-  当A开始OnPause流程后，焦点应用是A，焦点窗口是null
-  在zygote创建B的进程完毕后，焦点应用是B，焦点窗口是null
-  应用B的OnResume流程完成后，焦点应用是B，焦点窗口是B（的某一个窗口）

**在这个过程中，如果焦点窗口为 null的时间超过了5秒，那么当前焦点应用就会被报告为窗口获取焦点超时类的ANR**

需要注意的是会被报告为ANR的是“当前焦点应用”而不是B。理论上讲创建新应用进程的速度非常快，焦点应用总是能及时地切换到新应用B上，在理想情况下“当前焦点应用”和“新启动的应用B”是等价的。

可惜在实际操作中，某些情况下发生ANR时，被报出ANR的应用并不是真正发生ANR的应用。如果步骤3中zygote迟迟创建不出应用B的进程，那么焦点应用会一直保持在A上，超时后就会报出A发生ANR；此外**Android4.4上**为了适应多窗口逻辑的需要，WMS和InputDispatcher维护的焦点窗口和焦点应用可以不同步，且原生代码中存在Bug。比如在上述流程中，步骤3中应用B的进程创建完成，但是由于原生Bug导致焦点应用没有转换，超时后同样会报出A发生ANR。

因此在分析窗口获取焦点超时的ANR时，一定要注意分析当前焦点应用和焦点窗口是否一致，首先要明确ANR的真正应用是哪一个，后续的分析才会有价值

窗口获取焦点超时通常由以下4类原因导致:

- 应用程序创建慢。程序的OnCreate/OnStart/OnResume方法执行速度慢/存在死锁/死循环导致OnResume迟迟不能执行完毕，超时造成ANR
- 应用程序'OnPause'慢。对同一个应用而言，前一次OnPause执行完毕之前后一次OnResume不会执行。但不同应用之间不会互相影响
- 系统整体性能慢。由于系统性能原因，如CPU占用率高/平均等待队列长/内存碎片化/页错误高/GC慢/用户空间冻结/进程陷入不可打断的睡眠，会造成整体运行慢使ANR频繁发生
- 'WMS'异常。由于4.4上存在的原生Bug，有时应用OnResume执行完毕后8秒焦点仍然不会转换。导致ANR发生

##广播超时##
当应用程序主线程在执行BroadcastReceiver的onReceive方法时，超时没有执行完毕，就会报出广播超时类型的ANR。对于前台进程超时时间是10秒，后台进程超时时间是60秒。如果需要完成一项比较耗时的工作，应当通过发送Intent给应用的Service来完成，而不应长时间占用**OnReceive主线程**。**与前两类ANR不同，系统对这类ANR不会显示对话框提示，仅在slog中输出异常信息**

此类ANR发生时的提示语是：**Reason: Broadcast of Intent { act=android.net.wifi.WIFI_STATE_CHANGED flg=0x4000010 cmp=com.android.settings/.widget.SettingsAppWidgetProvider (has extras) }**

在小内存Android设备上，Kernel中的LowMemoryKiller会频繁地杀死一些后台应用以释放内存。如果一个应用恰好在开始执行OnReceive方法时被LMK杀死，那么在60秒后BoardcastQueue检查广播处理情况时此应用就一定会发生ANR。这种场景的关键特征是报出ANR时System.log中会显示ANR应用的PID为0。

为避免此类问题发生，提高Monkey测试首错时间，可以在BoardcastQueue中添加代码，检测广播超时ANR的PID，为0时不报ANR

##Service 执行超时##
**Service 的各个生命周期函数，如OnStart、OnCreate、OnStop也运行在主线程中，当这些函数超过20秒钟没有返回就会触发 ANR。同样对这种情况的 ANR 系统也不会显示对话框提示,仅是输出 log**。

此类ANR的提示语是：**Reason: Executing service com.android.bluetooth/.btservice. AdapterService**

##ContentProvider执行超时##
在Android4.4后继平台上，为方便ContentProvider进行Debug增加了此类ANR。当主线程在执行ContentProvider相关操作时没有在规定的时间内执行完毕就会发生ANR。**由程序开发者自行设置是否启用以及超时时间**。ANR发生时的提示语为：**Reason: ContentProvider not responding**。

#应用如何避免ANR#
造成ANR的原因可以分成两大类，**系统原因与应用原因**。系统原因是指在手机开发过程中，由于Kernel、FrameWork、驱动存在问题，导致系统工作不稳定，最终在应用层表现出来的ANR。如CPU驱动错误导致四核手机只有一个核运行、Kernel将用户空间冻结导致任何程序都不能执行、I/O吞吐量低下导致应用程序长时间等待I/O，HAL层实时进程长时间占用CPU导致调度队列过长、AMS原生Bug导致系统焦点不能正确转换等等。对于此类问题，如果底层无法在交付时确保系统稳定，就需要在分析大量ANR问题的基础上提炼出其共同规律，针对疑点添加debug信息，再通过长时间的复测才能加以解决。

应用原因是指应用程序主线程死锁、阻塞或者性能低下导致ANR。应用自身为避免发生ANR，应当在程序开发中注意避免将耗时的操作放在主线程，耗时操作包括：

- **数据库操作**: 数据库操作尽量采用异步方法做处理，Monkey测试中IOWait可能会很高，此时一个微不足道的数据库查询操作都可能需要很长时间才能返回
- **初始化的数据和控件太多**:可以用布局查看工具HierarchyViewer来优化UI设计，避免深层嵌套。此外还需要注意限制控件输入字符的数量，测试人员曾在一个设计输入40个字的输入框中输入了四十万字然后反复转屏，引发ANR
- **频繁的创建线程或者其它大对象**:有些应用在Monkey测试中会创建出800+子线程，显然应避免这样做
- **加载过大数据和图片**:对于彩信或Gallery，设计时应当考虑加载缩略图而不是原始图片，因为测试很喜欢用100M的jpg图片做压力测试
- **对大数据排序和循环操作**:曾有人写出复杂度为O(n^2)的通讯录联系人匹配算法，测试时发现匹配两千个联系人需要15分钟，最后被优化到25秒。显然这种操作应当放在子线程中处理
- **过多的广播和滥用广播**:如果处理一个广播需要花费较长时间，应当通过发送Intent给应用的Service来完成
- **大对象的传递和共享**:bundle的传递
- **访问网络**：4.4项目曾经遇到过一个从谷歌应用商店下载的程序启动后在主线程中访问Facebook，然后在GFW（Great Firewall）上撞得头破血流导致ANR
- **锁住主线程**：Gallery原生代码中会给主线程上一个无限等待的锁，然后由子线程来解锁。但是在Monkey测试中如果系统处于高负载解锁不及时就会发生ANR，应尽量避免

#如何分析ANR问题#
与Native Crash或者Java Crash发生时简单明确的崩溃堆栈不同，分析ANR问题时需要通盘考虑，综合log中各方面的信息，找出是系统原因还是应用原因导致ANR。片面地分析应用程序堆栈或者CPU信息通常都只能得出令人啼笑皆非的错误结论

在一份标准的sLog中，通常有以下文件可用于分析ANR的原因：

- **system.log**： 包含ANR发生时间点信息、ANR发生前的CPU信息，还包含大量系统服务输出的信息
- **main.log**： 包含ANR发生前应用自身输出的信息，可供分析应用是否有异常；此外还包含dalvik输出的GC信息，可供分析内存回收的速度，判断系统是否处于低内存或内存碎片化状态
- **event.log**： 包含AMS与WMS输出的应用程序声明周期信息，可供分析窗口创建速度以及焦点转换情况
- **kernel.log**： 包含kernel打出的信息，LowMemoryKiller杀进程、内存碎片化或内存不足，mmc驱动异常都可以在这里找到
- **napshot.log、trace.log**： 在有些项目中这两个文件可能被合并为一个，包含ANR发生时的应用堆栈信息、PS信息和meminfo信息。根据定制需要，还可以包含ANR发生时的system_server堆栈、SurfaceFlinger堆栈、文件句柄状况等信息
- **tomestone**： 有些应用的ANR是由于之前应用已经崩溃导致的，需要注意以下在ANR发生前如果在tomestone文件夹中此应用已经发生了Native Crash，那么ANR很有可能就是由此导致的
- **Dropbox**： 该文件会把snapshot中的信息备份一份，如果因为某些原因导致snapshot文件丢失，可以尝试在dropbox中寻找ANR发生时的堆栈信息

##System.log中的信息##
- **ANR第一时间点**

在分析ANR时首先要寻找的就是ANR的第一时间点。如第一次出现“Input event dispatching timed out sending to ……”字样，这就是ANR发生的第一时间点。在明确了这个时间点后，才能根据不同ANR的类型分析在超时时间段内系统和应用有什么异常信息。例如广播超时需要分析第一时间点前10秒（后台广播60秒）的广播队列信息；窗口转换超时需要分析第一时间点前5秒的窗口焦点转换过程和event.log中的窗口生命周期信息。通过ANR第一现场的时间和ANR类型，可以找到出错时间段，压缩需要分析的时间范围

- **ANR in信息** 

通常分析ANR问题时第一件事就是打开system.log，找到“ANR in”信息，然后大致了解一下发生ANR的是什么应用，CPU占用情况如何。接下来会详细讲解这段log中包含的诸多信息。

需要特别注意的是“ANR in”并不是ANR发生的第一时间点。它是在输出ANR应用堆栈和主要系统服务堆栈、ps、meminfo等信息后，ANR进程马上就要被杀死时才被输出的。如果ANR发生时输出的debug信息较多，或者当时的CPU或I/O资源比较紧张，“ANR in”可能在ANR发生的第一时间点之后10秒甚至20秒才被输出

ANR in相关log的信息主要有：

- **ANR进程名称与PID**：进程名和PID是找到发生ANR的应用的主要依据，但是有两种例外情况。如果PID为0，说明应用在发生ANR之前就已经被LowMemoryKiller杀死或者已经崩溃。这种情况下应用程序无法处理广播或按键消息，因此出现ANR；由于原生Bug，窗口获取焦点超时导致的ANR可能会报告在错误的应用上，这主要是因为焦点应用和焦点窗口不同步导致的
-  **ANR类别**：可以据此判断ANR超时时间，决定需要回溯多久查找ANR的原因。比如用户输入时间处理超时回溯5秒，广播超时回溯10秒。
-  **CPU统计时间段**：ago和Later分别表示 ANR 发生前后一段时间内的 CPU 使用率,并不是某一时刻的值。需要注意的是，这个统计本身也会收到CPU负载高的影响，可能无法统计到ANR发生之前的CPU状况
-  **进程CPU占用率**：单核手机上，如果进程CPU占用率较高，且截取到的是ANR发生之前的CPU情况，那么ANR可能与CPU占用率有关。
-  **页错误数量**：分为次要页错误和主要页错误，分别表示内存与缓存的命中情况。页错误过高说明内存频繁换页，会导致分配内存与GC速度显著变慢
- **新增进程**：ANR发生前出现大量的新增进程说明可能广播风暴或密集的延时重启
- **总CPU占用率**：在单核设备上可以保证准确，在支持热插拔的设备上一般不准确
- **线程CPU占用率**：可配合snapshot中的应用调用堆栈分析单个进程CPU占用率高问题

在分析System.log时，应当注意应用发生ANR之前是否出现了OutOfMemory错误、java Crash或者Native Crash。此外还应当注意应用相关的服务是否出现了异常，比如acore被LMK杀死contact就会发生ANR，camera handler发生崩溃会导致camera发生ANR

##Main.log中的信息##
Mian.log中主要是应用输出的信息，相对System.log而言会比较散乱。从中挖掘ANR的关键信息需要一定的耐心。在分析时应当关注以下几个方面：

- **反复打出相同log**: 如果应用CPU占用率很高，且反复打出相同log，很可能是出现死循环：

	    05-20 14:26:32.875  2896  2896 V BluetoothSettings: onPreferenceChange()shouldScanDisabled = false
		
		05-20 14:26:32.895  2896  2896 V BluetoothSettings: onPreferenceChange()shouldScanDisabled = false
		
		05-20 14:26:32.915  2896  2896 V BluetoothSettings: onPreferenceChange()shouldScanDisabled = false
		
		05-20 14:26:32.925  2896  2896 V BluetoothSettings: onPreferenceChange()shouldScanDisabled = false
		
		05-20 14:26:32.945  2896  2896 V BluetoothSettings: onPreferenceChange()shouldScanDisabled = false
		
		05-20 14:26:32.965  2896  2896 V BluetoothSettings: onPreferenceChange()shouldScanDisabled = false
		
		05-20 14:26:32.975  2896  2896 V BluetoothSettings: onPreferenceChange()shouldScanDisabled = false
		
		05-20 14:26:33.005  2896  2896 V BluetoothSettings: onPreferenceChange()shouldScanDisabled = false 05-20 14:26:33.025  2896  2896 V BluetoothSettings: onPreferenceChange()shouldScanDisabled = false

- **应用输出的log是否存在异常**：如下例，Camera尝试分配26214400 
Byte显存，但是从16:42:16开始申请直到16:42:27才分配完毕，这段时间主线程一直被阻塞，因此发生ANR。在应用程序容易出现性能问题的关键点适度添加log，对查找ANR问题非常有帮助

	    01-01 16:42:16.942   150  1676 I SprdCameraHardware: allocateCaptureMem, mJpegHeapSize = 26214400, mRawHeapSize = 26214400
	
		01-01 16:42:16.942   150  1676 I SprdCameraHardware: allocateCaptureMem:mRawHeap align size = 26214400 . count 1
		
		......
		
		01-01 16:42:27.142   150  1676 I SprdCameraHardware: allocCameraMem: mm_iova: phys_addr 0x20000000, data: 0xaf50d000, size: 0x1900000, phys_size: 0x1900000.
		
		01-01 16:42:27.142   150  1676 I SprdCameraHardware: allocateCaptureMem: initJpeg

- **是否有多个应用都打出相同的异常信息**：有时一些ANR问题是由共同的底层问题导致的。如下例，每次ANR前log中都会报出io异常，且多个应用都有相同现象。分析后发现问题出现在kernel中的mmc驱动
	
	    FileExplorer出现ANR前，log中报出大量的io异常：
		
		05-10 16:31:27.580  1875  1911 W System.err: Caused by: libcore.io.ErrnoException: read failed: EIO (I/O error)
		
		05-10 16:31:27.580  1875  1910 W System.err: java.io.IOException: read failed: EIO (I/O error)
		
		Gallery出现ANR前，log中也报出的了同样的io异常：
		
		05-10 16:31:45.620  1627  1627 W System.err: java.io.IOException: read failed: EIO (I/O error)
		
		05-10 16:31:45.620  1627  1627 W System.err:         at libcore.io.IoBridge.read(IoBridge.java:450)
		
		结合Kernel.log分析发现，是mmc驱动出现错误造成I/O异常，最终导致ANR。
		
		<3>[ 1208.760352] c3 mmcblk1: error -110 sending status command, aborting
		
		<4>[ 1208.760385] c3 mmc1: mmc_blk_reset return EEXIST
		
		<3>[ 1208.761006] c0 mmc1: !!!!! error in sending cmd:18, int:0x10000, err:-110
		
		<3>[ 1208.761023] c0 sdhci: =========== REGISTER DUMP (mmc1)===========
		
		<3>[ 1208.761036] c0 sdhci: 0x80a52fc0 | 0x00087200 | 0x00397a46 | 0x123a0033
		
		<3>[ 1208.761048] c0 sdhci: 0x00000000 | 0x00000000 | 0x00000000 | 0x00000000
		
		<3>[ 1208.761048] c0 sdhci: 0x00000000 | 0x01f00207 | 0x00000d00 | 0x080eef07

- **GC时间是否过长**：如果GC花费的时间占据了ANR超时时间的一半以上，就需要考虑是由于系统内存极端不足或内存碎片化导致ANR。注意计算GC时间时仅能计算超时时间段内ANR应用主线程GC时间，再加上WAIT_FOR_CONCURRENT_GC的时间。如下例，仅计算发生ANR的进程596的GC时间，和WAIT_FOR_CONCURRENT_GC的时间，一共1014+838+387+252ms，不能把非ANR应用23642的GC时间320ms也算进去

	    01-05 19:27:41.965 23642 23642 D dalvikvm: GC_FOR_ALLOC freed 391K, 20% free 9719K/12012K, paused 32ms, total 320ms
	
		01-05 19:27:43.725   596   600 D dalvikvm: GC_CONCURRENT freed 3772K, 43% free 12607K/22044K, paused 44ms+25ms, total 1014ms
		
		01-05 19:27:43.725   596   637 D dalvikvm: WAIT_FOR_CONCURRENT_GC blocked 838ms
		
		01-05 19:27:43.725   596   612 D dalvikvm: WAIT_FOR_CONCURRENT_GC blocked 387ms

##调用堆栈中的信息##
**AMS会抓取ANR发生瞬间应用程序全部线程的调用堆栈，供开发人员分析阻塞位置**

很多人看调用堆栈时都会首先去找**“held by”（死锁发生的关键词）**，看看是不是死锁。但ANR并不一定由死锁造成，如何从千奇百怪的堆栈信息中判断ANR的原因呢，主要应注意以下几个方面:

###线程状态###
线程状态：作为运行在Linux上的系统，Android对标准POSIX本地线程状态进行了进一步的封装，使之可以更准确地描述虚拟机内部的状态。参见下表： 
<table>
	<tr>
		<th>
			线程状态
		</th>
		<th>
			含义
		</th>
	</tr>
	<tr>
		<th>
			SUSPENDED
		</th>
		<th>
			线程暂停，可能是由于输出Trace、GC或debug被暂停
		</th>
	</tr>
	<tr>
		<th>
			NATIVE
		</th>
		<th>
			正在执行JNI本地函数
		</th>
	</tr>
	<tr>
		<th>
			MONITOR
		</th>
		<th>
			线程阻塞，等待获取对象锁
		</th>
	</tr>
	<tr>
		<th>
			WAIT
		</th>
		<th>
			执行了无线等待的wait函数
		</th>
	</tr>
	<tr>
		<th>
			TIMED_WAIT
		</th>
		<th>
			执行了带有超时参数的wait、sleep或join函数
		</th>
	</tr>
	<tr>
		<th>
			VMWAIT
		</th>
		<th>
			正在等待VM资源
		</th>
	</tr>
	<tr>
		<th>
			RUNNING/RUNNABLE
		</th>
		<th>
			线程可运行或正在运行
		</th>
	</tr>
	<tr>
		<th>
			INITALIZING
		</th>
		<th>
			新建，正在初始化，为其分配资源
		</th>
	</tr>
	<tr>
		<th>
			STARTING
		</th>
		<th>
			新建，正在启动
		</th>
	</tr>
	<tr>
		<th>
			ZOMBIE
		</th>
		<th>
			线程死亡，终止运行，等待父线程回收它
		</th>
	</tr>
	<tr>
		<th>
			UNKNOWN
		</th>
		<th>
			未知状态
		</th>
	</tr>
</table>

**SUSPENDED状态的含义是线程暂停，但是绝不能看到应用程序主线程处于SUSPENDED，就说是系统/Dalvik/CPU/内核/底层将应用主线程暂停导致ANR。必须依据调用堆栈，具体问题具体分析**

线程处于SUSPENDED有三种原因：

- 正在运行的线程需要输出调用堆栈信息
- 正在GC
- 或者被调试程序暂停

当输出一个线程的调用堆栈时，如果线程正处于NATIVE、MONITOR、WAIT、TIMED_WAIT、VMWAIT这样的安全状态时、可以直接输出线程的调用堆栈。这是由于处于上述状态的应用并没有在运行，而是在等待某个锁或者远程调用返回。而处于RUNNING/RUNNABLE状态的线程处于一种不安全的状态，必须先将其挂起，转为SUSPENDED才能输出调用堆栈。这就好像不能给一辆行驶中的卡车换轮胎，必须先把卡车停下来

如果线程是由于正在GC而处于SUSPENDED状态，那么线程的调用堆栈中就一定能显式地看到GC相关的方法，如下面例子中的黑体字。系统内存紧张或内存碎片化时，GC速度慢确实会导致ANR，但主线程直接阻塞在GC上比较少见。在实际操作中应当结合main.log中dalvik打出的GC时间信息分析，如果发生ANR的应用在超时时间段内GC花费的时间之和超过2.5秒，就需要怀疑是GC太慢导致ANR

	    "main" prio=5 tid=1 SUSPENDED
	
	  | group="main" sCount=1 dsCount=0 obj=0x41c71ca8 self=0x41c603c8
	
	  | sysTid=29123 nice=-6 sched=0/0 cgrp=apps handle=1074614612
	
	  | state=S schedstat=( 0 0 0 ) utm=10789 stm=4384 core=0
	
	  #00  pc 00021924  /system/lib/libc.so (__futex_syscall3+8)
	
	  #01  pc 0000ef74  /system/lib/libc.so (__pthread_cond_timedwait_relative+48)
	
	  #02  pc 0000efd4  /system/lib/libc.so (__pthread_cond_timedwait+64)
	
	  #03  pc 000535db  /system/lib/libdvm.so
	
	  #04  pc 00054c9b  /system/lib/libdvm.so
	
	  #05  pc 0005516b  /system/lib/libdvm.so (dvmSuspendAllThreads(SuspendCause)+18)
	
	  #06  pc 000297f8  /system/lib/libdvm.so (dvmCollectGarbageInternal(GcSpec const*)+1376)
	
	  #07  pc 00055bf3  /system/lib/libdvm.so (dvmCollectGarbage()+30)
	
	  #08  pc 00026fe0  /system/lib/libdvm.so
	
	......
	
	  #21  pc 0000e3fb  /system/lib/libc.so (__libc_init+50)
	
	  #22  pc 00000d7c  /system/bin/app_process
	
	  at java.lang.Runtime.gc(Native Method)
	
	  at android.os.StrictMode.decrementExpectedActivityCount(StrictMode.java:2008)
	
	  at android.app.ActivityThread.performDestroyActivity(ActivityThread.java:3504)
	
	......
	
	  at com.android.internal.os.ZygoteInit.main(ZygoteInit.java:595)
	
	  at dalvik.system.NativeStart.main(Native Method)

此外，调试程序时设置断点也会使线程处于SUSPENDED状态，这时堆栈信息会显示dsCount=1。但调试程序时不会报告ANR，对此种情况不再赘述

###调用堆栈状态###
由应用原因引发’ANR’的原因通常可分为四大类：死锁、阻塞、死循环、低性能

- 发生死锁时的调用堆栈 

见下例，inputmethod主线程中的getCurrentInputMethodSubtype方法进行了一次远程调用，跨进程调到了system_server的Binder_F中。此处发生了死锁，Binder_F等待main，main等待WindowManager 
，WindowManager 又等待Binder_F。因此inputmethod主线程的远程调用无法返回，导致ANR

	    md line: com.android.inputmethod.latin
	
	"main" prio=5 tid=1 NATIVE
	
	  | group="main" sCount=1 dsCount=0 obj=0x415e1ca8 self=0x4151b408
	
	  | sysTid=22186 nice=0 sched=0/0 cgrp=apps handle=1074639188
	
	  #00  pc 000205a8  /system/lib/libc.so (__ioctl+8)
	
	  #01  pc 0002cf17  /system/lib/libc.so (ioctl+14)
	
	......
	
	  at android.os.BinderProxy.transact(Native Method)
	
	  at com.android.internal.view.IInputMethodManager$Stub$Proxy.getCurrentInputMethodSubtype(IInputMethodManager.java:908) ......
	
	
	
	Cmd line: system_server
	
	"main" prio=5 tid=1 MONITOR
	
	  | group="main" sCount=1 dsCount=0 obj=0x415e1ca8 self=0x4151b408
	
	  | sysTid=611 nice=-2 sched=0/0 cgrp=apps handle=1074639188
	
	  at android.view.WindowManagerGlobal.addView(WindowManagerGlobal.java:~209)
	
	  - waiting to lock <0x41c5a650> (a java.lang.Object) held by tid=12 (WindowManager)......
	
	
	
	"WindowManager" prio=5 tid=12 MONITOR
	
	  | group="main" sCount=1 dsCount=0 obj=0x41c11680 self=0x53921218
	
	  | sysTid=625 nice=-4 sched=0/0 cgrp=apps handle=1402082928
	
	  at android.view.inputmethod.InputMethodManager.windowDismissed(InputMethodManager.java:~1215)
	
	  - waiting to lock <0x41c0c118> (a android.view.inputmethod.InputMethodManager$H) held by tid=61 (Binder_F)......
	
	
	
	"Binder_F" prio=5 tid=61 MONITOR
	
	  | group="main" sCount=1 dsCount=0 obj=0x41ebfc40 self=0x567f47d8
	
	  | sysTid=1144 nice=0 sched=0/0 cgrp=apps handle=1451181400
	
	  at com.android.server.InputMethodManagerService.getCurrentInputMethodSubtype(InputMethodManagerService.java:~3078)

- 执行Binder调用时的调用堆栈

在上面例子中发生ANR的应用主线程正在进行Binder调用，这样的调用堆栈在ANR的snapshot中是很常见的。可以看到下面用黑体字标出的Binder调用客户端和服务端关键log，注意此时主线程状态是NATIVE，说明线程正在执行JNI本地函数。当发现ANR应用正在进行Binder调用时，一定不能草率地作出“Binder机制出错因此没返回”，“底层没返回所以ANR”的结论。应当首先寻找服务端堆栈，检查服务端是否出现死锁、被IO阻塞或其他异常 

	    Cmd line: com.android.sprdlauncher2
	
	"main" prio=5 tid=1NATIVE
	
	  | group="main" sCount=1 dsCount=0 obj=0x415b8cc0 self=0x414d2428
	
	  | sysTid=833 nice=-1 sched=0/0 cgrp=[no-cpu-subsys] handle=1074315604
	
	  #00  pc 00020588  /system/lib/libc.so (__ioctl+8)
	
	  #01  pc 0002cef7  /system/lib/libc.so (ioctl+14)
	
	  #02  pc 0001ea9d  /system/lib/libbinder.so (android::IPCThreadState::talkWithDriver(bool)+140)
	
	  #03  pc 0001efd7  /system/lib/libbinder.so (android::IPCThreadState::waitForResponse(android::Parcel*, int*)+42)
	
	  #04  pc 0001f32d  /system/lib/libbinder.so (android::IPCThreadState::transact(int, unsigned int, android::Parcel const&, android::Parcel*, unsigned int)+204)
	
	  #05  pc 0001ae41  /system/lib/libbinder.so (android::BpBinder::transact(unsigned int, android::Parcel const&, android::Parcel*, unsigned int)+30)
	
	......
	
	  #21  pc 00001423  /system/bin/app_process
	
	  #22  pc 0000e403  /system/lib/libc.so (__libc_init+50)
	
	  #23  pc 00000f34  /system/bin/app_process
	
	  at android.os.'BinderProxy.transact(Native Method)<---'客户端
	
	  at android.app.ISearchManager$Stub$Proxy.getGlobalSearchActivity(ISearchManager.java:215)
	
	  at android.app.SearchManager.getGlobalSearchActivity(SearchManager.java:595)
	
	......
	
	
	
	Cmd line: system_server
	
	"Binder_C" prio=5 tid=59 MONITOR
	
	  | group="main" sCount=1 dsCount=0 obj=0x4232f5b8 self=0x51d4f210
	
	  | sysTid=17000 nice=-1 sched=0/0 cgrp=[no-cpu-subsys] handle=1439133664
	
	  at com.android.server.search.SearchManagerService.getSearchables(SearchManagerService.java:~91)
	
	 - waiting to lock <0x41fd4098> (a android.util.SparseArray) held by tid=14 (android.bg)
	
	  at com.android.server.search.SearchManagerService.getGlobalSearchActivity(SearchManagerService.java:230)
	
	  at android.app.ISearchManager$Stub.onTransact(ISearchManager.java:86)'<---'服务端
	
	  at android.os.Binder.execTransact(Binder.java:427)
	
	  at dalvik.system.NativeStart.run(Native Method)

- 主线程被上锁的调用堆栈 

有极少数应用如Gallery3D和Camera会给自己的主线程上一个无限等待的锁，在子线程完成特定操作后由子线程解锁主线程。理论上讲子线程应当很快解锁，但在系统负载高的时候很容易导致ANR。 

这种情况下snapshot中可见主线程调用堆栈处于Wait状态，主线程held by 
tid=1自己等待自己。解决这个问题需要在子线程中添加log，检查解锁不及时的原因。 
需注意仅有主线程给自己上无限等待锁才会导致ANR，子线程这样做是常见操作，不会导致ANR

    md line: com.android.gallery3d
	
	"main" prio=5 tid=1 WAIT
	
	  | group="main" sCount=1 dsCount=0 obj=0x41619cc0 self=0x415315c8
	
	  | sysTid=25931 nice=-1 sched=0/0 cgrp=apps handle=1074602324
	
	  | state=S schedstat=( 0 0 0 ) utm=182 stm=254 core=0
	
	  at java.lang.Object.wait(Native Method)
	
	 - waiting on <0x41619d90> (a java.lang.VMThread) held by tid=1 (main)
	
	  at java.lang.Thread.parkFor(Thread.java:1205)
	
	  at sun.misc.Unsafe.park(Unsafe.java:325)
	
	  at java.util.concurrent.locks.LockSupport.park(LockSupport.java:157)
	
	  at java.util.concurrent.locks.AbstractQueuedSynchronizer.parkAndCheckInterrupt(AbstractQueuedSynchronizer.java:813)
	
	  at java.util.concurrent.locks.AbstractQueuedSynchronizer.acquireQueued(AbstractQueuedSynchronizer.java:846)
	
	  at java.util.concurrent.locks.AbstractQueuedSynchronizer.acquire(AbstractQueuedSynchronizer.java:1175)
	
	  at java.util.concurrent.locks.ReentrantLock$NonfairSync.lock(ReentrantLock.java:180)
	
	  at java.util.concurrent.locks.ReentrantLock.lock(ReentrantLock.java:256)
	
	  at com.android.gallery3d.ui.GLRootView.dispatchTouchEvent(GLRootView.java:475)
	
	  at android.view.ViewGroup.dispatchTransformedTouchEvent(ViewGroup.java:2216)
	
	  at android.view.ViewGroup.dispatchTouchEvent(ViewGroup.java:1959)
	
	  at android.view.ViewGroup.dispatchTransformedTouchEvent(ViewGroup.java:2216)
	
	  at android.view.ViewGroup.dispatchTouchEvent(ViewGroup.java:1959)
	
	  at android.view.ViewGroup.dispatchTransformedTouchEvent(ViewGroup.java:2216)

- 主线程阻塞的调用堆栈 

除了各种死锁之外，阻塞也是导致ANR的一个重要原因。如下例，应用程序主线程正在进行IO操作，获取SD卡剩余空间但是向JNI层的调用却没有返回。如果出现这样的调用堆栈，且CPU信息中显示IOWait非常高，就要考虑是由I/O读写速度慢导致的ANR。此例中结合Kernel.log发现是由mmc驱动错误影响I/O速度阻塞主线程导致ANR
	
	"main" prio=5 tid=1 NATIVE
	
	  | group="main" sCount=1 dsCount=0 obj=0x41571cc0 self=0x41489418
	
	  | sysTid=10712 nice=-1 sched=0/0 cgrp=apps handle=1074012500
	
	  | state=S schedstat=( 0 0 0 ) utm=47 stm=199 core=0
	
	  (native backtrace unavailable)
	
	  at libcore.io.Posix.statvfs(Native Method)
	
	  at libcore.io.ForwardingOs.statvfs(ForwardingOs.java:132)
	
	  at android.os.StatFs.doStat(StatFs.java:44)
	
	  at android.os.StatFs.<init>(StatFs.java:39)
	
	  at com.android.camera.StorageUtil.getAvailableSpace(StorageUtil.java:144)
	
	  at com.android.camera.CameraActivity.updateStorageSpace(CameraActivity.java:1539)
	
	  at com.android.camera.PhotoModule.initializeFirstTime(PhotoModule.java:643)
	
	......
	
	  at android.os.Handler.dispatchMessage(Handler.java:102)
	
	  at android.os.Looper.loop(Looper.java:136)
	
	  at android.app.ActivityThread.main(ActivityThread.java:5281)
	
	  at java.lang.reflect.Method.invokeNative(Native Method)
	
	  at java.lang.reflect.Method.invoke(Method.java:515)
	
	  at com.android.internal.os.ZygoteInit$MethodAndArgsCaller.run(ZygoteInit.java:881)
	
	  at com.android.internal.os.ZygoteInit.main(ZygoteInit.java:697)
	
	  at dalvik.system.NativeStart.main(Native Method)
	
	
	
	Kernel.log
	
	01-01 10:02:28.997 <3>[ 7092.548578] c1 sdhci: =========== REGISTER DUMP (mmc0)===========
	
	01-01 10:02:28.997 <3>[ 7092.548598] c0 sdhci: 0x00000000 | 0x00000000 | 0x00000000 | 0x00000000
	
	01-01 10:02:28.997 <3>[ 7092.548598] c0
	
	01-01 10:02:28.997 <3>[ 7092.548608] c0 sdhci: 0x00000000 | 0x00000000 | 0x00000000 | 0x00000000
	
	01-01 10:02:28.997 <3>[ 7092.548638] c0
	
	01-01 10:02:28.997 <3>[ 7092.548654] c0 sdhci: 0x00000000 | 0x00000000 | 0x00000000 | 0x00000000
	
	01-01 10:02:28.997 <3>[ 7092.548658] c0
	
	01-01 10:02:28.997 <3>[ 7092.548660] c1 sdhci: ===========================================

- 出现死循环的调用堆栈 

如果在程序设计中对输入参数合法性检查不严格，代码（特别是字符串拆分操作）可能会陷入死循环。这类问题的特征在于应用程序用户空间的CPU占用率很高，往往接近100%。调用堆栈处于SUSPENDED状态（这说明在开始输出堆栈前它一直处于RUNNING）。此时就需要参照调用堆栈分析代码，在怀疑发生死循环的位置添加debug信息检查输入参数合法性，待重现后找到死循环的原因再加以修改

	"ANR in com.android.mms (com.android.mms/.ui.SearchActivity)
	
	PID: 28898
	
	Reason: Input dispatching timed out (Waiting because the focused window has not finished processing the input events that were previously delivered to it.)
	
	Load: 9.99 / 9.26 / 8.96
	
	CPU usage from 3013ms to -7584ms ago:
	
	  77% 28898/com.android.mms: 75% user + 1.5% kernel / faults: 3926 minor 65 major
	
	
	
	"main" prio=5 tid=1 SUSPENDED
	
	  | group="main" sCount=1 dsCount=0 obj=0x415cfcc0 self=0x414e7418
	
	  | sysTid=30269 nice=-1 sched=0/0 cgrp=apps handle=1074397524
	
	  | state=S schedstat=( 0 0 0 ) utm=1077 stm=130 core=3
	
	  at android.text.SpannableStringInternal.getSpans(SpannableStringInternal.java:~215)
	
	  at android.text.SpannableString.getSpans(SpannableString.java:25)
	
	  at android.text.SpannableStringInternal.sendSpanAdded(SpannableStringInternal.java:308)
	
	  at android.text.SpannableStringInternal.setSpan(SpannableStringInternal.java:136)
	
	  at android.text.SpannableString.setSpan(SpannableString.java:46)
	
	  at com.android.mms.ui.SearchActivity$TextViewSnippet.onLayout(SearchActivity.java:212)
	
	  at android.view.View.layout(View.java:14825)
	
	  at android.widget.LinearLayout.setChildFrame(LinearLayout.java:1671)
	
	  at android.widget.LinearLayout.layoutVertical(LinearLayout.java:1525)
	
	  at android.widget.LinearLayout.onLayout(LinearLayout.java:1434)
	
	......
	
	  at dalvik.system.NativeStart.main(Native Method)
	
	
	
	while (m.find(start)) {
	
	    spannable.setSpan(new StyleSpan(sTypefaceHighlight), m.start(), m.end(), 0);
	
	    spannable.setSpan(new ForegroundColorSpan(mTextColorForSearch), m.start(), m.end(), 0);
	
	    start = m.end();
	
	}

- 低性能的调用堆栈 

就以往项目的经验而言，前期由应用死锁、阻塞导致的ANR较多，项目中后期问题主要集中在性能上。由于手机内存、I/O性能存在瓶颈，或是程序算法或流程设计不合理等原因，在压力测试中会出现很多由性能问题导致的ANR。当发生这类问题时，程序并不会一直阻塞在一个特定位置，而是非常缓慢地前行。主线程看上去和阻塞很相似，但通常会停在一个被频繁调用的原生公共模块，通常是窗口绘制或布局相关的方法上。 

低性能问题通常比较难以判断，应当主要关注以下几个特征：

- meminfo中剩余内存在20M以下
- system.log中发生ANR的应用Kernel空间CPU占用率非常高
- system.log中的主要页错误超过600次/秒
- main.log中发生ANR的应用的GC速度大于200ms
- event.log中创建应用进程所需时间超过1秒，窗口生命周期个方法执行迟缓
- kernel.log中ANR时间点附近频繁调用LowMemoryKiller杀死进程
- kernel.log中有内存碎片log
- 一份log中多个模块反复出现ANR，但是出现问题时的堆栈各不相同

分析性能问题应注意避免几种错误做法：

- 只看调用堆栈：程序缓慢运行时抓取的调用堆栈经常会“恰好”停在一个被频繁调用调用的公共模块上，有点像玩击鼓传花，停在哪行代码上并不意味这行代码阻塞了整个程序的运行。不能看到调用堆栈在进行窗口布局就说程序卡在WMS里，看到调用堆栈正在绘制字体就说程序卡在libskia库里面。阻塞只是导致ANR的原因之一，而不是全部
- 将一切问题都归于系统性能：对于报出的ANR问题，应当先检查是否有死锁、阻塞，应用本身的算法和流程有没有问题。不要一看到LMK就说内存不足，一看到log中ANR较多就说是系统性能差。应当从上到下从局部到整体依次排除，在确认上层没有任何可怀疑的点后，再考虑是系统性能问题

下面是一个真实的例子，在android系统的原生代码中很多输入对话框并没有限制输入字符数，于是测试在wifi名称输入框中拷入了40万字，并反复旋转屏幕使控件重新计算布局，直到发生ANR。调用堆栈中可以看出ANR正在进行绘制字体的操作。显然字体库代码并不存在阻塞，而是由于截取堆栈时刚好运行到了这里

谢天谢地后来PM叫停了这种极(chun)限(cui)场(you)景(bing)的测试。不然要给每一个原生对话框添加字符串限制工程量实在是太大了

- NativePollOnce调用堆栈 

有时trace文件会输出一个NativePollOnce调用堆栈，这是一个正常的堆栈，当应用程序处于空闲状态消息循环等待下一个输入事件时就会显示这样的堆栈。绝对不是“主线程阻塞在消息循环上”，更不是“阻塞在Framework里”

###Snapshot中的信息###
**根据不同项目的定制情况不同，snapshot还能输出很多ANR发生时的系统状态信息，如纵内存信息、连续内存段数量、线程信息、文件句柄和磁盘使用量、binder状态、wakelock分配情况等等。其中和分析ANR相关的主要是内存信息和线程信息这两部分**

- **内存信息** 

内存信息是指用cat /proc/meminfo指令读出的内核信息，仅需要关注前4项。 
MemTotal: 403784 kB 所有可用内存大小，即物理内存减去内核占用、预留内存和显存 
MemFree: 3068 kB 完全空闲的内存 
Buffers: 3140 kB 用于文件缓存的内存 
Cached: 43772 kB 被高速缓冲存储器使用的内存大小 
在计算剩余内存时应该用MemFree+Cached，根据经验值，如果两者之和小于20M，就是说明系统处于极低内存状态，应用很可能出现由低内存导致的ANR

- **线程信息**

ANR发生时AMS会通过ps -t命令输出线程的状态信息，需要注意分析进程是否启动了数量异常的子线程，比如Launcher和Gallery3D出现过启动了500+子线程的例子；发生ANR时各个应用的内存使用量；是否启动了一些异常的进程，比如同时启动5个Monkey进程一起跑

	u0_a54    1195  154   847220 16560 ffffffff 400b5764 S com.android.launcher3
	
	u0_a54    1200  1195  847220 16560 c008d384 400b5930 S GC
	
	u0_a54    1202  1195  847220 16560 c0052964 400b5198 S Signal Catcher
	
	u0_a54    1204  1195  847220 16560 c0568ec0 400b5404 S JDWP
	
	u0_a54    1206  1195  847220 16560 c008d384 400b5930 S Compiler
	
	u0_a54    1207  1195  847220 16560 c008d384 400b5930 S ReferenceQueueD
	
	u0_a54    1208  1195  847220 16560 c008d384 400b5930 S FinalizerDaemon
	
	u0_a54    1210  1195  847220 16560 c008d384 400b5930 S FinalizerWatchd
	
	u0_a54    1212  1195  847220 16560 c0442574 400b45b4 S Binder_1
	
	u0_a54    1213  1195  847220 16560 c0442574 400b45b4 S Binder_2
	
	u0_a54    1238  1195  847220 16560 c01681ac 400b5764 S launcher-loader
	
	u0_a54    1239  1195  847220 16560 c008d384 400b5930 S AsyncTask #1
	
	u0_a54    1276  1195  847220 16560 c0442574 400b45b4 S Binder_3
	
	u0_a54    1872  1195  847220 16560 c008d384 400b5930 S droid.launcher3
	
	u0_a54    1873  1195  847220 16560 c008d384 400b5930 S droid.launcher3
	
	u0_a54    1874  1195  847220 16560 c0568ec0 400b5348 S droid.launcher3

此外还应注意线程的运行状态，其中S、R都是PS中常见的正常线程状态。需要特别注意的是D状态，在D状态说明进程处于不可中断的睡眠状态，此时它不会响应任何外部信号，甚至无法用Kill杀死进程。

处于此状态的线程通常是在等待I/O，比如磁盘I/O、网络I/O或者外设I/O。线程短时间处于D状态是正常现象，但是在I/O瓶颈较严重的手机上如果应用连续几秒或者更长时间都处于D状态无法响应外部信号，就会导致ANR
<table>
	<tr>
		<th>
			线程状态
		</th>
		<th>
			含义
		</th>
	</tr>
	<tr>
		<th>
			S
		</th>
		<th>
			可中断的睡眠
		</th>
	</tr>
	<tr>
		<th>
			R
		</th>
		<th>
			运行或就绪状态
		</th>
	</tr>
	<tr>
		<th>
			D
		</th>
		<th>
			不可中断的睡眠
		</th>
	</tr>
	<tr>
		<th>
			Z
		</th>
		<th>
			退出状态，作为僵尸进程等待回收
		</th>
	</tr>
	<tr>
		<th>
			X
		</th>
		<th>
			退出状态，即将被回收
		</th>
	</tr>
	<tr>
		<th>
			T
		</th>
		<th>
			暂停或跟踪状态
		</th>
	</tr>
</table>

###Event.log中的信息###
Event.log中主要是ActivityManager和WindowManager打出的信息。在分析由性能问题导致的ANR时，应用程序可能并没有死锁或者阻塞，而是在OnCreate中浪费4秒，留给OnResume执行的时间已经不够了。分析这类问题时就不能简单地看应用程序主线程堆栈停在哪里，而是要分析窗口生命周期各个方法的执行时间，找到运行迟缓的部分。Event.log中需要关注的信息主要有：
<table>
	<tr>
		<th>
			Log关键字
		</th>
		<th>
			含义
		</th>
	</tr>
	<tr>
		<th>
			am_proc_start
		</th>
		<th>
			开始创建应用进程
		</th>
	</tr>
	<tr>
		<th>
			am_proc_bound
		</th>
		<th>
			应用进程创建完毕
		</th>
	</tr>
	<tr>
		<th>
			am_restart_activity
		</th>
		<th>
			realActivityStart，创建进程完成后首次启动应用
		</th>
	</tr>
	<tr>
		<th>
			am_resume_activity
		</th>
		<th>
			窗口Resume开始
		</th>
	</tr>
	<tr>
		<th>
			am_on_resume_called
		</th>
		<th>
			窗口Resume完毕
		</th>
	</tr>
	<tr>
		<th>
			am_pause_activity
		</th>
		<th>
			窗口Pause开始
		</th>
	</tr>
	<tr>
		<th>
			am_on_pause_called
		</th>
		<th>
			窗口Pause完毕
		</th>
	</tr>
	<tr>
		<th>
			am_failed_to_pause
		</th>
		<th>
			窗口Pause超时
		</th>
	</tr>
	<tr>
		<th>
			am_finish_activity
		</th>
		<th>
			应用Finish开始
		</th>
	</tr>
	<tr>
		<th>
			am_proc_died
		</th>
		<th>
			进程死亡（比如被LowMemoryKiller杀死）
		</th>
	</tr>
</table>
在掌握了以上窗口生命周期log后，就可以从event.log中分析究竟是哪一个过程慢导致ANR发生，见下例：

- **创建进程慢**：正常情况下启动应用创建进程所需的时间应当是300～500ms，在系统内存碎片化分配不出连续内存段或者CPU变频不正常时进程创建速度就会明显变慢，列如：log中am_proc_start到am_proc_bound一共花费了4.5秒才创建出一个进程。这就可以排除应用问题，转而查找底层原因
- **窗口创建缓慢**：log中可见，进程启动速度正常，但是用了6秒才完成Resume走到am_on_resume_called。如果单一应用反复出现此现象，就需要在应用的声明周期方法中分段添加log查找执行缓慢的代码；如果多个应用随机出现此现象，就需要对系统整体性能进行分析，查找阻塞点

		07-12 12:39:53.830   592   608 I am_proc_start: [0,12735,10008,com.android.gallery3d,activity,com.android.gallery3d/.app.GalleryActivity]
		
		07-12 12:39:54.340   592  1322 I am_proc_bound: [0,12735,com.android.gallery3d]
		
		07-12 12:39:55.060   592  1322 I am_restart_activity: [0,1110839416,99,com.android.gallery3d/.app.GalleryActivity]
		
		07-12 12:40:01.470 12735 12735 I am_on_resume_called: [0,com.android.gallery3d.app.GalleryActivity]
		
		07-12 12:40:02.070   592   608 I am_anr  : [0,12735,com.android.gallery3d,14203973,Input dispatching timed out (Waiting because no window has focus but there is a focused application that may eventually add a window when it finishes starting up.)]

- **窗口Pause慢**：log中可见，SlideshowEditActivity在05:14:57就已经收到AMS的am_pause_activity命令，但直到05:15:08才完成am_on_paused_called走完Pause流程。对于单一应用而言，前一个窗口Pause不下去新窗口就没法Resume出来，焦点长时间处于null状态就会触发窗口焦点转换ANR。这个例子需要应用检查窗口Pause慢的原因

		06-09 05:14:57.913   628 25582 I am_pause_activity: [0,1115385600,com.android.mms/.ui.SlideshowEditActivity]
		
		06-09 05:14:58.413   628   644 I am_restart_activity: [0,1116031944,213,com.android.mms/.ui.SlideEditorActivity]
		
		06-09 05:15:08.933  2946  2946 I am_on_paused_called: [0,com.android.mms.ui.SlideshowEditActivity]
		
		06-09 05:15:08.933   628 25580 I am_failed_to_pause: [0,1115385600,com.android.mms/.ui.SlideshowEditActivity,(none)]
		
		06-09 05:15:09.193   628   644 I am_anr  : [0,2946,com.android.mms,1082703461,Input dispatching timed out (Waiting because no window has focus but there is a focused application that may eventually add a window when it finishes starting up.)]]

###Kernel log###
Kernel.log主要包含HAL层和Kernel层的信息，对于平时惯于分析system.log和mian.log的程序员而言，其中的信息会比较难于理解。其实在分析ANR过程中，大多数时候仅需要注意以下几个基本点即可：

- **LowMemoryKiller**：Android系统的内存管理原则是，允许启动尽可能多的应用，当内存不足时再由Kernel中的LowMemoryKiller根据特定算法杀死后台应用，为前台应用释放内存

小内存设备上由LowMemoryKiller导致的ANR通常有两种，一种是应用刚刚收到一个广播消息就被LMK杀死，消息无人处理导致广播超时发生ANR。因此分析广播超时ANR时需要注意在超时时间段内应用是否被LMK杀死。针对此问题可以修改AMS，当报出广播超时ANR前首先检查应用是否已经被杀死，如果应用已死就不再报出ANR

- **内存碎片或内存耗尽**： 当小内存设备高强度运行数个小时之后，内存会逐渐碎片化，较大的连续内存段越来越少，剩下的都是4kB、16kB的零碎内存段。这时如果应用程序需要分配一个32kB的连续内存段，Kernel就只能尝试调用LMK杀死一些后台进程来释放内存。如果释放内存花费时间过长就会导致等待内存分配的应用发生ANR。如下例：sh尝试分配order=2即4Kb*2^2=16kB的连续内存段，但此时内存已经碎片化，只有4kB和8kB的内存可用了。如果在ANR发生前后Kernel.log中出现了大量“invoked oom-killer”信息，就要考虑调整LMK回收策略更积极地释放内存，以减少由于系统性能问题导致ANR的概率
		
		09-22 15:36:44.117 <4>[ 1304.482941] c2 sh invoked oom-killer: gfp_mask=0xd0, order=2, oom_adj=-16, oom_score_adj=-941
		
		09-22 15:36:44.117 <4>[ 1304.483533] c2 lowmem_reserve[]: 0 0 0
		
		09-22 15:36:44.117 <4>[ 1304.483544] c2 Normal: 737*4kB 232*8kB 0*16kB 0*32kB 0*64kB 0*128kB 0*256kB 0*512kB 0*1024kB 0*2048kB 0*4096kB = 4804kB

- **一些特殊异常信息**

就像上层应用和服务会发生Java Crash和Native Crash，Kernel中同样会出现各式各样的崩溃和异常。在分析ANR时需要注意在超时时间段内Kernel中的log有没有明显的异常信息。需要指出的是，查找由Kernel引发的ANR需要较多的实践经验，千万不能在kernel.log中看到一段奇怪的log就认为它是ANR的原因，这样不仅不利于及时解决问题，还会给Kernel增加额外的工作量
