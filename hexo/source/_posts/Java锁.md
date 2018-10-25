---
title: Java锁
date: 2018-10-13 11:21:53
tags: [锁]
categories: [Java]
---

Java中的锁记录下<!--more-->

### 							JAVA中的锁

* 公平锁/非公平锁

  **非公平锁的性能要比公平锁高，因为不需要维护队列**

  * **公平锁:多个线程按照申请锁的顺序来获取锁，也就是说申请锁的顺序和获取锁的顺序是一致的，加锁前会检查是否有排队等待锁的线程，如果有那么，先来先得**

    * RetrantLock 默认是非公平锁，可以指定构造参数来构建公平锁。

      公平锁

    ```
            package com.convergence.support.lock;
    
            import java.util.concurrent.ExecutorService;
            import java.util.concurrent.Executors;
            import java.util.concurrent.locks.ReentrantLock;
    
            /**
            * 公平锁与非公平锁 之RetrantLock
            * 
            * @author andreby
            *
            */
            public class RetrantLockDemo {
    
            private ReentrantLock reentrantLock;
    
            public RetrantLockDemo(boolean isFair) {
            super();
            this.reentrantLock = new ReentrantLock(isFair);
            }
    
            public void testLock() {
            try {
            reentrantLock.lock();
            System.out.println("线程=" + Thread.currentThread().getName() + "已锁定");
            } catch (Exception e) {
            e.printStackTrace();
            System.out.println("Something is wrong with it ");
            } finally {
            reentrantLock.unlock();
            }
            }
            /**
            * 进行main方法测试公平锁
            * @param args
            */
            public static void main(String[] args) {
            // 指定使用公平锁
            RetrantLockDemo lockDemo = new RetrantLockDemo(true);
            Thread thread = new Thread(new Runnable() {
            @Override
            public void run() {
            System.out.println("进来了" + Thread.currentThread().getName());
            lockDemo.testLock();
            }
            });
            ExecutorService executorService = Executors.newCachedThreadPool();
            for (int i = 0; i < 10; i++) {
            executorService.execute(thread);
            }
            }
    
            }
    
    ```

    输出结果:

    ```
    进来申请锁了pool-1-thread-2
    进来申请锁了pool-1-thread-4
    进来申请锁了pool-1-thread-3
    进来申请锁了pool-1-thread-1
    线程=pool-1-thread-2已锁定
    进来申请锁了pool-1-thread-7
    进来申请锁了pool-1-thread-8
    进来申请锁了pool-1-thread-6
    进来申请锁了pool-1-thread-5
    线程=pool-1-thread-4已锁定
    线程=pool-1-thread-3已锁定
    线程=pool-1-thread-1已锁定
    进来申请锁了pool-1-thread-9
    进来申请锁了pool-1-thread-10
    线程=pool-1-thread-7已锁定
    线程=pool-1-thread-8已锁定
    线程=pool-1-thread-6已锁定
    线程=pool-1-thread-5已锁定
    线程=pool-1-thread-9已锁定
    线程=pool-1-thread-10已锁定
    ```

    那么可以看到申请锁的顺序和获得锁的顺序是一致的

  * **非公平锁：多个线程获取锁的顺序并不是按照申请锁的顺序，有可能后申请的线程比先申请 先获得锁，加锁时候不考虑排队等待问题，直接尝试获取锁，获取不到的话自动到队尾等待，这样的话 有可能出现优先级反转或饥饿现象**

    RetrantLock 非公平锁

    ```
    package com.convergence.support.lock;
    
    import java.util.concurrent.ExecutorService;
    import java.util.concurrent.Executors;
    import java.util.concurrent.locks.ReentrantLock;
    
    /**
     * 公平锁与非公平锁 之RetrantLock
     * 
     * @author andreby
     *
     */
    public class RetrantLockDemo {
    
    	private ReentrantLock reentrantLock;
    
    	public RetrantLockDemo(boolean isFair) {
    		super();
    		this.reentrantLock = new ReentrantLock(isFair);
    	}
    
    	public void testLock() {
    		try {
    			reentrantLock.lock();
    			System.out.println("线程=" + Thread.currentThread().getName() + "已锁定");
    		} catch (Exception e) {
    			e.printStackTrace();
    			System.out.println("Something is wrong with it ");
    		} finally {
    			reentrantLock.unlock();
    		}
    	}
    	/**
    	 * 进行main方法测试公平锁
    	 * @param args
    	 */
    	public static void main(String[] args) {
    		// 指定使用公平锁
    		RetrantLockDemo fairLock = new RetrantLockDemo(true);
    		Thread thread = new Thread(new Runnable() {
    			@Override
    			public void run() {
    				System.out.println("进来申请锁了" + Thread.currentThread().getName());
    				fairLock.testLock();
    			}
    		});
    		ExecutorService fairexecutorService = Executors.newCachedThreadPool();
    		for (int i = 0; i < 10; i++) {
    			fairexecutorService.execute(thread);
    		}
    		//使用非公平锁
    		RetrantLockDemo unFairLock = new RetrantLockDemo(false);
    		Thread unFairThread= new Thread(new Runnable() {
    			@Override
    			public void run() {
    				System.out.println("进来申请锁了" + Thread.currentThread().getName());
    				unFairLock.testLock();				
    			}
    		});
    		System.out.println("=====================非公平锁===========================");
    		ExecutorService unfairexecutorService = Executors.newCachedThreadPool();
    		for (int i = 0; i < 10; i++) {
    			unfairexecutorService.execute(unFairThread);
    		}
    
    	}
    
    }
    ```

    输出结果：

    ```
    =====================非公平锁===========================
    进来申请锁了pool-1-thread-8
    线程=pool-1-thread-8已锁定
    进来申请锁了pool-1-thread-1
    线程=pool-1-thread-1已锁定
    进来申请锁了pool-2-thread-2
    线程=pool-2-thread-2已锁定
    进来申请锁了pool-1-thread-3
    线程=pool-1-thread-3已锁定
    进来申请锁了pool-1-thread-4
    线程=pool-1-thread-4已锁定
    进来申请锁了pool-2-thread-3
    进来申请锁了pool-1-thread-5
    线程=pool-2-thread-3已锁定
    线程=pool-1-thread-5已锁定
    进来申请锁了pool-2-thread-8
    进来申请锁了pool-2-thread-4
    线程=pool-2-thread-8已锁定
    线程=pool-2-thread-4已锁定
    进来申请锁了pool-1-thread-9
    进来申请锁了pool-2-thread-9
    线程=pool-1-thread-9已锁定
    线程=pool-2-thread-9已锁定
    进来申请锁了pool-1-thread-10
    线程=pool-1-thread-10已锁定
    进来申请锁了pool-2-thread-1
    线程=pool-2-thread-1已锁定
    进来申请锁了pool-2-thread-5
    线程=pool-2-thread-5已锁定
    进来申请锁了pool-2-thread-6
    线程=pool-2-thread-6已锁定
    进来申请锁了pool-2-thread-7
    线程=pool-2-thread-7已锁定
    进来申请锁了pool-2-thread-2
    线程=pool-2-thread-2已锁定
    ```

    明显看到是谁申请锁就获取锁。

    Synchronize 非公平锁，不同于RetrantLock 它是不能变成公平锁的

    * 为方法加锁

      * 为静态方法加锁 锁对象是类。

        ```
        	public static synchronized void testStaticSyn() {
        		System.out.println("synchronized 获取到锁了,当前线程=" + Thread.currentThread().getName());
        		try {
        			TimeUnit.MILLISECONDS.sleep(10);
        		} catch (InterruptedException e) {
        			// TODO Auto-generated catch block
        			e.printStackTrace();
        		}
        	}
        		public static void main(String[] args) {
        		System.out.println("=====================非公平锁synchronized===========================");
        		Thread synThread = new Thread(new Runnable() {
        			@Override
        			public void run() {
        				System.out.println("进来申请锁了当前线程=" + Thread.currentThread().getName());
        				RetrantLockDemo.testStaticSyn();
        			}
        		});
        		ExecutorService synexecutorService = Executors.newCachedThreadPool();
        		for (int i = 0; i <5; i++) {
        			synexecutorService.execute(synThread);
        		}
        
        	}
        ```

        输出结果：注意synchronized 获取到锁了 开头的字样为获取到

        ```
        =====================非公平锁synchronized===========================
        进来申请锁了当前线程=pool-1-thread-1
        进来申请锁了当前线程=pool-1-thread-2
        进来申请锁了当前线程=pool-1-thread-3
        synchronized 获取到锁了,当前线程=pool-1-thread-1
        进来申请锁了当前线程=pool-1-thread-4
        进来申请锁了当前线程=pool-1-thread-5
        synchronized 获取到锁了,当前线程=pool-1-thread-2
        synchronized 获取到锁了,当前线程=pool-1-thread-5
        synchronized 获取到锁了,当前线程=pool-1-thread-4
        synchronized 获取到锁了,当前线程=pool-1-thread-3
        ```

      * 为非静态方法加锁 锁对象为当前实例

        **同一个对象在两个线程中分别访问该对象的加锁方法 会互斥**

        **不同对象在两个线程中调用同一个同步方法 不会互斥 **

        下面是不同对象的情况

        ```
        	public synchronized  void testNoStaticStaticSyn() {
        		System.out.println("synchronized 获取到锁了,当前线程=" + Thread.currentThread().getName());
        		try {
        			TimeUnit.MILLISECONDS.sleep(10);
        		} catch (InterruptedException e) {
        			// TODO Auto-generated catch block
        			e.printStackTrace();
        		}
        	}
        		public static void main(String[] args) {
        
        		System.out.println("=====================非公平锁实例方法synchronized===========================");
        
        		Thread synNoStaticThread = new Thread(new Runnable() {
        			@Override
        			public void run() {
        				System.out.println("进来申请锁了当前线程=" + Thread.currentThread().getName());
        				RetrantLockDemo demo= new RetrantLockDemo(false);
        				demo.testNoStaticStaticSyn();
        			}
        		});
        		ExecutorService synexecutorService = Executors.newCachedThreadPool();
        		for (int i = 0; i <5; i++) {
        			synexecutorService.execute(synNoStaticThread);
        		}
        
        	}
        ```

        输出，不互斥

        ```
        =====================非公平锁实例方法synchronized===========================
        进来申请锁了当前线程=pool-1-thread-2
        进来申请锁了当前线程=pool-1-thread-3
        synchronized 获取到锁了,当前线程=pool-1-thread-2
        synchronized 获取到锁了,当前线程=pool-1-thread-3
        进来申请锁了当前线程=pool-1-thread-1
        进来申请锁了当前线程=pool-1-thread-5
        synchronized 获取到锁了,当前线程=pool-1-thread-5
        synchronized 获取到锁了,当前线程=pool-1-thread-1
        进来申请锁了当前线程=pool-1-thread-4
        synchronized 获取到锁了,当前线程=pool-1-thread-4
        ```

        

    * 为代码块加锁

      

      

