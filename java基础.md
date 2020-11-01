###  java基础

#### wait和sleep区别

* 这两个方法来自不同的类分别是Thread和Object
* 最主要是sleep方法没有释放锁，而wait方法释放了锁，使得其他线程可以使用同步控制块或者方法。
* 使用范围：wait，notify和notifyAll只能在同步控制方法或者同步控制块里面使用，而sleep可以在任何地方使用。
* sleep必须捕获异常，而wait，notify和notifyAll不需要捕获异常
* sleep是Thread类的静态方法。sleep的作用是让线程休眠制定的时间，在时间到达时恢复，也就是说sleep将在接到时间到达事件事恢复线程执行。wait是Object的方法，也就是说可以对任意一个对象调用wait方法，调用wait方法将会将调用者的线程挂起，直到其他线程调用同一个对象的notify方法才会重新激活调用者。

#### synchronized底层原理

Synchronize是可重入锁，一个获得锁的线程可以再次进入一个同步块方法



#### 基本数据类型和字节数

整型：byte（1字节）(-128 - 127)、short（2字节）(-2^15 - 2^15-1)、int（4字节）(-2^31 -2^31-1)、long（8字节）(-2^63 - 2^63-1)

浮点型：float（4字节）储存大型浮点数组的时候可节省内存空间、double（8字节）

字符：char（2字节）

布尔：boolean（1位）

基本数据类型比较(string 除外)， == 和 Equals 两者都是比较值；

复合数据类型 当他们用（==）进行比较的时候，比较的是他们在内存中的存放地址，所以，除非是同一个new出来的对象，他们的比较后的结果为true，否则比较后结果为false。	

String中的equals方法其实比较的是字符串的内容是否一样，==比较的是内存地址

```java
String str1 = "Hello";
String str2 = new String("Hello");
String str3 = str2;
str1 == str2 //false
str1 == str3 //false
str2 == str3 //true
str1.equals(str2) 
    //true
```

字符串有intern方法。他的意思是检查字符串池里是否存在，如果存在了那就直接返回为true。因此在这里首先s1会在字符串池里面有一个，然后 s2.intern()一看池子里有了，就不再新建了，直接把s2指向它。

#### String，StringBuffer，StringBuilder

* StringBuilder不是线程安全的，StringBuffer是线程安全的，StringBuffer线程安全的一个原因是很多方法例如append里，是带有synchronized关键字。
* Java 中字符串属于对象，是final定义的，String的值是不可变的，这就导致每次对String的操作都会生成**新的String对象**。
* 当对字符串进行修改的时候，需要使用 StringBuffer 和 StringBuilder 类。和 String 类不同的是，StringBuffer 和 StringBuilder 类的对象能够被多次的修改，并且不产生新的未使用对象。
* 由于 StringBuilder 相较于 StringBuffer 有速度优势，**所以多数情况下建议使用 StringBuilder 类**。然而在应用程序要求线程安全的情况下，则必须使用 StringBuffer 类。

#### volatile关键字

保证了不同线程对这个变量进行操作时的可见性，即一个线程修改了某个变量的值，这新值对其他线程来说是立即可见的。

禁止进行指令重排序。



#### try，catch，final

其中`try{...}`这一块代码是需要被检测异常的代码；而`catch{...}`这一段是处理异常的代码；最后的`finally{...}`代码块是一定会被执行的代码。

#### Hashmap

<img src="C:\Users\Administrator\Desktop\IMG_20200806_132835.jpg" style="zoom:25%;" />

#### 排序    

<img src="C:\Users\Administrator\Desktop\IMG_20200806_132806.jpg" alt="IMG_20200806_132806" style="zoom:25%;" />

### Java 并发

#### 缓存不一致

如果一个变量在多个CPU中都存在缓存，可能出现缓存不一致的问题

两个解决方法：

* 通过在总线上加LOCK#锁的形式来解决缓存不一致的问题。因为CPU和其他部件进行通信都是通过总线来进行的，如果对总线加LOCK#锁的话，也就是说阻塞了其他CPU对其他部件访问（如内存），从而使得只能有一个CPU能使用这个变量的内存。但是在锁住总线期间，其他CPU无法访问内存，导致效率低下。
* 通过缓存一致性协议，比如 Intel 的**MESI**协议，MESI协议保证了每个缓存中使用的共享变量的副本是一致的。它核心的思想是：当CPU**写数据**时，如果发现操作的变量是**共享变量**，即在其他CPU中也存在该变量的副本，会发出信号通知其他CPU将该变量的缓存行置为**无效状态**，因此当其他CPU需要读取这个变量时，发现自己缓存中缓存该变量的缓存行是无效的，那么它就会从内存重新读取。

这两种都是硬件层面上提供的方式。

