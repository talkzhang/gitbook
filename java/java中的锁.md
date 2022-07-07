# ReentrantLock 和 synchronized 关键字区别

通过ReentrantLock 和 synchronized都是实现可重入锁、独享锁，synchronized是非公平锁，ReentrantLock构造函数默认是非公平锁，但可以通过构造函数设置为公平锁；ReentrantLock使用起来更为灵活，它可以通过lock方法实现和synchronized一样的功能，也可以通过tryLock尝试性的去获取锁，甚至可以精确到指定时间段内获取不到可以放弃获取，而synchronized只能让线程死等；ReentrantLock需要手动释放锁资源，如果不释放容易发生死锁，synchronized是java系统层面的关键字，它的锁资源不需要手动执行处理，系统会自动释放；实现方式不同，ReentrantLock是通过AQS队列来实现的，所以它有自动尝试机制，synchronized是基于jvm内部来实现；ReentrantLock可以通过condition条件来使满足条件的获取锁，而synchronized只能单一阻塞。

ReentrantLock代码举例说明：

```java
public class Test {

    ReentrantLock lock = new ReentrantLock();

    public static void main(String[] args) {
        Test t = new Test();
        for (int i = 0; i < 10; i++) {
            new Thread(() -> {
                t.lockMethod();
            }).start();
        }
    }

    public void lockMethod(){

        boolean l = false;
        try {
            if (l = lock.tryLock(2, TimeUnit.SECONDS)) {
                System.out.println(Thread.currentThread() + "执行...");
                TimeUnit.SECONDS.sleep(5);
            } else {
                System.out.println(Thread.currentThread() + "未获取到锁");
            }
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            if (l) {
                System.out.println(Thread.currentThread() + "已关闭锁资源");
                lock.unlock();
            }

        }
    }
}
```

# volatile关键字有哪些特性

volatile可以保证可见性，防止指令重排，但不能保证原子性。

它的可见性是怎么保证呢？从jmm，java内存模型说起，java中读取变量每个线程是从主存中读取到本地内存，然后写的时候，先写入本地内存，然后刷到主内存，如果这个变量是volatile修饰的，那么在读取变量时，先从主存读取到本地内存，然后写的时候直接强制写到主内存，并且使其他线程读到的本地内存中的该变量失效，这样其它线程读取该变量时保证了可见性。

指令重排，编译器在执行代码过程中，并不一定完全按照编码顺序去执行，只会保证最终一致性即可，这是从优化的角度实现的，这种情况单线程没问题，多线程情况下可能就会引发一些可见性问题，其实指令重排也是一定程度上保证了valitile修饰变量的可见性，指定重排是通过内存屏障来实现的，就是在执行valitile修饰的变量进行读写时，会前后加上类似锁的这么一个东西，使其不被编译器优化顺序。

volatile可以配合synchronized关键字保证原子性。

# synchronized原理及锁升级过程

首先说原理，synchronized 锁机制在 Java 虚拟机中的同步是基于进入和退出监视器锁对象 monitor 实现的，大概就是说锁对象的时候，对象头部内维护一个monitor对象，里面维护一个计数和线程id一类的东西，当有线程来，如果这个计数counter是0，那线程可以获取monitor，并将计数器+1，取锁就成功了，它的底层通过monitorenter 和 monitorexit 指令来实现，执行monitorexit时，会将计数的couter-1，当然这种情况是针对锁对象来讲的，当锁定的是方法时，处理方式就不一样了，首先如果是带有synchronized关键字的方法，会有一个标志位来标明当前方法是带锁的（ACC_SYNCHRONIZED），既然是带锁的，那就需要获取一个monitor，无论多少个线程大家都是在竞争同一个monitor。

在jdk1.6之前版本中，synchronized是一个重量级锁，之所以被称为重量级锁是因为它需要依靠系统mutex来完成锁机制，就会在用户态和内核态之间进行切换，这样做的代价是比较高的，非常影响效率，其实很多场景中，线程竞争资源并没有那么激烈，所以为了提高效率，jdk1.6引入了synchronized锁的升级过程从偏向锁>轻量级锁>重量级锁，说偏向锁之前先说下jvm中对象的组成部分，有对象头、对象元数据、填充数据，对象头里面记录了对象的gc次数，hash值等信息，元数据就是对象内属性的地址引用，填充数据是因为jvm是按8字节单位来处理任务的，所以内存中对象的内存大小必须是8的倍数，所以填充数据是为了让对象内存满足8的倍数

锁的状态就是对象头内维护的，当时偏向锁时，认为没有别的线程过来竞争资源，会在对象头中标记为偏向锁，并将当前线程的id保存到对象头中，此时当有另一个线程过来获取锁时，发现已经有别的线程占有了，所以这时第二个线程会把对象头锁状态信息改为轻量级锁，并把这个对象头的monitor复制到当前线程中，通过自旋的方式去修改线程id为自身，如果自旋达到一定次数还没有修改成功，那么就升级为重量级锁，后面来的线程直接阻塞等待锁释放。

# AQS是什么？ReentrantLock和它有什么关系？

aqs是juc java并发包下的一个核心组件，而reentrantlock的核心就是使用aqs来完成加锁和释放锁的。

aqs的具体过程概述：在aqs内维护了一个状态state和线程，state默认值是0，而线程默认是null，当有线程获取锁时，会将state=1，且将线程赋值为当前持有锁的线程，而第二个线程获取锁时，首先要看这个状态state是不是=0，如果不是0就要看线程是不是自己，这也是reentrantlock是可重入锁的原因，当其他线程发现state!=0并且线程也不是它自己，这时就会将该线程放入队列中，如果刚开始线程释放了锁，那么队列中的线程就会拿到锁，然后重复线程1的动作。

所以reentrantlock可以看成是最终继承的一个类，而aqs是完成功能锁的核心。

