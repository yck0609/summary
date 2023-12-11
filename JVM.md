# JVM

> JVM是Java平台的核心组件之一，用于执行Java字节码，提供跨平台执行环境，使得Java程序能够在不同的操作系统和硬件上运行

# 1 内存区域

> 内存区域是指运行Java程序时将JVM内存划分为不同的区域，用于存储不同类型的数据，执行不同的任务

![](https://www.zhuxianfei.com/upload/news/202108/16/2021081614355331.jpg)

## 1.1 程序计数器（Program Counter）

> 程序计数器用于记录当前线程执行的字节码指令的地址，指向当前线程正在执行的方法的下一条要执行的指令（如果正在执行的是本地方法则为空）。当线程切换时，JVM根据程序计数器的值确定下一条要执行的指令，确保线程切换后能够从正确的位置继续执行

- **线程私有** ：线程切换时需要保持各自的执行状态，因此每个线程都有独立的程序计数器，互不干扰

- **字节码指令地址** ：程序计数器存储字节码指令地址而不是实际的指令内容。字节码指令是Java虚拟机执行的最小指令单位，每个指令都有一个唯一的地址。程序计数器通过记录指令地址来控制线程的执行流程

- **异常处理** ：发生异常时，JVM会根据程序计数器的值来确定异常处理代码的位置，从而正确地处理异常并进行相应的跳转。程序计数器是JVM中唯一一个在Java虚拟机规范中没有规定任何`OutOfMemoryError`的区域

## 1.2 虚拟机栈（Virtual Machine Stack）

> 虚拟机栈以帧（Frame）为单位，每个方法对应一个帧，用于存储方法的局部变量、操作数栈、动态链接、方法出口等信息，管理方法的调用和执行过程，保证方法能够正确地进入和退出

- **线程私有** ：虚拟机栈存储线程私有的方法执行信息，不同线程之间的栈帧需要独立维护，以支持并发执行，每个线程各自的虚拟机栈之间互不干扰

- **栈容量** ：栈的大小是固定的，可以通过JVM参数设置。如果虚拟机栈的大小不够，可能会导致栈溢出（StackOverflowError）异常。可以通过 -Xss 这个虚拟机参数来指定每个线程的 Java 虚拟机栈内存大小，在 JDK 1.4 中默认为 256K，而在 JDK 1.5+ 默认为 1M

### 1.2.1 虚拟机栈储存内容

- **栈帧** ：虚拟机栈以栈帧为单位进行管理。每个方法在虚拟机栈中对应一个栈帧，栈帧由操作数栈、局部变量表、动态链接、方法出口等组成。栈帧的创建和销毁与方法的调用和返回相对应，它们按照后进先出（LIFO）的顺序进行操作

- **局部变量表** ：栈帧包含局部变量表，用于存储方法的局部变量。局部变量表是一个固定长度的数组，用于存储各种类型的局部变量，包括基本数据类型和对象引用

- **操作数栈** ：栈帧中包含操作数栈，用于执行方法的字节码指令。操作数栈是后进先出（LIFO）的数据结构，用于存储方法执行过程中的操作数和中间结果。操作数栈的大小在编译期间就确定了，由编译器根据方法的字节码指令确定

- **动态链接** ：虚拟机栈中的栈帧通过动态链接来实现方法调用的动态性。动态链接指的是在运行时将符号引用解析为实际的方法引用，以实现方法的动态绑定。动态链接的过程包括类的查找和方法的绑定，以确保方法调用的正确性

- **方法出口** ：每个栈帧中都包含一个方法出口，用于指示方法的返回地址。方法出口可以是方法的正常返回点，也可以是异常处理代码的入口点。方法的返回会将栈帧从虚拟机栈中弹出，并将控制权返回给调用方

## 1.3 本地方法栈（Native Method Stack）

> 本地方法栈用于支持Java程序调用和执行本地（非Java）方法。本地方法是使用非Java语言（如C、C++等）编写的方法，通过本地方法栈可以实现Java程序与本地代码的交互。本地方法栈保存了本地方法的执行信息，包括方法的参数、局部变量和操作数栈等

- **与虚拟机栈的区别**：本地方法栈与虚拟机栈的主要区别在于执行的方法类型不同。虚拟机栈主要用于执行Java方法，而本地方法栈主要用于执行本地方法。虚拟机栈中的栈帧保存了Java方法的执行信息，包括字节码指令等，而本地方法栈中的栈帧保存了本地方法的执行信息

## 1.4 堆（Heap）

> 堆是JVM中最大的内存区域，用于存储对象实例和数组等动态分配的数据，是Java程序运行时的动态数据区域，也是垃圾回收的主要工作区域

![](https://s7.51cto.com/oss/202206/04/f720fa16571a082d0d935677e63938eaa66da1.png)

### 1.4.1 新生代（Young Generation）

> 新生代用于存储新创建的对象，由Eden空间和Survivor空间组成

#### 1.4.1.1 Eden空间

> `Eden` 空间用于存储刚创建的对象。如果在`Eden`空间中的对象经历一系列的垃圾回收过程后仍然存活，它们将被移动到`Survivor`空间

#### 1.4.1.2 From Survivor / To Survivor

> `Survivor` 空间用于存储从Eden空间晋升而来的存活对象，由两个相等大小的区域组成，分别称为From空间和To空间。垃圾回收时，Eden 和 From Survivor 幸存的对象将被复制到To Survivor，并且From空间会被清空。随后，From Survivor 和 To Survivor 角色互换。经过多次垃圾回收后，仍然存活的对象会逐渐被移到老年代

### 1.4.2 老年代（Old Generation）

> 老年代用于存储长时间存活的对象和大对象，当`Survivor` 空间无法容纳存活的对象时，这些对象将被晋升到老年代。此外，如果对象的年龄达到一定的阈值（通常是15），也会被直接晋升到老年代。这个过程称为对象的晋升（promotion）

## 1.5 方法区(Method Area)

> 方法区也被称为永久代（Permanent Generation）或元空间（Metaspace）（在Java 8及之后的版本中），用于存储类的元数据、静态变量、常量以及编译后的方法字节码等信息。方法区是所有线程共享的内存区域，它在JVM启动时被创建，并且在JVM关闭时被销毁，其大小可以通过命令行参数或JVM配置进行调整

- **作用**：用于存储类的信息，包括类的结构、字段、方法、常量池等，是所有线程共享的内存区域，存储了整个程序的类信息和静态数据
- **存储内容**：方法区存储了类的结构信息，包括类的字段、方法、构造函数等。它还包含了常量池（Constant Pool），常量池是一种特殊的数据结构，用于存储编译时生成的各种字面量和符号引用。方法区还存储了静态变量和即时编译器（Just-In-Time Compiler，JIT）编译后的代码
- **类加载**：当需要使用一个类时，JVM会先到方法区中查找该类的信息。如果方法区中没有该类的信息，则会通过类加载器（ClassLoader）将类的字节码加载到方法区中，并进行相应的验证、解析和初始化操作
- **垃圾回收**：垃圾回收器会扫描方法区，标记出不再使用的类和常量，并进行相应的回收操作。方法区的垃圾回收主要针对无用的类和常量，以释放内存空间
- **内存布局**：在JVM的内存布局中，方法区位于堆的逻辑上方，它的大小可以通过JVM参数进行配置

# 2 垃圾收集

> 垃圾收集主要是针对堆和方法区进行。程序计数器、虚拟机栈和本地方法栈这三个区域属于线程私有的，只存在于线程的生命周期内，线程结束之后就会消失，因此不需要对这三个区域进行垃圾回收

## 2.1 对象回收判定

### 2.1.1 引用计数算法

> 为对象添加引用计数器，当对象增加一个引用时计数器加 1，引用失效时计数器减 1。引用计数为 0 的对象可被回收。但是在两个对象出现循环引用时引用计数器永远不为 0，导致无法对它们进行回收。正是因为循环引用的存在，因此 Java 虚拟机不使用引用计数算法

### 2.1.2 可达性分析算法

> 以 GC Roots 为起始点进行搜索，可达的对象都是存活的，不可达的对象可被回收。GC Roots 一般包含以下内容：
> 
> - 虚拟机栈中局部变量表中引用的对象
> - 本地方法栈中 JNI 中引用的对象
> - 方法区中类静态属性引用的对象
> - 方法区中的常量引用的对象

## 2.2 垃圾收集算法

> 垃圾回收算法用于自动管理内存，识别和释放不再使用的对象

### 2.2.1 标记-清除算法（Mark - Sweep）

> 1. 标记阶段 ：程序检查每个对象是否为活动对象，如果是活动对象，则在对象头部打上标记
> 
> 2. 清除阶段 ：进行对象回收并取消标志位，判断回收后的分块与前一个空闲分块是否连续，若连续则合并分块
> - 优点 ：标记和清除过程效率都不高
> 
> - 不足 ：会产生大量不连续的内存碎片，导致无法给大对象分配内存

### 2.2.2 标记-复制算法（Mark - Copying）

> 复制算法将内存空间划分为`From`空间和`To`空间。在垃圾回收过程中，垃圾回收器将存活对象从`From`空间复制到`To`空间，并按顺序紧凑排列。然后，清空`From`空间中的所有对象，使其成为空白内存
> 
> - 复制算法解决了标记-清除算法的内存碎片问题，但它浪费了一半的内存空间

### 2.2.3 标记-整理算法（Mark - Compact）

> 标记-压缩算法标记存活对象，然后将它们压缩到内存的一端，清理未被标记的对象，并更新对象引用
> 
> - 优点:不会产生内存碎片
> 
> - 不足:需要移动大量对象，处理效率比较低

## 2.3 垃圾收集器

> 垃圾收集算法是内存回收的方法论，垃圾收集器就是内存回收的实践者

### 2.3.1 Serial 收集器（Serial Collector）

> Serial收集器以串行的方式执行，使用单线程进行垃圾收集，即暂停所有应用线程，进行垃圾回收。优点是简单高效，在单个 CPU 环境下，由于没有线程交互的开销，因此拥有最高的单线程收集效率

### 2.3.2 Serial Old 收集器

> Serial Old 收集器是 Serial 收集器的老年代版本，在 JDK 1.5 以及之前版本（Parallel Old 诞生以前）中与 Parallel Scavenge 收集器搭配使用。作为 CMS 收集器的后备预案，在并发收集发生 Concurrent Mode Failure 时使用

### 2.3.3 ParNew 收集器

> ParNew 收集器是 Serial 收集器的多线程版本，使用多线程进行垃圾收集

### 2.3.4 Parallel Scavenge 收集器

> Parallel Scavenge 收集器是多线程收集器，目标是达到一个可控制的吞吐量，因此被称为“吞吐量优先”收集器。所谓吞吐量就是CPU用于运行用户代码的时间与CPU总消耗时间的比值，即吞吐量 = 运行用户代码时间 / (运行用户代码时间 + 垃圾收集时间)。停顿时间越短就越适合需要与用户交互的程序，良好的响应速度能提升用户体验。而高吞吐量则可以高效率地利用 CPU 时间，尽快完成程序的运算任务，适合在后台运算而不需要太多交互的任务

### 2.3.5 Parallel Old 收集器

> Parallel Old 收集器是 Parallel Scavenge 收集器的老年代版本。在注重吞吐量以及 CPU 资源敏感的场合，都可以优先考虑 Parallel Scavenge 加 Parallel Old 收集器

### 2.3.6 CMS 收集器（Concurrent Mark Sweep Collector）

> CMS收集器是一种以获取最短回收停顿时间为目标的并发垃圾收集器，在垃圾收集过程中尽可能减少应用程序的停顿时间，通过在应用程序运行的同时进行部分垃圾回收来实现低延迟。CMS收集器适用于具有较大堆内存和更短停顿时间要求的应用程序，但会导致更高的CPU使用率

#### 2.3.6.1 工作流程

> 1. 初始标记：停止用户线程，标记GC Roots能直接关联到的对象
> 
> 2. 并发标记：进行 GC Roots Tracing ，也就是从根节点往下去寻找引用对象的过程，不需要停顿
> 
> 3. 重新标记：为了修正并发标记期间因用户程序继续运作而导致标记产生变动的那一部分对象的标记记录，需要停顿。停止用户线程，修正并发标记期间，因用户程序继续运作而导致标记产生变动的那一部分对象的标记记录，这个阶段的停顿时间一般会比初始标记阶段稍长一些，但远比并发标记的时间短
> 
> 4. 并发清除：对未被标记的对象做清除工作，这个阶段如果有新增对象产生，会不做任何处理，等待下一次GC，不需要停顿
> 
> 5. 并发重置：重置本次GC过程中的标记数据

#### 2.3.6.2 优缺点

> 1. 吞吐量低：低停顿时间是以牺牲吞吐量为代价的，导致 CPU 利用率不够高
> 
> 2. 无法处理浮动垃圾，可能出现 Concurrent Mode Failure。浮动垃圾是指并发清除阶段由于用户线程继续运行而产生的垃圾，这部分垃圾只能到下一次 GC 时才能进行回收。由于浮动垃圾的存在，因此需要预留出一部分内存，意味着 CMS 收集不能像其它收集器那样等待老年代快满的时候再回收。如果预留的内存不够存放浮动垃圾，就会出现 Concurrent Mode Failure，这时虚拟机将临时启用 Serial Old 来替代 CMS
> 
> 3. 标记 - 清除算法导致的空间碎片，往往出现老年代空间剩余，但无法找到足够大连续空间来分配当前对象，不得不提前触发一次 Full GC

### 2.3.7 G1 收集器（Garbage-First Collector）

> G1收集器是一种面向服务器的垃圾收集器，旨在提供更可控的停顿时间和更高的吞吐量。它使用分代和区域划分的方式，将堆内存划分为多个区域，可以并行和并发地执行垃圾回收，适用于具有大堆内存和低停顿时间要求的应用程序，可以更好地控制停顿时间分布。 G1 可以直接对新生代和老年代一起回收。G1 把堆划分成多个大小相等的独立区域（Region），新生代和老年代不再物理隔离。通过引入 Region 的概念，从而将原来的一整块内存空间划分成多个的小空间，使得每个小空间可以单独进行垃圾回收。这种划分方法带来了很大的灵活性，使得可预测的停顿时间模型成为可能。通过记录每个 Region 垃圾回收时间以及回收所获得的空间（这两个值是通过过去回收的经验获得），并维护一个优先列表，每次根据允许的收集时间，优先回收价值最大的 Region。每个 Region 都有一个 Remembered Set，用来记录该 Region 对象的引用对象所在的 Region。通过使用 Remembered Set，在做可达性分析的时候就可以避免全堆扫描

#### 2.3.7.1 工作流程

> 1. **初始标记**：同CMS的初始标记
> 
> 2. **并发标记**：同CMS的并发标记
> 
> 3. 最终标记：为了修正在并发标记期间因用户程序继续运作而导致标记产生变动的那一部分标记记录，虚拟机将这段时间对象变化记录在线程的 Remembered Set Logs 里面，最终标记阶段需要把 Remembered Set Logs 的数据合并到 Remembered Set 中。这阶段需要停顿线程，但是可并行执行
> 
> 4. 筛选回收：首先对各个 Region 中的回收价值和成本进行排序，根据用户所期望的 GC 停顿时间来制定回收计划。此阶段其实也可以做到与用户程序一起并发执行，但是因为只回收一部分 Region，时间是用户可控制的，而且停顿用户线程将大幅度提高收集效率。停止用户线程，首先排序各个Region的回收价值和成本，然后根据用户期望的GC停顿时间来制定回收计划。最后按计划回收一些价值高的Region中的垃圾对象。回收时采用复制算法，从一个或多个Region块中复制存活对象到堆上的另一个空的Region块，并且在此过程中压缩和释放内存。G1不会像CMS那样回收完因为有很多内存碎片还需要整理一次

#### 2.3.7.2 优缺点

- 空间整合：整体来看是基于“标记 - 整理”算法实现的收集器，从局部（两个 Region 之间）上来看是基于“复制”算法实现的，这意味着运行期间不会产生内存空间碎片。
- 可预测的停顿：能让使用者明确指定在一个长度为 M 毫秒的时间片段内，消耗在 GC 上的时间不得超过 N 毫秒。

使用G1收集器时，Java堆的内存布局与其他收集器有很大差别，它将整个Java堆划分为多个大小相等的独立区域（Region），即Region块。有**Eden块**、**Survivor块**、**Old块**和专门用来存放大对象的**Humongous块**（当一个对象的存储空间超过一个Region块大小的50%的时候，就会被放入到Humongous块中。如果一个大对象太大，可能会横跨多个Region块来存储）之分。虽然还保留有新生代和老年代的概念，但新生代和老年代不再是物理隔阂了，它们都是一部分（可以不连续）Region的集合

G1垃圾收集的分类如下，其与一般的GC也不太相同：

- **Young GC**：不同于常规的Young GC在Eden区满了的时候会进行，G1会计算这次Eden区回收大概需要多少时间，如果回收时间远小于设定好的停顿时间值，就会选择增加新的年轻代的Region块，以此来存放新的对象，直到下一次Eden区满。如果下一次GC的预测时间接近设定的值，那么就会触发Young GC
- **Mixed GC**：老年代的空间占用率达到参数设定的值时会触发，回收所有的Young区和部分的Old和Humongous区，使用复制算法的时候如果发现没有足够的空Region块能承载拷贝对象的时候，会触发一次Full GC
- **Full GC**：停止系统线程，然后采用单线程进行清理工作，空闲出的一批Region块以供下一次的Mixed GC来使用，这个过程是比较耗时的（如上所说，Shenandoah收集器已经优化成多线程进行收集了）

### 2.3.8 Shenandoah收集器

> Shenandoah收集器是一种低延迟垃圾收集器，基于读屏障（Read Barrier）和写屏障（Write Barrier）实现并发标记和并发压缩阶段，实现几乎无停顿的垃圾回收，适用于对低延迟要求非常高的大型应用程序

### 2.3.9 ZGC收集器（Z Garbage Collector）

> ZGC收集器是一种低延迟垃圾收集器，旨在实现几乎无停顿的垃圾回收，使用了一种基于读屏障（Read Barrier）的算法，适用于对低延迟要求非常高的大型应用程序。ZGC收集器是一款基于Region内存布局的，暂时不设分代，使用了读屏障、颜色指针等技术来实现可并发的标记-整理算法的，以低延迟为首要目标的一款垃圾收集器

ZGC中的Region块分为大、中、小三种容量：

- **小型Region**：容量固定为2MB，用于放置小于256KB的小对象；
- **中型Region**：容量固定为32MB，用于放置大于等于256KB但小于4MB的对象；
- **大型Region**：容量不固定，可以动态变化，但必须为2MB的整数倍，用于放置4MB或以上的大对象。

#### 2.3.9.1 工作流程

> 1. **并发标记**：与G1一样，并发标记是遍历对象图做可达性分析的阶段。它的初始标记和最终标记也会出现短暂的停顿，与G1不同的是，ZGC的标记是在指针上而不是在对象上进行的，标记阶段会更新染色指针中的Marked 0、Marked 1标志位
> 
> 2. **并发预备重分配**：这个阶段需要根据特定的查询条件统计得出本次收集过程要清理哪些Region，将这些Region组成重分配集（Relocation Set）。ZGC每次回收都会扫描所有的Region，用范围更大的扫描成本换取省去G1中记忆集的维护成本
> 
> 3. **并发重分配**：重分配是ZGC执行过程中的核心阶段，这个过程要把重分配集中的存活对象复制到新的Region上，并为重分配集中的每个Region维护一个转发表（Forward Table），记录从旧对象到新对象的转向关系。ZGC收集器能仅从引用上就明确得知一个对象是否处于重分配集之中，如果用户线程此时并发访问了位于重分配集中的对象，这次访问将会被预置的内存屏障（读屏障）所截获，然后立即根据Region上的转发表记录将访问转发到新复制的对象上，并同时修正更新该引用的值，使其直接指向新对象，ZGC将这种行为称为指针的“自愈”（Self-Healing）能力
> 
> 4. **并发重映射**：重映射所做的就是修正整个堆中指向重分配集中旧对象的所有引用，但是ZGC中对象引用存在“自愈”功能，所以这个重映射操作并不是很迫切。ZGC很巧妙地把并发重映射阶段要做的工作，合并到了下一次垃圾收集循环中的并发标记阶段里去完成，反正它们都是要遍历所有对象的，这样合并就节省了一次遍历对象图的开销。一旦所有指针都被修正之后， 原来记录新旧对象关系的转发表就可以释放掉了

### 2.3.10 Epsilon收集器

> Epsilon收集器是一种实验性的垃圾收集器，目的是实现垃圾收集的最低延迟。它不执行任何实际的垃圾回收操作，而是简单地忽略所有的垃圾回收请求。Epsilon收集器适用于对垃圾回收没有要求的特殊场景，如性能测试或仅在内存耗尽时才进行应急垃圾回收。这些垃圾收集器在HotSpot虚拟机中提供了不同的垃圾回收策略和性能特点，可以根据应用程序的需求进行选择和配置。Java 11中还加入了一个比较特殊的垃圾收集器——Epsilon，该垃圾收集器被称为“no-op”收集器，将处理内存分配而不实施任何实际的内存回收机制。 也就是说，这是一款不做垃圾回收的垃圾回收器。这个垃圾回收器看起来并没什么用，主要可以用来进行性能测试、内存压力测试等，Epsilon GC可以作为度量其他垃圾回收器性能的对照组。大神Martijn说，Epsilon GC至少能够帮助理解GC的接口，有助于成就一个更加模块化的JVM

# 4 内存分配与回收策略

## Minor GC 和 Full GC

- Minor GC：回收新生代，因为新生代对象存活时间很短，因此 Minor GC 会频繁执行，执行的速度一般也会比较快

- Full GC：回收老年代和新生代，老年代对象其存活时间长，因此 Full GC 很少执行，执行速度会比 Minor GC 慢很多

## 内存分配策略

### 1. 对象优先在 Eden 分配

大多数情况下，对象在新生代 Eden 上分配，当 Eden 空间不够时，发起 Minor GC。

### 2. 大对象直接进入老年代

大对象是指需要连续内存空间的对象，最典型的大对象是那种很长的字符串以及数组。

经常出现大对象会提前触发垃圾收集以获取足够的连续空间分配给大对象。

-XX:PretenureSizeThreshold，大于此值的对象直接在老年代分配，避免在 Eden 和 Survivor 之间的大量内存复制。

### 3. 长期存活的对象进入老年代

为对象定义年龄计数器，对象在 Eden 出生并经过 Minor GC 依然存活，将移动到 Survivor 中，年龄就增加 1 岁，增加到一定年龄则移动到老年代中。

-XX:MaxTenuringThreshold 用来定义年龄的阈值。

### 4. 动态对象年龄判定

虚拟机并不是永远要求对象的年龄必须达到 MaxTenuringThreshold 才能晋升老年代，如果在 Survivor 中相同年龄所有对象大小的总和大于 Survivor 空间的一半，则年龄大于或等于该年龄的对象可以直接进入老年代，无需等到 MaxTenuringThreshold 中要求的年龄。

### 5. 空间分配担保

在发生 Minor GC 之前，虚拟机先检查老年代最大可用的连续空间是否大于新生代所有对象总空间，如果条件成立的话，那么 Minor GC 可以确认是安全的。

如果不成立的话虚拟机会查看 HandlePromotionFailure 的值是否允许担保失败，如果允许那么就会继续检查老年代最大可用的连续空间是否大于历次晋升到老年代对象的平均大小，如果大于，将尝试着进行一次 Minor GC；如果小于，或者 HandlePromotionFailure 的值不允许冒险，那么就要进行一次 Full GC。

## Full GC 的触发条件

对于 Minor GC，其触发条件非常简单，当 Eden 空间满时，就将触发一次 Minor GC。而 Full GC 则相对复杂，有以下条件：

### 1. 调用 System.gc()

只是建议虚拟机执行 Full GC，但是虚拟机不一定真正去执行。不建议使用这种方式，而是让虚拟机管理内存。

### 2. 老年代空间不足

老年代空间不足的常见场景为前文所讲的大对象直接进入老年代、长期存活的对象进入老年代等。

为了避免以上原因引起的 Full GC，应当尽量不要创建过大的对象以及数组。除此之外，可以通过 -Xmn 虚拟机参数调大新生代的大小，让对象尽量在新生代被回收掉，不进入老年代。还可以通过 -XX:MaxTenuringThreshold 调大对象进入老年代的年龄，让对象在新生代多存活一段时间。

### 3. 空间分配担保失败

使用复制算法的 Minor GC 需要老年代的内存空间作担保，如果担保失败会执行一次 Full GC。具体内容请参考上面的第 5 小节。

### 4. JDK 1.7 及以前的永久代空间不足

在 JDK 1.7 及以前，HotSpot 虚拟机中的方法区是用永久代实现的，永久代中存放的为一些 Class 的信息、常量、静态变量等数据。

当系统中要加载的类、反射的类和调用的方法较多时，永久代可能会被占满，在未配置为采用 CMS GC 的情况下也会执行 Full GC。如果经过 Full GC 仍然回收不了，那么虚拟机会抛出 java.lang.OutOfMemoryError。

为避免以上原因引起的 Full GC，可采用的方法为增大永久代空间或转为使用 CMS GC。

### 5. Concurrent Mode Failure

执行 CMS GC 的过程中同时有对象要放入老年代，而此时老年代空间不足（可能是 GC 过程中浮动垃圾过多导致暂时性的空间不足），便会报 Concurrent Mode Failure 错误，并触发 Full GC。

# 5 类文件结构

## 5.1 Class 文件结构

### 5.1.1 魔数

### 5.1.2 常量池

### 5.1.3 访问标志

### 5.1.4 字段表集合

### 5.1.5 方法表集合

### 5.1.6 属性表集合

## 5.2 字节码指令

### 5.2.1 字节码与数据类型

### 5.2.2 加载和存储指令

### 5.2.3 运算指令

### 5.2.4 类型转换指令

### 5.2.5 对象创建与访问指令

### 5.2.6 操作数栈管理指令

### 5.2.7 控制转移指令

### 5.2.8 方法调用和返回指令

### 5.2.9 异常处理指令

### 5.2.10 同步指令

# 6 类加载机制

> 在Class文件中描述的各类信息，最终都需要加载到虚拟机中之后才能被运行和使用

## 类的生命周期

- **加载（Loading）**
- **验证（Verification）**
- **准备（Preparation）**
- **解析（Resolution）**
- **初始化（Initialization）**
- 使用（Using）
- 卸载（Unloading）

## 类加载的时机

## 类加载过程

> 包含了加载、验证、准备、解析和初始化这 5 个阶段

### 1. 加载

加载是类加载的一个阶段，注意不要混淆。

加载过程完成以下三件事：

- 通过类的完全限定名称获取定义该类的二进制字节流。
- 将该字节流表示的静态存储结构转换为方法区的运行时存储结构。
- 在内存中生成一个代表该类的 Class 对象，作为方法区中该类各种数据的访问入口。

其中二进制字节流可以从以下方式中获取：

- 从 ZIP 包读取，成为 JAR、EAR、WAR 格式的基础。
- 从网络中获取，最典型的应用是 Applet。
- 运行时计算生成，例如动态代理技术，在 java.lang.reflect.Proxy 使用 ProxyGenerator.generateProxyClass 的代理类的二进制字节流。
- 由其他文件生成，例如由 JSP 文件生成对应的 Class 类。

### 2. 验证

确保 Class 文件的字节流中包含的信息符合当前虚拟机的要求，并且不会危害虚拟机自身的安全。

### 3. 准备

类变量是被 static 修饰的变量，准备阶段为类变量分配内存并设置初始值，使用的是方法区的内存。

实例变量不会在这阶段分配内存，它会在对象实例化时随着对象一起被分配在堆中。应该注意到，实例化不是类加载的一个过程，类加载发生在所有实例化操作之前，并且类加载只进行一次，实例化可以进行多次。

初始值一般为 0 值，例如下面的类变量 value 被初始化为 0 而不是 123。

```java
public static int value = 123;
```

如果类变量是常量，那么它将初始化为表达式所定义的值而不是 0。例如下面的常量 value 被初始化为 123 而不是 0。

```java
public static final int value = 123;
```

### 4. 解析

将常量池的符号引用替换为直接引用的过程。

其中解析过程在某些情况下可以在初始化阶段之后再开始，这是为了支持 Java 的动态绑定。

<div data="补充为什么可以支持动态绑定 --> <--"></div>
###  5. 初始化

<div data="modify -->"></div>
初始化阶段才真正开始执行类中定义的 Java 程序代码。初始化阶段是虚拟机执行类构造器 <clinit>() 方法的过程。在准备阶段，类变量已经赋过一次系统要求的初始值，而在初始化阶段，根据程序员通过程序制定的主观计划去初始化类变量和其它资源。

&lt;clinit>() 是由编译器自动收集类中所有类变量的赋值动作和静态语句块中的语句合并产生的，编译器收集的顺序由语句在源文件中出现的顺序决定。特别注意的是，静态语句块只能访问到定义在它之前的类变量，定义在它之后的类变量只能赋值，不能访问。例如以下代码：

```java
public class Test {
    static {
        i = 0;                // 给变量赋值可以正常编译通过
        System.out.print(i);  // 这句编译器会提示“非法向前引用”
    }
    static int i = 1;
}
```

由于父类的 &lt;clinit>() 方法先执行，也就意味着父类中定义的静态语句块的执行要优先于子类。例如以下代码：

```java
static class Parent {
    public static int A = 1;
    static {
        A = 2;
    }
}

static class Sub extends Parent {
    public static int B = A;
}

public static void main(String[] args) {
     System.out.println(Sub.B);  // 2
}
```

接口中不可以使用静态语句块，但仍然有类变量初始化的赋值操作，因此接口与类一样都会生成 &lt;clinit>() 方法。但接口与类不同的是，执行接口的 &lt;clinit>() 方法不需要先执行父接口的 &lt;clinit>() 方法。只有当父接口中定义的变量使用时，父接口才会初始化。另外，接口的实现类在初始化时也一样不会执行接口的 &lt;clinit>() 方法。

虚拟机会保证一个类的 &lt;clinit>() 方法在多线程环境下被正确的加锁和同步，如果多个线程同时初始化一个类，只会有一个线程执行这个类的 &lt;clinit>() 方法，其它线程都会阻塞等待，直到活动线程执行 &lt;clinit>() 方法完毕。如果在一个类的 &lt;clinit>() 方法中有耗时的操作，就可能造成多个线程阻塞，在实际过程中此种阻塞很隐蔽。

## 类初始化时机

### 1. 主动引用

虚拟机规范中并没有强制约束何时进行加载，但是规范严格规定了有且只有下列五种情况必须对类进行初始化（加载、验证、准备都会随之发生）：

- 遇到 new、getstatic、putstatic、invokestatic 这四条字节码指令时，如果类没有进行过初始化，则必须先触发其初始化。最常见的生成这 4 条指令的场景是：使用 new 关键字实例化对象的时候；读取或设置一个类的静态字段（被 final 修饰、已在编译期把结果放入常量池的静态字段除外）的时候；以及调用一个类的静态方法的时候。

- 使用 java.lang.reflect 包的方法对类进行反射调用的时候，如果类没有进行初始化，则需要先触发其初始化。

- 当初始化一个类的时候，如果发现其父类还没有进行过初始化，则需要先触发其父类的初始化。

- 当虚拟机启动时，用户需要指定一个要执行的主类（包含 main() 方法的那个类），虚拟机会先初始化这个主类；

- 当使用 JDK 1.7 的动态语言支持时，如果一个 java.lang.invoke.MethodHandle 实例最后的解析结果为 REF_getStatic, REF_putStatic, REF_invokeStatic 的方法句柄，并且这个方法句柄所对应的类没有进行过初始化，则需要先触发其初始化；

### 2. 被动引用

以上 5 种场景中的行为称为对一个类进行主动引用。除此之外，所有引用类的方式都不会触发初始化，称为被动引用。被动引用的常见例子包括：

- 通过子类引用父类的静态字段，不会导致子类初始化。

```java
System.out.println(SubClass.value);  // value 字段在 SuperClass 中定义
```

- 通过数组定义来引用类，不会触发此类的初始化。该过程会对数组类进行初始化，数组类是一个由虚拟机自动生成的、直接继承自 Object 的子类，其中包含了数组的属性和方法。

```java
SuperClass[] sca = new SuperClass[10];
```

- 常量在编译阶段会存入调用类的常量池中，本质上并没有直接引用到定义常量的类，因此不会触发定义常量的类的初始化。

```java
System.out.println(ConstClass.HELLOWORLD);
```

## 类与类加载器

> 两个类相等，需要类本身相等，并且使用同一个类加载器进行加载。这是因为每一个类加载器都拥有一个独立的类名称空间。这里的相等，包括类的 Class 对象的 equals() 方法、isAssignableFrom() 方法、isInstance() 方法的返回结果为 true，也包括使用 instanceof 关键字做对象所属关系判定结果为 true。

## 类加载器

### 类加载器分类

从 Java 虚拟机的角度来讲，只存在以下两种不同的类加载器：

- 启动类加载器（Bootstrap ClassLoader），使用 C++ 实现，是虚拟机自身的一部分；

- 所有其它类的加载器，使用 Java 实现，独立于虚拟机，继承自抽象类 java.lang.ClassLoader。

从 Java 开发人员的角度看，类加载器可以划分得更细致一些：

- 启动类加载器（Bootstrap ClassLoader）此类加载器负责将存放在 &lt;JRE_HOME>\lib 目录中的，或者被 -Xbootclasspath 参数所指定的路径中的，并且是虚拟机识别的（仅按照文件名识别，如 rt.jar，名字不符合的类库即使放在 lib 目录中也不会被加载）类库加载到虚拟机内存中。启动类加载器无法被 Java 程序直接引用，用户在编写自定义类加载器时，如果需要把加载请求委派给启动类加载器，直接使用 null 代替即可。

- 扩展类加载器（Extension ClassLoader）这个类加载器是由 ExtClassLoader（sun.misc.Launcher$ExtClassLoader）实现的。它负责将 &lt;JAVA_HOME>/lib/ext 或者被 java.ext.dir 系统变量所指定路径中的所有类库加载到内存中，开发者可以直接使用扩展类加载器。

- 应用程序类加载器（Application ClassLoader）这个类加载器是由 AppClassLoader（sun.misc.Launcher$AppClassLoader）实现的。由于这个类加载器是 ClassLoader 中的 getSystemClassLoader() 方法的返回值，因此一般称为系统类加载器。它负责加载用户类路径（ClassPath）上所指定的类库，开发者可以直接使用这个类加载器，如果应用程序中没有自定义过自己的类加载器，一般情况下这个就是程序中默认的类加载器。

## 双亲委派模型

> 应用程序是由三种类加载器互相配合从而实现类加载，除此之外还可以加入自己定义的类加载器。下图展示了类加载器之间的层次关系，称为双亲委派模型（Parents Delegation Model）。该模型要求除了顶层的启动类加载器外，其它的类加载器都要有自己的父类加载器。这里的父子关系一般通过组合关系（Composition）来实现，而不是继承关系（Inheritance）。

### 1. 工作过程

一个类加载器首先将类加载请求转发到父类加载器，只有当父类加载器无法完成时才尝试自己加载。

### 2. 好处

使得 Java 类随着它的类加载器一起具有一种带有优先级的层次关系，从而使得基础类得到统一。

例如 java.lang.Object 存放在 rt.jar 中，如果编写另外一个 java.lang.Object 并放到 ClassPath 中，程序可以编译通过。由于双亲委派模型的存在，所以在 rt.jar 中的 Object 比在 ClassPath 中的 Object 优先级更高，这是因为 rt.jar 中的 Object 使用的是启动类加载器，而 ClassPath 中的 Object 使用的是应用程序类加载器。rt.jar 中的 Object 优先级更高，那么程序中所有的 Object 都是这个 Object。

### 3. 实现

以下是抽象类 java.lang.ClassLoader 的代码片段，其中的 loadClass() 方法运行过程如下：先检查类是否已经加载过，如果没有则让父类加载器去加载。当父类加载器加载失败时抛出 ClassNotFoundException，此时尝试自己去加载。

```java
public abstract class ClassLoader {
    // The parent class loader for delegation
    private final ClassLoader parent;

    public Class<?> loadClass(String name) throws ClassNotFoundException {
        return loadClass(name, false);
    }

    protected Class<?> loadClass(String name, boolean resolve) throws ClassNotFoundException {
        synchronized (getClassLoadingLock(name)) {
            // First, check if the class has already been loaded
            Class<?> c = findLoadedClass(name);
            if (c == null) {
                try {
                    if (parent != null) {
                        c = parent.loadClass(name, false);
                    } else {
                        c = findBootstrapClassOrNull(name);
                    }
                } catch (ClassNotFoundException e) {
                    // ClassNotFoundException thrown if class not found
                    // from the non-null parent class loader
                }

                if (c == null) {
                    // If still not found, then invoke findClass in order
                    // to find the class.
                    c = findClass(name);
                }
            }
            if (resolve) {
                resolveClass(c);
            }
            return c;
        }
    }

    protected Class<?> findClass(String name) throws ClassNotFoundException {
        throw new ClassNotFoundException(name);
    }
}
```

## 自定义类加载器实现

以下代码中的 FileSystemClassLoader 是自定义类加载器，继承自 java.lang.ClassLoader，用于加载文件系统上的类。它首先根据类的全名在文件系统上查找类的字节代码文件（.class 文件），然后读取该文件内容，最后通过 defineClass() 方法来把这些字节代码转换成 java.lang.Class 类的实例。java.lang.ClassLoader 的 loadClass() 实现了双亲委派模型的逻辑，自定义类加载器一般不去重写它，但是需要重写 findClass() 方法。

```java
public class FileSystemClassLoader extends ClassLoader {

    private String rootDir;

    public FileSystemClassLoader(String rootDir) {
        this.rootDir = rootDir;
    }

    protected Class<?> findClass(String name) throws ClassNotFoundException {
        byte[] classData = getClassData(name);
        if (classData == null) {
            throw new ClassNotFoundException();
        } else {
            return defineClass(name, classData, 0, classData.length);
        }
    }

    private byte[] getClassData(String className) {
        String path = classNameToPath(className);
        try {
            InputStream ins = new FileInputStream(path);
            ByteArrayOutputStream baos = new ByteArrayOutputStream();
            int bufferSize = 4096;
            byte[] buffer = new byte[bufferSize];
            int bytesNumRead;
            while ((bytesNumRead = ins.read(buffer)) != -1) {
                baos.write(buffer, 0, bytesNumRead);
            }
            return baos.toByteArray();
        } catch (IOException e) {
            e.printStackTrace();
        }
        return null;
    }

    private String classNameToPath(String className) {
        return rootDir + File.separatorChar
                + className.replace('.', File.separatorChar) + ".class";
    }
}
```

# IO / NIO

# 常见问题

## JVM常见的启动参数

```java
java -Xss2M HackTheJava
```

```
-Xms：初始大小内存，默认为物理内存1/64，等价于-XX:InitialHeapSize
-Xmx：最大分配内存，默认为物理内存1/4，等价于-XX:MaxHeapSize
-Xss：设置单个线程的大小，一般默认为512K~1024K，等价于-XX:ThreadStackSize
-Xmn：设置年轻代大小
-XX:MetaspaceSize：设置元空间大小，-Xms10m -Xmx10m -XX:MetaspaceSize=1024m -            XX:+PrintFlagsFinal
-XX:+PrintGCDetails：输出详细GC收集日志信息
```

- `-Xmn`参数用于指定新生代堆的大小，可以设置新生代的初始大小和最大大小
  
  ```shell
  java -Xmn512m -Xmx2g MyClass // 将新生代堆的初始大小设置为512MB，最大大小为2GB
  ```

- `-XX:SurvivorRatio`参数指定了Eden空间和Survivor空间的大小比例，默认值为8，表示Eden空间占新生代的8/10，两个Survivor空间各占1/10
  
  ```shell
  java -XX:SurvivorRatio=6 MyClass // 将Eden空间和Survivor空间比例调整为6:1:1
  ```



```
-XX:MaxTenuringThreshold：设置垃圾最大年龄
```

```
java -XX:+PrintFlagsInitial   查看初始默认值
java -XX:+PrintCommandLineFlags -version 查看GC是哪个收集器
```

```
如何查看一个正在运行中的java程序，它的某个JVM参数手否开启，具体值是多少？
1.命令jps -l 在idea下的terminal窗口下得到类的进程号 ID；
2.jinfo -flag PrintGCDetails ID //查看某一个正在运行的java程序是否开启打印GC收集细节
```

#### 几种常用的内存调试工具

```
jps:查看虚拟机进程的状况，如进程ID；
jmap: 用于生成堆转储快照文件(某一时刻的)。
jhat: 对生成的堆转储快照文件进行分析。
jstack: 用来生成线程快照(某一时刻的)。生成线程快照的主要目的是定位线程长时停顿的原因(如死锁,死循环,等待             I/O 等), 通过查看各个线程的调用堆栈,就可以知道没有响应的线程在后台做了什么或者等待什么资源。 
jstat: 虚拟机统计信息监视工具。如显示垃圾收集的情况,内存使用的情况。
Jconsole: 主要是内存监控和线程监控。内存监控:可以显示内存的使用情况。线程监控:遇到线程停顿时,可以使用这个            功能。
```

## 请谈谈你对ooM的认识

##### 1、Java.lang.StackOverflowError

##### 2.Java.lang.OutOfMemoryError:Java heap space

##### 3、Java.lang.OutOfMemeoryError:GC overhead limit exceeded

##### 4、Java.lang.OutOfMemeoryError:Direct buffer memory

##### 5、Java.lang.OutOfMemeoryError:unable to create new native thread

##### 6、Java.lang.OutOfMemeoryError:Metaspace

```
使用Java -XX:+PrintFlagsInitial命令查看本机的初始化参数，-XX:MetaspaceSize为21810376B(约20M)
```

## 假如生产环境出现CPU占用过高，请谈谈你的分析思路和定位

##### 案例步骤

```
1、先用top命令找出CPU占比最高的
```

```
2、 ps -ef或者jps进一步定位，得知是一个怎么样的一个后台程序
```

```
3、  定位到具体线程或者代码
ps -mp 进程 -o THREAD,tid,time
参数解释：
-m 显示所有线程
-p pid进程使用cpu的时间
-o 该参数后是用户自定义格式
```

```
4、将需要的线程ID转换为16进制格式(英文小写格式)
    printf "%x\n"  有问题的线程ID
```

5、jstack 进程ID | grep tid(16进制线程ID小写英文) -A60

## 引用类型

无论是通过引用计数算法判断对象的引用数量，还是通过可达性分析算法判断对象是否可达，判定对象是否可被回收都与引用有关。Java 提供了四种强度不同的引用类型。

### 1. 强引用

被强引用关联的对象不会被回收。使用 new 一个新对象的方式来创建强引用。

```java
Object obj = new Object();
```

### 2. 软引用

被软引用关联的对象只有在内存不够的情况下才会被回收。使用 SoftReference 类来创建软引用。

```java
Object obj = new Object();
SoftReference<Object> sf = new SoftReference<Object>(obj);
obj = null;  // 使对象只被软引用关联
```

### 3. 弱引用

被弱引用关联的对象一定会被回收，也就是说它只能存活到下一次垃圾回收发生之前。使用 WeakReference 类来创建弱引用。

```java
Object obj = new Object();
WeakReference<Object> wf = new WeakReference<Object>(obj);
obj = null;
```

### 4. 虚引用

又称为幽灵引用或者幻影引用，一个对象是否有虚引用的存在，不会对其生存时间造成影响，也无法通过虚引用得到一个对象。为一个对象设置虚引用的唯一目的是能在这个对象被回收时收到一个系统通知。使用 PhantomReference 来创建虚引用。

```java
Object obj = new Object();
PhantomReference<Object> pf = new PhantomReference<Object>(obj, null);
obj = null;

```