* 可重入锁

  RetrantLock就是一个可重入锁。可重入锁就是递归锁。指同一个线程在外层方法获取锁的时候，进入内层方法 时会自动获取锁，synchronized也是个可重入锁。可重入锁在一定程度上可以避免思索。

  synchronized也是可重入锁

  ```
  synchronized void setA() throws Exception{
      Thread.sleep(1000);
      setB();
  }
  //对象当setA获取到锁的时候SetA调用的setB方法会自动获取该对象的锁
  synchronized void setB() throws Exception{
      Thread.sleep(1000);
  }
  ```

  

* 独享锁/共享锁

  * 独享锁也是互斥锁，同时只有一个线程能获得这个锁。

    ReentrantLock  为独享锁，ReadWriteLock 也为独享锁。

  * 共享锁 即为多个线程可以同时获得这个锁。

    Semaphore  、CountDownLatch  为共享锁。

    ReadWriteLock  中的读锁为共享锁。写锁为独享锁。

* 互斥锁/读写锁

  互斥锁或读写锁为独享锁/共享锁的具体实现

  互斥锁的具体实现就是ReentrantLock  

  读写锁的具体实现就是ReadWriteLock  

* 乐观锁/悲观锁

  经典的乐观锁/悲观锁的应用场景大概是数据库中的乐观锁与悲观锁

  * 数据库中乐观锁

    表中有个字段为版本号 比如这个字段为version,当第一次进行数据库操作的时候，获取到这个字段，假设其他用户不会进行操作，但是在最后的事务提交阶段，需要进行版本检查，如果这个字段和第一次的时候一样那么就进行提交，否则就不提交。其实这算是应用逻辑层的方式，不是从数据库层面进行加锁。

  * 数据库中的悲观锁

    读取的时候为后面的更新加锁，之后再来的读操作都会等待。这种是数据库锁 ，是从数据库层面进行的操作。

    比如ORM框架 hibernate，可以使用session.lock（）锁定对象实现悲观锁。

    mybatis的话一般用select xxx for update （行锁）lock xxx（表锁） 。但是一般很少有人用悲观锁吧，开销比较大

  java中的乐观锁与悲观锁

  * 悲观锁：

    synchronized 是独占锁，获得该锁的线程才可以执行被锁住的代码，申请该锁的线程只能挂起等待直到该锁释放后才唤醒，拿到锁并进行执行。那么这样 的话开销很大，等待锁的线程在等待期间不能做任何事情，所以它是一种悲观锁。

  * 乐观锁：

    获得锁后会一直持有锁以防本线程再次申请锁导致无谓的解锁加锁开销，或者假设没有冲突而去完成同步代码块如果冲突再循环重试，或者采取申请锁失败后不立刻挂起而是稍微等待再次尝试获取 等待策略，以减少线程因为挂起、阻塞、唤醒（发生 CPU 的调度切换） 而造成的开销。 也就是说失败后会充实避免过大的性能开销。

    偏向锁，轻量级锁(CAS轮询)、自旋锁就属于上述的乐观锁。

