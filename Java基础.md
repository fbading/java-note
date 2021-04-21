

## 一、线程

### 1.1线程的本质

>线程的本质 -------thread ---------------- JVM ------------ OS【linux】

### 1.1.1 线程启动

>thread start() -----> hotspot---> start()------> thread.c [start()]---->pthread_create()----->linux的源码

`Java线程模型与操作系统是一一对应的`

`java中的线程就是通过jni调用jvm源码当中的pthread_create的函数创建的线程`

### 1.1.2 os三种同步的源语

>* 互斥量 ：pthread_mutex_t （互斥锁）发生竞争时候如果拿不到锁则睡眠
>* 自旋锁 ：pthread_spin_t  (pthread_spin_t)空转os
>* 信号量

### 1.1.3 synchronized 

>* 底层是mutex，是互斥锁
>* 当synchronized临界区的代码发生异常的时候，会释放当前锁【monitorexit javac级别汇编指令中已经做了monitorexit的处理】

#### synchronized 源码

>`monitorenter`:
>
>`monitorexit`:

`总结`

>`首先会判断线程的mark word是否是可偏向模式，如果没有禁用偏向模式，当线程t2来来执行的时候看看是否偏向自己，在看看是否是匿名偏向，然后我们就cas来循环对象的是否与无锁状态相同【00】，若能成功我们就把lock record的的displace_header的值改为无锁的转态，若t3线程也来执行加锁，锁会膨胀升级为重量锁。`

### 1.1.4 内核态用户态

>* 跟线程无关，跟`cpu`有关

### 1.1.5 判断对象的大小

`引入jar包`

```xml
<dependency>
	 <groupId>org.openjdk.jol</groupId>
    <artifactId>jol-core</artifactId>
    <version>0.10</version>
</dependency>
```



`代码`

```java
ClassLayout.parseInstance(l).toPrinttable();
```

`对象头组成`

>* 对象头 ：由`mark work` = 8【64bit】 和 `klass pointer` = 4【32bit】组成
>* 实例数据
>* 对齐填充

>![image-20210328150406164](https://gitee.com/dingdingxiaolong/typora_img/raw/master/img/20210328150410.png)
>
>* ud = unuserd = 没有使用
>* bl = biased_lock = 偏向标识 `不可偏向为0 ，可偏向为1`
>* ep epoch = 偏向时间戳
>
>代码结果
>
>![image-20210328155950514](https://gitee.com/dingdingxiaolong/typora_img/raw/master/img/20210328155953.png)
>
>`此图显示无锁可偏向状态，但是没有偏向`

### 1.1.6 锁状态

>* 偏向锁 ：01
>* 轻量锁 ：00
>* 重量锁 ：10
>* gc  ：11

### 1.1.7 锁的膨胀过程

>**前提没有计算hashCode**--**偏向锁打开**
>
>* `当一把锁第一次被线程持有的时候是变向锁，如果这个线程再次加锁还是偏向锁，如果其他线程（交替执行）轻量锁，如果资源竞争重量锁。`
>
>* `轻量锁[00]释放以后会变成无锁状态`**[001]**

>`当线程有计算**hashCode**不可偏向的变向锁，如果加锁会直接变化成轻量锁。`



### 1.1.8 公平锁和非公平锁

>* 所谓的公平锁和非公平锁他们首先会在加锁的时候去抢锁，如果加锁失败，他就会进入队列（**不会睡眠**），然后再次看一下自己是否有拿锁资格，若还是拿不到就睡眠（**排队**）
>
>* **公平锁**：第一次加锁的时候，他不会去尝试拿锁，他会去看一下我前面有没有人排队，如果有人排队，我则进入队列（并不等于排队），然后不死心，再次看一下我有没有拿锁的资格，如果有继续拿，拿不到则睡眠（排队）
>
> 总结： `一朝排队，永远排队`

#### 区别

>* 非公平锁代码首先就会尝试拿锁，要是拿不到在等待的时候又会在尝试拿锁
>* 公平锁在拿锁的时候就首先会判断队列是否有值，然后再尝试拿锁











## 二、 死锁

### 1.1条件

>* 至少两个线程
>* 至少两把锁
>* 线程数`大于等于`锁的个数

## 1.2 查找方法

> * jps -v
> * jstack 序号值
> * 

## 1.3 死锁代码

```java
public class NormalDeadLock {
    private static Object valueFirst = new Object();//第一个锁
    private static Object valueSecond = new Object();//第二个锁

    //先拿第一个锁，再拿第二个锁
    private static void fisrtToSecond() throws InterruptedException {
        String threadName = Thread.currentThread().getName();
        synchronized (valueFirst){
            System.out.println(threadName+" get 1st");
            Thread.sleep(100);
            synchronized (valueSecond){
                System.out.println(threadName+" get 2nd");
            }
        }
    }

    //先拿第二个锁，再拿第一个锁
    private static void SecondToFisrt() throws InterruptedException {
        String threadName = Thread.currentThread().getName();
        synchronized (valueFirst){
            System.out.println(threadName+" get 2nd");
            Thread.sleep(100);
            synchronized (valueSecond){
                System.out.println(threadName+" get 1st");
            }
        }
    }

    private static class TestThread extends Thread{
        private String name;
        public TestThread(String name) {
            this.name = name;
        }
        public void run(){
            Thread.currentThread().setName(name);
            try {
                SecondToFisrt();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }

    public static void main(String[] args) {
        Thread.currentThread().setName("TestDeadLock");
        TestThread testThread = new TestThread("SubTestThread");
        testThread.start();
        try {
            fisrtToSecond();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}
```

## 1.4 活锁

##1.4.1 关键代码

>```java
>while(true){
>    if(from.getLock().tryLock()){
>        System.out.println(Thread.currentThread().getName()
>                +" get"+from.getName());
>        try{
>            if(to.getLock().tryLock()){
>                try{
>                    System.out.println(Thread.currentThread().getName()
>                            +" get"+to.getName());
>                    from.flyMoney(amount);
>                    to.addMoney(amount);
>                    System.out.println(from);
>                    System.out.println(to);
>                    break;
>                }finally{
>                    to.getLock().unlock();
>                }
>            }
>        }finally {
>            from.getLock().unlock();
>        }
>
>    }
>    Thread.sleep(r.nextInt(2));
>}
>```

# AQS

## 1.1同步队列：

> 同步器**aqs**内部会维护一个队列，当多个线程争夺一把锁的时候，获取这把锁失败的线程会被通过**cas**加入队列的尾节点，加入了以后并且会在队列上进行自旋，当有线程释放锁以后，会唤醒队列的后续节点，当后续节点被自旋唤醒以后就会尝试去拿锁。

## 1.2阻塞队列：

>**condition**本质就是队列，而且节点和同步队列使用的节点是同一个，但是实际使用的时候点调用`await`方法以后，会把当前节点打包成condition下的节点放入到condition队列
> 的尾部，并且当它进入阻塞状态【**park状态**】，当线程调用了condition下signal方法，会把condition下的头节点唤醒加入到同步队列的尾节点重新争抢锁。







