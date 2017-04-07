---
layout: post
title: Concurrent-Basic
date: 2017-04-08 
tag: java
---

# Basic

## 1. Thread

Java提供了两种方式来创建线程：

+ 继承Thread类，并覆盖run()方法。
+ 创建一个实现了Runnable接口的类。使用带参数的Thread构造器来创建Thread对象。

### 实验

首先，我们通过写一个Calculator类继承Runnable对象，并重写run方法，其中run方法里面放的是我们具体的业务逻辑代码，然后在主线程中开启10个线程来执行这个Runnable对象。

```java
public class Calculator implements Runnable{
   private int number;

   public Calculator(int number) {
       this.number = number;
   }

   @Override
   public void run() {
       for (int i = 1; i <= 10; i++) {
           System.out.printf("%s : %d * %d = %d\n",Thread.currentThread().getName()
           , number, i, i*number);
       }
   }
}
```
```java
public class Main {
   public static void main(String[] args) {
       for(int i=0; i<=10; i++){
           Calculator calculator = new Calculator(i);
           Thread thread = new Thread(calculator);
           thread.start();
       }
   }
}
```
实验结果如下:
![](/images/posts/concurrent/result1.png)

通过这个实验，我们得到的结论如下：

+ 多个线程的执行顺序不可测，它们经常表现出交替执行的顺序，这是因为CPU在多个线程之间切换，得到CPU资源的那个线程才能执行代码。

+ Thread对象才是真正创建线程，开启线程的对象。

+ Runnable更像是一种策略设计模式，线程把具体的执行执行策略或者说业务逻辑交给开发者去实现，具体的方法是实现Runnable对象，并重新Runnable的run方法。

### 验证结论
通过查看源码,我们可以得到如下的注释信息:

```java
 * <p>
 * The other way to create a thread is to declare a class that
 * implements the <code>Runnable</code> interface. That class then
 * implements the <code>run</code> method. An instance of the class can
 * then be allocated, passed as an argument when creating
 * <code>Thread</code>, and started. The same example in this other
 * style looks like the following:
 * <hr><blockquote><pre>
 *     class PrimeRun implements Runnable {
 *         long minPrime;
 *         PrimeRun(long minPrime) {
 *             this.minPrime = minPrime;
 *         }
 *
 *         public void run() {
 *             // compute primes larger than minPrime
 *             &nbsp;.&nbsp;.&nbsp;.
 *         }
 *     }
 * </pre></blockquote><hr>
 * <p>

```
在Thread的start()方法中调用了native修饰start0()方法，这是一个与底层交互的C++代码，我们不需要知道它做了什么，但是我们可以肯定要启动线程，只能通过Thread的start()方法才行。

这是Thread中的run方法，我们发现，如果我们不重新run方法，或者不传入Runnable对象(我们传入的Runnable对象会对target初始化，如果不传，默认为null)

通过对比实验结果与Java源码及注释，我们可以对Thread有个大概的认识和总结：

+ 创建Thread对象并不会创建新的线程，必须调用Thread的start方法才会创建新的线程。

+ 线程执行的代码放在run方法中，我们可以继承Thread对象，然后重写run方法。我们也可以把实现了Runnable接口的对象传给Thread，这样的话，Thread中target会被初始化，然后调用target的run方法。

+ start0()是native方法，会被start()方法调用。

+ Runnable是给线程执行的对象，必须重写run方法，该方法供对象调用.


## 1.1 Thread线程信息
* 线程名 
* 优先级 最大是10 最小是1 默认初始化的线程优先级是5
* 线程组
* 线程状态
* 线程id 默认是自增
* 线程组 ThreadGroup
---
## 1.2 join()方法

	在一些情形下，我们必须等待线程的终止，我们的程序在执行其他的任务时，必须先初始化一些必须的资源。当一个线程对象的join()方法被调用时，调用它的线程被挂起，直到这个线程对象完成它的任务。
---
## 1.3 守护线程

	守护线程的优先级很低，通常被用来作为同一个程序中普通线程的服务提供者，它们通常是无限循环的，以等待服务请求者或者执行线程的任务。当守护线程是程序中唯一运行的线程时，守护线程执行结束后，JVM也就结束了这个程序。
---
## 1.4 不可控异常

	当线程对象的run()方法抛出非运行异常时，我们必须捕获并且处理它们，当运行时异常从run()方法中抛出时，我们通过向Thread设置不可控异常处理器来处理这种异常。ExceptionHandler对象必须实现UncaughtExceptionHandler接口并且实现了这个接口的uncaughtException()方法。当一个线程抛出了异常并且没有被捕获时，JVM检查这个线程是否被预置了未捕获异常处理器，如果找到，JVM将调用线程对象的这个方法，并将线程对象和异常作为传入参数。
