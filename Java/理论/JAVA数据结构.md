# JAVA数据结构
## 基础知识
### 基本类型
#### 4种整型、2种浮点型、1种字符型、1种布尔型

|类型|存储空间|范围|
|:----|:----:|----:|
|int|**32**bit|[-2147483648,2147483647]|
|short|**16**bit|[-32768,32767]|
|long|**64**bit|[-9223372036854775808,9223372036854775807]|
|byte|**8**bit|[-128,127]|
|float|**32**bit|[-3.4E38,3.4E38]|
|double|**64**bit|[-1.7E308,1.7E308]|
|char|**16**bit|Unicode字符|
|boolean|**1**bit|true,false|

#### 基本类型转换
- 自动转换
    - byte &rarr; short &rarr; int &rarr; long &rarr; float &rarr; double
    - char &rarr; int &rarr; long &rarr; float &rarr; double
        - 注：char 是单个字符时，按字符对应的ASCII码
- 强制转换（会存在失真情况）
    - double &rarr; float &rarr; long &rarr; int &rarr; short &rarr; byte

#### 运算符
- 算术运算符：+（加）、-（减）、*（乘）、/（除）、%（求余）
- 关系运算符：

    |运算符|含义|备注|
    |:----:|:----|:----|
    |==|是否相等||
    |<|小于||
    |&gt;|大于||
    |<=|小于等于||
    |&gt;=|大于等于||
    |!=|不等于||
    |&&|逻辑与||
    |\|\||逻辑或||
    |！|逻辑非||
    |&|与|转为2进制，两个数都为1则为1，否则为0|
    |\||或|转为2进制，两个数只要有一个为1则为1，否则为0|
    |^|异或|2进制的比较，相同为0，不相同为1，得出比较后的2进制转10进制|
    |~|非|转为2进制，并按类型字节补位，然后对2进制取反，0转1，1转0|
    |&gt;&gt;|右移|转2进制后，按类型补位，向右移N位，右边的N位去除，前面的补0|
    |<<|左移|转2进制后，按类型补位，向左移N位，左边的N位去除，后面的补0|
    |&gt;&gt;&gt;|无符号右移|高位填充0的右移,正数和>>没有区别|
    |?:|三元运算符||

### 数据结构
#### 按逻辑结构分：

    集合（数据之间无关系）、线性结构（一对一）、树形结构（一对多）、图形结构（多对多）

#### 按物理结构分：

    顺序存储、链表存储、索引存储、散列存储

#### 常用数据结构：

    数组、栈、队列、链表、树、图、堆、散列表

#### JAVA常用结构：
##### Array([])：
- 在内在中连续存放，大小固定，不可变。

##### List
List接口为Collection直接接口。List所代表的是有序的Collection，即它用某种特定的插入顺序来维护元素顺序。  

用户可以对列表中每个元素的插入位置进行精确地控制，同时可以根据元素的整数索引（在列表中的位置）访问元素，并搜索列表中的元素。  

实现List接口的集合主要有：ArrayList、LinkedList、Vector、Stack    

- ArrayList

ArrayList是一个动态数组，也是我们最常用的集合。它允许任何符合规则的元素插入甚至包括null。
  
每一个ArrayList都有一个初始容量（10），该容量代表了数组的大小。随着容器中的元素不断增加，容器的大小也会随着增加。在每次向容器中增加元素的同时都会进行容量检查，当快溢出时，就会进行扩容操作。所以如果我们明确所插入元素的多少，最好指定一个初始容量值，避免过多的进行扩容操作而浪费时间、效率。 
 
size、isEmpty、get、set、iterator 和 listIterator 操作都以固定时间运行。add 操作以分摊的固定时间运行，也就是说，添加 n 个元素需要 O(n) 时间（由于要考虑到扩容，所以这不只是添加元素会带来分摊固定时间开销那样简单）。  

ArrayList擅长于随机访问。同时ArrayList是非同步的。

- LinkedList

同样实现List接口的LinkedList与ArrayList不同，ArrayList是一个动态数组，而LinkedList是一个双向链表。所以它除了有ArrayList的基本操作方法外还额外提供了get，remove，insert方法在LinkedList的首部或尾部。  

由于实现的方式不同，LinkedList不能随机访问，它所有的操作都是要按照双重链表的需要执行。在列表中索引的操作将从开头或结尾遍历列表（从靠近指定索引的一端）。这样做的好处就是可以通过较低的代价在List中进行插入和删除操作。  

与ArrayList一样，LinkedList也是非同步的。如果多个线程同时访问一个List，则必须自己实现访问同步。一种解决方法是在创建List时构造一个同步的List：   
List list = Collections.synchronizedList(new LinkedList(...));

- Vector

与ArrayList相似，但是Vector是同步的。所以说Vector是线程安全的动态数组。它的操作与ArrayList几乎一样。

- Stack

Stack继承自Vector，实现一个后进先出的堆栈。Stack提供5个额外的方法使得Vector得以被当作堆栈使用。基本的push和pop 方法，还有peek方法得到栈顶的元素，empty方法测试堆栈是否为空，search方法检测一个元素在堆栈中的位置。Stack刚创建后是空栈。


##### Set

Set是一种不包括重复元素的Collection。它维持它自己的内部排序，所以随机访问没有任何意义。与List一样，它同样运行null的存在但是仅有一个。由于Set接口的特殊性，所有传入Set集合中的元素都必须不同，同时要注意任何可变对象，如果在对集合中元素进行操作时，导致e1.equals(e2)==true，则必定会产生某些问题。实现了Set接口的集合有：EnumSet、HashSet、TreeSet。

- EnumSet

是枚举的专用Set。所有的元素都是枚举类型。

- HashSet

 HashSet堪称查询速度最快的集合，因为其内部是以HashCode来实现的。它内部元素的顺序是由哈希码来决定的，所以它不保证set 的迭代顺序；特别是它不保证该顺序恒久不变。

- TreeSet

基于TreeMap，生成一个总是处于排序状态的set，内部以TreeMap来实现。它是使用元素的自然顺序对元素进行排序，或者根据创建Set 时提供的 Comparator 进行排序，具体取决于使用的构造方法。

##### Map

Map与List、Set接口不同，它是由一系列键值对组成的集合，提供了key到Value的映射。同时它也没有继承Collection。在Map中它保证了key与value之间的一一对应关系。也就是说一个key对应一个value，所以它不能存在相同的key值，当然value值可以相同。实现map的有：HashMap、TreeMap、HashTable、Properties、EnumMap。

- HashMap

以哈希表数据结构实现，查找对象时通过哈希函数计算其位置，它是为快速查询而设计的，其内部定义了一个hash表数组（Entry[] table），元素会通过哈希转换函数将元素的哈希地址转换成数组中存放的索引，如果有冲突，则使用散列链表的形式将所有相同哈希地址的元素串起来，可能通过查看HashMap.Entry的源码它是一个单链表结构。

- TreeMap

键以某种排序规则排序，内部以red-black（红-黑）树数据结构实现，实现了SortedMap接口

- HashTable

也是以哈希表数据结构实现的，解决冲突时与HashMap也一样也是采用了散列链表的形式，不过性能比HashMap要低


##### Queue 先进行出FIFO

 队列，它主要分为两大类，一类是阻塞式队列，队列满了以后再插入元素则会抛出异常，主要包括ArrayBlockQueue、PriorityBlockingQueue、LinkedBlockingQueue。另一种队列则是双端队列，支持在头、尾两端插入和移除元素，主要包括：ArrayDeque、LinkedBlockingDeque、LinkedList。
