## 代码:多线程交替打印奇偶数

### 正确的实现方式

```java
public class CountDemo {
    static class Wrapper {
        private Integer count;
        public Wrapper(Integer count) {
            this.count = count;
        }
        public Integer getCount() {
            return count;
        }
        public void setCount(Integer count) {
            this.count = count;
        }
    }
    static class PrintOdd implements Runnable {
        private Wrapper wrapper;
        public PrintOdd(Wrapper wrapper) {
            this.wrapper = wrapper;
        }
        @Override
        public void run() {
            try {
                synchronized (wrapper) {
                    while (wrapper.getCount() <= 100) {
                        if (wrapper.getCount() % 2 == 0) {
                            wrapper.wait();
                        } else {
                            System.out.println("PrintOdd thread print..." + wrapper.getCount());
                            wrapper.setCount(wrapper.getCount() + 1);
                            wrapper.notify();
                        }
                    }
                }
            } catch (Exception ex) {
                ex.printStackTrace();
            }
        }
    }
    static class PrintEven implements Runnable {
        private Wrapper wrapper;
        public PrintEven(Wrapper wrapper) {
            this.wrapper = wrapper;
        }
        @Override
        public void run() {
            try {
                synchronized (wrapper) {
                    while (wrapper.getCount() <= 100) {
                        if (wrapper.getCount() % 2 == 1) {
                            wrapper.wait();
                        } else {
                            System.out.println("PrintEven thread print..." + wrapper.getCount());
                            wrapper.setCount(wrapper.getCount() + 1);
                            wrapper.notify();
                        }
                    }
                }
            } catch (Exception ex) {
                ex.printStackTrace();
            }
        }
    }
    public static void main(String[] args) {
        Wrapper wrapper = new Wrapper(1);
        PrintOdd printOdd = new PrintOdd(wrapper);
        PrintEven printEven = new PrintEven(wrapper);
        new Thread(printOdd).start();
        new Thread(printEven).start();
    }
}
```

代码执行结果如下：

> PrintOdd thread print...1
> PrintEven thread print...2
> PrintOdd thread print...3
> PrintEven thread print...4
> ...(省略展示)...
> PrintOdd thread print...99
> PrintEven thread print...100

### wait/notify正确的使用姿势



```java
synchronized (sharedObject) { 
    while (condition) { 
         // 条件不满足，当前线程阻塞等待
         sharedObject.wait(); 
    } 
    // 执行条件满足时的逻辑
    doSomething();
    // 通知阻塞的线程可以唤醒
    sharedObject.notify();
}
```

### 利用ReentrantLock锁实现

众所周知，除了synchronized关键字的加锁方式，还可以使用Lock锁实现对共享变量的同步控制。利用Lock对象还可以生成Condition条件，Condition条件中包含了await/signal方法，可以用来替代Object对象的wait/notify方法。
 下面的代码演示了利用ReentrantLock可重入锁实现两个线程交替打印奇偶数的实现，逻辑与前面的代码相似。

```java
public class CountLockDemo {

    public static void main(String[] args) {
        ReentrantLock lock = new ReentrantLock();
        Condition condition = lock.newCondition();
        Wrapper wrapper = new Wrapper(1);
        new Thread(new PrintOdd(lock, condition, wrapper)).start();
        new Thread(new PrintEven(lock, condition, wrapper)).start();
    }

    static class Wrapper {
        private Integer count;
        public Wrapper(Integer count) {
            this.count = count;
        }
        public Integer getCount() {
            return count;
        }
        public void setCount(Integer count) {
            this.count = count;
        }
    }

    static class PrintOdd implements Runnable {
        private volatile Wrapper wrapper;
        private ReentrantLock lock;
        private Condition condition;
        public PrintOdd(ReentrantLock lock, Condition condition, Wrapper wrapper) {
            this.lock = lock;
            this.condition = condition;
            this.wrapper = wrapper;
        }

        @Override
        public void run() {
            while (wrapper.getCount() <= 100) {
                lock.lock();
                try {
                    if (wrapper.getCount() % 2 == 0) {
                        condition.await();
                    } else {
                        System.out.println("PrintOdd thread print..." + wrapper.getCount());
                        wrapper.setCount(wrapper.getCount() + 1);
                        condition.signal();
                    }
                } catch (Exception ex) {
                    ex.printStackTrace();
                } finally {
                    lock.unlock();
                }
            }
        }
    }

    static class PrintEven implements Runnable {
        private volatile Wrapper wrapper;
        private ReentrantLock lock;
        private Condition condition;
        public PrintEven(ReentrantLock lock, Condition condition, Wrapper wrapper) {
            this.lock = lock;
            this.condition = condition;
            this.wrapper = wrapper;
        }

        @Override
        public void run() {
            while (wrapper.getCount() <= 100) {
                lock.lock();
                try {
                    if (wrapper.getCount() % 2 == 1) {
                        condition.await();
                    } else {
                        System.out.println("PrintEven thread print..." + wrapper.getCount());
                        wrapper.setCount(wrapper.getCount() + 1);
                        condition.signal();
                    }
                } catch (Exception ex) {
                    ex.printStackTrace();
                } finally {
                    lock.unlock();
                }
            }
        }
    }
}
```

### 总结

使用synchronized关键字的时候，要保证被锁住的对象不会发生变更。本文所使用的Integer对象以及String之类的基本对象，执行过程中都容易发生对象变更，synchronized使用时要选择不变化的对象。