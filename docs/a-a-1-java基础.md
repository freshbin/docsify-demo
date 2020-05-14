# 一、java基础
## 1、数据类型
一个字节等于8bit
基本数据类：boolean（1），byte（1），char（2），int（4），short（2），float（4），double（8），long（8）
封装类型：Boolean，Byte，Character，Integer，Short，Float，Double，Long
### 1.1 包装类型
Integer x = 2 // 装箱，调用了Integer.valueOf(2)
int y = 2 // 拆箱，调用了x.intValue()

### 1.2 缓存池
new Integer(123)与Integer.valueOf(123)的区别在于：前者每次都会创建一个新对象，而后者会使用缓存池中的对象，多次调用也会是同一个对象的引用，因此就有两个new Integer(x)出来的对象比较会是false，而两个Integer.valueOf()赋值的对象比较出来的结果会是true。
JDK1.8中缓存池常量范围-128~127
Integer的缓存上界默认是127，但是可以在jvm启动的时候，通过XX:AutoBoxCacheMax=<size>参数来调整

## 2、字符串
String定义的字符串不可改变长度，jdk8中用char数组存储数据，该数组被final修饰，String不可变的好处：1、可以缓存hash值，常用语HashMap的key，不可变的特性使得hash值也不可变，因此只需要进行一次计算；2、StringPool存放创建过的String对象；3、安全性：作为网络连接参数；4、线程安全：不可变性具备线程安全，所以可以在多个线程中安全使用。

StringBuffer和StringBuilder可以改变字符串长度，StringBuffer是线程安全的，StringBuilder不是线程安全的。

new String("abc")的方式会创建两个字符串对象，前提是StringPool中没有该字符串对象，那么就会在堆中创建一个字符串对象，StringPool也会创建一个字符串对象，指向这个字符串字面量；该方式创建出来的对象用“==”比较的是两个对象地址，因此是false，而直接用String a = "abc"的方式会从常量池获取，因此创建出来的对象是同一个地址引用，String的intern()方法会把字符串放到StringPool中，之后返回这个字符串的引用。

```java
String s1 = new String("aaa");
String s2 = new String("aaa");
System.out.println(s1 == s2);           // false
String s3 = s1.intern();
String s4 = s1.intern();
System.out.println(s3 == s4);           // true
String s5 = s2.intern();
System.out.println(s3 == s5);           // true
```
```java
String s5 = "bbb";
String s6 = "bbb";
System.out.println(s5 == s6);  // true
```

jdk7之前,StringPool放在运行时常量池中，属于永久代，但是java7之后，被移到了堆中，因为永久代空间有限，大量使用字符串的场景会导致OOM。
## 3、集合

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200511084447314.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2ZyZXNoYmluMDAw,size_16,color_FFFFFF,t_70)

1）List以特定索引存储对象，可以有重复元素，Set不能存放重复元素，Map以键值对存放对象。

2）ArrayList和Vector都是用数组存数据，查询快，插入慢，Vector的方法有synchronized修饰，所以是线程安全，性能比ArrayList差，LinkedList用链表存数据，插入快，查询慢，可以用Collections.synchronizedList()方法转成线程安全的。每次插入的时候都会先进行扩容检验、所以要指定list的大小，不然默认会是10个大小。

3）List和Set继承Collection接口。常用的集合有List和Map。

**4）快速失败和安全失败区别**：针对迭代器而言，安全失败是基于对底层集合做拷贝，安全失败的迭代器不会抛出异常，而快速失败的迭代器会抛出ConcurrentModificationException异常，java.util包下的集合类都是快速失败的，java.util.concurrent包下的容器都是安全失败的。