#### 并发编程三个概念

原子性问题，可见性问题，有序性问题

* 原子性：即一个操作或者多个操作 要么全部执行并且执行的过程不会被任何因素打断，要么就都不执行。

* 可见性：是指当多个线程访问同一个变量时，一个线程修改了这个变量的值，其他线程能够立即看得到修改的值。

* 有序性：即程序执行的顺序按照代码的先后顺序执行。

   指令重排序 ：一般来说，处理器为了提高程序运行效率，可能会对输入代码进行优化，它不保证程序中各个语句的执行先后顺序同代码中的顺序一致，但是它会保证程序最终执行结果和代码顺序执行的结果是一致的。

  怎么保证结果一致？

  会考虑指令之间的数据依赖性，如果一个指令Instruction 2必须用到Instruction 1的结果，那么处理器会保证Instruction 1会在Instruction 2之前执行。

  ```java
  //线程1:
  context = loadContext();  //语句1
  inited = true;       //语句2
  //线程2:
  while(!inited ){
   sleep()
  }
  doSomethingwithconfig(context);
  
  ```

  语句1和语句2没有数据依赖性，因此可能会被重排序，重排序会导致错误发生

  因此，指令重排序不会影响单个线程的执行，但是会影响到线程并发执行的正确性

#### Java内存模型

Java内存模型规定所有的变量都是存在主存当中（类似于前面说的物理内存），每个线程都有自己的工作内存（类似于前面的高速缓存）。线程对变量的所有操作都必须在工作内存中进行，而不能直接对主存进行操作。并且每个线程不能访问其他线程的工作内存。

* Java内存模型只保证了**基本读取和赋值是原子性操作**，如果要实现更大范围操作的原子性，可以通过**synchronized和Lock**来实现。由于synchronized和Lock能够保证任一时刻只有一个线程执行该代码块，那么自然就不存在原子性问题了，从而保证了原子性。
* Java提供了**volatile关键字**来保证可见性,当一个共享变量被volatile修饰时，它会保证修改的值会立即被更新到主存，当有其他线程需要读取时，它会去内存中读取新值。而普通的共享变量不能保证可见性，因为普通共享变量被修改之后，什么时候被写入主存是不确定的，当其他线程去读取时，此时内存中可能还是原来的旧值，因此无法保证可见性。通过**synchronized和Lock**也能够保证可见性，synchronized和Lock能保证同一时刻只有一个线程获取锁然后执行同步代码，并且在释放锁之前会将对变量的修改刷新到主存当中。

* 可以通过**volatile关键字**来保证一定的“有序性”。另外可以**通过synchronized和Lock**来保证有序性，很显然，synchronized和Lock保证每个时刻是有一个线程执行同步代码，相当于是让线程顺序执行同步代码，自然就保证了有序性。

  Java内存模型具备一些先天的“有序性”，即不需要通过任何手段就能够得到保证的有序性，这个通常也称为 **happens-before 原则**。如果两个操作的执行次序无法从happens-before原则推导出来，那么它们就不能保证它们的有序性，虚拟机可以随意地对它们进行重排序。

  ##### 八条原则：

  - 程序次序规则：一个线程内，按照代码顺序，书写在前面的操作先行发生于书写在后面的操作

  - 锁定规则：一个unLock操作先行发生于后面对同一个锁的lock操作

  - volatile变量规则：对一个变量的写操作先行发生于后面对这个变量的读操作

  - 传递规则：如果操作A先行发生于操作B，而操作B又先行发生于操作C，则可以得出操作A先行发生于操作C

  - 线程启动规则：Thread对象的start()方法先行发生于此线程的每个一个动作

  - 线程中断规则：对线程interrupt()方法的调用先行发生于被中断线程的代码检测到中断事件的发生

  - 线程终结规则：线程中所有的操作都先行发生于线程的终止检测，我们可以通过Thread.join()方法结束、Thread.isAlive()的返回值手段检测到线程已经终止执行

  - 对象终结规则：一个对象的初始化完成先行发生于他的finalize()方法的开始

    第一个规则，这个规则是用来保证程序在单线程中执行结果的正确性，但无法保证程序在多线程中执行的正确性。

    第二个规则，无论在单线程中还是多线程中，同一个锁如果出于被锁定的状态，那么必须先对锁进行了释放操作，后面才能继续进行lock操作

    第三个规则，如果一个线程先去写一个变量，然后一个线程去进行读取，那么写入操作肯定会先行发生于读操作。

接下来跳回volatile关键字



### 计算机网络

#### 状态码

1xx 服务器收到请求，需要请求者继续执行操作  

2xx ok，请求成功  

3xx 重定向，资源已经重新分配  

4xx 客户端请求错误，403 forbidden请求资源被拒绝，404 not found找不到请求资源  

