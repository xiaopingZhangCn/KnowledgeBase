# 深入理解JAVA虚拟机
## 运行时数据区

Java虚拟机管理的内存：方法区、虚拟机栈、本地方法栈、堆、程序计数器。方法区和堆是线程共享，栈、程序计数器是线程隔离区，生命周期随线程。

### 程序计数器

程序计数器是一块较小的内存，它可以看做是当前线程所执行的行号指示器。字节码解释器工作的时候就是通过改变这个计数器的值来选取下一条需要执行的字节码的指令，分支、循环、跳转、异常处理、线程恢复等基础功能都需要依赖这个计数器来完成。  

如果线程正在执行的是一个java方法，这个计数器记录的是正在执行的虚拟机字节码指地址；如果正在执行的是Native方法，这个计数器则为空。  

此内存区域是唯一一个在java虚拟机规范中没有规定任何OutOfMemotyError情况的区域。

此区域是线程私有的，由于java虚拟机的多线程是通过线程轮流切换并分配处理器执行时间的方式来实现，在任何一个确定的时间，一个处理器（对多核处理器来说是一个内核）只会执行一条线程中的指令。因此为了线程切换能够恢复到正确的执行位置上，每条线程都有一个独立的线程计算器，各条线程之间计数器互不影响，独立存储，我们叫这类内存区域线程私有的内存。

### java虚拟机栈

虚拟机栈描述的是Java方法执行的内存模型：每个方法在执行的同时都会创建一个栈帧用于储存局部变量表、操作数栈、动态链接、方法出口等信息。每个方法从调用直至完成的过程，就对应着一个栈帧在虚拟机中入栈到出栈的过程。  

栈内存就是虚拟机栈，或者说是虚拟机栈中局部变量表的部分。  

局部变量表存放了编译期可知的各种基本数据类型（boolean、byte、char、short、int、float、long、double）、对象引用类型和returnAddress类型（指向一条字节码指令的地址）  

其中64位长度的long和double类型的数据会占用两个局部变量空间，其余的数据类型只占用1个。  

局部变量表所需要的内存空间在编译期间完成分配，当进入一个方法时，这个方法需要在帧中分配多大的局部变量空间是完全确定的，在方法运行期间不会改变局部变量表的大小。  

Java虚拟机规范对这个区域规定了两种异常状况：如果线程请求的栈深度大于虚拟机所允许的深度，将抛出StackOverflowError异常。如果虚拟机扩展时无法申请到足够的内在，就会跑出OutOfMemoryError异常。

### 本地方法栈

本地方法栈和虚拟机栈发挥的作用是非常类似的，他们的区别是虚拟机栈为虚拟机执行Java方法（也就是字节码）服务，而本地方法栈则为虚拟机使用到Native方法服务。  

本地方法栈区域也会抛出StackOverflowError和OutOfMemoryError异常。

### java堆

堆是Java虚拟机所管理的内存中最大的一块，Java堆是被所有线程共享的一块内存区域，在虚拟机启动的时候创建，此内存区域的唯一目的是存放对象实例，几乎所有的对象实例都在这里分配内存。所有的对象实例和数组都在堆上分配。  

Java堆是垃圾收集器管理的主要区域，采用分代收集算法。Java堆细分为新生代（Eden、From Survivor、To Survivor）和老年代。划分的目的都是为了更好的回收内存，或者更快地分配内存。  

Java堆可以处于物理上不连续的内在空间中，只要逻辑上是连续的即可。如果在堆中没有完成实例分配，并且堆也夫法在扩展时将会抛出OutOfMemoryError异常。

### 方法区

方法区它用于储存已被虚拟机加载的类信息、常量、静态变量、即时编译器编译后的代码等数据。 HotSpot虚拟机的设计团队把GC分代收集扩展至方法区，或者说使用永久代来代替方法区。在目前已经发布的JDK1.7的HotSpot中，已经把原本放在永久代的字符串常量池移出了。

除了Java堆一样不需要连续的内存和可以选择固定大小或者可扩展外，还可以选择不实现垃圾收集。这个区域的内存回收目标主要是针对常量池的回收和卸载。

当方法区无法满足内存分配需求时，将抛出OutOfMemoryError异常。

### 运行时常量池

它是方法区的一部分。Class文件中除了有类的版本、字段、方法、接口等描述信息外、还有一项信息是常量池，用于存放编译期生成的各种字面量和符号引用，这部分内容将在类加载后进入方法区的运行时常量池中存放。  