### 3.1 List
**1）Array和ArrayList**
Array可以包含基本类型和对象类型，ArrayList只能是对象类型，Array的大小固定，ArrayList大小不固定，当处理固定大小的基本数据类型时，可以使用Array，操作会比ArrayList快，因为ArrayList需要自动装箱。ArrayList初始化的时候需要指定大小，不然扩容会变为原来的1.5倍，频繁扩容会影响性能，由于list中的数组元素并不会全部使用，因此会使用transient来修饰声明保存数据的数组不会被序列化，同时ArrayList使用writeObject()和readObject(0方法来控制数组中有元素填充的内容。
**初始化一个线程安全的ArrayList方法**：
1、可以使用`List<String> list = new ArrayList<>(); List<String> synList = Collections.synchronizedList(list);
`，
2、也可以使用concurrent并发包下的CopyOnWriteArrayList，`List<String> list = new CopyOnWriteArrayList<>();`。
CopyOnWriteArrayList是**读写分离**的，写操作在一个复制的数组上进行，读操作在原数组进行。写操作会加锁，写操作完成之后，会让原数组指向新的复制数组。适合读多写少的场景。缺陷：**内存占用高**；数据不一致，**读写操作不能做到实时同步**。

**2）Vector**：线程安全。扩容默认翻倍。

**3）LinkedList**：双向链表，可作队列。

### 3.2 Set
**TreeSet**：基于红黑树实现，有序操作。
**HashSet**：基于哈希表实现，支持快速查找，不支持有序操作。
**LinkedHashSet**：具有HashSet的查找效率，内部使用双向链表维护插入顺序。

### 3.3 Queue
**1）LinkedList**：双向队列。

**2）PriorityQueue**：基于堆结构实现，可以实现优先级队列。

### 3.3 map的分类和常见情况
四个实现类：HashMap，HashTable，LinkedHashMap，TreeMap。map以键值对存在，不允许键重复，但是值可以重复。

1）**HashMap**：允许键重复，不支持线程同步，但是可以用Collections.synchronizedMap方法使HashMap同步，或者使用**ConcurrentHashMap**（推荐使用）。有两个重要参数：容量和负载因子。默认容量是16，负载因子是0.75，当size>16*0.75时，就会发生扩容。put和get方法都会把key计算成hashcode值，该值通过与map存储空间的数组长度做取模运算，结果就是数组中的index下标，hashMap规定数组的长度为2的n次方，那么使用2的n次方-1做位运算结果与取模一样，但是效率高出许多。如果计算出的index已经有数据，那么就使用头插法在该位置生成链表，如果是get，那么就遍历链表，比较key和链表节点一致就返回。
遍历方式：

```java
Iterator<Map.Entry<String, Integer>> entryIterator = map.entrySet().iterator();
        while (entryIterator.hasNext()) {
            Map.Entry<String, Integer> next = entryIterator.next();
            System.out.println("key=" + next.getKey() + " value=" + next.getValue());
        }
```
```java
map.forEach((key,value)->{
    System.out.println("key=" + key + " value=" + value);
});
```
**注意：** 在高并发下，发生扩容，容易出现环形链表，如果访问一个不存在map中的key，该key的index恰好落在环形链表的数组上，那么就会出现死循环。
jdk8中当冲突hashCode的链表长度超过了阈值(默认为8)并且table的长度不小于64（否则扩容一次），链表就会转换为红黑树。
2）**HashTable**：和HashMap一样继承Dictionary类，支持线程同步，不允许空键，写入比HashMap慢。

3）**LinkedHashMap**：是HashMap的子类，保存了记录的插入顺序，因此可以获取对象的插入顺序，LinkedHashMap的遍历速度与容量无关，与实际数据有关，而HashMap的遍历速度和容量有关，因此，当HashMap容量很大，实际数据比较小，速度可能不如LinkedHashMap。

4）**TreeMap**：可以根据自定义的排序规则对键进行排序，因此，如果按照自然顺序或自定义顺序遍历键的时候，用TreeMap会比较好，所以有排序需求的话，用这个map会更好。

## 4、其他
### 4.1 计算小数问题
类似4.0-3-6的结果不是0.4，会出现很长的小数，这是由于二进制存储十进制的误差导致的，可以使用BigDecimal来处理这些需要精度的数字操作，new BigDecimal(String.class)，里面的入参必须是String类型才能正常计算，还可以通过 BigDecimal.setScale(小数位)来进行小数的截取
### 4.2 “==”与“equal”
“==”比较的是内存地址，如果是基本数据类型，就是比较值。“equal”需要重写hashCode()方法，因为不重写hashCode()，那么调用equal也会去比较两个对象的hashCode是否相等，而默认的hashCode()方法其实就是计算对象的内存地址，所以如果不重写hashCode，那么两个对象的内存地址肯定不一样，这就会导致equal比较出来的结果是false。
HashSet和HashMap的key是使用了hashCode()计算对象应该存储的位置，因此如果放入hash表的对象没有重写hashCode()方法，那么就会被当做两个不同的对象。
重写hashCode()的方法，
1、首先初始化一个整型变量，赋值为一个不为0的常数值，比如int result = 17
2、将result*31+成员变量，因为任何数乘以31都可以被jvm优化成(x<<5)-x的形式，移位和减法的操作比普通的乘法效率更高
```java
@Override
public int hashCode() {
    int result = 17;
    result = 31 * result + i; // i是基本类型
    result = 31 * result + s.hashCode(); // s是封装类型或者其他类对象
    return result;
}

```

### 4.3 final，finally与finalize
1）final：final可以修饰变量，方法和类。修饰的变量，基本类型的值不可改变，对象类型指向的地址不可改变；方法不可以重写；类不可以继承。

2）finally是异常处理中，总是执行的部分。

3）finalize：是垃圾处理中回调的方法，调用回收对象的此方法，当该方法被系统调用，表示这个对象即将死亡，但是主动调用并不会触发死亡。

### 4.4 Synchronized，lock和volatile
1）synchronized：是java的关键字，用来修饰一个方法或者一个代码块的时候，能够保证线程安全。加锁顺序：无锁-偏向锁-轻量级锁（自旋锁，CAS）-重量级锁。 修饰静态方法和同步代码块的用法锁是类锁，修饰的成员方法，需要获取的是对象锁。

2）lock：是一个接口，lock发生异常时，如果没有unLock()方法，会造成死锁，因此需要在finally中释放锁，但是synchronized在发生异常会自动释放锁；lock可以让等待的线程响应中断，synchronized等待的线程会一直等待下去；Lock可以知道有没有成功获取锁，而synchronized不可以。

3）volatile：可以保证有序性和可见性。代码经过编译器之后，会重新排序，加入volatile，可以保证代码执行的有序；在翻译成汇编代码时，如果使用了volatile，那么代码会加入一个lock的前缀指令，防止读取旧值之后，其他线程对该值修改，保证读取的旧值和新值一致。

### 4.5 值传递与引用传递
java只有值传递，对象作为参数也不会改变指向的内存地址。
如果方法参数是以对象传入，那么改变该对象的成员变量等字段值，也会相应的改变原对象的字段值，但是如果把该对象赋值为其他对象，那么此时该方法的对象参数和调用时候的对象参数指针已经不是指向同一个了，比如下面这个mc对象在方法里面被赋予了新值，那么就和main方法中的mc不是同一个了，即使在test方法改变mc对象里面的成员字段值，也不会影响到main方法中的mc对象的成员字段值。
```java
public static void main(String[] args) {
	MyClass mc = new MyClass("mc1");
	canChange(mc);
	cannotChange(mc);
}
public static void cannotChange(MyClass mc) {
	mc = new MyClass("newMc"); // 此处mc对象指针指向了新的MyClass对象。
}
public static void canChange(MyClass mc) {
	mc.setName("mc2"); // 这里就会改变外部mc对象的name字段值。
}
```

### 4.6 泛型
泛型可以确保正确类型的对象放入集合，防止运行时出现ClassCastException。不能把`List<String>`传递给一个接收`List<Object>`参数的方法，这会导致编译出错。
1）**extend和super**
上界：<? extend Fruit>：表示所有继承Fruit的子类，只能get，不能add。因为get，都可以用父类Fruit兜底，可以用Fruit fruit= lists.get(0)接收，但是add的话 ，无法确定add的类型。

下界：<? super Apple>：表示Apple的所有父类，只能add，不能get。add的时候，可以add对象Apple的所有子类，因为不管是什么子类，都可以向上转型为Apple及其所有的父类，但是get的时候，无法确定用什么Apple的什么父类接收。
不是特别理解，记住是由于编译器支持向上转型，但不支持向下转型吧。

### 4.7 隐式类型转换
下面代码会编译出错，因为java不支持隐式向下转型
```java
short i = 1;
i = i+1; // 报错
```
但是如果是强制类型转换的话就可以，比如下面的语句
```java
short i = 0;
i+=1;
```
这个+=被编译成下列的表达
```java
i = (short)(i + 1);
```

### 4.8 switch
swith不支持long类型。

### 4.9 static
静态方法中不能有this和super。
静态内部类不需要通过外部类创建，但是非静态内部类需要通过外部类创建；静态内部类不能访问外部类的非静态变量和方法。
静态语句块与普通语句块的初始化顺序如下：
其中静态变量和静态语句块的顺序根据定义的先后顺序执行。
父类（静态变量、静态语句块）
子类（静态变量、静态语句块）
父类（实例变量、普通语句块）
父类（构造函数）
子类（实例变量、普通语句块）
子类（构造函数）

### 4.10 clone
1、不重写clone方法，无法使用该方法
2、重写clone方法之后，需要实现Cloneable接口，否则会抛CloneNotSupportedException异常
3、clone有深拷贝和浅拷贝，浅拷贝之后，拷贝前后的引用类型的成员变量指向的是同一个地址，深拷贝之后才是独立开一块内存地址保存。
4、建议使用拷贝函数或者拷贝工厂来拷贝。

### 4.11 继承
子类重写父类的方法时，访问级别不能低于父类的访问权限，这是为了确保里氏替换原则。
子类继承父类之后，调用子类的方法顺序为
```java
this.func(this)
super.func(this)
this.func(super)
super.func(super)
```

## 5、设计原则
1）单一职责原则：一个类或者方法只做一件事。

2）开闭原则：对修改关闭，对扩展开放。

3）依赖倒转原则：面向接口编程。

4）里氏替换原则：能用父类的地方都能用子类代替。

5）接口隔离原则：接口拥有的方法尽量小。

6）迪米特法则：最少知道原则，一个对象对另一个对象知道的东西尽量少。

## 6、反射
可以通过获取类来操作类的方法和变量。可以使用类对象调用newInstance()方法创建对象，也可以用类对象获取构造器之后调用newInstance()方法创建对象。
主要包含以下三个类：
1）Field：可以使用get()和set()方法读取和修改Field对象关联的字段；
2）Method：可以使用invoke()方法调用与Method对象关联的方法；
3）Constructor：可以使用Constructor的newInstance()创建新的对象。
**反射的优点**：
1）**可扩展性**：应用程序可以利用全限定名创建可扩展对象的实例，来使用来自外部的用户自定义类。
2）**类浏览器和可视化开发环境**：一个类浏览器需要枚举类的成员，可视化开发环境可以从利用反射中可用的类型信息中收益，以帮助程序员编写正确的代码。
3）**调试器和测试工具**：调试器需要能够检查一个类里的私有成员，测试工具可以利用反射来自动地调用类里定义的可被发现的API定义，以确保一组测试中有较高的代码覆盖率。
**反射的缺点：**
1）**性能开销**：反射涉及了动态类型的 解析，所以JVM无法对这些代码优化。
2）**安全限制**：反射要求使用的环境在一个没有安全限制的环境中。
3）**内部暴露**：反射允许访问私有的属性和方法，可能会导致意外，以及破坏代码可移植性。

## 7、异常
Throwable分类两种：Error和Exception。其中Error表示JVM无法处理的错误，Exception分为两种：受检异常和非受检异常。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200510153024520.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2ZyZXNoYmluMDAw,size_16,color_FFFFFF,t_70)

