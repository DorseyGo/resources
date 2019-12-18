# Java4
## Java 并发编程
### 底层实现原理
#### 2.1 Volatile的应用
volatile是轻量级的synchronized，它在多处理器任务开发中保证了共享变量的“可见性”。可见性，指当一个线程修改一个共享变量时，另外一个线程能读到这个修改的值。
##### 2.1.1 volatile的定义与实现原理
**定义**，
- 允许线程访问<font color='blue' size='3'>共享变量</font>，为了确保变量能准确和一致性的更新，线程应该确保通过<font color='blue' size='3'>排他锁</font>单独获得这个变量

为了提高处理速度，处理器不直接与内存进行通信， 而是先将系统内存的数据读到<font size='3' color='red'>内部缓存（L1, L2或其他）</font>后再进行操作，但操作不知道何时会写到内存。如果对声明了volatile的变量进行写操作，JVM就会向处理器发送一条LOCK前缀的指令，将这个变量所在缓存行的数据写回系统内存（由<font size='3' color='red'>缓存一致性协议</font>保证，处理器会重新从系统内存中读取修改的变量）。

volatile**两条**实现原则，
- <font color='blue' size='2'>Lock前缀指令会引起处理器缓存回写至内存</font>。
- <font color='blue' size='2'>一个处理器的缓存回写到内存会导致其他处理器的内存无效</font>。

#### 2.2 synchronized的实现原理与应用
Java中的每一个对象都可以作为锁，具体表现为3种形式，
- 对于普通同步方法，锁是当前实例对象
- 对于静态同步方法，锁是当前类的class对象
- 对于同步方法块，锁是synchronized括号里配置的对象

代码同步是使用<font color='red' size='2'>monitorenter</font>和<font color='red' size='2'>monitorexit</font>指令来实现的，monitorenter指令是在编译后插入到同步代码块的开始位置，而monitorexit是插入到方法结束处和异常处。

##### 2.2.1 锁的升级和对比
在Java SE1.6中，锁一共有4种状态（级别从低到高），
* 无锁状态
* 偏向锁状态
* 轻量级锁状态
* 重量级锁状态

<i>锁可以升级，但不能降级</i>。
###### 2.2.1.1 偏向锁
当一个线程访问同步代码块并获取锁时，会在对象头和栈帧中的锁记录里存储锁偏向的线程ID，以后该线程在进入和退出同步块时不需要进行CAS操作来加锁和解锁。
```flow
st=>start: start
e=>end: end
op=>operation: 获取Mark Word中偏向锁的标识
cond=>condition: 对象头的Mark Word是否存储当前线程的偏向锁?
cond1=>condition: 标识是否为1？
op1=>operation: 线程获得锁
op2=>operation: 使用CAS竞争锁
op3=>operation: 使用CAS将对象头的偏向锁指向当前线程

st->cond
cond(yes)->op1->e
cond(no)->cond1
cond1(yes)->op2->e
cond1(no)->op3->e
```
<center><font size='2'>图2-1 偏向锁</font></center>
<br />
1 <b>关闭偏向锁</b>
<br />在Java6和Java7中默认开启，在程序启动几秒钟后才激活，如有必要可以使用JVM参数来关闭延迟：<font color='blue'><b>-XX:BiasedLockingStartupDelay=0</b></font>。如果你确定应用程序里所有的锁通常情况下处于竞争，可通过JVM参数关闭偏向锁：<font color='blue'><b>-XX:BiasedLocking=false</b></font>。

##### 2.2.2 锁的优缺点对比
<center><small>表2-2锁的优缺点对比</small></center><br />

| 锁   |      优点      |  缺点 | 适用场景 |
|----------|:-------------:|:-----------|:------------|
| 偏向锁 |  加锁和解锁不需要额外的消耗，和执行非同步方法相比仅存在纳秒级别的差距 | 如果锁之间存在竞争，会带来额外的锁撤销的消耗 | 适用于只有一个线程访问同步块场景 |
| 轻量级锁 |    竞争的线程不会阻塞，提高了程序的响应性   |   如果始终得不到锁竞争的线程，使用自旋会消耗CPU | 追求响应时间 <br /> 同步块执行速度非常快 |
| 重量级锁 | 线程竞争不使用自旋，不会消耗CPU |    线程阻塞，响应时间缓慢 | 追求吞吐量 <br /> 同步块执行速度较长 |

## 2 Java性能分析
### 2.1 线程的作用
线程是瞬时快照，包含线程调用状态及调用关系。一般线程性能分析以下问题，
* 系统无缘无故的CPU过高
* 系统挂起，无响应
* 系统运行的越来越慢
* 性能瓶颈（无法充分利用CPU等）
* 线程死锁、死循环等
* 由于线程数过多，导致内存溢出

### 2.2 线程状态
线程有以下几种状态，
* new
  - 线程刚被创建，但还没调用start方法
* Runnable
  - 从虚拟机角度看，线程正在运行当中，当然可能是某种耗时计算IO等待的操作/CPU时间切片等，这个状态下发生的等待都是其他的系统资源，不是锁、sleep等
* blocked
  * 线程处于阻塞状态，正等待一个<font size='3' color='red'>monitor lock</font>。通常情况下，是因为当前线程与其他线程公用了一个锁，其他线程正在使用这个锁进入某个synchronized同步方法块或者方法
* waiting
  * 改线程拥有了某个锁之后，调用了他的<font color='red' size='3'>wait</font>方法，等待其他线程/锁拥有者调用notify/notifyall
  * 当线程调用以下方法会进入waiting状态，
    * object#wait()
    * Thread#join()
    * LockSupport#park()
* timed_waiting
  * 线程正在等待，通过使用了<font size='2' color='blue'>sleep，wait，join</font>或者是<font size='2' color='blue'>park</font>方法
* terminated
  * 线程终止

> <strong>Note</strong>
> * 系统慢，必须特别关注<font color='red'>blocked，waiting on condition</font>
> * 系统<font color='red'>CPU耗高</font>，肯定是线程执行有死循环，此时需关注<font color='red'>Runnable</font>状态
