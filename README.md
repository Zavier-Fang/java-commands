## java内存分析常用命令

### 一、jps：虚拟机进程状况工具（常用）
> java process status（jps） 用于查看正在运行的java程序的状态

    # 用法
    $ jps [ options ] [ hostid ]

    jps -m  （常用）显示主函数输入的参  数
    jps -l  （常用）显示应用程序主类完  整/全限定包类名或jar完整名称
    jps -v  （常用）列出程序启动时的    jvm参数
    jps -V   输出通过.hotsportrc或  -XX:Flags=<filename>指定的jvm参数

### 二、jstat：虚拟机统计信息监视工具
> jstat是jvm statistics 的缩写

**用法**

    $ jstat [ option lvmid [ interval[s|ms] [count] ] ]

**类加载统计**

    $ jstat -class 7658
    Loaded  Bytes  Unloaded  Bytes     Time   
    10381 19456.3       80   117.8      6.7

* Loaded：已加载的class数量
* Bytes：已加载所占用空间大小
* Unloaded：未加载的class数量
* Bytes：未加载所占用空间大小
* Time：加载所用时间

**常用的统计命令**

    $ jstat -gcutil 7658 5s 2
     S0     S1     E      O      M     CCS    YGC     YGCT    FGC    FGCT     GCT   
    62.50   0.00  67.59  46.13  98.26  97.07  10662   36.514     3    0.286   36.800
    62.50   0.00  73.00  46.13  98.26  97.07  10662   36.514     3    0.286   36.800

* S0、S1：分别表示新生代的两个Survivor区
* E ：Eden区使用占比
* O：老年代使用占比
* M：元数据空间使用占比
* CCS：压缩使用占比
* YGC：年轻代垃圾回收次数
* FGC：老年代垃圾回收次数
* FGCT：老年代垃圾回收消耗时间
* GCT：垃圾回收消耗总时间

### 三、jmap：Java内存影像工具（非常重要）
**用法**

    $ jmap [option] <pid>
        (to connect to running process)
    $ jmap [option] <executable <core>
        (to connect to a core file)
    $ jmap [option] [server_id@]<remote server IP or hostname>
        (to connect to remote debug server)

**举例与说明**

    $ jmap -heap 7658
    Attaching to process ID 7658, please wait...
    Debugger attached successfully.
    Server compiler detected.
    JVM version is 25.171-b11   

    using thread-local object allocation.
    Parallel GC with 8 thread(s)    

    Heap Configuration:         # Heap堆配置信息
       MinHeapFreeRatio         = 0 # JVM堆最小空闲比率(default 40)
       MaxHeapFreeRatio         = 100  # JVM堆最大空闲比率(default 70)
       MaxHeapSize              = 536870912 (512.0MB) # 堆最大值，可以通过 -XX:MaxHeapSize 或 -Xmx设置
       NewSize                  = 44564480 (42.5MB) # 新生代大小
       MaxNewSize               = 178782208 (170.5MB) # 新生代的最大大小
       OldSize                  = 89653248 (85.5MB) # 老年代大小
       NewRatio                 = 2 # 新生代与老生代的大小比率 1:2
       SurvivorRatio            = 8 # 年轻代中Eden区与两个Survivor区的比值。注意Survivor区有两个。如：8，表示Eden：Survivor=8：2
       MetaspaceSize            = 21807104 (20.796875MB) # jdk1.7有永久代，jdk1.8更换成了Metaspace
       CompressedClassSpaceSize = 1073741824 (1024.0MB)  # class信息存放的空间大小
       MaxMetaspaceSize         = 17592186044415 MB # Metaspace是使用的直接内存，理论上可以无限大（内存+硬盘缓冲区）
       G1HeapRegionSize         = 0 (0.0MB) # 设置的 G1 区域的大小。值是 2 的幂，范围是 1 MB 到 32 MB 之间。目标是根据最小的 Java 堆大小划分出约 1024/2048 个区域。 

    Heap Usage: # 堆内存的使用情况
    PS Young Generation # 新生代
    Eden Space: # Eden空间
       capacity = 50331648 (48.0MB) # 容量
       used     = 37253928 (35.528114318847656MB) # 已使用容量
       free     = 13077720 (12.471885681152344MB) # 空闲容量
       74.01690483093262% used # 使用率
    From Space: # From Survivor 空间
       capacity = 524288 (0.5MB) # 容量
       used     = 327680 (0.3125MB) # 已使用容量
       free     = 196608 (0.1875MB) # 空闲容量
       62.5% used # 使用率
    To Space: # To Survivor 空间
       capacity = 524288 (0.5MB) # 容量
       used     = 0 (0.0MB) # 已使用容量
       free     = 524288 (0.5MB) # 空闲容量
       0.0% used # 使用率
    PS Old Generation # 老年代
       capacity = 125304832 (119.5MB) # 容量
       used     = 57833832 (55.154640197753906MB) # 已使用容量
       free     = 67471000 (64.3453598022461MB) # 空闲容量
       46.15451062573548% used # 使用率 

    28116 interned Strings occupying 3331608 bytes.

**当然也可以生成内存快照**

    $ jmap -dump:live,format=b,file=jmap-7658.hprof 7658
    Dumping heap to /opt/applications/xxx/data/jmap-7658.hprof ...
    Heap dump file created