throw：用来明确抛出异常，可以是自定义的异常，写在方法内部。
throws：写在方法上，表明该方法会抛出什么异常。
finally：总会执行。
catch：执行捕获的异常。

## 8、抽象类和接口
区别：1、抽象类用abstract，接口用interface；
2、抽象类可以有默认方法实现，jdk8以前接口不能有方法实现，jdk8以后可以有默认方法实现；
3、子类用extend继承抽象类，用interface实现接口，
4、抽象类可以有构造器，接口不能有构造器；
5、抽象方法可以用public/protected/default等修饰符，接口默认是public修饰符，不能用其他修饰符；
6、一个子类只能继承一个抽象类，可以实现多个接口。
7、接口的字段默认是static和final的。

## 9、类加载机制与双亲委派模型
某个类加载器在接到加载请求时，先委托父类加载器加载，如果父类可以完成加载任务，就成功返回，否则由自己去加载。

好处：防止重复名字的类放在同一个类路径中，类都由模型最顶端的Bootstrap ClassLoader进行加载，双亲委派模型可以保证同一个类在一个类加载环境中只创建一个。

## 10、多线程
使用线程的三种方式：**实现Runnable接口**，**实现Callable接口**，**继承Thread类**，推荐实现Runnable接口，因为java不支持多继承，继承了Thread就不能继承其他类了；而且继承Thread相比实现Runnable开销过大。
### 10.1 上下文切换
 CPU不停的切换线程是非常耗效率的，线程从等待切换到运行状态都是一次上下文切换。解决方案：采用无锁编程，用hashId取模分段，让线程处理各自的分段数据；可以采用CAS算法；需要合理的创建线程数。
 
 ### 10.2 死锁
 解决方案：尽量一个线程只用一个锁；一个线程只占用一个资源；尝试使用定时锁，至少能保证锁最终会被释放。
 