* 分段锁

  一个容器中有多把锁，每一把锁用于锁容器中一部分数据，那么当多线程访问容器中不同数据段的数据的时候，线程就不存在锁竞争，从而可以有效的提高并发访问效率，这个在ConcurrentHashMap中 就有用到。

  比如：在 ConcurrentHashMap 中使用了一个包含 16 个锁的数组，每个锁保护所有散列桶的 1/16，其中第 N 个散列桶由第（N mod 16）个锁来保护。假设使用合理的散列算法使关键字能够均匀的分部，那么这大约能使对锁的请求减少到越来的 1/16。也正是这项技术使得 ConcurrentHashMap 支持多达 16 个并发的写入线程。

  缺点是因为对数据容器进行分段加锁，那么锁的维护就是个比较大的开销。

* 偏向锁/轻量级锁/重量级锁

  这三种锁，指的是锁的状态，针对synchronized来说的

  偏向锁只允许一个线程获得锁，加锁和解锁不需要额外的消耗，和执行非同步方法相比仅存在纳秒级的差距，但是如果线程间存在锁竞争，会带来额外的锁撤销的消耗，适用于只有一个线程访问同步块的场景。

  轻量级锁允许多个线程获得锁。竞争的线程不会阻塞，提高相应速度，但是如果始终得不到锁竞争的线程，会自旋消耗cpu,适用于追求响应速度，同步块执行速度非常快的场景。

  重量级锁：线程竞争不使用自旋，不会消耗cpu，但是会导致线程阻塞，响应时间缓慢，适用于追求吞吐量，同步块执行速度比较长的场景。