运行时常量池相对于Class文件常量池，具有动态性，运行期间也可以将新的常量放入常量池，比如String类的intern()方法。

Java语言并不要求常量一定只有编译器才产生，也就是可能将新的常量放入池中，这种特性被开发人员利用得比较多的便是String类的intern()方法。

### 直接内存

并不是运行时区域的一部分，JDK 1.4加入的NIO 它可以使用Native函数库直接分配堆外内存，然后通过Java堆中的DirectByteBuffer对象作为这块内存的引用进行操作。

## Hotspot虚拟机对象
### 对象的创建
#### 检查

虚拟机遇到一条new指令时，首先将去检查这个指令的参数是否能在常量池中定位到一个类的符号引用，并且检查这个符号引用代表的类是否已经被加载、解析和初始化，如果没有，那必须先执行相应的类加载过程。

#### 分配内存

接下来将为新生对象分配内存，对象所需内存在类加载完毕之后就可以完全确定，为对象分配内存空间的任务等同于把一块确定的大小内在从Java堆中划分出来。

假设Java堆中内存是绝对规整的，所有用过的内存放在一遍，空亲的内存放在另一边，中间放着一个指针作为分界点的指示器，那所分配内存就仅仅是把那个指针指向空亲空间那边挪动一段与对象大小相等的距离，这个分配方式叫做“指针碰撞”。

如果Java堆中的内存并不是规整的，已使用的内存和空亲的内存相互交错，那就没办法简单地进行指针碰撞了，虚拟机就必须维护一个列表，记录上哪些内存块是可用的，在分配的时候从列表中找一块足够大的空间划分给对象实例，并更新列表上的记录，这种分配方式成为“空闲列表”

选择哪种分配方式由Java堆是否规整决定，而Java堆是否规整又由所采用的垃圾收集器是否带有压缩整理功能决定。Serial，ParNew等带有Compact过程的收集器，采用的分配算法是“指针碰撞”。而CMS这种基于Mark-Sweep算法的收集器，通常采用“空闲列表”分配方式。

在分配内存的时候会出现并发的问题，比如在给A对象分配内在的时候，指针还没有来得及修改，对象B又同时使用了原来的指针进行了内存的分片。

有两种解决方案：

- 对分配的内存进行同步处理：CAS失败重试的方式保证更新操作的原子性。
- 把内存分配的动作按照线程划分在不同的空间之中进行，即每个线程在Java堆中分配一块小内存，称为本地缓冲区，哪个线程需要分配内存，就需要在本地缓冲区上进行，只有当缓冲区用完并分配新的缓冲区的时候，才需要同步锁定。

#### Init

执行new指令之后会接着执行Init方法，进行初始化，这样一个对象才算产生出来。

### 对象的内存布局

在Hotspot虚拟机中，对象在内存中储存的布局可以分为3块区域：对象头、实例数据、对齐填充。

对象头包括两部分：

- 储存对象自身的运行时数据，如哈希码、GC分代年龄、锁状态标志、线程持有的锁、偏向线程ID、偏向时间戳。
- 另一部分是指类型指针，即对象指向它的类元数据的指针，虚拟机通过这个指针来确定这个对象是哪个类的实例。另外，如果对象是一个Java数组，那在对象头中还必须有一块用于记录数组长度的数据，因为虚拟机可以通过普通Java对象的元数据信息确定Java对象的大小，但是从数组的元数据中无法确定数组的大小。

实例数据：

是对象正常储存的有效信息，也是程序代码中所定义的各种类型的字段内容。无论是从父类继承下来的，还是在子类中定义的，都需要记录下来。

对齐填充：

不是必然存在的，仅仅是起到占位符的作用。对象的大小必须是8字节的整数位，而对象头刚好是8字节的整数倍（1倍或者2倍），当实例数据没有对齐的时候，就需要通过对齐填充来补全。

### 对象的访问定位

- 使用句柄访问

Java堆中将会划分出一块内存来作为句柄池，reference中存储的就是对象的句柄地址，而句柄中包含了对象实例数据与类型数据各自的具体地址。

优势：reference中存储的是稳定的句柄地址，在对象被移动（垃圾收集时移动对象是普遍的行为）时，只会改变句柄中的实例数据指针，而reference本身不需要修改。