### 10.3 synchroized 
 **三种使用方式**
 1）同步普通方法，锁的是当前对象
 2）同步静态方法，锁的是Class对象
 3）同步块，锁的是()中的对象
 **实现原理**：JVM通过进入、退出对象监视器来实现对方法，同步块的同步。在调用方法前加入monitor.enter指令，在退出和异常处插入monitor.exit指令，没有获取锁的线程会被阻塞到方法入口处。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200511164011431.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2ZyZXNoYmluMDAw,size_16,color_FFFFFF,t_70)
 **锁优化**： synchronized称为重量级锁，为了减少获取和释放锁带来的消耗引入了偏向锁和轻量锁。
 **轻量锁：** 轻量锁的解锁过程利用CAS来实现。具体不是很懂，大概好像是说会记录对象头的Mark Word，当需要修改记录的时候，再比较一下刚刚的记录值，一致就进入同步代码，不一致，jvm会检查锁对象的Mark Word是否指向当前线程的锁记录，是则进入同步代码块，否则就膨胀为重量级锁（也不知道是不是要自旋一定次数之后膨胀为重量级锁）
 
 ### 10.4 多线程三大核心

 **1）原子性：** 一个操作要么全部成功，要么全部失败。
 几个原子性的体现代码如下，可见volatile不支持原子性：