5xx 服务器错误，500 服务器故障，503 服务器超载或停机维护



200 请求成功 ， 301 资源（网页等）被永久转移到其它URL，302 资源临时移动，403 forbidden 服务器理解请求客户端的请求，但是拒绝执行此请求。请求资源被拒绝，通常原因是服务器上某些文件或目录设置了权限，客户端权限不够 ， 404 not found 用户输入错误的链接 该链接指向的网页不存在 除此之外 也可以在服务器端拒绝请求且不想说明理由时使用， 

500，internal server error 服务器内部错误（比如浏览器代理除了问题，ip，端口不对等）该状态码表明服务器端在执行请求时发生了错误。也有可能是Web应用存在的bug或某些临时的故障。

502 bad gateway 作为网关或者代理工作的服务器尝试执行请求时，从远程服务器接收到了一个无效的响应

503，服务器超载或停机维护   504，Gateway Timeout网关超时 服务器作为网关或代理，未及时从上游服务器接收请求。

#### DNS查找的过程

0.浏览器从接收到的url中抽取出域名地址，将域名传给DNS应用的客户端  

1.检查浏览器缓存、本地hosts文件是否有这个网址的映射，如果有，就调用这个IP地址映射   

  2.   如果没有，则查找本地DNS解析器缓存是否有这个网址的映射，如果有，返回映射  

  3.   如果没有，则向DNS服务器提出查询请求  

  4.   服务器接收到查询时，查询本地配置区域资源，查到就返回结果  

  5.   如果查不到，但服务器缓存了此网址映射关系，返回查找结果  

  6.   如果没有缓存，就继续间请求转发至上一级DNS服务器进行查询。最终将解析结果依次返回本地DNS服务器，本地DNS服务器在返回给客户端，并把这个映射存到服务器的缓存中

#### 浏览器中输入一个URL后，按下回车后发生了什么

URL，统一资源定位符，简单点就是网址=ip或域名 + 端口号 + 资源位置 + 参数 + 锚点  

  1．输入一个网址之后，首先浏览器通过查询DNS，查找这个URL的IP地址，（通过层层向上级DNS服务器查找直到找到对应URL的IP地址）  

  2．得到目标服务器的IP地址及端口号（http 80端口，https 443端口），会调用系统库函数socket，请求一个TCP流套接字。客户端向服务器发送HTTP请求报文  

  （1）应用层：客户端发送HTTP请求报文。  

  （2）传输层：（加入源端口、目的端口）建立连接。实际发送数据之前，三次握手客户端和服务器建立起一个TCP连接。  

  （3）网络层：（加入IP头）路由寻址。  

  （4）数据链路层：（加入frame头）传输数据。  

  （5）物理层：物理传输bit。  

  3．服务器端经过物理层→数据链路层→网络层→传输层→应用层，解析请求报文，发送HTTP响应报文。  

  4．关闭连接，TCP四次挥手。  

  5．客户端解析HTTP响应报文，浏览器开始显示HTML

### 题

#### 反转链表

```java
/*
public class ListNode {
    int val;
    ListNode next = null;

    ListNode(int val) {
        this.val = val;
    }
}*/
public class Solution {
    public ListNode ReverseList(ListNode head) {
        ListNode newhead=null;
        ListNode node;
        while(head!=null){
            node=head;
            head=head.next;
            
            node.next=newhead;
            newhead=node;
        }
        return newhead;
        
    }
}
```

head不断往后，每次将上一次的newhead连接到下次的newhead直到最后 

https://blog.csdn.net/qq_42351880/article/details/88637387

#### 最长无的重复子串的长度

```java
import java.util.*
public class Solution {
    public int maxLength (int[] arr) {
        // write code here
        int ans=0;
        int n=arr.length;
        HashMap<Integer, Integer> map = new HashMap<>();
        for(int i=0,j=0;j<arr.length;j++){
            if(map.containsKey(arr[j])==true){
                i=Math.max(map.get(arr[j]),i);
            }
            map.put(arr[j],j+1);//遇到重复的直接去右边一位
            ans=Math.max(ans,j-i+1);  
        }
        return ans;
    }
}
```

用hashmap储存位置，遇到相同的就换位置，字符串同理。

#### 判断链表是否有环

```java
public class Solution {
    public boolean hasCycle(ListNode head) {
        if(head==null)
            return false;
        ListNode i=head;
        ListNode j=head;
        while(j!=null&&j.next!=null){
            j=j.next.next;
            i=i.next;
            if(i==j)
                return true;
        }
        return false;
    }
}
```

快慢指针,空间O(1)  相遇就是有环，循环终止条件是快指针为null或者next为null， 也可以用hashmap，但是空间O(n)

