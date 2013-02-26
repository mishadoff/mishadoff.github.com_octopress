---
layout: post
title: "Java Magic. Part 4: sun.misc.Unsafe"
date: 2013-02-26 02:37
comments: true
categories: [programming, java]
published: true
---

Java is a safe programming language and prevents programmer
from doing memory corruption mistakes. But, there is a way to
do such *mistakes* intentionally, using `Unsafe` class.

This article is quick overview of `sun.misc.Unsafe` *public* API and few
interesting cases of its usage.

<!-- more -->

### Unsafe instantiation

Before usage, we need to create instance of `Unsafe` object.
There is no simple way to do it like `Unsafe unsafe = new Unsafe()`,
because `Unsafe` class has private constructor and `getUnsafe()` method
available only from trusted code.

We can make our code "trusted".

`Unsafe` class contains its instance called `theUnsafe`, which marked as `private`.
We can steal that variable via java reflection.

``` java
Field f = Unsafe.class.getDeclaredField("theUnsafe");
f.setAccessible(true);
Unsafe unsafe = (Unsafe) f.get(null);
```

*Note:* Ignore your IDE. For example, eclipse show error "Access restriction..."
but if you run code, all works just fine. If the error is annoying, ignore errors on
`Unsafe` usage in:

    Preferences -> Java -> Compiler -> Errors/Warnings ->
    Deprecated and restricted API -> Forbidden reference -> Warning

### Unsafe API

