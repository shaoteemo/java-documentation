本文档翻译官方文档：https://docs.oracle.com/javase/tutorial/essential/concurrency/index.html。

版权信息请参考：https://www.oracle.com/java/technologies/javase-documentation.html。

# Java™ 教程中文版之并发编程(官方版)

> *The Java Tutorials have been written for JDK 8. Examples and practices described in this page don't take advantage of improvements introduced in later releases and might use technology no longer available.*
> *See* [Java Language Changes](https://docs.oracle.com/pls/topic/lookup?ctx=en/java/javase&id=java_language_changes) *for a summary of updated language features in Java SE 9 and subsequent releases.*
> *See* [JDK Release Notes](https://www.oracle.com/technetwork/java/javase/jdk-relnotes-index-2162236.html) *for information about new features, enhancements, and removed or deprecated options for all JDK releases.*

计算机用户认为他们的系统一次可以同时做不止一件事是理所当然的。他们假设他们可以继续在文字处理器中工作，而其他应用程序下载文件、管理打印队列和流式传输音频。即使是单个应用程序，通常也期望一次做不止一件事。例如，流音频应用程序必须同时从网络读取数字音频、解压缩、管理播放和更新其显示。即使文字处理器也应该随时准备好响应键盘和鼠标事件，无论它重新格式化文本或更新显示有多忙。可以做这些事情的软件被称为*并发*软件。

Java 平台从头开始设计以支持并发编程，在 Java 编程语言和 Java 类库中具有基本的并发支持。从 5.0 版开始，Java 平台还包含高级并发 API。本课介绍了平台的基本并发支持，并总结了 `java.util.concurrent` 包中的一些高级 API。

## 进程与线程

在并发编程中，有两个基本的执行单元：进程和线程。在 Java 编程语言中，并发编程主要关注线程。然而，进程也很重要。

计算机系统通常有许多活动进程和线程。即使在只有一个执行核心的系统中也是如此，因此在任何给定时刻只有一个线程实际执行。单个内核的处理时间通过称为**时间片**的操作系统功能在进程和线程之间共享。

计算机系统具有多个处理器或具有多个执行核心的处理器变得越来越普遍。这极大地增强了系统并发执行进程和线程的能力——但即使在没有多个处理器或多个执行核心的简单系统上也可以实现并发。

### 进程

一个进程有一个自包含的执行环境。一个进程通常有一套完整的、私有的基本运行时资源；特别的是，每个进程都有自己的内存空间。

进程通常被视为程序或应用程序的同义词。然而，用户看到的单个应用程序实际上可能是一组协作进程。为了促进进程之间的通信，大多数操作系统都支持**进程间通信** (IPC： *Inter Process Communication*) 资源，例如管道和套接字。IPC 不仅用于同一系统上的进程之间的通信，还用于不同系统上的进程之间的通信。

Java 虚拟机的大多数实现都作为单个进程运行。Java 应用程序可以使用`ProcessBuilder` 对象创建其他进程。多进程应用程序不在本课程范围内。

### 线程

线程有时被称为轻量级进程。进程和线程都提供了一个执行环境，但是创建一个新线程比创建一个新进程需要更少的资源。

线程存在于一个进程中——每个进程至少有一个。线程共享进程的资源，包括内存和打开的文件。这有助于实现高效通信但也可能存在一些问题。

多线程执行是 Java 平台的一个基本特性。每个应用程序至少有一个线程，如果你算上执行内存管理和信号处理之类的“系统”线程的话此时应用程序有多个线程。但从应用程序员的角度来看，您只从一个线程开始，称为主线程。这个线程能够创建额外的线程，我们将在下一节中演示。

## Java中的线程对象

每个线程都与 Thread 类的一个实例相关联。使用 Thread 对象创建并发应用程序有两种基本策略。

- 要直接控制线程的创建和管理，只需在每次应用程序需要启动异步任务时实例化 Thread。
- 要从应用程序的其余部分抽象线程管理，请将应用程序的任务传递给执行程序。

本节记录了 Thread 对象的使用。 Executors 与其他高级并发对象一起讨论。

### 定义和启动线程

创建 `Thread` 实例的应用程序必须提供将在该线程中运行的代码。有两种方法可以做到这一点：

- 提供一个 `Runnable` 对象。 Runnable 接口定义了一个单独的方法 `run`，用于包含在线程中执行的代码。 Runnable 对象被传递给 Thread 构造函数，如 `HelloRunnable` 示例中所示：

  ```java
  public class HelloRunnable implements Runnable {
  
      public void run() {
          System.out.println("Hello from a thread!");
      }
  
      public static void main(String args[]) {
          (new Thread(new HelloRunnable())).start();
      }
  
  }
  ```

  - 子类线程（即继承 Thread）。 `Thread` 类本身实现了 `Runnable`，尽管它的 `run` 方法什么也不做。应用程序可以子类化 `Thread`，提供自己的 `run` 实现，如 `HelloThread` 示例中所示：

  ```java
  public class HelloThread extends Thread {
  
      public void run() {
          System.out.println("Hello from a thread!");
      }
  
      public static void main(String args[]) {
          (new HelloThread()).start();
      }
  
  }
  ```

请注意，两个示例都调用 `Thread.start` 以启动新线程。

您应该使用以下哪些方式？第一个使用 `Runnable` 对象的习惯用法更通用，因为 `Runnable` 对象可以子类化除 `Thread` 之外的类。第二个习惯用法在简单的应用程序中更容易使用，但受到任务类必须是 `Thread` 的子类这一事实的限制（译注：失去了多继承）。本课重点介绍第一种方法，它将 `Runnable` 任务与执行任务的 `Thread` 对象分开。这种方法不仅更加灵活，而且适用于后面介绍的高级线程管理 API。

Thread 类定义了许多对线程管理有用的方法。其中包括静态方法，它提供有关调用该方法的当前线程的信息或影响其状态。其他方法从参与管理线程和 Thread 对象的其他线程调用。我们将在以下部分中研究其中一些方法。

### 使用睡眠暂停执行

`Thread.sleep` 使当前线程暂停执行一段指定的时间。这是使让出处理器资源可用于应用程序的其他线程或可能在计算机系统上运行的其他应用程序的有效方法。`sleep` 方法也可用于调步，如下面的示例所示，并等待另一个具有时间要求的任务的线程，如后面部分中的 `SimpleThreads` 示例。

提供了两种重载的睡眠方法：一种将睡眠时间指定为毫秒，另一种将睡眠时间指定为纳秒。但是，不能保证这些睡眠时间是精确的，因为它们受到底层操作系统提供的设施的限制。此外，睡眠周期可以通过中断终止，我们将在后面的部分中讲解。在任何情况下，您都不能假设调用 sleep 将在指定的时间段内暂停线程。

`SleepMessages` 示例使用 `sleep` 以四秒的间隔打印消息：

```java
public class SleepMessages {
    public static void main(String args[])
        throws InterruptedException {
        String importantInfo[] = {
            "Mares eat oats",
            "Does eat oats",
            "Little lambs eat ivy",
            "A kid will eat ivy too"
        };

        for (int i = 0;
             i < importantInfo.length;
             i++) {
            //Pause for 4 seconds
            Thread.sleep(4000);
            //Print a message
            System.out.println(importantInfo[i]);
        }
    }
}
```

注意 `main` 方法声明它抛出 `InterruptedException`。这是当 `sleep` 处于活动状态时另一个线程中断当前线程时 `sleep` 抛出的异常。由于这个应用程序没有定义另一个线程来引起中断，所以它不会费心去捕获 `InterruptedException`。

### 线程中断

*中断*是对线程的指示，它应该停止正在执行的操作并执行其他操作。由程序员决定线程如何响应中断，但线程终止是很常见的。这是本课强调的用法。

线程通过对要中断的线程调用 Thread 对象上的`interrupt`来发送中断。为了使中断机制正常工作，被中断的线程必须支持自己的中断。

#### 中断支持

一个线程如何支持自己的中断？这取决于它目前在做什么。如果线程频繁调用抛出 `InterruptedException`的方法，它会在捕获该异常后简单地从 `run` 方法返回。例如，假设 `SleepMessage` 示例中的打印消息循环位于线程的 `Runnable` 对象的 `run` 方法中。那么它可能会修改成如下代码以支持中断：

```java
for (int i = 0; i < importantInfo.length; i++) {
    // Pause for 4 seconds
    try {
        Thread.sleep(4000);
    } catch (InterruptedException e) {
        // We've been interrupted: no more messages.
        return;
    }
    // Print a message
    System.out.println(importantInfo[i]);
}
```

许多抛出 `InterruptedException` 的方法，例如 `sleep`，旨在取消其当前操作并在收到中断时立即返回。

如果一个线程运行很长时间没有调用抛出 `InterruptedException` 的方法怎么办？然后它必须定期调用 `Thread.interrupted`，如果接收到中断，则返回 true。例如：

```java
for (int i = 0; i < inputs.length; i++) {
    heavyCrunch(inputs[i]);
    if (Thread.interrupted()) {
        // We've been interrupted: no more crunching.
        return;
    }
}
```

在这个简单的例子中，代码只是简单地测试中断并在收到中断后退出线程。在更复杂的应用程序中，抛出 `InterruptedException` 可能更有意义：

```java
if (Thread.interrupted()) {
    throw new InterruptedException();
}
```

这允许将中断处理代码集中在 catch 子句中。

#### 中断状态标志（Interrupts）

中断机制是使用称为中断状态的内部标志实现的。调用 `Thread.interrupt` 设置此标志。当线程通过调用静态方法 `Thread.interrupted` 检查中断时，中断状态被清除。一个线程使用非静态 `isInterrupted` 方法来查询另一个线程的中断状态，它不会更改中断状态标志。

按照惯例，任何通过抛出 `InterruptedException` 退出的方法都会在它这样做时清除中断状态。然而，中断状态总是有可能被另一个调用中断的线程立即再次设置。

### Joins

`join` 方法允许一个线程等待另一个线程的完成。如果 t 是其线程当前正在执行的 `Thread` 对象， `t.join();`导致当前线程暂停执行，直到 t 的线程终止。`join` 的重载允许程序员指定一个等待期。但是，与 `sleep` 一样，`join` 依赖于操作系统的计时，因此您不应假设 `join` 将等待您指定的时间。

### `SimpleThreads` 例子

下面的例子汇集了本节的一些概念。 `SimpleThreads` 由两个线程组成。第一个是每个 Java 应用程序都有的主线程。主线程从 `Runnable` 对象 `MessageLoop` 创建一个新线程，并等待它完成。 如果 `MessageLoop` 线程完成时间过长，主线程就会中断它。

`MessageLoop` 线程打印出一系列消息。 如果在打印所有消息之前被中断，`MessageLoop` 线程将打印一条消息并退出。

```JAVA
/*
 * Copyright (c) 1995, 2008, Oracle and/or its affiliates. All rights reserved.
 *
 * Redistribution and use in source and binary forms, with or without
 * modification, are permitted provided that the following conditions
 * are met:
 *
 *   - Redistributions of source code must retain the above copyright
 *     notice, this list of conditions and the following disclaimer.
 *
 *   - Redistributions in binary form must reproduce the above copyright
 *     notice, this list of conditions and the following disclaimer in the
 *     documentation and/or other materials provided with the distribution.
 *
 *   - Neither the name of Oracle or the names of its
 *     contributors may be used to endorse or promote products derived
 *     from this software without specific prior written permission.
 *
 * THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS "AS
 * IS" AND ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO,
 * THE IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR
 * PURPOSE ARE DISCLAIMED.  IN NO EVENT SHALL THE COPYRIGHT OWNER OR
 * CONTRIBUTORS BE LIABLE FOR ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL,
 * EXEMPLARY, OR CONSEQUENTIAL DAMAGES (INCLUDING, BUT NOT LIMITED TO,
 * PROCUREMENT OF SUBSTITUTE GOODS OR SERVICES; LOSS OF USE, DATA, OR
 * PROFITS; OR BUSINESS INTERRUPTION) HOWEVER CAUSED AND ON ANY THEORY OF
 * LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY, OR TORT (INCLUDING
 * NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE OF THIS
 * SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.
 */
 
public class SimpleThreads {
 
    // Display a message, preceded by
    // the name of the current thread
    static void threadMessage(String message) {
        String threadName =
            Thread.currentThread().getName();
        System.out.format("%s: %s%n",
                          threadName,
                          message);
    }
 
    private static class MessageLoop
        implements Runnable {
        public void run() {
            String importantInfo[] = {
                "Mares eat oats",
                "Does eat oats",
                "Little lambs eat ivy",
                "A kid will eat ivy too"
            };
            try {
                for (int i = 0;
                     i < importantInfo.length;
                     i++) {
                    // Pause for 4 seconds
                    Thread.sleep(4000);
                    // Print a message
                    threadMessage(importantInfo[i]);
                }
            } catch (InterruptedException e) {
                threadMessage("I wasn't done!");
            }
        }
    }
 
    public static void main(String args[])
        throws InterruptedException {
 
        // Delay, in milliseconds before
        // we interrupt MessageLoop
        // thread (default one hour).
        long patience = 1000 * 60 * 60;
 
        // If command line argument
        // present, gives patience
        // in seconds.
        if (args.length > 0) {
            try {
                patience = Long.parseLong(args[0]) * 1000;
            } catch (NumberFormatException e) {
                System.err.println("Argument must be an integer.");
                System.exit(1);
            }
        }
 
        threadMessage("Starting MessageLoop thread");
        long startTime = System.currentTimeMillis();
        Thread t = new Thread(new MessageLoop());
        t.start();
 
        threadMessage("Waiting for MessageLoop thread to finish");
        // loop until MessageLoop
        // thread exits
        while (t.isAlive()) {
            threadMessage("Still waiting...");
            // Wait maximum of 1 second
            // for MessageLoop thread
            // to finish.
            t.join(1000);
            if (((System.currentTimeMillis() - startTime) > patience)
                  && t.isAlive()) {
                threadMessage("Tired of waiting!");
                t.interrupt();
                // Shouldn't be long now
                // -- wait indefinitely
                t.join();
            }
        }
        threadMessage("Finally!");
    }
}
```

## 同步（Synchronization）

线程主要通过共享对字段和引用字段所引用的对象的访问来进行通信。这种形式的通信非常有效，但会导致两种错误：*线程干扰*和*内存一致性错误*。防止这些错误所需的方法是同步。

但是，同步会引入线程抢占，当两个或多个线程尝试同时访问同一资源并导致 Java 运行时更慢地执行一个或多个线程，甚至挂起它们的执行时，就会发生这种情况。饥饿和活锁是线程争用的形式。有关更多信息，请参阅活跃度部分。

本节涵盖以下主题：

- **线程干扰（Thread Interference）**描述了当多个线程访问共享数据时如何引入错误。
- **内存一致性错误（Memory Consistency Errors）**描述了由共享内存的不一致视图导致的错误。
- **同步方法（Synchronized Methods）** 描述了一个简单的习惯用法，可以有效地防止线程干扰和内存一致性错误。
- **隐式锁和同步（Implicit Locks and Synchronization）**描述了一个更通用的同步习惯用法，并描述了同步是如何基于隐式锁的。
- **Atomic Access** 谈到了不能被其他线程干扰的操作的一般思想。

### 线程干扰（Thread Interference）

考虑一个名为 `Counter` 的简单类

```JAVA
class Counter {
    private int c = 0;

    public void increment() {
        c++;
    }

    public void decrement() {
        c--;
    }

    public int value() {
        return c;
    }

}
```

`Counter`被设计成每次调用`increment`都会给 c 加 1，每次调用`decrement`都会从 c 中减去 1。但是，如果一个`Counter`对象被多个线程引用，线程之间的干扰可能会阻止这种情况按预期发生。

当在不同线程中运行但作用于相同数据的两个操作交错时，就会发生干扰。这意味着这两个操作由多个步骤组成，并且步骤序列重叠。

`Counter` 实例上的操作似乎不可能交错，因为`c`上的两个操作都是单个简单的语句。但是，即使是简单的语句也可以通过虚拟机转换为多个步骤。我们不会检查虚拟机采取的具体步骤——知道单个表达式 `c++` 可以分解为三个步骤就足够了：

1. 检索 c 的当前值。
2. 将检索到的值增加 1。
3. 将增加的值存储回 c。

表达式`c--`可以用同样的方式分解，除了第二步是递减而不是递增。

假设线程 A 调用`increment`大约在线程 B 调用`decrement`的同时。如果 c 的初始值为 0，则它们的交错操作可能遵循以下顺序：

1. 线程 A：检索 c。
2. 线程 B：检索 c。
3. 线程A：递增检索值；结果是 1。
4. 线程 B：减少检索到的值；结果是-1。
5. 线程 A：将结果存储在 c 中； c 现在是 1。
6. 线程 B：将结果存储在 c 中； c 现在是 -1。

线程 A 的结果丢失，被线程 B 覆盖。这种特殊的交织只是一种可能性。在不同的情况下，可能是线程 B 的结果丢失了，也可能根本没有错误。因为它们是不可预测的，线程干扰错误可能难以检测和修复。

### 内存一致性错误（Memory Consistency Errors）

当不同的线程对应该是相同数据的内容有不一致的看法时，就会发生内存一致性错误。内存一致性错误的原因很复杂，超出了本教程的范围。幸运的是，程序员不需要详细了解这些原因。所需要的只是避免它们的策略。

避免内存一致性错误的关键是理解*happens-before*关系。这种关系只是保证一个特定语句写入的内存对另一个特定语句可见。要了解这一点，请考虑以下示例。假设定义并初始化了一个简单的`int`字段：

```JAVA
int counter = 0;
```

`counter`字段在两个线程 A 和 B 之间共享。假设线程 A 递增计数器：

```JAVA
counter++;
```

然后，不久之后，线程 B 打印出`counter`：

```java
System.out.println(counter);
```

如果这两个语句是在同一个线程中执行的，那么可以安全地假设打印出的值是“1”。但是如果这两个语句在不同的线程中执行，打印出来的值很可能是“0”，因为不能保证线程 A 对`counter`的更改对线程 B 是可见的——除非程序员在两者之间建立了happens-before 关系这两个声明。

有几个操作可以创建 happens-before 关系。其中之一是同步，我们将在以下部分中讲述。

- 当一个语句调用`Thread.start`时，与该语句有happens-before每个语句也与新线程执行的每个语句有happens-before。
- 当一个线程终止并导致另一个线程中的`Thread.join`返回时，则被终止线程执行的所有语句与成功加入之后的所有语句都具有happens-before 关系。

有关创建happens-before的操作列表，请参阅`java.util.concurrent`包的摘要页面。

### 同步方法（Synchronized Methods）

Java 编程语言提供了两种基本的同步习惯用法：*同步方法*和*同步语句*。下一节将描述两个同步语句中更复杂的语句。本节是关于同步方法的。

```java
public class SynchronizedCounter {
    private int c = 0;

    public synchronized void increment() {
        c++;
    }

    public synchronized void decrement() {
        c--;
    }

    public synchronized int value() {
        return c;
    }
}
```

如果`count`是`SynchronizedCounter`的实例，那么使这些方法同步有两个效果：

- 首先，对同一对象的同步方法的两次调用不可能交错。当一个线程正在为一个对象执行同步方法时，所有其他调用同一个对象的同步方法的线程都会阻塞（挂起执行），直到第一个线程完成对对象的处理。
- 其次，当一个同步方法退出时，它会自动建立一个发生在同一个对象的同步方法的任何后续调用之前的关系。这保证了对象状态的更改对所有线程都是可见的。

请注意，构造函数不能同步——在构造函数中使用 synchronized 关键字是一个语法错误。同步构造函数没有意义，因为只有创建对象的线程才能在构造对象时访问它。

> 警告：在构造将在线程之间共享的对象时，请务必小心，以免对对象的引用过早地“泄漏”。例如，假设您要维护一个名为实例的列表，其中包含类的每个实例。您可能想在构造函数中添加以下行：
>
> ```JAVA
> instances.add(this);
> ```
>
> 但是其他线程可以在对象的构造完成之前使用实例来访问该对象。

同步方法启用了一种防止线程干扰和内存一致性错误的简单策略：如果一个对象对多个线程可见，则对该对象变量的所有读取或写入都通过`synchronized`方法完成。（一个重要的例外：final 字段，在对象被构造后不能被修改，一旦对象被构造，就可以通过非同步方法安全地读取）这种策略是有效的，但会带来活跃度方面的问题，我们将在本课后面看到。

### 内部锁和同步

同步是围绕称为内在锁或监视器锁的内部实体构建的。（API 规范通常将此实体简称为“监视器”。）内在锁在同步的两个方面都发挥作用：强制对对象状态进行独占访问并建立对可见性至关重要的发生前关系（happens-before）。

每个对象都有一个与之关联的内在锁。按照惯例，需要对对象字段进行独占且一致访问的线程必须在访问对象之前获取对象的内在锁，然后在访问完成后释放内在锁。线程在获取锁和释放锁之间被称为拥有内在锁。只要一个线程拥有一个内在锁，其他线程就不能获得相同的锁。另一个线程在尝试获取锁时会阻塞。

当一个线程释放一个内在锁时，在该操作和相同锁的任何后续获取之间建立了一个happens-before 关系。

#### 同步方法中的锁

当线程调用同步方法时，它会自动获取该方法对象的内在锁，并在该方法返回时释放它。即使返回是由未捕获的异常引起的，也会释放锁。

您可能想知道调用静态同步方法时会发生什么，因为静态方法与类相关联，而不是与对象相关联。在这种情况下，线程获取与类关联的 Class 对象的内在锁。因此，对类的静态字段的访问由与类的任何实例的不同的锁控制。

#### 同步语句

创建同步代码的另一种方法是使用同步语句。与同步方法不同，同步语句必须指定提供内在锁的对象：

```java
public void addName(String name) {
    synchronized(this) {
        lastName = name;
        nameCount++;
    }
    nameList.add(name);
}
```

在这个例子中，`addName`方法需要同步对`lastName`和`nameCount`的更改，但也需要避免同步其他对象的方法的调用。（从同步代码调用其他对象的方法可能会产生在 Liveness 部分中描述的问题。）如果没有同步语句，就必须有一个单独的、非同步的方法，其唯一目的是调用`nameList.add`。

同步语句对于通过细粒度同步提高并发性也很有用。例如，假设类`MsLunch`有两个从未一起使用的实例字段`c1`和`c2`。这些字段的所有更新都必须同步，但没有理由阻止`c1`的更新与`c2`的更新交错 ---- 这样做会通过创建不必要的阻塞来减少并发性。我们不使用同步方法或以其他方式使用与`此(this)`关联的锁，而是创建两个对象来单独提供锁。

```java
public class MsLunch {
    private long c1 = 0;
    private long c2 = 0;
    private Object lock1 = new Object();
    private Object lock2 = new Object();

    public void inc1() {
        synchronized(lock1) {
            c1++;
        }
    }

    public void inc2() {
        synchronized(lock2) {
            c2++;
        }
    }
}
```

使用这个习语要格外小心。您必须绝对确定交错访问受影响的字段确实是安全的。

#### 重入同步

回想一下，一个线程不能获得另一个线程拥有的锁。但是一个线程可以获得一个它已经拥有的锁。允许一个线程多次获取同一个锁可以实现可重入同步。这描述了一种情况，即同步代码直接或间接调用了一个也包含同步代码的方法，并且两组代码使用相同的锁。如果没有可重入同步，同步代码将不得不采取许多额外的预防措施来避免线程导致自身阻塞（译注：死锁。）。

### 原子访问（Atomic Access）

在编程中，原子操作是一次有效地发生的操作。原子操作不能在中间停止：它要么完全发生，要么根本不发生。在操作完成之前，原子操作的副作用是不可见的。

我们已经看到增量表达式，例如 c++，不描述原子操作。即使是非常简单的表达式也可以定义可以分解为其他动作的复杂动作。但是，您可以指定一些原子操作：

- 对于引用变量和大多数原始变量（除 long 和 double 之外的所有类型），读取和写入是原子的。
- 对于所有声明为 volatile 的变量（包括 long 和 double 变量），读取和写入都是原子的。

原子操作不能交错，因此可以使用它们而不必担心线程干扰。然而，这并不能消除所有同步原子操作的需要，因为内存一致性错误仍然可能发生。使用`volatile`变量可以降低内存一致性错误的风险，因为对`volatile`变量的任何写入都会与对该同一变量的后续读取建立先发生关系。这意味着对`volatile`变量的更改始终对其他线程可见。更重要的是，这也意味着当一个线程读取一个`volatile`变量时，它不仅会看到对`volatile`的最新更改，还会看到导致更改的代码的副作用。

使用简单的原子变量访问比通过同步代码访问这些变量更有效，但需要程序员更加小心以避免内存一致性错误。额外的努力是否值得取决于应用程序的大小和复杂性。

`java.util.concurrent`包中的一些类提供不依赖于同步的原子方法。我们将在高级并发对象部分讨论它们。

## Liveness

并发应用程序及时执行的能力被称为活跃度（*liveness*）。本节描述最常见的一种活性问题，即死锁（deadlock），并继续简要描述另外两种活性问题，饥饿（starvation）和活锁（livelock）。

### 死锁（Deadlock）

死锁描述了两个或多个线程被永远阻塞、互相等待的情况。这是一个例子。

阿尔方斯和加斯顿是朋友，也是礼貌的忠实信徒。一个严格的礼貌规则是，当你向朋友鞠躬时，你必须保持鞠躬，直到你的朋友有机会还礼为止。不幸的是，这条规则没有考虑到两个朋友可能同时向对方鞠躬的可能性。这个示例应用程序`Deadlock`模拟了这种可能性：

```JAVA
public class Deadlock {
    static class Friend {
        private final String name;
        public Friend(String name) {
            this.name = name;
        }
        public String getName() {
            return this.name;
        }
        public synchronized void bow(Friend bower) {
            System.out.format("%s: %s"
                + "  has bowed to me!%n", 
                this.name, bower.getName());
            bower.bowBack(this);
        }
        public synchronized void bowBack(Friend bower) {
            System.out.format("%s: %s"
                + " has bowed back to me!%n",
                this.name, bower.getName());
        }
    }

    public static void main(String[] args) {
        final Friend alphonse =
            new Friend("Alphonse");
        final Friend gaston =
            new Friend("Gaston");
        new Thread(new Runnable() {
            public void run() { alphonse.bow(gaston); }
        }).start();
        new Thread(new Runnable() {
            public void run() { gaston.bow(alphonse); }
        }).start();
    }
}
```

当死锁运行时，两个线程在尝试调用`bowBack`时极有可能会阻塞。两个块都不会结束，因为每个线程都在等待另一个退出`bow`。

### 饥饿和活锁（Starvation and Livelock）

饥饿和活锁比死锁更不常见，但仍然是每个并发软件设计者可能会遇到的问题。

#### 线程饥饿（Starvation）

饥饿描述了线程无法定期访问共享资源并且无法取得对应进展的情况。当共享资源被“贪婪（greedy）”线程长时间不可用时，就会发生这种情况。例如，假设一个对象提供了一个通常需要很长时间才能返回的同步方法。如果一个线程频繁调用这个方法，其他同样需要频繁同步访问同一对象的线程也会经常被阻塞。

#### 活锁（Livelock）

一个线程通常会响应另一个线程的操作。如果另一个线程的动作也是对另一个线程动作的响应，则可能会导致活锁。与死锁一样，活锁线程无法取得进一步进展。然而，线程并没有被阻塞——它们只是忙于相互响应而无法恢复工作。这类似于两个人在走廊中试图通过对方：阿尔方斯向左移动让加斯顿通过，而加斯顿向右移动让阿尔方斯通过。看到他们还在互相阻挡，阿尔方斯移动到他的右边，而加斯顿移动到他的左边。他们还在互相挡着，所以……

## 保护块（Guarded Blocks）

线程通常必须协调它们的行为。最常见的协调习惯用法是**受保护的块**。这样的块首先轮询块可以继续之前必须为true的条件。要正确执行此操作，需要遵循许多步骤。

假设，例如，`guardedJoy`是一个方法，在另一个线程设置共享变量`joy`之前，该方法不得继续。理论上，这种方法可以简单地循环直到条件满足，但该循环是多余的，因为它在等待时持续执行。

```java
public void guardedJoy() {
    // Simple loop guard. Wastes
    // processor time. Don't do this!
    while(!joy) {}
    System.out.println("Joy has been achieved!");
}
```

更有效的守护调用`Object.wait`来挂起当前线程。在另一个线程发出可能发生了某些特殊事件的通知之前，wait 的调用不会返回——尽管不一定是该线程正在等待的事件：

```java
public synchronized void guardedJoy() {
    // This guard only loops once for each special event, which may not
    // be the event we're waiting for.
    while(!joy) {
        try {
            wait();
        } catch (InterruptedException e) {}
    }
    System.out.println("Joy and efficiency have been achieved!");
}
```

> 注意：始终在测试等待条件的循环内调用 wait。不要假设中断是针对您正在等待的特定条件，或者条件仍然为真。

像许多挂起执行的方法一样，`wait`可以抛出`InterruptedException`。在这个例子中，我们可以忽略那个异常——我们只关心`joy`的价值。

为什么这个版本的`guardedJoy`是同步的？假设`d`是我们用来调用`wait`的对象。当一个线程调用`d.wait`时，它必须拥有 d 的内部锁——否则会抛出错误。在同步方法中调用等待是获取内部锁的简单方法。

当调用`wait`时，线程释放锁并挂起执行。在未来的某个时间，另一个线程将获取相同的锁并调用`Object.notifyAll`，通知所有等待该锁的线程发生了一些重要的事情：

```JAVA
public synchronized notifyJoy() {
    joy = true;
    notifyAll();
}
```

在第二个线程释放锁一段时间后，第一个线程重新获取锁并通过从`wait`调用返回来恢复运行状态。

> 注意：还有第二种通知方法`notify`，它可以唤醒单个线程。因为`notify`不允许您指定被唤醒的线程，所以它仅在大规模并行应用程序中有用——也就是说，具有大量线程的程序都在做类似的工作。在这样的应用程序中，您不关心哪个线程被唤醒。

让我们使用受保护的块来创建一个生产者-消费者应用程序。这种应用程序在两个线程之间共享数据：创建数据的生产者和处理数据的消费者。这两个线程使用共享对象进行通信。协调是必不可少的：在生产者线程交付数据之前，消费者线程不得尝试检索数据，如果消费者尚未检索旧数据，则生产者线程不得尝试交付新数据。

在这个例子中，数据是一系列文本消息，它们通过一个`Drop`类型的对象共享：

```JAVA
public class Drop {
    // Message sent from producer
    // to consumer.
    private String message;
    // True if consumer should wait
    // for producer to send message,
    // false if producer should wait for
    // consumer to retrieve message.
    private boolean empty = true;

    public synchronized String take() {
        // Wait until message is
        // available.
        while (empty) {
            try {
                wait();
            } catch (InterruptedException e) {}
        }
        // Toggle status.
        empty = true;
        // Notify producer that
        // status has changed.
        notifyAll();
        return message;
    }

    public synchronized void put(String message) {
        // Wait until message has
        // been retrieved.
        while (!empty) {
            try { 
                wait();
            } catch (InterruptedException e) {}
        }
        // Toggle status.
        empty = false;
        // Store message.
        this.message = message;
        // Notify consumer that status
        // has changed.
        notifyAll();
    }
}
```

`Producer`中定义的生产者线程发送一系列消息。字符串“DONE”表示所有消息都已发送。为了模拟现实世界应用程序的不可预测性，生产者线程暂停消息之间的随机间隔。

```JAVA
import java.util.Random;

public class Producer implements Runnable {
    private Drop drop;

    public Producer(Drop drop) {
        this.drop = drop;
    }

    public void run() {
        String importantInfo[] = {
            "Mares eat oats",
            "Does eat oats",
            "Little lambs eat ivy",
            "A kid will eat ivy too"
        };
        Random random = new Random();

        for (int i = 0;
             i < importantInfo.length;
             i++) {
            drop.put(importantInfo[i]);
            try {
                Thread.sleep(random.nextInt(5000));
            } catch (InterruptedException e) {}
        }
        drop.put("DONE");
    }
}
```

在`Consumer`中定义的消费者线程只是检索消息并将它们打印出来，直到它检索到“DONE”字符串。该线程也会随机暂停。

```JAVA
import java.util.Random;

public class Consumer implements Runnable {
    private Drop drop;

    public Consumer(Drop drop) {
        this.drop = drop;
    }

    public void run() {
        Random random = new Random();
        for (String message = drop.take();
             ! message.equals("DONE");
             message = drop.take()) {
            System.out.format("MESSAGE RECEIVED: %s%n", message);
            try {
                Thread.sleep(random.nextInt(5000));
            } catch (InterruptedException e) {}
        }
    }
}
```

最后，这是在`ProducerConsumerExample`中定义的主线程，它启动生产者和消费者线程。

```JAVA
public class ProducerConsumerExample {
    public static void main(String[] args) {
        Drop drop = new Drop();
        (new Thread(new Producer(drop))).start();
        (new Thread(new Consumer(drop))).start();
    }
}
```

> 注意：编写 Drop 类是为了演示受保护的块。为避免重新发明轮子，请在尝试编写自己的数据共享对象之前检查 Java 集合框架中的现有数据结构。有关更多信息，请参阅问题和练习部分。

## 不可变对象（Immutable Objects）

如果对象在构造后其状态无法更改，则该对象被认为是不可变的。最大程度地依赖不可变对象被广泛接受为创建简单、可靠代码的合理策略。

不可变对象在并发应用程序中特别有用。因为它们不能改变状态，所以它们不会被线程干扰破坏或在不一致的状态下被观察到。

程序员通常不愿意使用不可变对象，因为他们担心创建新对象而不是更新对象的成本。对象创建的影响通常被高估，并且可以被与不可变对象相关的一些效率所抵消。 这些包括由于垃圾收集而减少的开销，以及消除保护可变对象免受损坏所需的代码。

以下小节采用一个实例可变的类，并从中派生出一个具有不可变实例的类。 这样做时，他们给出了这种转换的一般规则，并展示了不可变对象的一些优点。

### 同步类示例

`SynchronizedRGB`类定义了表示颜色的对象。 每个对象将颜色表示为三个代表原色值的整数和一个给出颜色名称的字符串。

```JAVA
public class SynchronizedRGB {

    // Values must be between 0 and 255.
    private int red;
    private int green;
    private int blue;
    private String name;

    private void check(int red,
                       int green,
                       int blue) {
        if (red < 0 || red > 255
            || green < 0 || green > 255
            || blue < 0 || blue > 255) {
            throw new IllegalArgumentException();
        }
    }

    public SynchronizedRGB(int red,
                           int green,
                           int blue,
                           String name) {
        check(red, green, blue);
        this.red = red;
        this.green = green;
        this.blue = blue;
        this.name = name;
    }

    public void set(int red,
                    int green,
                    int blue,
                    String name) {
        check(red, green, blue);
        synchronized (this) {
            this.red = red;
            this.green = green;
            this.blue = blue;
            this.name = name;
        }
    }

    public synchronized int getRGB() {
        return ((red << 16) | (green << 8) | blue);
    }

    public synchronized String getName() {
        return name;
    }

    public synchronized void invert() {
        red = 255 - red;
        green = 255 - green;
        blue = 255 - blue;
        name = "Inverse of " + name;
    }
}
```

`SynchronizedRGB`必须谨慎使用，以避免在不一致的状态下被看到。 例如，假设一个线程执行以下代码：

```JAVA
SynchronizedRGB color =
    new SynchronizedRGB(0, 0, 0, "Pitch Black");
...
int myColorInt = color.getRGB();      //Statement 1
String myColorName = color.getName(); //Statement 2
```

如果另一个线程在语句 1 之后但在语句 2 之前调用 color.set，则 myColorInt 的值将与 myColorName 的值不匹配。 为了避免这种结果，两个语句必须绑定在一起：

```JAVA
synchronized (color) {
    int myColorInt = color.getRGB();
    String myColorName = color.getName();
} 
```

这种不一致，只适用于可变对象——对于`SynchronizedRGB`的不可变版本不会有问题。

### 定义不可变对象的策略

以下规则定义了创建不可变对象的简单策略。并非所有记录为“不可变”的类都遵循这些规则。这并不一定意味着这些类的创建者马虎——他们可能有充分的理由相信他们的类的实例在构建后永远不会改变。但是，此类策略需要复杂的分析，不适合初学者。

1. 不要提供“setter”方法——修改字段或字段引用的对象的方法。
2. 将所有字段设为`final`和`private`。
3. 不允许子类覆盖方法。最简单的方法是将类声明为 final。 更复杂的方法是使构造函数私有并在工厂方法中构造实例。
4. 如果实例字段包含对可变对象的引用，则不允许更改这些对象：
   - 不要提供修改可变对象的方法。
   - 不要共享对可变对象的引用。 永远不要存储对传递给构造函数的外部可变对象的引用； 如有必要，创建副本并存储对副本的引用。 同样，必要时创建内部可变对象的副本，以避免在方法中返回原始对象。

将此策略应用于`SynchronizedRGB`会有以下步骤：

1. 这个类中有两个`setter`方法。 第一个`set`任意转换对象，并且在类的不可变版本中没有位置。 第二个，`invert`，可以通过让它创建一个新对象而不是修改现有对象来进行调整。
2. 所有字段都已经是`private`； 他们进一步被定义为`final`。
3. 类本身被声明为 final。
4. 只有一个字段引用一个对象，并且该对象本身是不可变的。 因此，不需要防止更改“包含”可变对象的状态的保护措施。

在这些更改之后，我们有了`ImmutableRGB`：

```JAVA
final public class ImmutableRGB {

    // Values must be between 0 and 255.
    final private int red;
    final private int green;
    final private int blue;
    final private String name;

    private void check(int red,
                       int green,
                       int blue) {
        if (red < 0 || red > 255
            || green < 0 || green > 255
            || blue < 0 || blue > 255) {
            throw new IllegalArgumentException();
        }
    }

    public ImmutableRGB(int red,
                        int green,
                        int blue,
                        String name) {
        check(red, green, blue);
        this.red = red;
        this.green = green;
        this.blue = blue;
        this.name = name;
    }


    public int getRGB() {
        return ((red << 16) | (green << 8) | blue);
    }

    public String getName() {
        return name;
    }

    public ImmutableRGB invert() {
        return new ImmutableRGB(255 - red,
                       255 - green,
                       255 - blue,
                       "Inverse of " + name);
    }
}
```

## 高级并发对象

到目前为止，本课程的重点是从一开始就成为 Java 平台一部分的低级 API。这些 API 足以满足非常基本的任务，但更高级的任务需要更高级别的构建块。 对于充分利用当今多处理器和多核系统的大规模并发应用程序尤其重要。

在本节中，我们将了解 Java 平台 5.0 版引入的一些高级并发特性。 大多数这些特性在新的`java.util.concurrent`包中实现。`Java Collections Framework`中还有新的并发数据结构。

- 锁定对象（Lock objects）支持简化许多并发应用程序的锁定习惯用法。
- `Executors`定义了一个用于启动和管理线程的高级 API。`java.util.concurrent`提供的`Executor`实现提供了适合大型应用程序的线程池管理。
- 并发集合（Concurrent collections）可以更轻松地管理大型数据集合，并且可以大大减少对同步的需求。
- 原子变量（Atomic variables）具有最小化同步并有助于避免内存一致性错误的功能。
- `ThreadLocalRandom`（JDK 7）提供了从多个线程高效生成伪随机数的功能。

### 锁定对象（Lock objects）

同步代码依赖于一种简单的可重入锁。 这种锁使用方便，但也有很多局限性。`java.util.concurrent.locks`包支持更复杂的锁定习惯用法。 我们不会详细研究这个包，而是将重点放在它最基本的接口`Lock`上。

锁对象的工作方式与同步代码使用的隐式锁非常相似。与隐式锁一样，一次只有一个线程可以拥有一个`Lock`对象。`Lock`对象还通过其关联的`Condition`对象支持等待/通知机制。

`Lock`对象相对于隐式锁的最大优势是它们能够退出获取锁的尝试。 如果锁定不立即可用或在超时到期之前（如果指定），则`tryLock`方法将退出。 如果另一个线程在获取锁之前发送中断，则`lockInterruptably`方法会退出。

让我们使用`Lock`对象来解决我们在 `Liveness` 中看到的死锁问题。Alphonse 和Gaston 已经训练自己注意朋友何时要鞠躬。我们通过要求我们的`Friend`对象在继续鞠躬之前必须为两个参与者获取锁来模拟这种改进。这是改进模型 `Safelock` 的源代码。 为了证明这个习语的多功能性，我们假设 Alphonse 和 Gaston 对他们新发现的安全鞠躬能力如此着迷，以至于他们无法停止互相鞠躬：

```java
import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;
import java.util.Random;

public class Safelock {
    static class Friend {
        private final String name;
        private final Lock lock = new ReentrantLock();

        public Friend(String name) {
            this.name = name;
        }

        public String getName() {
            return this.name;
        }

        public boolean impendingBow(Friend bower) {
            Boolean myLock = false;
            Boolean yourLock = false;
            try {
                myLock = lock.tryLock();
                yourLock = bower.lock.tryLock();
            } finally {
                if (! (myLock && yourLock)) {
                    if (myLock) {
                        lock.unlock();
                    }
                    if (yourLock) {
                        bower.lock.unlock();
                    }
                }
            }
            return myLock && yourLock;
        }
            
        public void bow(Friend bower) {
            if (impendingBow(bower)) {
                try {
                    System.out.format("%s: %s has"
                        + " bowed to me!%n", 
                        this.name, bower.getName());
                    bower.bowBack(this);
                } finally {
                    lock.unlock();
                    bower.lock.unlock();
                }
            } else {
                System.out.format("%s: %s started"
                    + " to bow to me, but saw that"
                    + " I was already bowing to"
                    + " him.%n",
                    this.name, bower.getName());
            }
        }

        public void bowBack(Friend bower) {
            System.out.format("%s: %s has" +
                " bowed back to me!%n",
                this.name, bower.getName());
        }
    }

    static class BowLoop implements Runnable {
        private Friend bower;
        private Friend bowee;

        public BowLoop(Friend bower, Friend bowee) {
            this.bower = bower;
            this.bowee = bowee;
        }
    
        public void run() {
            Random random = new Random();
            for (;;) {
                try {
                    Thread.sleep(random.nextInt(10));
                } catch (InterruptedException e) {}
                bowee.bow(bower);
            }
        }
    }
            

    public static void main(String[] args) {
        final Friend alphonse =
            new Friend("Alphonse");
        final Friend gaston =
            new Friend("Gaston");
        new Thread(new BowLoop(alphonse, gaston)).start();
        new Thread(new BowLoop(gaston, alphonse)).start();
    }
}
```

### Executors

在前面的所有示例中，由新线程完成的任务（由其`Runnable`对象定义）与线程本身（由`Thread`对象定义）之间存在密切联系。这适用于小型应用程序，但在大型应用程序中，将线程管理和创建与应用程序的其余部分分开是有意义的。封装这些函数的对象称为Executors。以下小节详细描述了执行程序。

- 执行器接口定义了三种执行器对象类型。
- 线程池是最常见的执行器实现类型。
- Fork/Join 是一个利用多个处理器的框架（JDK 7 中的新增功能）。

#### `Executor`接口

`java.util.concurrent`包定义了三个执行器接口：

- `Executor`, 支持启动新任务的简单接口。
- `ExecutorService`, `Executor`的一个子接口，它添加了有助于管理生命周期的功能，包括单个任务和 executor 本身。
- `ScheduledExecutorService`, `ExecutorService` 的子接口，支持未来某个时间执行和（或）定期执行任务。

通常，引用执行程序对象的变量被声明为这三种接口类型之一，而不是执行程序类类型。

##### `Executor` 接口

`Executor` 接口提供了一个单一的方法，`execute`，旨在替代常见的线程创建习惯用法。如果 `r` 是一个 `Runnable` 对象，而 `e` 是一个 `Executor` 对象，你可以替换

```java
(new Thread(r)).start();
```

为

```java
e.execute(r);
```

但是，`execute`的定义不太具体。低级习惯用法创建一个新线程并立即启动它。根据`Executor`的实现，`execute`可能会做同样的事情，但更有可能使用现有的工作线程来运行 `r`，或者将 `r` 放入队列以等待工作线程变为可用。（我们将在线程池一节中描述工作线程。）

`java.util.concurrent`中的执行器实现旨在充分利用更高级的 `ExecutorService` 和 `ScheduledExecutorService` 接口，尽管它们也与基本的 `Executor` 接口一起使用。

##### `ExecutorService` 接口

`ExecutorService` 接口用类似但更通用的提交方法补充了 `execute`。和 `execute` 一样，`submit` 接受 `Runnable` 对象，但也接受 `Callable` 对象，它允许任务返回一个值。`submit` 方法返回一个 `Future` 对象，该对象用于检索 `Callable` 返回值并管理 `Callable` 和 `Runnable` 任务的状态。

`ExecutorService` 还提供了用于提交大量 `Callable` 对象集合的方法。最后，`ExecutorService` 提供了许多管理执行器关闭的方法。为了支持立即关闭，任务应该正确处理中断（interrupts）。

##### `ScheduledExecutorService` 接口

`ScheduledExecutorService` 接口用 `schedule` 补充其父 `ExecutorService` 的方法，在指定的延迟后执行 `Runnable` 或 `Callable` 任务。此外，该接口还定义了 `scheduleAtFixedRate` 和 `scheduleWithFixedDelay`，它们以定义的时间间隔重复执行指定的任务。

#### 线程池（Thread Pools）

`java.util.concurrent` 中的大多数执行器实现都使用*线程池*，它由工作线程组成。这种线程与其执行的 `Runnable` 和 `Callable` 任务分开存在，通常用于执行多个任务。

使用工作线程可以最大限度地减少线程创建造成的开销。线程对象使用大量内存，在大型应用程序中，分配和取消分配许多线程对象会产生大量内存管理开销。

一种常见的线程池类型是*fixed thread pool*（固定线程池）。这种类型的池总是有指定数量的线程在运行；如果一个线程在它仍在使用时以某种方式终止，它会自动替换为一个新线程。任务通过内部队列提交到池中，只要活动任务多于线程，该队列就会保存额外的任务。

fixed thread pool的一个重要优点是使用它的应用程序可以优雅地降级。要理解这一点，请考虑一个 Web 服务器应用程序，其中每个 HTTP 请求都由一个单独的线程处理。如果应用程序只是为每个新的 HTTP 请求创建一个新线程，并且系统收到的请求数量超过了它可以立即处理的数量，那么当所有这些线程的开销超过系统的容量时，应用程序将突然停止响应所有请求。由于可以创建的线程数量受到限制，应用程序不会像 HTTP 请求进入时那样快速地为它们提供服务，但它会在系统能够承受的情况下尽快为它们提供服务。

创建使用fixed thread pool的执行器的一种简单方法是调用 `java.util.concurrent.Executors` 中的 `newFixedThreadPool` 工厂方法 该类还提供了以下工厂方法：

- `newCachedThreadPool` 方法创建一个具有可扩展线程池的执行器。此执行程序适用于启动许多短期任务的应用程序。
- `newSingleThreadExecutor` 方法创建一个执行器，一次执行一个任务。
- 几个工厂方法是上述执行器的 `ScheduledExecutorService` 版本。

如果上述工厂方法提供的执行器都不能满足您的需求，则构造 `java.util.concurrent.ThreadPoolExecutor` 或 `java.util.concurrent.ScheduledThreadPoolExecutor` 的实例将为您提供额外的选择。

#### Fork/Join

fork/join 框架是 `ExecutorService` 接口的实现，可帮助您利用多个处理器。它专为可以递归分解为更小的部分的工作而设计。目标是使用所有可用的处理能力来增强应用程序的性能。

与任何 `ExecutorService` 实现一样，fork/join 框架将任务分配给线程池中的工作线程。fork/join 框架与众不同，因为它使用了工作窃取算法。无事可做的工作线程可以从仍然忙碌的其他线程中获取任务。

fork/join 框架的中心是 `ForkJoinPool` 类，它是 `AbstractExecutorService` 类的扩展。`ForkJoinPool`实现了核心工作窃取算法，可以执行 `ForkJoinTask` 进程。

##### 基本用途

使用 fork/join 框架的第一步是编写代码来执行一部分工作。您的代码应该类似于以下伪代码：

```java
if (my portion of the work is small enough)
  do the work directly
else
  split my work into two pieces
  invoke the two pieces and wait for the results
```

将此代码包装在 `ForkJoinTask` 子类中，通常使用其更专业的类型之一，`RecursiveTask`（可以返回结果）或 `RecursiveAction`。

在 `ForkJoinTask` 子类准备好后，创建代表所有要完成的工作的对象，并将其传递给 `ForkJoinPool` 实例的 `invoke()` 方法。

##### Blurring for Clarity

为了帮助您了解 fork/join 框架的工作原理，请考虑以下示例。假设您要模糊图像。原始源图像由整数数组表示，其中每个整数包含单个像素的颜色值。模糊的目标图像也由与源大小相同的整数数组表示。

执行模糊是通过一次处理一个像素的源阵列来完成的。每个像素与其周围的像素进行平均（对红色、绿色和蓝色分量进行平均），并将结果放置在目标数组中。由于图像是一个大数组，这个过程可能需要很长时间。您可以通过使用 fork/join 框架实现算法来利用多处理器系统上的并发处理。这是一种可能的实现：

```java
public class ForkBlur extends RecursiveAction {
    private int[] mSource;
    private int mStart;
    private int mLength;
    private int[] mDestination;
  
    // Processing window size; should be odd.
    private int mBlurWidth = 15;
  
    public ForkBlur(int[] src, int start, int length, int[] dst) {
        mSource = src;
        mStart = start;
        mLength = length;
        mDestination = dst;
    }

    protected void computeDirectly() {
        int sidePixels = (mBlurWidth - 1) / 2;
        for (int index = mStart; index < mStart + mLength; index++) {
            // Calculate average.
            float rt = 0, gt = 0, bt = 0;
            for (int mi = -sidePixels; mi <= sidePixels; mi++) {
                int mindex = Math.min(Math.max(mi + index, 0),
                                    mSource.length - 1);
                int pixel = mSource[mindex];
                rt += (float)((pixel & 0x00ff0000) >> 16)
                      / mBlurWidth;
                gt += (float)((pixel & 0x0000ff00) >>  8)
                      / mBlurWidth;
                bt += (float)((pixel & 0x000000ff) >>  0)
                      / mBlurWidth;
            }
          
            // Reassemble destination pixel.
            int dpixel = (0xff000000     ) |
                   (((int)rt) << 16) |
                   (((int)gt) <<  8) |
                   (((int)bt) <<  0);
            mDestination[index] = dpixel;
        }
    }
    ……
```

现在您实现了抽象的 `compute()` 方法，该方法要么直接执行模糊处理，要么将其拆分为两个较小的任务。一个简单的数组长度阈值有助于确定工作是执行还是拆分。

```java
protected static int sThreshold = 100000;

protected void compute() {
    if (mLength < sThreshold) {
        computeDirectly();
        return;
    }
    
    int split = mLength / 2;
    
    invokeAll(new ForkBlur(mSource, mStart, split, mDestination),
              new ForkBlur(mSource, mStart + split, mLength - split,
                           mDestination));
}
```

如果前面的方法在 RecursiveAction 类的子类中，那么将任务设置为在 ForkJoinPool 中运行很简单，包括以下步骤：

1. 创建一个代表所有要完成的工作的任务。

   ```
   // source image pixels are in src
   // destination image pixels are in dst
   ForkBlur fb = new ForkBlur(src, 0, src.length, dst);
   ```

2. 创建将运行任务的 ForkJoinPool。

   ```java
   ForkJoinPool pool = new ForkJoinPool();
   ```

3. 运行任务。

   ```java
   pool.invoke(fb);
   ```

有关完整的源代码，包括一些创建目标图像文件的额外代码，请参阅 ForkBlur 示例。

```java

/*
* Copyright (c) 2010, 2013, Oracle and/or its affiliates. All rights reserved.
*
* Redistribution and use in source and binary forms, with or without
* modification, are permitted provided that the following conditions
* are met:
*
*   - Redistributions of source code must retain the above copyright
*     notice, this list of conditions and the following disclaimer.
*
*   - Redistributions in binary form must reproduce the above copyright
*     notice, this list of conditions and the following disclaimer in the
*     documentation and/or other materials provided with the distribution.
*
*   - Neither the name of Oracle or the names of its
*     contributors may be used to endorse or promote products derived
*     from this software without specific prior written permission.
*
* THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS "AS
* IS" AND ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO,
* THE IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR
* PURPOSE ARE DISCLAIMED.  IN NO EVENT SHALL THE COPYRIGHT OWNER OR
* CONTRIBUTORS BE LIABLE FOR ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL,
* EXEMPLARY, OR CONSEQUENTIAL DAMAGES (INCLUDING, BUT NOT LIMITED TO,
* PROCUREMENT OF SUBSTITUTE GOODS OR SERVICES; LOSS OF USE, DATA, OR
* PROFITS; OR BUSINESS INTERRUPTION) HOWEVER CAUSED AND ON ANY THEORY OF
* LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY, OR TORT (INCLUDING
* NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE OF THIS
* SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.
*/
 
import java.awt.image.BufferedImage;
import java.io.File;
import java.util.concurrent.ForkJoinPool;
import java.util.concurrent.RecursiveAction;
import javax.imageio.ImageIO;
 
/**
 * ForkBlur implements a simple horizontal image blur. It averages pixels in the
 * source array and writes them to a destination array. The sThreshold value
 * determines whether the blurring will be performed directly or split into two
 * tasks.
 *
 * This is not the recommended way to blur images; it is only intended to
 * illustrate the use of the Fork/Join framework.
 */
public class ForkBlur extends RecursiveAction {
 
    private int[] mSource;
    private int mStart;
    private int mLength;
    private int[] mDestination;
    private int mBlurWidth = 15; // Processing window size, should be odd.
 
    public ForkBlur(int[] src, int start, int length, int[] dst) {
        mSource = src;
        mStart = start;
        mLength = length;
        mDestination = dst;
    }
 
    // Average pixels from source, write results into destination.
    protected void computeDirectly() {
        int sidePixels = (mBlurWidth - 1) / 2;
        for (int index = mStart; index < mStart + mLength; index++) {
            // Calculate average.
            float rt = 0, gt = 0, bt = 0;
            for (int mi = -sidePixels; mi <= sidePixels; mi++) {
                int mindex = Math.min(Math.max(mi + index, 0), mSource.length - 1);
                int pixel = mSource[mindex];
                rt += (float) ((pixel & 0x00ff0000) >> 16) / mBlurWidth;
                gt += (float) ((pixel & 0x0000ff00) >> 8) / mBlurWidth;
                bt += (float) ((pixel & 0x000000ff) >> 0) / mBlurWidth;
            }
 
            // Re-assemble destination pixel.
            int dpixel = (0xff000000)
                    | (((int) rt) << 16)
                    | (((int) gt) << 8)
                    | (((int) bt) << 0);
            mDestination[index] = dpixel;
        }
    }
    protected static int sThreshold = 10000;
 
    @Override
    protected void compute() {
        if (mLength < sThreshold) {
            computeDirectly();
            return;
        }
 
        int split = mLength / 2;
 
        invokeAll(new ForkBlur(mSource, mStart, split, mDestination),
                new ForkBlur(mSource, mStart + split, mLength - split, 
                mDestination));
    }
 
    // Plumbing follows.
    public static void main(String[] args) throws Exception {
        String srcName = "red-tulips.jpg";
        File srcFile = new File(srcName);
        BufferedImage image = ImageIO.read(srcFile);
         
        System.out.println("Source image: " + srcName);
         
        BufferedImage blurredImage = blur(image);
         
        String dstName = "blurred-tulips.jpg";
        File dstFile = new File(dstName);
        ImageIO.write(blurredImage, "jpg", dstFile);
         
        System.out.println("Output image: " + dstName);
         
    }
 
    public static BufferedImage blur(BufferedImage srcImage) {
        int w = srcImage.getWidth();
        int h = srcImage.getHeight();
 
        int[] src = srcImage.getRGB(0, 0, w, h, null, 0, w);
        int[] dst = new int[src.length];
 
        System.out.println("Array size is " + src.length);
        System.out.println("Threshold is " + sThreshold);
 
        int processors = Runtime.getRuntime().availableProcessors();
        System.out.println(Integer.toString(processors) + " processor"
                + (processors != 1 ? "s are " : " is ")
                + "available");
 
        ForkBlur fb = new ForkBlur(src, 0, src.length, dst);
 
        ForkJoinPool pool = new ForkJoinPool();
 
        long startTime = System.currentTimeMillis();
        pool.invoke(fb);
        long endTime = System.currentTimeMillis();
 
        System.out.println("Image blur took " + (endTime - startTime) + 
                " milliseconds.");
 
        BufferedImage dstImage =
                new BufferedImage(w, h, BufferedImage.TYPE_INT_ARGB);
        dstImage.setRGB(0, 0, w, h, dst, 0, w);
 
        return dstImage;
    }
}
```

##### 标准实现

除了使用 fork/join 框架为要在多处理器系统上并发执行的任务实现自定义算法（例如上一节中的 `ForkBlur.java` 示例）之外，Java SE 中还有一些通常有用的功能已经使用fork/join 框架。在 Java SE 8 中引入的一个这样的实现被 `java.util.Arrays` 类用于它的 `parallelSort()` 方法。这些方法类似于 `sort()`，但通过 fork/join 框架利用并发性。在多处理器系统上运行时，大型数组的并行排序比顺序排序更快。但是，这些方法究竟如何利用 fork/join 框架超出了 Java 教程的范围。有关此信息，请参阅 Java API 文档。

fork/join 框架的另一个实现由 `java.util.streams` 包中的方法使用，该包是计划用于 Java SE 8 版本的 Project Lambda 的一部分。有关更多信息，请参阅 Lambda 表达式部分。

### 并发集合

`java.util.concurrent`包，包括对 Java Collections Framework 的许多补充。这些最容易通过提供的集合接口进行分类：

- `BlockingQueue` 定义了一个先进先出的数据结构，当你试图添加到一个完整的队列或从一个空的队列中检索时，它会阻塞或超时。
- `ConcurrentMap` 是 `java.util.Map` 的一个子接口，它定义了有用的原子操作。这些操作仅在键存在时删除或替换键值对，或者仅在键不存在时添加键值对。使这些操作原子化有助于避免同步。 `ConcurrentMap` 的标准通用实现是 `ConcurrentHashMap`，它是 `HashMap` 的并发模拟。
- `ConcurrentNavigableMap` 是 `ConcurrentMap` 的子接口，支持近似匹配。`ConcurrentNavigableMap` 的标准通用实现是 `ConcurrentSkipListMap`，它是 `TreeMap` 的并发模拟。

所有这些集合都通过定义将对象添加到集合的操作与访问或删除该对象的后续操作之间的发生前关系来帮助避免内存一致性错误。

### 原子变量（Atomic Variables）

`java.util.concurrent.atomic` 包定义了支持对单个变量进行原子操作的类。所有类都有 `get` 和 `set` 方法，它们的工作方式类似于对 `volatile` 变量的读取和写入。也就是说，一个集合与同一个变量上的任何后续 `get` 都有一个发生在之前的关系。原子 `compareAndSet` 方法也具有这些内存一致性特性，适用于整数原子变量的简单原子算术方法也是如此。

为了看看这个包是如何使用的，让我们回到我们最初用来演示线程干扰的 `Counter` 类：

```java
class Counter {
    private int c = 0;

    public void increment() {
        c++;
    }

    public void decrement() {
        c--;
    }

    public int value() {
        return c;
    }

}
```

使 `Counter` 免受线程干扰的一种方法是使其方法同步，如 `SynchronizedCounter`：

```java
class SynchronizedCounter {
    private int c = 0;

    public synchronized void increment() {
        c++;
    }

    public synchronized void decrement() {
        c--;
    }

    public synchronized int value() {
        return c;
    }

}
```

对于这个简单的类，同步是一个可以接受的解决方案。但是对于更复杂的类，我们可能希望避免不必要的同步对活跃度的影响。用 `AtomicInteger` 替换 `int` 字段允许我们在不诉诸同步的情况下防止线程干扰，就像在 `AtomicCounter` 中一样：

```java
import java.util.concurrent.atomic.AtomicInteger;

class AtomicCounter {
    private AtomicInteger c = new AtomicInteger(0);

    public void increment() {
        c.incrementAndGet();
    }

    public void decrement() {
        c.decrementAndGet();
    }

    public int value() {
        return c.get();
    }

}
```

### 并发随机数

在 JDK 7 中，`java.util.concurrent` 包括一个便利类 `ThreadLocalRandom`，用于希望使用来自多个线程或 `ForkJoinTasks` 的随机数的应用程序。

对于并发访问，使用 `ThreadLocalRandom` 而不是 `Math.random()` 会减少争用并最终提高性能。

您需要做的就是调用 `ThreadLocalRandom.current()`，然后调用其方法之一来检索随机数。 这是一个例子：

```java
int r = ThreadLocalRandom.current() .nextInt(4, 77);
```

## 进一步阅读

- Java 中的并发编程：Doug Lea 的 *Design Principles and Pattern (2nd Edition)*。 一位领先专家的综合著作，他也是 Java 平台并发框架的架构师。
- Java 并发实践作者：Brian Goetz、Tim Peierls、Joshua Bloch、Joseph Bowbeer、David Holmes 和 Doug Lea。 一本实用指南，旨在供新手使用。
- Joshua Bloch 的 Effective Java Programming Language Guide（第 2 版）。 虽然这是一个通用的编程指南，但它关于线程的章节包含并发编程的基本“最佳实践”。
- 并发：*State Models & Java Programs (2nd Edition)*，作者 Jeff Magee 和 Jeff Kramer。 通过结合建模和实际示例介绍并发编程。
- *[Java Concurrent Animated](http://sourceforge.net/projects/javaconcurrenta/):* 显示并发功能使用情况的动画。

## 问题和练习：并发

### Questions

1. 你能将 Thread 对象传递给 Executor.execute 吗？ 这样的调用有意义吗？

### Exercises

1. 编译并运行 BadThreads.java：

   ```java
   public class BadThreads {
   
       static String message;
   
       private static class CorrectorThread
           extends Thread {
   
           public void run() {
               try {
                   sleep(1000); 
               } catch (InterruptedException e) {}
               // Key statement 1:
               message = "Mares do eat oats."; 
           }
       }
   
       public static void main(String args[])
           throws InterruptedException {
   
           (new CorrectorThread()).start();
           message = "Mares do not eat oats.";
           Thread.sleep(2000);
           // Key statement 2:
           System.out.println(message);
       }
   }
   ```

   应用程序应该打印出“Mares do eat oats.”。 是否保证总是这样做？ 如果没有，为什么不呢？ 更改两次 Sleep 调用的参数会有所帮助吗？ 您如何保证对消息的所有更改都将在主线程中可见？

2. 修改 <u>Guarded Blocks</u><sup>在本文中</sup> 中的生产者-消费者示例以使用标准库类而不是 `Drop` 类。

### 参考答案

#### Answers to Questions and Exercises: Concurrency

##### Questions

1. Question:Can you pass a `Thread` object to `Executor.execute`? Would such an invocation make sense? Why or why not?

   **Answer:** `Thread` implements the `Runnable` interface, so you can pass an instance of `Thread` to `Executor.execute`. However it doesn't make sense to use `Thread` objects this way. If the object is directly instantiated from `Thread`, its `run` method doesn't do anything. You can define a subclass of `Thread` with a useful `run` method — but such a class would implement features that the executor would not use.

##### Exercises

1. Exercise:Compile and run`BadThreads.java`:

   ```java
   public class BadThreads {
   
       static String message;
   
       private static class CorrectorThread
           extends Thread {
   
           public void run() {
               try {
                   sleep(1000); 
               } catch (InterruptedException e) {}
               // Key statement 1:
               message = "Mares do eat oats."; 
           }
       }
   
       public static void main(String args[])
           throws InterruptedException {
   
           (new CorrectorThread()).start();
           message = "Mares do not eat oats.";
           Thread.sleep(2000);
           // Key statement 2:
           System.out.println(message);
       }
   }
   ```

   The application should print out "Mares do eat oats." Is it guaranteed to always do this? If not, why not? Would it help to change the parameters of the two invocations of `Sleep`? How would you guarantee that all changes to `message` will be visible to the main thread?

   **Solution:** The program will almost always print out "Mares do eat oats." However, this result is not guaranteed, because there is no happens-before relationship between "Key statement 1" and "Key statement 2". This is true even if "Key statement 1" actually executes before "Key statement 2" — remember, a happens-before relationship is about visibility, not sequence.

   There are two ways you can guarantee that all changes to `message` will be visible to the main thread:

   - In the main thread, retain a reference to the `CorrectorThread` instance. Then invoke `join` on that instance before referring to `message`
   - Encapsulate `message` in an object with synchronized methods. Never reference `message` except through those methods.

   Both of these techniques establish the necessary happens-before relationship, making changes to `message` visible.

   A third technique is to simply declare `message` as `volatile`. This guarantees that any write to `message` (as in "Key statement 1") will have a happens-before relationship with any subsequent reads of `message` (as in "Key statement 2"). But it does not guarantee that "Key statement 1" will *literally* happen before "Key statement 2". They will *probably* happen in sequence, but because of scheduling uncertainties and the unknown granularity of `sleep`, this is not guaranteed.

   Changing the arguments of the two `sleep` invocations does not help either, since this does nothing to guarantee a happens-before relationship.

2. Exercise:Modify the producer-consumer example in`Guarded Blocks`to use a standard library class instead of the`Drop`class.

   **Solution:** The `java.util.concurrent.BlockingQueue` interface defines a `get` method that blocks if the queue is empty, and a `put` methods that blocks if the queue is full. These are effectively the same operations defined by `Drop` — except that `Drop` is not a queue! However, there's another way of looking at Drop: it's a queue with a capacity of zero. Since there's no room in the queue for *any* elements, every `get` blocks until the corresponding `take` and every `take` blocks until the corresponding `get`. There is an implementation of `BlockingQueue` with precisely this behavior: `java.util.concurrent.SynchronousQueue`

   `BlockingQueue` is almost a drop-in replacement for `Drop`. The main problem in `Producer`is that with `BlockingQueue`, the `put` and `get` methods throw `InterruptedException`. This means that the existing `try` must be moved up a level:

   ```java
   import java.util.Random;
   import java.util.concurrent.BlockingQueue;
   
   public class Producer implements Runnable {
       private BlockingQueue<String> drop;
   
       public Producer(BlockingQueue<String> drop) {
           this.drop = drop;
       }
   
       public void run() {
           String importantInfo[] = {
               "Mares eat oats",
               "Does eat oats",
               "Little lambs eat ivy",
               "A kid will eat ivy too"
           };
           Random random = new Random();
   
           try {
               for (int i = 0;
                    i < importantInfo.length;
                    i++) {
                   drop.put(importantInfo[i]);
                   Thread.sleep(random.nextInt(5000));
               }
               drop.put("DONE");
           } catch (InterruptedException e) {}
       }
   }
   ```

   Similar changes are required for`Consumer`:

   ```java
   import java.util.Random;
   import java.util.concurrent.BlockingQueue;
   
   public class Consumer implements Runnable {
       private BlockingQueue<String> drop;
   
       public Consumer(BlockingQueue<String> drop) {
           this.drop = drop;
       }
   
       public void run() {
           Random random = new Random();
           try {
               for (String message = drop.take();
                    ! message.equals("DONE");
                    message = drop.take()) {
                   System.out.format("MESSAGE RECEIVED: %s%n",
                                     message);
                   Thread.sleep(random.nextInt(5000));
               }
           } catch (InterruptedException e) {}
       }
   }
   ```

   For`ProducerConsumerExample`, we simply change the declaration for the`drop`object:

   ```java
   import java.util.concurrent.BlockingQueue;
   import java.util.concurrent.SynchronousQueue;
   
   public class ProducerConsumerExample {
       public static void main(String[] args) {
           BlockingQueue<String> drop =
               new SynchronousQueue<String> ();
           (new Thread(new Producer(drop))).start();
           (new Thread(new Consumer(drop))).start();
       }
   }
   ```

   