```java
public class AtomicTest {
    private int inc = 0;
    private volatile int incVolatile = 0;
    private AtomicInteger atomicInteger = new AtomicInteger();
    private int incSynchronized = 0;

    public void addInc() {
        inc += 1;
    }
    public int getInc() {
        return inc;
    }

    public void addIncVolatile() {
        incVolatile += 1;
    }
    public int getIncVolatile() {
        return incVolatile;
    }

    public void addIncAtomicInteger() {
        atomicInteger.incrementAndGet();
    }
    public int getIncAtomicInteger() {
        return atomicInteger.get();
    }

    public synchronized void addIncSynchronized() {
        incSynchronized += 1;
    }
    public int getIncSynchronized() {
        return incSynchronized;
    }

    public static void main(String[] arg) throws InterruptedException {
        int threadCount = 1000;
        AtomicTest atomicTest = new AtomicTest();
        CountDownLatch countDownLatch = new CountDownLatch(threadCount);
        ThreadPoolExecutor threadPoolExecutor = new ThreadPoolExecutor(0, Integer.MAX_VALUE,
                60L, TimeUnit.SECONDS,
                new SynchronousQueue<Runnable>());
        for(int i = 0; i < threadCount; i++) {
            threadPoolExecutor.execute(() -> {
                atomicTest.addInc();
                atomicTest.addIncVolatile();
                atomicTest.addIncAtomicInteger();
                atomicTest.addIncSynchronized();
                countDownLatch.countDown();
            });
        }
        countDownLatch.await();
        threadPoolExecutor.shutdown();
        System.out.println("inc:" + atomicTest.getInc());
        System.out.println("incVolatile:" + atomicTest.getIncVolatile());
        System.out.println("incAto:" + atomicTest.getIncAtomicInteger());
        System.out.println("incSyn:" + atomicTest.getIncSynchronized());
    }
}
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200513094030843.png)

 **2）可见性：** 线程会从缓存先读取数据，之后再更新到主内存，而volatile能够保证修饰的变量修改之后会立即刷新到主线程，并将其余缓存中该变量的值清空，其他线程就会到主内存读取最新值。synchronized和final修饰的字段也可以保证可见性。
 
 **3）顺序性：** 一段代码的执行顺序会由于指令重排序，而引发一些bug，比如下面的单例模式
```java
public class Singleton {
        private static volatile Singleton singleton;
        private Singleton() {
        }
        public static Singleton getInstance() {
            if (singleton == null) {
                synchronized (Singleton.class) {
                    if (singleton == null) {
                        singleton = new Singleton();
                    }
                }
            }
            return singleton;
        }
    }
```
singleton=new Singleton()的代码分为三步
1）分配内存空间
2）初始化对象
3）将singleton对象指向分配的内存地址。
如果不使用volatile，那么可能第三步会在第二步之前执行，导致某个线程拿到的单例对象还没有初始化，而报错。
又比如下面的代码
```java
private volatile boolean flag ;
    private void run(){
        new Thread(new Runnable() {
            @Override
            public void run() {
                while (flag) {
                    doSomeThing();
                }
            }
        });
    }
    private void stop(){
        flag = false ;
    }
```
如果不加volatile，那么如果有个线程修改了flag的值，但是没有刷新到主内存，那么线程的方法就会不断执行。
因此，volatile可以保证可见性和顺序性（禁止指令重排序），但是无法保证原子性。

### 10.5 ReentrantLock
分为**公平锁**与**非公平锁**，公平锁每次都判断队列是否有其他线程，有的话就不尝试获取锁；而非公平锁是抢占模式，不需要管队列情况。因此公平锁会造成大量的线程上下文切换，非公平锁的效率就比公平锁高。
### 10.6 循环打印奇偶数
主要是用到线程等待（wait）和唤醒（notify）

```java
package com.freshbin.other.thread.printEvenAndOddNum;

/**
 * 交替打印奇偶数
 *
 * @author freshbin
 * @date 2020/5/12 9:41
 */
public class AlternateNum {
    private volatile int startNum = 0;

    public int getStartNum() {
        return this.startNum;
    }

    public void setStartNum(int startNum) {
        this.startNum = startNum;
    }

    public static void main(String[] arg) {
        AlternateNum alternateNum = new AlternateNum();
        Thread threadEvenNum = new Thread(new EvenNum(alternateNum));
        threadEvenNum.setName("evenThread");
        Thread threadOddNum = new Thread(new OddNum(alternateNum));
        threadOddNum.setName("oddThread");
        threadEvenNum.start();
        threadOddNum.start();
    }
}
```
```java
package com.freshbin.other.thread.printEvenAndOddNum;

/**
 * 打印奇数
 *
 * @author freshbin
 * @date 2020/5/12 9:42
 */
public class EvenNum implements Runnable {

    private AlternateNum alternateNum;

    EvenNum(AlternateNum alternateNum) {
        this.alternateNum = alternateNum;
    }