<p align="center">
    <img src="https://raw.githubusercontent.com/xiaopingZhangCn/KnowledgeBase/master/Java/resource/687309-20180430172222987-651423543.jpg" width="60%"> 
    <br />    <small> 句柄方式访问对象 </small>
</p>

- 使用直接指针访问

Java堆对象的布局就必须考虑如何访问类型数据的相关信息，而refernce中存储的就是对象的地址。

优势：速度更快，节省了一次指针定位的时间开销，由于对象的访问在Java中非常频繁，因此这类开销积少成多后也是一项非常可观的执行成本。

<p align="center">
    <img src="https://raw.githubusercontent.com/xiaopingZhangCn/KnowledgeBase/master/Java/resource/123434123123.jpg" width="60%">
    <br />    <small> 直接指针方式访问对象 </small>
</p> 

## OutOfMemoryError 异常（OOM）
### Java堆溢出

Java堆用于存储对象实例，只要不断的创建对象，并且保证GC Roots到对象之间有可达路径来避免垃圾回收机制清除这些对象，那么在数量到达最大堆的容量限制后就会产生内存溢出异常。

如果是内存泄漏，可进一步通过工具查看泄漏对象到GC Roots的引用链。于是就能找到泄漏对象是通过怎样的路径与GC Roots相关联并导致垃圾收集器无法自动回收它们的。掌握了泄漏对象的类型信息及GC Roots引用链的信息，就可以比较准确地定位出泄漏代码的位置。

如果不存在泄漏，换句话说，就是内存中的对象确实都还必须存活着，那就应当检查虚拟机的堆参数-Xmx与-Xms，与机器物理内存对比看是否还可以调大，从代码上查检是否存在某些对象生命周期过长、持有状态时间过长的情况，尝试减少程序运行期的内存消耗。

-Xms 堆的最小值；-Xmx 堆的最大值；-XX:+HeapDumpOnOutOfMemoryError  内存溢出异常时Dump出当前的内存堆转储快照以便日后分析

### 虚拟机栈和本地方法栈溢出

对于HotSpot来说，虽然-Xoss参数（设置本地方法栈大小）存在，但实际上是无效的，栈容量只由-Xss参数设定。关于虚拟机栈和本地方法栈，在Java虚拟机规范中描述了两种异常：

- 如果线程请求的栈深度大于虚拟机所允许的最大深度，将抛出StackOverflowError异常。
- 如果虚拟机在扩展栈时无法申请到足够在内存空间，则抛出OutOfMemoryError异常。

在单线程下，无论由于栈帧太大还是虚拟机栈容量太小，当内存无法分配的时候，虚拟机抛出的都是StackOverflowError异常。

如果是多线程导致的内存溢出，与栈空间是否足够大并不存在任何联系，这个时候每个线程的栈分配的内存越大，反而越容易产生内存溢出异常。解决的时候是在不能减少线程数或更换64位的虚拟机的情况下，就只能通过减少最大堆和减少栈容量来换取更多的线程。

### 方法区和运行时常量池溢出

String.intern()是一个Native方法，它的作用是：如果字符串常量池中已经包含一个等于此String对象的字符串，则返回代表池中这个字符串的String对象；否则，将此String对象包含的字符串添加到常量池中，并且返回此String对象的引用。

由于常量池分配在永久代中，可以通过-XX:PermSize和-XX:MaxPermSize限制方法区大小，从而间接限制其中常量池的容量。

String.intern()：

- JDK1.6 intern()方法会把首次遇到的字符串实例复制到永久代，返回的也是永久代中这个字符串实例的引用，而由StringBuilder创建的字符串实例在Java堆上，所以必然不是一个引用。
- JDK1.7 intern()方法的实现不会再复制实例，只是在常量池中记录首次出现的实例引用，因此intern()返回的引用和由StringBuilder创建的那个字符串实例是同一个。

多次调用String.intern()方法可以产生内存溢出异常。JDK 1.6之间，可以通过 -XX:PermSize  和 -XX:MaxPermSize  限制永久代大小，从而达到限制方法区大小的目的

### 本地直接内存溢出

可以通过 -XX:MaxDirectMemorySize 指定。如果不指定，则默认和Java堆最大值（-Xmx 指定）一样

## java垃圾收集器

程序计数器、虚拟机栈、本地方法栈3个区域随线程而生，随线程而来，在这几个区域内就不需要过多考虑回收的问题，因为方法结束或者线程结束时，内存自然就跟随着回收了。

