title: ThreadLocal
date: 2015-01-06 23:01:42
tags:
---

##ThreadLocal

###是什么？
是线程的局部变量，每个线程都可以获得自己的一份拷贝，修改等操作，不会影响其他线程的拷贝。

###什么时候用？
1. 每个线程都需要一个上下文环境。
2. 某个对象的操作不是线程安全的，但想避免使用线程同步来访问。

###怎么用？
#### 每个线程都需要一个上下文环境
```java
class StaticClass {
  static private ThreadLocal threadLocal = new ThreadLocal<User>();

  static getThreadLocal() {
    return threadLocal;
  }
}

doGet(HttpServletRequest req, HttpServletResponse resp) {
  User user = getLoggedInUser(req);
  StaticClass.getThreadLocal().set(user)
  try {
    doSomething()
    doSomethingElse()
    renderResponse(resp)
  }
  finally {
    StaticClass.getThreadLocal().remove()
  }
}

```
Now any code that requires the user object at any time can get hold of it by extracting it from the thread local, without needing to resort to those pesky extra parameters:
```java
User user = StaticClass.getThreadLocal().get()
```

**If we do not use `ThreadLocal`, we need to pass the context info through the parameters.**
```java
doGet(HttpServletRequest req, HttpServletResponse resp) {
  User user = getLoggedInUser(req);
  doSomething(user) // every method call into to pass in the context info
  doSomethingElse(user)
  renderResponse(resp,user)
}
```

#### 某个对象的操作不是线程安全的，但想避免使用线程同步来访问。
```java
public class Foo
{
    // SimpleDateFormat is not thread-safe, so give one to each thread
    private static final ThreadLocal<SimpleDateFormat> formatter = new ThreadLocal<SimpleDateFormat>(){
        @Override
        protected SimpleDateFormat initialValue()
        {
            return new SimpleDateFormat("yyyyMMdd HHmm");
        }
    };

    public String formatIt(Date date)
    {
        return formatter.get().format(date);
    }
}
```

###源码参考

**`ThreadLocal<T>` 提供相应方法来操作`Thread`持有的`ThreadLocal.ThreadLocalMap`, 该map存有该线程对应的那份拷贝.**

```java
public class ThreadLocal<T> {
    /**
     * ThreadLocals rely on per-thread hash maps attached to each thread
     * (Thread.threadLocals and inheritableThreadLocals).  The ThreadLocal
     * objects act as keys, searched via threadLocalHashCode.  This is a
     * custom hash code (useful only within ThreadLocalMaps) that eliminates
     * collisions in the common case where consecutively constructed
     * ThreadLocals are used by the same threads, while remaining well-behaved
     * in less common cases.
     */
    private final int threadLocalHashCode = nextHashCode();

    /**
     * The next hash code to be given out. Accessed only by like-named method.
     */
    private static int nextHashCode = 0;

    /**
     * The difference between successively generated hash codes - turns
     * implicit sequential thread-local IDs into near-optimally spread
     * multiplicative hash values for power-of-two-sized tables.
     */
    private static final int HASH_INCREMENT = 0x61c88647;

    /**
     * Compute the next hash code. The static synchronization used here
     * should not be a performance bottleneck. When ThreadLocals are
     * generated in different threads at a fast enough rate to regularly
     * contend on this lock, memory contention is by far a more serious
     * problem than lock contention.
     */
    private static synchronized int nextHashCode() {
        int h = nextHashCode;
        nextHashCode = h + HASH_INCREMENT;
        return h;
    }

    /**
     * Creates a thread local variable.
     */
    public ThreadLocal() {
    }

    /**
     * Returns the value in the current thread's copy of this thread-local
     * variable.  Creates and initializes the copy if this is the first time
     * the thread has called this method.
     *
     * @return the current thread's value of this thread-local
     */
    public T get() {
        Thread t = Thread.currentThread();
        ThreadLocalMap map = getMap(t);
        if (map != null)
            return (T)map.get(this);

        // Maps are constructed lazily.  if the map for this thread
        // doesn't exist, create it, with this ThreadLocal and its
        // initial value as its only entry.
        T value = initialValue();
        createMap(t, value);
        return value;
    }

    /**
     * Sets the current thread's copy of this thread-local variable
     * to the specified value.  Many applications will have no need for
     * this functionality, relying solely on the {@link #initialValue}
     * method to set the values of thread-locals.
     *
     * @param value the value to be stored in the current threads' copy of
     *        this thread-local.
     */
    public void set(T value) {
        Thread t = Thread.currentThread();
        ThreadLocalMap map = getMap(t);
        if (map != null)
            map.set(this, value); // Thread 持有的Map会保存(ThreadLocal, T)
        else
            createMap(t, value);
    }

    /**
     * Get the map associated with a ThreadLocal. Overridden in
     * InheritableThreadLocal.
     *
     * @param  t the current thread
     * @return the map
     */
    ThreadLocalMap getMap(Thread t) {
        return t.threadLocals;
    }

    /**
     * Create the map associated with a ThreadLocal. Overridden in
     * InheritableThreadLocal.
     *
     * @param t the current thread
     * @param firstValue value for the initial entry of the map
     * @param map the map to store.
     */
    void createMap(Thread t, T firstValue) {
        t.threadLocals = new ThreadLocalMap(this, firstValue);
    }

    .......

    /**
     * ThreadLocalMap is a customized hash map suitable only for
     * maintaining thread local values. No operations are exported
     * outside of the ThreadLocal class. The class is package private to
     * allow declaration of fields in class Thread.  To help deal with
     * very large and long-lived usages, the hash table entries use
     * WeakReferences for keys. However, since reference queues are not
     * used, stale entries are guaranteed to be removed only when
     * the table starts running out of space.
     */
    static class ThreadLocalMap {

    ........

    }

}

```

**`Thread`类持有`ThreadLocal.ThreadLocalMap`**
```java
public class Thread implements Runnable {
    ......

    /* ThreadLocal values pertaining to this thread. This map is maintained
     * by the ThreadLocal class. */
    ThreadLocal.ThreadLocalMap threadLocals = null;  
    ......
}
```
##注意

**如果`ThreadLocal`引用了非线程安全的对象，那么该对象依然是非线程安全的**

##引用
1. [When and how should I use a ThreadLocal variable?](http://stackoverflow.com/questions/817856/when-and-how-should-i-use-a-threadlocal-variable)
2. [Purpose of ThreadLocal?](stackoverflow.com/questions/1490919/purpose-of-threadlocal)
3. [正确理解ThreadLocal](http://www.iteye.com/topic/103804)