* 自旋锁

  指尝试获取锁的线程不会立即阻塞，而是采用循环的方式去尝试获取锁，当循环条件被其他线程改变时，才能进入临界区，这样可以减少线程上下文的消耗，但是会消耗cpu资源，比如Disruptor这个框架中就是用了自旋锁

  ```
  public class SpinLock {
   
    private AtomicReference<Thread> sign =new AtomicReference<>();
   
    public void lock(){
      Thread current = Thread.currentThread();
      while(!sign .compareAndSet(null, current)){
      }
    }
    public void unlock (){
      Thread current = Thread.currentThread();
      sign .compareAndSet(current, null);
    }
  }
  ```

  使用了 CAS 原子操作，lock 函数将 owner 设置为当前线程，并且预测原来的值为空。unlock 函数将 owner 设置为 null，并且预测值为当前线程。 

  当有第二个线程调用 lock 操作时由于 owner 值不为空，导致循环一直被执行，直至第一个线程调用 unlock 函数将 owner 设置为 null，第二个线程才能进入临界区。

  由于自旋锁只是将当前线程不停地执行循环体，不进行线程状态的改变，所以响应速度更快。但当线程数不停增加时，性能下降明显，因为每个线程都需要执行，占用 CPU 时间。如果线程竞争不激烈，并且保持锁的时间段。适合使用自旋锁。

  注：该例子为非公平锁，获得锁的先后顺序，不会按照进入 lock 的先后顺序进行。