栈中的栈帧随着方法的进入和退出就有条不紊的执行者出栈和入栈的操作，每一个栈分配多少个内存基本都是在类线路构确定下来的时候就已经确定了，这几个区域内存分配和回收都具有确定性。

而堆和方法区则没，一个接口的实现是多种多样的，多个实现类需要的内存可能不一样，一个方法中多个分支需要的内存也不一样，我们只能在程序运行的期间知道需要创建那些对象，分配多少内存，这部分的内存分配和回收都是动态的。

### 判断对象存活
- 引用计数器法

给对象添加一个引用计数器，每当由一个地方引用它时，计数器值就加1；当引用失效时，计数器就减1；任何时刻计数器为0的对象不是不可能再被使用的。

优点是判断简单，效率也很高。缺点是无法解决相互循环引用的问题。

- 可达性分析算法 

通过一系列的成为“GC Roots”的对象作为起始点，从这些节点开始向下搜索，搜索所走过的路径成为引用链，当一个对象GC Roots没有任何引用链相连时，则证明此对象时不可用的。

Java语言中的GC Roots的对象包括下面几种：

- 虚拟机栈（栈桢中的本地变量表）中引用的对象
- 方法区中类静态属性引用的对象
- 方法区中常量引用的对象
- 本地方法栈INI（Native方法）引用的对象

### 引用

- 强引用：就是在程序代码之中变遍存在的，类似Object obj = new Object()这类的引用，只要强引用还存在，垃圾收集器永远不会回收掉被引用的对象
- 软引用：用来描述一些还有用但并非必须的元素。对于它在系统将要发生内存溢出异常之前，将会把这些对象列进回收范围之中进行第二次回收，如果这次回收还没有足够的内存才会抛出内存溢出异常
- 弱引用：用来描述非必须对象的，但是它的强度比软引用更弱一些，被引用关联的对象只能生存到下一次垃圾收集发生之前，当垃圾收集器工作时，无论当前内存是否足够都会回收掉只被弱引用关联的对象
- 虚引用：唯一目的就是能在这个对象被收集器回收时收到一个系统通知

### finalize()方法

当对象进行了可达性分析，没有与GC Roots相连的引用链，将会被第一次标记，并根据是否需要执行finalize()方法进行一次筛选，对象没有重写finalize()或者虚拟机已经调用过finalize()，都被视为不需要执行。

如果对象有必要执行finalize方法，会被放入到F-Queue队列中，并在稍后由虚拟机自动创建的低优先级的Finalize线程去触发它，并不保证等待此方法执行结束。

如果对象在finalize方法执行中，重新和GC Roots产生了引用链，则可以逃脱此次被回收的命运，但finalize方法只能运行一次，所以并不能通过此方法逃脱下一次被回收。

不建议使用这个方法，建议大学完全忘记这个方法的存在。

### 回收方法区

永久代的垃圾收集主要回收两部分内容：废弃常量和无用的类

- 废弃常量：假如一个字符串abc已经进入了常量池中，如果当前系统没有任何一个String对象abc，也就是没有任何String对象引用常量池的abc常量，也没有其它地方引用的这个字面量，这个时候发生内存回收这个常量就会被清理出常量池。
- 无用的类（满足以下第件才可以被回收卸载）：
    - 该类所有的实例都已经被回收，就是Java堆中不存在该类的任何实例
    - 加载该类的ClassLoader已经被回收
    - 该类的java.lang.Class对象没有在任何地方被引用，无法再任何地方通过反射访问该类的方法
    - 通过垃圾收集算法
- HotSpot虚拟机通过-Xnoclassgc参数进行控制是否启用类卸载功能。在大量使用反射、动态代理、CGLib等框架，需要虚拟机具备类卸载功能，避免方法区发生内在溢出。

### 垃圾回收算法
#### 标记-清除算法（mark-sweep）

算法分为标记和清除两个阶段：首先标记出所有需要回收的对象，在标记完成后统一回收所有被标记的对象。

不足：一个是效率问题，标记和清除两个过程的效率都不高；另一个是空间问题，标记清楚之后会产生大量不连续的内存碎片，空间碎片太多可能会导致以后再程序运行过程中需要分配较大的对象时，无法找到足够的连续内存而不得不提前触发另一次拉圾收集动作。

