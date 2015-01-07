title: Java Thread
date: 2015-01-07 23:01:42
tags:
---

## Java Thread

###Synchronization
The most basic of these methods is *synchronization*, which is implemented using **monitors**. Each object in Java is associated with a monitor, which a thread can *lock* or *unlock*. Only one thread at a time may hold a lock on a monitor. Any other threads attempting to lock that monitor are blocked until they can obtain a lock on that monitor. A thread `t` may lock a particular monitor multiple times; each unlock reverses the effect of one lock operation.

The `synchronized` statement computes a reference to an object; it then attempts to perform a lock action on that object's monitor and does not proceed further until the lock action has successfully completed. After the lock action has been performed, the body of the `synchronized` statement is executed. If execution of the body is ever completed, either normally or abruptly, an unlock action is automatically performed on that same monitor.


A `synchronized` method automatically performs a lock action when it is invoked:

1. If the method is an instance method, it locks the monitor associated with the **instance** for which it was invoked (that is, the object that will be known as `this` during execution of the body of the method). 
2. If the method is static, it locks the monitor associated with the **Class object** that represents the class in which the method is
defined. 

###Wait Sets and Notification

Every object, in addition to having an associated monitor, has an associated *wait set*. A wait set is a set of threads.

When an object is first created, its wait set is empty. Elementary actions that add threads to and remove threads from wait sets are atomic. Wait sets are manipulated solely through the methods `Object.wait`, `Object.notify`, and `Object.notifyAll`.

Wait set manipulations can also be affected by the interruption status of a thread, and by the Thread class's methods dealing with interruption. Additionally, the `Thread` class's methods for sleeping and joining other threads have properties derived from those of wait and notification actions.

**调用`wait()`之后，会释放lock，当resume的时候，会再次去lock**

###示例

```java
public class Main extends Thread {
    @Override
    public void run() {
        System.out.println("start");
        synchronized (this) { // wait/notify/notifyAll use the same
                                // synchronization resource
								// try to lock the monitor of this instance
            try {
            	System.out.println("wait start");
                this.wait(); // when this call is finished, will unlock the monitor
                System.out.println("wait end"); // when this thread is resumed, this 
												// thread will try to lock the monitor of this instance
												// and then invoke any operations after `wait()`
            } catch (InterruptedException e) {
                e.printStackTrace(); // notify won't throw exception
            }
        }
    }
 
    public static void main(String[] args) {
        Thread thread = new Main();
        thread.start();
        try {
            sleep(2000);
        } catch (InterruptedException e) {
        }
        synchronized (thread) {
            System.out.println("Wait() will release the lock!");
            thread.notify();
        }
    }
}
```

The out put is:

```java
start
wait start
Wait() will release the lock!
wait end
```

###参考
> The Java Language Specification Java SE 7 Edition