    @Override
    public void run() {
        while (alternateNum.getStartNum() <= 100) {
            synchronized (alternateNum) {
                if (alternateNum.getStartNum() % 2 != 0) {
                    System.out.println(Thread.currentThread().getName() + "==奇数打印：" + alternateNum.getStartNum());
                    alternateNum.setStartNum(alternateNum.getStartNum() + 1);
                    alternateNum.notify();
                } else {
                    try {
                        alternateNum.wait();
                    } catch (InterruptedException e) {
                        System.out.println("EvenNum.run中调用线程等待异常！");
                    }
                }
            }
        }
    }
}
```
```java
package com.freshbin.other.thread.printEvenAndOddNum;

/**
 * 打印偶数
 *
 * @author freshbin
 * @date 2020/5/12 9:42
 */
public class OddNum implements Runnable {
    private AlternateNum alternateNum;

    OddNum(AlternateNum alternateNum) {
        this.alternateNum = alternateNum;
    }

    @Override
    public void run() {
        while (alternateNum.getStartNum() <= 100) {
            synchronized (alternateNum) {
                if (alternateNum.getStartNum() % 2 == 0) {
                    System.out.println(Thread.currentThread().getName() + "==偶数打印：" + alternateNum.getStartNum());
                    alternateNum.setStartNum(alternateNum.getStartNum() + 1);
                    alternateNum.notify();
                } else {
                    try {
                        alternateNum.wait();
                    } catch (InterruptedException e) {
                        System.out.println("OddNum.run中调用线程等待异常！");
                    }
                }
            }
        }
    }
}
```
### 10.7 数组实现堵塞队列

```java
package com.freshbin.other.thread.arrayqueue;

/**
 * 数组实现带锁队列
 * @author freshbin
 * @date 2020/5/12 14:52
 */
public class ArrayQueue<T> {
    private Integer count = 0;
    private Object[] array;
    /**
     * 队满锁
     */
    private Object full = new Object();
    /**
     * 队空锁
     */
    private Object empty = new Object();
    private Integer getIndex = 0;
    private Integer putIndex = 0;
    ArrayQueue(int size) {
        this.array = new Object[size];
    }

    public void put(T value) {
        synchronized (full) {
            while(count == array.length) {
                try{
                    full.wait();
                } catch (InterruptedException e) {
                    System.out.println("put方法中full.wait()异常");
                    break;
                }
            }
        }
        synchronized (empty) {
            array[putIndex] = value;
            putIndex++;
            count++;
            if(putIndex == array.length) {
                putIndex = 0;
            }
            empty.notify();
        }
    }

    public T get() {
        synchronized (empty) {
            while(count == 0) {
                try{
                    empty.wait();
                } catch (InterruptedException e) {
                    System.out.println("get方法中empty.wait()异常");
                    return null;
                }
            }
        }
        synchronized (full) {
            Object value = array[getIndex];
            getIndex++;
            count--;
            if(getIndex == array.length) {
                getIndex = 0;
            }
            full.notify();
            return (T)value;
        }
    }