#### 复制算法（copying）

它将可用内存按照容量划分为大小相等的两块，每次只使用其中的一块。当这块的内存用过多了，就将还存活着的对象复制到另外一块上面，然后再把已使用过的内在空间一次清理掉。这样使用每次都是对整个半区进行内存回收，内存分配时也就不用考虑内在碎片等复杂情况，只要移动堆顶指针，按顺序分配内存即可。

不足：将内存缩小了原来的一半

实际中我们并不需要按照1：1比例来划分内存空间，而是将内存分为一块较大的Eden空间和两块较小的Survivor空间，默认比例8：1。回收时，将Eden和一个Survival的存活对象全部放入到另一个Survival空间中，最后清理掉刚刚的Eden和Survival空间。当Survival空间不够时，由老年代进行内存分配担保。

#### 标记整理算法（mark-compact）

根据老年代对象的特点，先标记存活对象。让所有存活的对象都向一端移动，然后直接清理掉端边界以外的内存

#### 分代收集算法

只是根据对象存活周期的不同将内存划分为几块。一般是把java堆分为新生代和老年代，这样就可以根据各个年代的特点采用最适当的收集算法。在新生代中，每次垃圾收集时都发现有大批对象死去，只有少量存活，那就选用复制算法，只需要付出少量存活对象的复制成本就可以完成收集。而老年代中因为对象存活率高、没有额外空间对它进行分配担保，就必须使用标记清理或者标记整理算法来进行回收。

### HotSpot算法实现

枚举根节点实现

- 可达性分析时会进行GC停顿，停顿所有的java线程。
- HotSpot进行的是准确式GC，当系统停顿下来后，虚拟机有办法得知哪些地方存在着对象引用，HotSpot中使用一组称为OopMap的数据结构来达到这个目的

安全点

- HotSpot没有为每个指令都生成OopMap，只在特定的位置记录这些信息，这些位置称为安全点。安全点的选定不能太少，也不能太频繁，安全点的选定以“是否让程序长时间执行”为标准
- 采用主动式中断的方式让所有线程都跑到最近的安全点上停顿下来。设置一个标志，各个程序执行的时候轮询这个标志，发现中断标志为真时，自己就中断挂起

安全区域

- 解决没有分配CPU时间的暂时不执行的程序停顿

### 垃圾收集器

如果两个收集器之间有连线，说明可以搭配使用。没有最好的收集器，也没有万能的收集器，只有对应具体应用最合适的收集器。
<p align="center">
    <img src="https://github.com/xiaopingZhangCn/KnowledgeBase/blob/master/Java/resource/201905300001.png" width="60%">
    <br />
</p>

- Serial收集器

新生代收集器，单线程回收，它在进行垃圾收集时，必须暂停其它所有的工作线程，直至收集结束。优点在于，简单高效，对于运行在Client模式下的虚拟机来说是一个好的选择（比如用户的桌面应用）

参数：-XX:UseSerialGC 打开开关后，使用Serial +Serial Old的收集器组合进行内存回收。

- ParNew收集器

新生代收集器，Serial的多线程版本，除了Serial收集器之外，只有它能与CMS收集器配合工作。

-XX:+UseConcMarkSweepGC 选项后默认的新生代收集器，也可以使用-XX:+UseParNewGC 选项来强制指定它

ParNew收集器在单CPU的环境中，效果不如Serial好，随着CPU的增加，对于GC时系统资源的利用还是很有效的

默认开启的收集线程数和CPU数相等，可以使用-XX:ParallelGCTreads指定

并行：指多条垃圾收集器线程并行工作，但此时用户线程仍然外于等待状态。并发：指用户线程与垃圾收集线程同时执行（不一定是并行的，可能会交替执行），用户程序在继续执行，而垃圾收集程序运行于另一个CPU上。

- Parallel Scavenge

新生代收集器，并行收集器，复制算法，和其它收集器不同，关注点是吞吐量（垃圾回收时间占总时间的比例）。提供了两个参数用于控制吞吐量。

-XX:MaxGCPauseMillis 最大垃圾收集停顿时间，减少GC的停顿时间是以牺牲吞吐量和新生代空间来换取的，不是设置的越小越好

-XX:GCTimeRatio 设置吞吐量大小，值是大于0小于100的范围，相当于吞吐量倒数，比如设置成99，吞吐量就为1/（1+99） = 1%

