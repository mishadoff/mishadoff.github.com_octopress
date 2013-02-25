---
layout: post
title: "Java Magic. Part 4: sun.misc.Unsafe"
date: 2013-02-25 02:44
comments: true
categories: [programming, java]
published: false
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

#### Lock-free hashtables

![TODO]()

`compareAndSwap` methods are atomic and can be used to implement
high-performance lock-free data structures.

### Bonus

Documentation for `park` method from `Unsafe` class:

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

Never use `Unsafe`.