    public int size() {
        return count;
    }
}
```
### 10.8 不加锁的线程安全
**1）栈封闭：** 多个线程访问同一个方法的局部变量，不会出现线程安全问题，因为局部变量存储在虚拟机栈中，属于线程私有的。
**2）线程本地存储：** 使用ThreadLocal实现线程本地存储功能，可能会由于ThreadLocal.ThreadLocalMap的底层数据结构导致内存泄露问题，因此需要在每次使用ThreadLocal之后手动调用remove()。
**3）可重入代码（Reentrant code）：** 可以在代码的任意时刻中断，去执行其他代码（包括递归调用自身），当程序返回时，原来的程序不会出现错误。
### 10.9 J.U.C
**1）CountDownLatch：** 维护了一个计数器cnt，每次调用countDown()方法会让计数器-1，当计数器为0时，因为调用await()方法在等待的线程就会被唤醒，代码如下：
```java
public class CountDownLatchTest {
    public static void main(String[] arg) throws InterruptedException {
        int threadCount = 10;
        CountDownLatch countDownLatch = new CountDownLatch(threadCount);
        ThreadPoolExecutor threadPoolExecutor =  new ThreadPoolExecutor(5, 5,
                10, TimeUnit.SECONDS, new ArrayBlockingQueue<Runnable>(10));
        for(int i = 0; i < threadCount; i++) {
            threadPoolExecutor.execute(() -> {
                System.out.print("run ");
                countDownLatch.countDown();
            });
        }
        countDownLatch.await();
        System.out.println("所有线程执行完毕！");
        threadPoolExecutor.shutdown();
    }
}
```
**2）CycliBarrier：** 有两个构造函数，其中parties指示计数器的初始值，barrierAction在所有线程都到达屏障的时候会执行一次。当执行await()方法之后，计数器会-1，当为0时，所有调用await()方法的线程会继续执行。调用reset()方法可以循环使用。
```java
public class CycliBarrierTest {
    public static void main(String[] arg) {
        int threadNum = 10;
        CyclicBarrier cyclicBarrier = new CyclicBarrier(threadNum);
        ThreadPoolExecutor threadPoolExecutor =  new ThreadPoolExecutor(10, 10,
                10, TimeUnit.SECONDS, new ArrayBlockingQueue<Runnable>(10));
        for(int i = 0; i < threadNum; i++) {
            threadPoolExecutor.execute(() -> {
                System.out.print("before..");
                try {
                    cyclicBarrier.await(); // 如果线程池的可用线程太少，而这时候就会卡住不动，后面的线程无法进入队列，前面的线程还没出队
                } catch (InterruptedException e) {
                    e.printStackTrace();
                } catch (BrokenBarrierException e) {
                    e.printStackTrace();
                }
                System.out.print("after..");
            });
        }
        threadPoolExecutor.shutdown();
    }
}
```
**3）Semaphore：** 类似操作系统中的信号量，可以控制对互斥资源的访问线程数。
```java
public class SemaphoreTest {
    private static Random rand= new Random(3000);
    public static void main(String[] arg) {
        ThreadPoolExecutor threadPoolExecutor = new ThreadPoolExecutor(10, 10,
                10, TimeUnit.SECONDS, new ArrayBlockingQueue<Runnable>(10));
        Semaphore semaphore = new Semaphore(3);
        for(int i = 0; i < 10; i++) {
            threadPoolExecutor.execute(() -> {
                try {
                    semaphore.acquire();
                    System.out.println(semaphore.availablePermits() + "==进入==");
                    TimeUnit.MILLISECONDS.sleep(rand.nextInt(10000));
                    System.out.println(semaphore.availablePermits() + "==退出==");
                } catch (InterruptedException e) {
                    e.printStackTrace();
                } finally {
                    semaphore.release();
                }
            });
        }
        threadPoolExecutor.shutdown();
    }
}
```
**4）FutureTask：** 可以异步获取实现了Callable接口的线程返回结果，FutureTask实现了RunableFuture接口，该接口继承了Runnable和Future接口。当子线程比较耗时，而且主线程需要子线程的返回结果时，可以使用该方法。
```java
public class FutureTaskTest {
    public static void main(String[] arg) throws InterruptedException, ExecutionException {
        FutureTask futureTask = new FutureTask(new Callable() {
            @Override
            public Integer call() throws Exception {
                int result = 0;
                for(int i = 0; i < 100; i++) {
                    Thread.sleep(100);
                    result += 1;
                }
                return result;
            }
        });
        System.out.println("main执行");
        Thread thread = new Thread(futureTask);
        thread.start();
        System.out.println("thread已经启动执行");
        System.out.println("子线程执行完毕：" + futureTask.get()); // 会等待futureTask执行返回才往下执行，有点像调用了join()方法
        System.out.println("main执行完毕");
    }
}
```
**5）BlockingQueue：** 有FIFO队列和优先级队列的实现。
```java
public class BlockQueueTest {
    private static BlockingQueue blockingQueue = new ArrayBlockingQueue(5);
    private static class Producer extends Thread {
        @Override
        public void run() {
            try {
                blockingQueue.put("product");
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println("producer ");
        }
    }

    private static class Consumer extends Thread {
        @Override
        public void run() {
            try {
                blockingQueue.take();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println("consumer ");
        }
    }