-XX:UseAdaptiveSizePolicy 这是一个开关参数，打开之后，就不需要设置新生代大小（-Xmn）、Eden和Survival的比例（-XX:SurvivalRatio）、晋升老年代对象年龄（-XX:PertenureSizeThreshold）等细节参数，收集器会自动调节这些参数。

- Serial Old 收集器

单线程收集器，老年代，主要意义是在Client模式下的虚拟机使用。在Server端，用于在JDK1.5以及之前版本和Parallel Scavenge配合使用，或者作为CMS的后备预案。

- Parallel Old 收集器

是Parallel Scavenge的老年代版本。在注重吞吐量的场合，都可以优先考虑Parallel Scavenge 和Palallel Old 配合使用

- CMS收集器

CMS并非没有暂停，而是用两次短暂停来替代串行标记整理算法的长暂停，它的收集周期是这样：

　　初始标记(CMS-initial-mark) -> 并发标记(CMS-concurrent-mark) -> 重新标记(CMS-remark) -> 并发清除(CMS-concurrent-sweep) ->并发重设状态等待下次CMS的触发(CMS-concurrent-reset)。

　　其中的1，3两个步骤需要暂停所有的应用程序线程的。第一次暂停从root对象开始标记存活的对象，这个阶段称为初始标记；第二次暂停是在并发标记之后， 暂停所有应用程序线程，重新标记并发标记阶段遗漏的对象（在并发标记阶段结束后对象状态的更新导致）。第一次暂停会比较短，第二次暂停通常会比较长，并且 remark这个阶段可以并行标记。
　　而并发标记、并发清除、并发重设阶段的所谓并发，是指一个或者多个垃圾回收线程和应用程序线程并发地运行，垃圾回收线程不会暂停应用程序的执行，如果你有多于一个处理器，那么并发收集线程将与应用线程在不同的处理器上运行，显然，这样的开销就是会降低应用的吞吐量。Remark阶段的并行，是指暂停了所有应用程序后，启动一定数目的垃圾回收进程进行并行标记，此时的应用线程是暂停的

　　CMS收集器是基于标记清除算法实现的，整个过程分为4个步骤：

　　1.初始标记2.并发标记3.重新标记4.并发清除

　　优点：并发收集、低停顿

　　缺点：

　　1.CMS收集器对CPU资源非常敏感，CMS默认启动的回收线程数是（CPU数量+3）/4，

　　2.CMS收集器无法处理浮动垃圾，可能出现Failure失败而导致一次Full G场地产生

　　3.CMS是基于标记清除算法实现的

- G1收集器

包括新生代和老年代的垃圾回收。和其他收集器相比的优点：并行和并发，分代收集，标记-整理，可预测的停顿。垃圾回收分为以下几个步骤：

- 初始标记：标记GC Roots能够直接关联到的对象，这阶段需要停顿线程，时间很短
- 并发标记：进行可达性分析，这阶段耗时较长，可与用户程序并发执行
- 最终标记：修正发生变化的记录，需要停顿线程，但是可并行执行
- 筛选回收：对各个Region的回收价值和成本进行排序，根据用户所期望的停顿时间来执行回收计划

### 内存分配与回收策略

MinorGC:清理新生代、MajorGC:清理老年代、FullGC:清理整个堆空间

#### 对象优先在Eden分配

对象优先在新生代Eden分配，当新生区Eden没有足够的空间分配时，通过分配担保机制提前转移到老年代中去

#### 大对象直接进入老年代

大对象是指需要大量连续内存空间的Java对象，最典型的大对象就是那种很长的字符串以及数组。这样做的目的是避免Eden区及两个Servivor之间发生大量的内存复制，虚拟机提供了参数 -XX:PretenureSizeThreshold（只对Serial，PerNew两个回收器起效），令大于这个值得对象直接在老年代分配，避免了Eden和两个Survival之间发生大量的内存复制。

#### 长期存活的对象将进入老年代

虚拟机给每个对象定义了对象年龄计数器（Age），如果对象在Eden出生，经过第一次Minor GC后依然存活，并且能被Survival容纳的话，将被移动到Survival，对象年龄设为1。对象在Survival中每熬过一次Major GC，年龄就增加1，达到一定程度（默认是15），就会被晋升到老年代。对象晋升老年代的阈值，可以通过参数-XX:MaxTenuringThreShold指定