---
## 1.5 ThreadGroup

	我们可以对线程进行分组，对组内的线程进行访问控制。Java提供了ThreadGroup类来表示一组对象，线程组可以包含线程对象，也可以包含其他的线程组对象，它时一个树形结构。

* 如果不指定线程组，默认就是和父类线程一个组。
* list()方法 → 打印线程组对象的信息。
* activeCount()方法 →  获取线程组包含的线程数目
* enumerate() → 获取线程组包含的线程列表
* interrupt() → 中断线程组的其余线程

```java
public class SearchTask implements Runnable{
    private Result result;
    public SearchTask(Result result) {
        this.result = result;
    }

    @Override
    public void run() {
        String name = Thread.currentThread().getName();
        System.out.printf("Thread %s: Start\n", name);
        try {
            doTask();
            result.setName(name);
        } catch (InterruptedException e) {
            System.out.printf("Thread %s: Interrupted\n",name);
            return;
        }
        System.out.printf("Thread %s: End\n", name);
    }

    private void doTask() throws InterruptedException{
        Random random = new Random((new Date()).getTime());
        int value = (int)(random.nextDouble()*100);
        System.out.printf("Thread %s: %d\n", Thread.currentThread().getName(), value);
        TimeUnit.SECONDS.sleep(value);
    }
}
   @Test
    public void testThreadGroup(){
        ThreadGroup threadGroup = new ThreadGroup("Searcher");
        Result result = new Result();
        SearchTask searchTask = new SearchTask(result);
        for(int i=0; i<5; i++){
            Thread thread = new Thread(threadGroup, searchTask);
            thread.start();
            try {
                TimeUnit.SECONDS.sleep(1);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }

        System.out.printf("Number of Threads: %d\n", threadGroup.activeCount());
        System.out.printf("Information about the Thread Group\n");
        threadGroup.list();

        Thread[] threads = new Thread[threadGroup.activeCount()];
        threadGroup.enumerate(threads);
        for(int i=0; i<threadGroup.activeCount(); i++){
            System.out.printf("Thread %s : %s\n",threads[i].getName(), threads[i].getState());
        }

        waitFinish(threadGroup);

        threadGroup.interrupt();
    }

  private void waitFinish(ThreadGroup threadGroup) {
        while(threadGroup.activeCount() > 9){
            try {
                TimeUnit.SECONDS.sleep(1);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }

```
---
## 1.6 ThreadFactory

	ThreadFactory接口实现了线程对象工厂，用以生成个性化名称的线程并且保存这些线程对象的统计信息。

```java
public class MyThreadFactory implements ThreadFactory {
    private int counter;
    private String name;
    private List<String> stats;

    public MyThreadFactory(String name) {
        counter = 0;
        this.name = name;
        stats = new ArrayList<String>();
    }

    @Override
    public Thread newThread(Runnable r) {
        Thread thread = new Thread(r, name + "-Thread_" + counter);
        counter++;
        stats.add(String.format("Created thread %d with name %s on %s\n", thread.getId(),thread.getName(), new Date()));
        return thread;
    }

    public String getStats(){
        StringBuffer buffer = new StringBuffer();
        Iterator<String> iterator = stats.iterator();
        while(iterator.hasNext()){
            buffer.append(iterator.next());
            buffer.append("\n");
        }
        return buffer.toString();
    }
}
```
---
# 2. Resource

	共享数据是并发程序最核心的概念，如果资源在多个线程内部，那么线程不能共享数据。让多个线程共享资源的方法：

1. 将资源声明为static，这是最简单的方法，但也是最不适用的方法。
2. 将要执行的业务逻辑封装称一个Runnable对象，将资源放到Runnable对象中，然后把Runnable对象传递给线程来执行，不同的执行线程都可以访问此Runnable。

但是，在某些情况下，这个对象的属性不需要被所有线程共享，Java并发API提供了一个干净的机制，即线程局部变量。

## 2.1 ThreadLocal
	从概念上来讲，可以将ThreadLocal<T>视为包含了Map<Thread,T>的对象，其中保存了特定于该线程的值。当某个线程初次调用ThreadLocal.get方法时，就会调用initialValue来获取初始值。

```java
public class SafeTask implements Runnable{
   private static ThreadLocal<Date> startDate  = new ThreadLocal<Date>(){
       @Override
       protected  Date initialValue(){
           return new Date();
       }
   };

   @Override
   public void run() {
       System.out.printf("Starting Thread: %s : %s\n",Thread.currentThread().getId(), startDate.get());
       try {
           TimeUnit.SECONDS.sleep((int)Math.rint(Math.random() * 10));
       } catch (InterruptedException e) {
           e.printStackTrace();
       }
       System.out.printf("Thread Finished: %s : %s\n", Thread.currentThread().getId(), startDate.get());
   }
}
```