    public static void main(String[] arg) {
        for(int i = 0; i < 2; i++) {
            Producer producer = new Producer();
            producer.start();
        }

        for(int i = 0; i < 5; i++) {
            Consumer consumer = new Consumer();
            consumer.start();
        }

        for(int i = 0; i < 3; i++) {
            Producer producer = new Producer();
            producer.start();
        }
    }
}
```
**6）ForkJoin：** 把多任务拆分成多个小任务，看起来有点像是分治算法的思想，以后面试有问到这东西再补充完善。

### 10.10 线程池
使用线程池的目的：
1、**线程**是**稀缺**资源，不能频繁创建。
2、**解耦**作用；线程的创建与执行完全分开，方便维护。
3、应当将其放入一个池子中，可以给其他任务进行**复用**。

**1）实现方式：** 通过ThreadPoolExecutor创建线程池。如果是使用springboot框架，那么可以自定义一个线程池，将ThreadPoolExecutor注入自定义线程池类中，在需要使用线程的地方注入该线程池。

**2）如何配置线程：** 
1、IO密集型任务，可以配置CPU个数*2；
2、CPU密集型任务，配置相当CPU个数的线程数。

**3）关闭线程池：** 
1、shutdown：执行后停止接受新任务，会把队列的任务执行完毕；
2、shutdownNow：停止接受新任务，还会中断所有的任务，将线程池状态变为stop。

## 11、JVM
### 11.1 运行时内存划分
**1）程序计数器：** 记录当前线程所执行的字节码行号，用于获取下一条执行的字节码。每个线程私有。
**2）java虚拟机栈：** 由一个个栈帧组成，在每一个方法调用时产生，每个栈帧由局部变量区，操作数栈等组成。线程私有。
**3）本地方法栈：** 与java虚拟机栈类似，区别是该栈为本地方法服务。
**4）堆：** 可利用参数-Xms -Xmx进行堆内存控制，这块区域也是垃圾回收器重点管理的区域，堆内存分为新生代，老年代。线程共享区域。
**5）方法区（元数据区）：** JDK1.7称为方法区也是永久代，JDK1.8移除。使用元数据区代替。
**6）运行时常量池：** 存放符号引用。当new 一个对象时，会检查这个区域是否有这个符号的引用。
**7）直接内存：** 又称为堆外内存，不由JVM虚拟机管理。可以借助老年代产生的fullGC顺便进行回收，也可以显示调用System.gc()回收。

### 11.2 垃圾收集
**1）判断一个对象是否可回收**
* **引用计数算法：** 为对象添加一个引用计数器，增加一个引用时，计数器加1，引用失效则减1，为0时表示对象可被回收。
* **可达性分析算法：** 以GC Roots为起始点进行搜索，可达的对象都是存活的，不可达的对象可被回收。GC Roots包含以下内容：
     	* **虚拟机栈中局部变量表中引用的对象**
     	* **本地方栈中JNI中引用的对象**
     	* **方法区中静态属性引用的对象**
     	* **方法区中的常量引用的对象**
* **方法区的回收：** 方法区存放永久代对象。这里的回收是对常量池的回收和对类的卸载。
* **finalize()：**

**2）引用类型**
无论是哪种回收，都与对象的引用有关。
* 强引用
* 软引用
* 弱引用
* 虚引用

**3）垃圾收集算法**
* **标记清除：** 标记活动对象和非活动对象，之后进行清除回收，标记清除过程效率都不高，会产生大量不连续的内存碎片，导致无法给大对象分配内存。
*  **标记整理：** 让所有存活的对象都向一端移动，然后直接清理掉端边界以外的内存。不会产生内存碎片，由于需要移动大量对象，处理效率低。
*  **复制：** 将内存分为大小相等的两块，当一块内存用完了，就把活着的对象复制到另一块内存，把使用过的内存进行清理。只使用了内存的一半，所以有点浪费空间。现在的商业虚拟机都采用这种收集算法，但是不分为大小相等的两块，而是有一块较大的Eden空间和两块较小的Survivor空间，每次使用Eden和其中一块Survivor。在回收时，将Eden和Survivor中还存活着的对象全部复制到另一块Survivor上，最后清理Eden和使用过的Survivor。HotSpot虚拟机的Eden和Survivor大小比例默认为8:1，保证了内存的利用率达到90%。如果每次回收有多于10%的对象存活，那么一块Survivor就不够用，此时需要依赖于老年代进行空间分配担保，也就是借用老年代的空间存存储放不下的对象。
（好像有点明白Eden，survivor和老年代了） 
*  **分代收集：** 根据存活周期将内存划分为几块，不同块采用适当的手机算法，一般将堆分为新生代和老年代。新生代使用复制算法；老年代使用：标记-清除或者标记-整理算法

**4）垃圾收集器**
* **Serial收集器：** 单线程收集器，简单高效。Client场景下默认的新生代收集器。
*  **ParNew收集器：** Serial收集器的多线程版本，Server场景下默认的新生代收集器。
*  **Parallel Scavenge收集器：** 多线程收集器。吞吐量优先收集器。
*  **Serial Old收集器：** Serial收集器的老年代版本，Client场景下的虚拟机使用，如果用在Server场景下，有两大用途：1、与Parallel Scavenge收集器搭配使用（Parallel Old诞生以前）；2、作为CMS收集器后备预案。
*  ** Parallel Old收集器：** Parallel Scavenge收集器的老年代版本。
*  **CMS收集器：** 分为四个流程：
   * 初始标记：仅仅只是标记一下GC Roots能直接关联到的对象，速度很快，需要停顿。
   * 并发标记：进行GC Roots Tracing的过程，它在整个回收过程汇总耗时最长，不需要停顿。
   * 重新标记：为了修正并标记期间因用户程序继续运作而导致标记产生变动的那一部分对象的标记记录，需要停顿。
   * 并发清除：不需要停顿。

* ** G1收集器：** 整体来看是基于标记整理算法实现的收集器，局部来看是基于复制算法实现的。

4）Full GC触发条件
Minor GC是当Eden空间满了，就触发一次Minor GC。而Full GC的条件如下：
* 调用System.gc()
* 老年代空间不足
*  空间分配担保失败
*  JDK1.7及以前的永久代空间不足
*  Concurrent Mode Failure

### 11.3 类的加载机制
类是在第一次使用时动态加载的，而不是一次性加载所有类，因为一次性加载，会占用很多的内存。
**1）类的生命周期：** 加载->验证->准备->解析->初始化->使用->卸载。