#### 动态对象年龄判定

为了更好地适应不同程序的内存状况，虚拟机并不是永远地要求对象的年龄必须达到了MaxTenuringThreshold才能晋级到老年代，如果在Servivor空间中相同年龄所有对象的大小总和大于Survivor空间的一半，年龄大于或等于该年龄的对象就可以直接进入到老年代，无须登到MaxTenuringThreshold中要求的年龄

#### 空间分配担保

在发生Minor GC 之前，虚拟机会检查老年代最大可用的连续空间是否大于新生代所有对象总空间，如果这个条件成立，那么Minor GC可以确保是安全的。如果不成立，则虚拟机会查看HandlePromotionFailure设置值是否允许担保失败。如果允许那么会继续检查老年代最大可用的连续空间是否大于晋级到老年代对象的平均大小，如果大于，将尝试进行一次Minor GC，尽管这次MinorGC 是有风险的：如果小于，或者HandlePromotionFailure设置不允许冒险，那这时也要改为进行一次Full GC

## 虚拟机类加载机制

虚拟机把描述类的数据从Class文件加载到内存，并对数据进行校验、转换解析和初始化，最终形成可以被虚拟机直接使用的java类型，这就是虚拟机的类加载机制，在java语言里面，类型的加载、连接和初始化过程都是在程序运行期间完成的。

### 类加载的时机

从类被加载到虚拟机内存中开始到卸载为止，整个生命周期包括：加载、验证、准备、解析、初始化、使用和卸载7个阶段。

加载、验证、准备、初始化、卸载这5个阶段的顺序是确定的，类的加载过程必须按照这种顺序按部就班地开始，而解析阶段不一定，它在某些情况下可以再初始化阶段之后再开始，这个是为了支持java语言运行时绑定（也称为动态绑定或晚期绑定）

虚拟机规范规定有且只有5种情况必须立即对类进行初始化：

- 遇到new、getstatic、putstatic、invokestatic这4条字节码指令时，如果类没有进行初始化，则需要触发其初始化。生成这4条指令的最常见的java代码场景是：使用new关键字实例化对象、读取或设置一个类的静态属性（被final修饰、已在编译期把结果放入常量池的静态字段除外）的时候，以及调用一个类的静态类的方法的时候。
- 使用java.lang.reflect包的方法对类进行反射调用的时候，如果类没有进行过初始化，则需要先触发其初始化。
- 当初化一个类的时候，如果发现其父类还没有进行过初始化，则需要先触发其父类的初始化。
- 当虚拟机启动的时候，用户需要指定一个要执行的主类（包括main()方法的那个类），虚拟机会先初化这个主类。
- 当使用JDK1.7的动态语言支持时，如果一个java.lang.invoke.MethodHandle实例最后的解析结果REF_getStatic、REF_putStatic、REF_invokeStatic的方法句柄，并且这个方法句柄所对应的类没有进行过初始化，则需要先触发其初始化。

被动引用：

- 通过子类引用父类的静态字段，不会导致子类初始化。
- 通过数组定义来引用类，不用触发此类的初始化。
- 常量在编译阶段会存入调用类的常量池中，本质上并没有直接引用到定义常量的类，因此不会触发定义常量的类的初始化。

接口的初始化：

接口在初始化时，并不要求其父接口全部完成类初始化，只有正在使用父接口的时候（如引用接口中定义的常量）才会初始化。

### 类加载过程

#### 加载
#### 验证
#### 准备
#### 解析
#### 初始化


### 类的加载器

#### 双亲委派模型
#### 双亲委派模型的工作过程
#### 优点

## Java内存模型与线程
### 内存间的交互操作
### 重排序
### 对于volatile型变量的特殊规则
### 对于long和double型变量的特殊规则
### 原子性、可见性和有序性
### 先行发生原则
### Java线程调度
### 状态转换
## 线程安全
### 线程安全的实现方法
### 锁优化
## 逃逸分析
## 虚拟机性能监控与故障处理工具
### JPS
### Jstat 监视JVM内存工具
### Jinfo 查看和修改JVM运行参数
### Jmap 命令用于生成Heap dump文件
### Jstack Java堆栈跟踪工具
## 常见JVM配置说明
### G1
### CMS + ParNew
### 参数说明
## JVM调优案例分析与实践
### Minor GC、Major GC、Full GC之间的区别
### 常用命令
### 问题排查