Class [sun.misc.Unsafe](http://www.docjar.com/docs/api/sun/misc/Unsafe.html)
consists of `105` methods. There are, actually,
few groups of important methods for manipulating with various entities.
Here is some of them:

* **Info**. Just returns some low-level memory information.
  - `addressSize`
  - `pageSize`

* **Objects**. Provides methods for object and its fields manipulation.
  - `allocateInstance`
  - `objectFieldOffset`

* **Classes**. Provides methods for classes and static fields manipulation.
  - `staticFieldOffset`
  - `defineClass`
  - `defineAnonymousClass`
  - `ensureClassInitialized`

* **Arrays**. Arrays manipulation.
  - `arrayBaseOffset`
  - `arrayIndexScale`

* **Synchronization**. Low level primitives for synchronization.
  - `monitorEnter`
  - `tryMonitorEnter`
  - `monitorExit`
  - `compareAndSwapInt`
  - `putOrderedInt`

* **Memory**. Direct memory access methods.
  - `allocateMemory`
  - `copyMemory`
  - `freeMemory`
  - `getAddress`
  - `getInt`
  - `putInt`

### Interesting use cases

#### Avoid initialization

`allocateInstance` method can be *useful* when you need to skip object initialization phase
or bypass security checks in constructor. Consider following class:

``` java
class A {
    private long a; // not initialized value

    public A() {
        this.a = 1; // initialization
    }

    public long a() { return this.a; }
}
```

Instantiating it using constructor, reflection and unsafe gives
different results.

``` java
A o1 = new A(); // constructor
o1.a(); // prints 1

A o2 = A.class.newInstance(); // reflection
o2.a(); // prints 1

A o3 = (A) unsafe.allocateInstance(A.class); // unsafe
o3.a(); // prints 0
```

#### **sizeOf**

Using `objectFieldOffset` method we can implement C-style `sizeof` function.
This implementation returns *shallow* size of object:

``` java
public static long sizeOf(Object o) {
    Unsafe u = getUnsafe();
    HashSet<Field> fields = new HashSet<Field>();
    Class c = o.getClass();
    while (c != Object.class) {
        for (Field f : c.getDeclaredFields()) {
            if ((f.getModifiers() & Modifier.STATIC) == 0) {
                fields.add(f);
            }
        }
        c = c.getSuperclass();
    }

    // get offset
    long maxSize = 0;
    for (Field f : fields) {
        long offset = u.objectFieldOffset(f);
        if (offset > maxSize) {
            maxSize = offset;
        }
    }

    return ((maxSize/8) + 1) * 8;   // padding
}
```

Algorithm is the following: go through all *non-static* fields including all
superclases, get offset for each field, find maximum and add padding.
Probably, I missed something, but idea is clear.

In fact, for good and accurate `sizeof` function better to use
[java.lang.instrument](http://docs.oracle.com/javase/7/docs/api/java/lang/instrument/package-summary.html) package

#### **Big Arrays**

As you know `Integer.MAX_VALUE` constant is a max size of java array.
Using direct memory allocation we can create arrays with size limited by only heap size.

Here is `SuperArray` implementation:

``` java
class SuperArray {
    private final static int BYTE = 1;

    private long size;
    private long address;

    public SuperArray(long size) {
        this.size = size;
        address = getUnsafe().allocateMemory(size * BYTE);
    }

    public void set(long i, byte value) {
        getUnsafe().putByte(address + i * BYTE, value);
    }

    public int get(long idx) {
        return getUnsafe().getByte(address + idx * BYTE);
    }

    public long size() {
        return size;
    }
}
```

And sample usage:

``` java
long SUPER_SIZE = (long)Integer.MAX_VALUE * 2;
SuperArray array = new SuperArray(SUPER_SIZE);
System.out.println("Array size:" + array.size()); // 4294967294
for (int i = 0; i < 100; i++) {
    array.set((long)Integer.MAX_VALUE + i, (byte)3);
    sum += array.get((long)Integer.MAX_VALUE + i);
}
System.out.println("Sum of 100 elements:" + sum);  // 300
```

#### Memory corruption

This one is usual for every C programmer.
By the way, its common technique for security bypass.

Consider some easy class that check acces rules:

``` java
class Guard {
    private int ACCESS_ALLOWED = 1;

    public boolean giveAccess() {
        return 42 == ACCESS_ALLOWED;
    }
}
```

The client code is *very secure* and calls
`giveAccess()` to check access rules. Unfortunately, for clients,
it always returns `false`. Only privileged users *somehow* can change
value of `ACCESS_ALLOWED` constant and get access.

In fact, it's not true. Here is the code demostrates it:

``` java
Guard guard = new Guard();
guard.giveAccess();   // false, no access

// bypass
Unsafe unsafe = getUnsafe();
Field f = guard.getClass().getDeclaredField("ACCESS_ALLOWED");
unsafe.putInt(guard, unsafe.objectFieldOffset(f), 42); // memory corruption

guard.giveAccess(); // true, access granted
```

Now all clients will get unlimited access.

#### Concurrency

`compareAndSwap` methods are atomic and can be used to implement
high-performance lock-free data structures.

For example, consider the problem to increment value in the shared object
using lot of threads.

First we define simple interface `Counter`:

``` java
interface Counter {
    void increment();
    long getCounter();
}
```

Then we define worker thread `CounterClient`, that uses `Counter`:

``` java
class CounterClient implements Runnable {
    private Counter c;
    private int num;

    public CounterClient(Counter c, int num) {
        this.c = c;
        this.num = num;
    }

    @Override
    public void run() {
        for (int i = 0; i < num; i++) {
            c.increment();
        }
    }
}
```

And this is testing code:

``` java
int NUM_OF_THREADS = 1000;
int NUM_OF_INCREMENTS = 100000;
ExecutorService service = Executors.newFixedThreadPool(NUM_OF_THREADS);
Counter counter = ... // creating instance of specific counter
long before = System.currentTimeMillis();
for (int i = 0; i < NUM_OF_THREADS; i++) {
    service.submit(new CounterClient(counter, NUM_OF_INCREMENTS));
}
service.shutdown();
service.awaitTermination(1, TimeUnit.MINUTES);
long after = System.currentTimeMillis();
System.out.println("Counter result: " + c.getCounter());
System.out.println("Time passed in ms:" + (after - before));
```

First implementation is not-synchronized counter:

``` java
class StupidCounter implements Counter {
    private long counter = 0;

    @Override
    public void increment() {
        counter++;
    }

    @Override
    public long getCounter() {
        return counter;
    }
}
```

Output:

```
Counter result: 99542945
Time passed in ms: 679
```

Working fast, but no threads management at all, so result is inaccurate.
Second attempt, add easiest java-way synchronization:

``` java
class SyncCounter implements Counter {
    private long counter = 0;

    @Override
    public synchronized void increment() {
        counter++;
    }

    @Override
    public long getCounter() {
        return counter;
    }
}
```

Output:

```
Counter result: 100000000
Time passed in ms: 10136
```

Radical synchronization always work. But timings is awful.
Let's try `ReentrantReadWriteLock`:

``` java
class LockCounter implements Counter {
    private long counter = 0;
    private WriteLock lock = new ReentrantReadWriteLock().writeLock();

    @Override
    public void increment() {
        lock.lock();
        counter++;
        lock.unlock();
    }

    @Override
    public long getCounter() {
        return counter;
    }
}
```

Output:

```
Counter result: 100000000
Time passed in ms: 8065
```

Still correct, and timings are better. What about atomics?

``` java
class AtomicCounter implements Counter {
    AtomicLong counter = new AtomicLong(0);

    @Override
    public void increment() {
        counter.incrementAndGet();
    }

    @Override
    public long getCounter() {
        return counter.get();
    }
}
```

Output:

```
Counter result: 100000000
Time passed in ms: 6552
```

`AtomicCounter` is even better. Finally, try `Unsafe`
primitive `compareAndSwapLong` to see if it is really privilegy to use it.

``` java
class CASCounter implements Counter {
    private long counter = 0;
    private Unsafe unsafe;
    private long offset;

    public CASCounter() throws Exception {
        unsafe = getUnsafe();
        offset = unsafe.objectFieldOffset(CASCounter.class.getDeclaredField("counter"));
    }

    @Override
    public void increment() {
        long before = counter;
        while (!unsafe.compareAndSwapLong(this, offset, before, before + 1)) {
            before = counter;
        }
    }

    @Override
    public long getCounter() {
        return counter;
    }
```

Output:

```
Counter result: 100000000
Time passed in ms: 6454
```

Hmm, seems equal to atomics. Maybe atomics use `Unsafe`? (*YES*)

In fact this example is easy enough, but it shows real power of `Unsafe`.

### Bonus

Documentation for `park` method from `Unsafe` class contains
longest English sentence I've ever seen:

> Block current thread, returning when a balancing
> unpark occurs, or a balancing unpark has
> already occurred, or the thread is interrupted, or, if not
> absolute and time is not zero, the given time nanoseconds have
> elapsed, or if absolute, the given deadline in milliseconds
> since Epoch has passed, or spuriously (i.e., returning for no
>"reason"). Note: This operation is in the Unsafe class only
> because unpark is, so it would be strange to place it
> elsewhere.

### Conclusion

Although, `Unsafe` has a bunch of useful applications, never use it.