### 四、jhat 堆分析工具
> 全称Java Heap Analyse Tool，Java堆分析工具。是对堆内存转储快照文件分析的一种命令工具，它会启动一个Web Server，然后通过浏览器就可以分析转储快照了。
如果你在使用 jdk11，也没安装其他版本的jdk，进而不能使用 jvisualvm 这个工具，现在你可以使用 jhat 来分析堆转储快照了。

    $ jhat -J-Xmx1024m jmap-7658.hprof
    Reading from jmap-7658.hprof...
    Dump file created Tue Nov 13 15:37:02 CST 2018
    Snapshot read, resolving...
    Resolving 655822 objects...
    Chasing references, expect 131 dots...................  ......................................................    ......................................................  ....
    Eliminating duplicate references......................  ......................................................    ......................................................  .
    Snapshot resolved.
    Started HTTP server on port 7000
    Server is ready.

上面 -Xmx1024m 参数可以控制启动应用的堆内存大小。

然后通过浏览器访问界面：http://111.111.108.78:7000/

### 五、jstack 堆栈跟踪工具（非常重要）
> jstack 是java虚拟机自带的一种堆栈跟踪工具。

**用法**

    $ jstack [-l] <pid>
    (to connect to running process)
    $ jstack -F [-m] [-l] <pid>
        (to connect to a hung process)
    $ jstack [-m] [-l] <executable> <core>
        (to connect to a core file)
    $ jstack [-m] [-l] [server_id@]<remote server IP or     hostname>
        (to connect to a remote debug server)

    # -F:强制dump线程信息 
    # -m:dump虚拟机栈和本地方法栈 
    # -l:长期监听，dump锁信息

### 六、linux系统命令top
> top 命令是Linux下常用的 性能分析工具，能够实时显示系统中各个进程的资源占用状况，类似于Windows的任务管理器。

    $ top -hv | -bcHiOSs -d secs -n max -u|U user -p pid(s) -o field -w [cols]
    # -d -n -p 都是非常常用的参数，分布表示 刷新时间间隔(单位秒)、刷新次数、指定的进程id

top 命令的几个值：

    $ top
    top - 17:32:26 up 207 days,  3:38,  2 users,  load  average: 0.02, 0.05, 0.05
    Tasks: 133 total,   1 running, 132 sleeping,   0    stopped,   0 zombie
    %Cpu(s):  0.5 us,  0.3 sy,  0.0 ni, 99.2 id,  0.0   wa,  0.0 hi,  0.0 si,  0.0 st
    KiB Mem : 16267428 total,   225800 free, 15516244   used,   525384 buff/cache
    KiB Swap:        0 total,        0 free,        0   used.   414824 avail Mem 

      PID USER      PR  NI    VIRT    RES    SHR S  %CPU    %MEM     TIME+     COMMAND                                                       
     9504 root      20   0 7779792 1.458g    516 S   4.0    9.4   1851:46 java

最上面展示了日期、任务、CPU、内存、交换区等使用情况。     
* PID：进程id
* USER：当前用户
* PR：进程优先级
* NI：nice值，负值表示优先级高，正值表示优先级低
* VIRT：进程使用的虚拟内存，单位kb。VIRT=SWAP+RES
* RES： 进程使用的、未被换出的物理内存大小，单位kb。RES=CODE+DATA
* SHR：共享内存大小，单位kb
* S：进程状态。D：不可中断的睡眠状态；R：运行； S：睡眠 ；T：跟踪/停止；Z：僵尸进程
* %CPU：CPU使用百分比（注：一个CPU最大百分比是100%，两个CPU最大百分比是200%，以此类推）
* %MEM：内存使用百分比
* TIME+：进程累计使用CPU的时间，单位1/100s
* COMMAND：进程名称

### 七、linux系统命令vmstat
> vmstat是Virtual Meomory Statistics（虚拟内存统计）的缩写，可对操作系统的虚拟内存、进程、IO、系统、CPU活动进行监控。

命令说明：

![](https://upload-images.jianshu.io/upload_images/2843224-0812d6b8c197c320.png?imageMogr2/auto-orient/strip|imageView2/2/w/868/format/webp)

该命令默认显示监控5部分：procs、memory、swap、io、system、cpu信息。具体列信息说明如下：

#### Procs（进程）
* r: 等待运行的进程数
* b: 处在非中断睡眠状态的进程数

#### Memory（内存） 单位：KB
* swpd: 虚拟内存使用大小
* free: 空闲的内存
* buff: 用作缓冲的内存大小
* cache: 用作缓存的内存大小

#### Swap （单位：KB）
* si: 从交换区写到内存的大小
* so: 每秒写入交换区的内存大小

#### IO （单位：块/秒）
* bi: 每秒读取的块数
* bo: 每秒写入的块数

#### System （系统）
* in: 每秒中断数，包括时钟中断。
* cs: 每秒上下文切换数。

#### CPU（以百分比表示）：
* us: 用户进程执行时间(user time)
* sy: 系统进程执行时间(system time)
* id: 空闲时间(包括IO等待时间),中央处理器的空闲时间 。以百分比表示。
* wa: 等待IO时间


> 参考链接：
> https://zhuanlan.zhihu.com/p/92955445
> https://www.jianshu.com/p/6d9e5ad2751c