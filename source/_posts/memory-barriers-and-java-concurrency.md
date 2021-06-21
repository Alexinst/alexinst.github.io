---
title: 「Java」内存屏障和 Java 并发
date: 2020-04-05 16:06:46
tags:
	- Java
	- concurrency
	- memory barrier
	- visibility
categories:
	- Java
---

**简述：**本文是 InfoQ 上一篇[文章](https://www.infoq.com/articles/memory_barriers_jvm_concurrency/)的译文，文中略有不通顺之处。

<!-- more -->

<br />

内存屏障，或者说内存栅栏，是一组用于限制内存操作的执行顺序的 CPU 指令。本文将解析内存屏障对多线程程序的影响，具体到内存屏障和 JVM 并发结构（如 `volatile`、`synchronized` 和 `Atomic` 类）之间的关系。读者需要对这些概念和 Java 内存模型有扎实的理解。不过，文章并不打算阐述互斥（`mutual exclusion`），并行（`parallelism`）或者原子性（`atomicity`）。内存屏障用于实现一个很重要的并发编程基本特性，即 `可见性`。

感谢 Brian Goetz 和 Eric Yew 对文章的审阅工作，同时也感谢 Christian Thalinger 提供的 `SPARC` 框架设备。

<br />



# 1 为什么内存屏障如此重要

访问主内存（`main memory`）一次需要数以百计个时钟周期（`clock cycle`）。CPU 使用缓存（`cache`）将内存延迟（`memory latency`）的开销降低了几个数量级。为了提高性能，缓存会对挂起（`pending`）的内存操作进行重排序（`re-order`）。也就是说，程序的读/写操作不一定按照代码顺序来执行。 

当数据是不可变的（`immutable`）或被封闭在某个线程内（`confined to the scope of one thread`）时，这种优化不会改变程序结果。然而，当指令重排序遇上对称多处理（`symmetric multi-processing`）和共享可变状态（`shared mutable state`）这些情况时，可能会是一场噩梦。 对共享可变数据的内存操作进行重排序时，程序运行结果会变得不确定，比如一个线程可能以与代码顺序不一致的顺序来修改对另一个线程可见（`visible`）的值。不过，内存屏障可以通过让处理器序列化（`serialize`）挂起（`pending`）的内存操作来避免这个问题。

<br />



# 2 内存屏障作为协议

JVM 并不直接显露内存屏障。（Memory barriers are not directly exposed by the JVM. ）反之，为了保证语言级的并发原语语义，它们被 JVM 插入到指令序列中。（Instead they are inserted into the instruction sequence by the JVM in order to uphold the semantics of language level concurrency primitives. ）我们将会看一些 Java 简单源码和其汇编指令，以了解其中原理。

让我们用 `Dekker's algorithm` ，来学习内存屏障的速成课程。该算法使用三个 volatile 变量（`intentFirst`、`intentSecond`、`turn`）来协调（`coordinate`）两个线程之间对共享资源的访问。

```Java
    // 线程 1                          // 线程 2
 1  intentFirst = true;               intentSecond = true;
 2
 3  while (intentSecond) {            while (intentFirst) {     // volatile 读
 4    	if (turn != 0) {                  if (turn != 1) {      // volatile 读
 5    	    intentFirst = false;              intentSecond = false;
 6          while (turn != 0) {}              while (turn != 1) {}
 7         intentFirst = true;               intentSecond = true;
 8      }                                 }
 9  }                                 }
10  criticalSection();                criticalSection();
11
12  turn = 1;                         turn = 0;                 // volatile 写
13  intentFirst = false;              intentSecond = false;     // volatile 写
```

不要关注这个算法的细节。那么，该看些什么呢？

仔细看，每个线程都试图在第一行通过将自己的 `intent` 置为 `true` 来进入临界区（`critical section`）。如果一个线程在第三行观察（`observe`）到冲突（即两个线程的 `intent` 都为 `true`），那么将通过轮流执行（`turn taking`）来解决冲突。在给定的时间点上，只有一个线程可以访问临界区。

硬件优化（`hardware optimizations`）在没有内存屏障的情况下会使代码运行结果变得不确定，即使编译器以代码顺序来编译这些内存操作。

- 看第 3 行和第 4 行上的两个连续的 `volatile` 读操作。每个线程都会检查另一个线程是否有进入临界区的 `intent`，再检查轮到谁了。
- 看第 12 行和第 13 行上的两个连续的 `volatile` 写操作。每个线程将变量 `turn` 改为另一个线程对应的值，并且将自己进入临界区的 `intent` 置为 `false`。

读线程不应该在另一个线程将其 `intent` 置为 `false` 后才观察到那个线程对变量 `turn` 的写操作，这将是一场灾难（A reading thread should never expect to observe the other thread's write to the turn variable after the other thread's withdrawal of intent. This would be a disaster. ）。但如果没有用 `volatile` 修饰这些变量，就会发生这种情况！比如，假如没有 `volatile` ，在线程 1 对变量 `turn` 写入(线程 1 倒数第二行)之前，线程 2 可能可以观察到线程 1 对 `intentFirst` 的写入(线程 1 最后一行)。

关键字 `volatile` 可以避免这个问题，因为它在对 `turn` 变量的写入和对 `intentFirst` 变量的写入之间建立了一个 *`happens before`* 关系（The keyword volatile prevents this problem because it establishes a *happens before* relationship between the write to the turn variable and the write to the intentFirst variable. ）。编译器不能对这些写操作重排序，如果必须的话，它就会使用内存屏障来禁止处理器的重排序。

HotSpot 选项 `PrintAssembly` 是 JVM 的一个诊断标志（diagnostic flag），它能够让我们获取 JIT 编译器生成的汇编指令。这需要最新的 OpenJDK 版本（update 14 或以上）或新版 HotSpot。另外还需要一个反汇编插件。Kenai 就有适合 Solaris、Linux 和 BSD 平台的插件。hsdis 插件可以作为 Windows 平台上的替代方案。

源码中第 3 行连续的两个读操作中的第一个体现在下面的汇编指令中。环境是多核 CPU Itanium 2，JDK 1.6（Update 17）。

下面的所有汇编指令，所有指令流都是按照左侧的行号进行排序的。相关的读操作、写操作和内存屏障指令都有前缀 `*`。建议读者不要陷入对每条指令的语义思考中。

```assembly
(Itanium)
1  0x2000000001de819c:  adds r37=597, r36      ;...84112554
2* 0x2000000001de81a0:  ld1.acq r38=[r37]      ;...0b30014a a010
3  0x2000000001de81a6:  nop.m 0x0              ;...00000002 00c0
4  0x2000000001de81ac:  sxt1 r38=r38           ;...00513004
5  0x2000000001de81b0:  cmp4.eq p0, p6=0, r38  ;...1100004c 8639
6  0x2000000001de81b6:  nop.i 0x0              ;...00000002 0003
7  0x2000000001de81bc:  br.cond.dpnt.many 0x2000000001de8220;;
```

这些简短的指令说来就话长了。第一个 `volatile` 读在第 2 行。Java 内存模型保证 JVM 会在第二次 `volatile` 读之前，按**程序顺序**将第一个 `volatile` 读传递给 CPU 。但这还不够，因为 CPU 仍然可以乱序执行这些操作。为了维护 Java 内存模型的一致性，JVM 使用带参数的 `ld.acq` （`load acquire`）来注释（`annotate`）第一个 `volatile` 读操作。通过使用 `ld.acq` ，编译器可以确保第 2 行上的读操作在后续的读操作之前完成。这样，问题就解决了。

注意，这影响的是读，而不是写。这里需要介绍一下单向内存屏障和双向内存屏障。

- **单向**内存屏障：对读 **或** 写强制排序。`ld.acq` 就是一个例子。
- **双向**内存屏障：对读 **和** 写强制排序。

一致性是双向的。（Consistency is a two way street. ）**如果另一个线程没有将写操作和写操作分开（`separate`），那么读线程在两次读操作之间插入一个内存屏障有什么用呢？**为了让线程间进行通信，它们都必须遵守协议（`protocol`），就像网络中的节点，或者团队中的人。如果一个线程不遵守协议，那么其他线程的工作（`effort`）就没有意义。在 `Dekker's algorithm` 中最后两行代码（即两个 `volatile` 写操作）对应的汇编指令中，我们会看到一个内存屏障。

> $ java -XX:+UnlockDiagnosticVMOptions -XX:PrintAssemblyOptions=hsdis-print-bytes -XX:CompileCommand=print,WriterReader.write WriterReader

```assembly
(Itanium)
 1  0x2000000001de81c0: adds r37=592,r36    ;...0b284149 0421
 2  0x2000000001de81c6: st4.rel [r37]=r39   ;...00389560 2380
 3  0x2000000001de81cc: adds r36=596,r36    ;...84112544
 4* 0x2000000001de81d0: st1.rel [r36]=r0    ;...09000048 a011
 5* 0x2000000001de81d6: mf                  ;...00000044 0000
 6  0x2000000001de81dc: nop.i 0x0           ;...00040000
 7  0x2000000001de81e0: mov r12=r33         ;...00600042 0021
 8  0x2000000001de81e6: mov.ret b0=r35,0x2000000001de81e0
 9  0x2000000001de81ec: mov.i ar.pfs=r34    ;...00aa0220
10  0x2000000001de81f0: mov r6=r32          ;...09300040 0021
```

在第 4 行，我们可以看到第二个写操作用**显式**内存屏障进行了注释（`annotate`）。通过使用带参数的 `st.rel`（或 `store release`），编译器可以确保第一个写操作在第二个写操作之前是**可见的**。这就达成了协议的双向性（This completes both sides of the protocol），因为第一个写操作发生在第二个写操作之前。

指令 `st.rel` 是**单向**内存屏障，就像 `ld.acq` 一样。但是，在第 5 行，编译器还添加了一个**双向**内存屏障。指令 `mf`（`memory fence`），是 Itanium 2 指令集中的一个**完全**内存屏障（`fully fence`）。

<br />



# 3 内存屏障是硬件特性

本文不打算对所有内存屏障做全面概述，这太费时费力了。重要的是要认识到，内存屏障的指令在不同的硬件架构中有相当大的差异。下面是在多核 CPU Intel Xeon 上获取的连续 `volatile` 写操作的汇编指令。本文中余下所有汇编指令都是基于 Intel Xeon 的，除非另有说明。

```assembly
 （Intel Xeon x86)
 1  0x03f8340c: push   %ebp                     ;...55
 2  0x03f8340d: sub    $0x8, %esp               ;...81ec0800 0000
 3  0x03f83413: mov    $0x14c, %edi             ;...bf4c0100 00
 4  0x03f83418: movb   $0x1, -0x505a72f0(%edi)  ;...c687108d a5af01
 5  0x03f8341f: mfence                          ;...0faef0
 6  0x03f83422: mov    $0x148, %ebp             ;...bd480100 00
 7  0x03f83427: mov    $0x14d, %edx             ;...ba4d0100 00
 8  0x03f8342c: movsbl -0x505a72f0(%edx), %ebx  ;...0fbe9a10 8da5af
 9  0x03f83433: test   %ebx, %ebx               ;...85db
10  0x03f83435: jne    0x03f83460               ;...7529
11  0x03f83437: movl   $0x1, -0x505a72f0(%ebp)  ;...c785108d a5af01
12  0x03f83441: movb   $0x0, -0x505a72f0(%edi)  ;...c687108d a5af00
13* 0x03f83448: mfence                          ;...0faef0
14  0x03f8344b: add    $0x8,%esp                ;...83c408
15  0x03f8344e: pop    %ebp                     ;...5d
```

基于 x86 Xeon 的汇编指令第 11 行和第 12 行，我们看到 **`volatile` 写操作**。第二个写操作后有一个 `mfence` 指令（第 13 行），这是一个**显式**的**双向**内存屏障。

下面是基于 `SPARC` 的连续 `volatile` 写操作。

```assembly
(SPARC)
 1  0xfb8ecc84: ldub [ %l1 + 0x155 ], %l3    ;...e60c6155
 2  0xfb8ecc88: cmp  %l3, 0                  ;...80a4e000
 3  0xfb8ecc8c: bne, pn %icc, 0xfb8eccb0     ;...12400009
 4  0xfb8ecc90: nop                          ;...01000000
 5  0xfb8ecc94: st  %l0, [ %l1 + 0x150 ]     ;...e0246150
 6  0xfb8ecc98: clrb  [ %l1 + 0x154 ]        ;...c02c6154
 7* 0xfb8ecc9c: membar  #StoreLoad           ;...8143e002
 8  0xfb8ecca0: sethi  %hi(0xff3fc000), %l0  ;...213fcff0
 9  0xfb8ecca4: ld  [ %l0 ], %g0             ;...c0042000
10  0xfb8ecca8: ret                          ;...81c7e008
11  0xfb8eccac: restore                      ;...81e80000
```

在第 5 行和第 6 行可以看到 **`volatile` 写操作**。第二个写操作后跟随有 `membar` 指令（第 7 行），这也是一个**显式**的**双向**内存屏障。

`x86` 和 `SPARC ` 的指令流与 `Itanium` 的指令流之间有一个重要的区别。**JVM 在 `x86` 和 `SPARC` 上在连续的写操作后设置了内存屏障，但是在这两个写操作之间没有设置内存屏障。然而，`Itanium` 的指令流在两个写操作之间有一个内存屏障。**

为什么 JVM 在不同的硬件架构中表现不同？

因为每种硬件架构都有一个内存模型，每个内存模型都有自己的一套一致性保证（`consistent guarantee`）体系。比如 `x86` 或 `SPARC` 的内存模型，有非常强大的一致性保证。其他内存模型，如 `Itanium`、`PowerPC` 或 `Alpha`，则较为宽松（`relaxed`）。比如，`x86` 和 `SPARC` 不会重排序连续的写操作，所以不需要内存屏障。而 `Itanium`、`PowerPC` 和 `Alpha` 会对连续的写操作进行重排序，因此 JVM 必须在写操作之间设置一个内存屏障。

**也就是说，JVM 使用内存屏障来消除 Java 内存模型和硬件的内存模型之间的差异。**

<br />



# 4 隐式内存屏障

显式的指令 `fence` 不是序列化（`serialize`）内存操作的唯一方法。让我们来看看 `Counter` 类这个例子。

```java
class Counter{
    static int counter = 0;

    public static void main(String[] _) {
        for (int i = 0; i < 100000; i++)
            inc();
    }

    static synchronized void inc() { 
        counter += 1; 
    }
}
```

`Counter` 类中有一个经典的“读 - 修改 - 写”操作。因为这三个操作一定是原子操作（即组合起来就不是原子操作）， 所以不能用 `volatile` 修饰静态字段 `counter` ，得用 `synchronized` 修饰方法 `inc()`。我们可以使用以下命令编译 `Counter` 类并查看方法 `inc()` 生成的汇编指令。Java 内存模型为 `synchronized` 区域的退出操作（`exiting of synchronized regions`） 提供了与 `volatile` 内存操作（`volatile memory operations`）相同的可见性语义（`visibility semantics`），因此我们会看到另一种内存屏障。

> $ java -XX:+UnlockDiagnosticVMOptions -XX:PrintAssemblyOptions=hsdis-print-bytes -XX:-UseBiasedLocking -XX:CompileCommand=print,Counter.inc Counter

```assembly
(Intel Xeon)
 1  0x04d5eda7: push   %ebp               ;...55
 2  0x04d5eda8: mov    %esp,%ebp          ;...8bec
 3  0x04d5edaa: sub    $0x28,%esp         ;...83ec28
 4  0x04d5edad: mov    $0x95ba5408,%esi   ;...be0854ba 95
 5  0x04d5edb2: lea    0x10(%esp),%edi    ;...8d7c2410
 6  0x04d5edb6: mov    %esi,0x4(%edi)     ;...897704
 7  0x04d5edb9: mov    (%esi),%eax        ;...8b06
 8  0x04d5edbb: or     $0x1,%eax          ;...83c801
 9  0x04d5edbe: mov    %eax,(%edi)        ;...8907
10* 0x04d5edc0: lock cmpxchg %edi,(%esi)  ;...f00fb13e
11  0x04d5edc4: je     0x04d5edda         ;...0f841000 0000
12  0x04d5edca: sub    %esp,%eax          ;...2bc4
13  0x04d5edcc: and    $0xfffff003,%eax   ;...81e003f0 ffff
14  0x04d5edd2: mov    %eax,(%edi)        ;...8907
15  0x04d5edd4: jne    0x04d5ee11         ;...0f853700 0000
16  0x04d5edda: mov    $0x95ba52b8,%eax   ;...b8b852ba 95
17  0x04d5eddf: mov    0x148(%eax),%esi   ;...8bb04801 0000
18* 0x04d5ede5: inc    %esi               ;...46
19  0x04d5ede6: mov    %esi,0x148(%eax)   ;...89b04801 0000
20  0x04d5edec: lea    0x10(%esp),%eax    ;...8d442410
21  0x04d5edf0: mov    (%eax),%esi        ;...8b30
22  0x04d5edf2: test   %esi,%esi          ;...85f6
23  0x04d5edf4: je     0x04d5ee07         ;...0f840d00 0000
24  0x04d5edfa: mov    0x4(%eax),%edi     ;...8b7804
25* 0x04d5edfd: lock cmpxchg %esi,(%edi)  ;...f00fb137
26  0x04d5ee01: jne    0x04d5ee1f         ;...0f851800 0000
27  0x04d5ee07: mov    %ebp,%esp          ;...8be5
28  0x04d5ee09: pop    %ebp               ;...5d
```

不出意外，`synchronized` 生成的指令数量比 `volatile` 的多。多出部分可以在第 18 行找到，但 JVM 并没有插入**显式**内存屏障。相反，JVM 在第 10 行和第 25 行使用了两次带 `lock ` 前缀的 `cmpxchg` 指令。（解释 `cmpxchg` 指令的语义超出了本文的范围。）值得注意的是，`lock cmpxchg` 不仅自动执行写操作，它还会刷新挂起（`flush pending`）的读和写操作。写操作将在所有后续内存操作之前都可见。如果我们使用 `java.util.concurrent.atomic` 来重构并运行 `Counter` 类，我们可以看到同样的技巧（`trick`）。

```java
import java.util.concurrent.atomic.AtomicInteger;

class Counter{
    static AtomicInteger counter = new AtomicInteger(0);

    public static void main(String[] args){
        for(int i = 0; i < 1000000; i++)
        	counter.incrementAndGet();
    }
}
```

> $ java -XX:+UnlockDiagnosticVMOptions -XX:PrintAssemblyOptions=hsdis-print-bytes -XX:CompileCommand=print,*AtomicInteger.incrementAndGet Counter

```assembly
(Intel Xeon)
 1  0x024451f7: push   %ebp               ;...55
 2  0x024451f8: mov    %esp,%ebp          ;...8bec
 3  0x024451fa: sub    $0x38,%esp         ;...83ec38
 4  0x024451fd: jmp    0x0244520a         ;...e9080000 00
 5  0x02445202: xchg   %ax,%ax            ;...6690
 6  0x02445204: test   %eax,0xb771e100    ;...850500e1 71b7
 7  0x0244520a: mov    0x8(%ecx),%eax     ;...8b4108
 8  0x0244520d: mov    %eax,%esi          ;...8bf0
 9  0x0244520f: inc    %esi               ;...46
10  0x02445210: mov    $0x9a3f03d0,%edi   ;...bfd0033f 9a
11  0x02445215: mov    0x160(%edi),%edi   ;...8bbf6001 0000
12  0x0244521b: mov    %ecx,%edi          ;...8bf9
13  0x0244521d: add    $0x8,%edi          ;...83c708
14* 0x02445220: lock cmpxchg %esi,(%edi)  ;...f00fb137
15  0x02445224: mov    $0x1,%eax          ;...b8010000 00
16  0x02445229: je     0x02445234         ;...0f840500 0000
17  0x0244522f: mov    $0x0,%eax          ;...b8000000 00
18  0x02445234: cmp    $0x0,%eax          ;...83f800
19  0x02445237: je     0x02445204         ;...74cb
20  0x02445239: mov    %esi,%eax          ;...8bc6
21  0x0244523b: mov    %ebp,%esp          ;...8be5
22  0x0244523d: pop    %ebp               ;...5d
```

在第 14 行，我们再次看到写操作有 `lock` 前缀。这将确保在所有后续内存操作之前，变量的新值对其他线程都可见。

<br />



# 5 内存屏障可被消除

JVM 知道如何消除不必要的内存屏障。

如果硬件内存模型的一致性保证（`consistency guarantee`）强于等于 Java 内存模型的一致性保证，这种情况下就比较简单，JVM 只会插入 `no op`，而不是实际的内存屏障。例如，`x86` 和 `SPARC` 硬件的内存模型的一致性保证足够强大，在读取 `volatile` 变量时就无需设置内存屏障。

还记得在 `Itanium` 上用来分隔两个读操作的显式单向内存屏障（`ld.acq`）吗？没错，在 `x86` 上 `Dekker's algorithm` 中连续的 `volatile` 读操作的汇编指令没有内存屏障。

在 `x86` 上，对共享内存的连续读操作。（A read followed by a read of shared memory on x86.）

```assembly
(x86 Dekker)
 1  0x03f83422: mov    $0x148,%ebp             ;...bd480100 00
 2  0x03f83427: mov    $0x14d,%edx             ;...ba4d0100 00
 3* 0x03f8342c: movsbl -0x505a72f0(%edx),%ebx  ;...0fbe9a10 8da5af
 4  0x03f83433: test   %ebx,%ebx               ;...85db
 5  0x03f83435: jne    0x03f83460              ;...7529
 6  0x03f83437: movl   $0x1,-0x505a72f0(%ebp)  ;...c785108d a5af01
 7  0x03f83441: movb   $0x0,-0x505a72f0(%edi)  ;...c687108d a5af00
 8  0x03f83448: mfence                         ;...0faef0
 9  0x03f8344b: add    $0x8,%esp               ;...83c408
10  0x03f8344e: pop    %ebp                    ;...5d
11  0x03f8344f: test   %eax,0xb78ec000         ;...850500c0 8eb7
12  0x03f83455: ret                            ;...c3
13  0x03f83456: nopw   0x0(%eax,%eax,1)        ;...66660f1f 840000
14* 0x03f83460: mov    -0x505a72f0(%ebp),%ebx  ;...8b9d108d a5af
15  0x03f83466: test   %edi,0xb78ec000         ;...853d00c0 8eb7
```

`volatile` 读操作位于第 3 行和第 14 行。它们都没有配以内存屏障。换句话说，在 `x86` 上（或者在 `SPARC` 上）执行 `volatile` 读操作时，唯一的性能损失就是不能对指令重排优化，指令本身与普通读操作没有什么不同。

另外，单向内存屏障的开销自然要比双向的低。当 JVM 知道单向内存屏障已经足够时，它就不会使用双向内存屏障，本文中的第一个示例证明了这一点。我们看到 `Itanium` 上两个连续的 `volatile` 读操作中的第一个使用一个单向内存屏障（`ld.acq`）进行注释（`annotate`）。如果使用显式的双向内存屏障对读操作进行注释）进行注释（`annotate`），程序仍然是正确的，但延迟开销（`latency cost`）会增大。

<br />



# 6 动态编译

静态编译器在构建时所知道的事情，动态编译器在运行时都会知道，甚至更多。更多的信息意味着更多的优化可能。例如，让我们看看 JVM 在单处理器上运行时如何使用内存屏障。下面的指令流是 `Dekker` 算法中两个连续的 `volatile` 写操作的运行时编译结果。环境是 VMWare WorkStation 里的单处理器模式 `x86` 镜像。

```assembly
(x86) 
 1  0x017b474c: push   %ebp                    ;...55
 2  0x017b474d: sub    $0x8,%esp               ;...81ec0800 0000
 3  0x017b4753: mov    $0x14c,%edi             ;...bf4c0100 00
 4  0x017b4758: movb   $0x1,-0x507572f0(%edi)  ;...c687108d 8aaf01
 5  0x017b475f: mov    $0x148,%ebp             ;...bd480100 00
 6  0x017b4764: mov    $0x14d,%edx             ;...ba4d0100 00
 7  0x017b4769: movsbl -0x507572f0(%edx),%ebx  ;...0fbe9a10 8d8aaf
 8  0x017b4770: test   %ebx,%ebx               ;...85db
 9  0x017b4772: jne    0x017b4790              ;...751c
10* 0x017b4774: movl   $0x1,-0x507572f0(%ebp)  ;...c785108d 8aaf01
11* 0x017b477e: movb   $0x0,-0x507572f0(%edi)  ;...c687108d 8aaf00
12  0x017b4785: add    $0x8,%esp               ;...83c408
13  0x017b4788: pop    %ebp                    ;...5d
```

在单处理器系统中，JVM 为**所有**内存屏障插入 `no op`，因为内存操作已经序列化（`serialize`）了。写操作（第 10 行和第 11 行）后不会有内存屏障。JVM 对 `atomic` 类进行了类似的优化（第 14 行）。下面是在相同的 `VMWare` 镜像中， `AtomicInteger.incrementAndGet()` 的运行时编译结果。

```assembly
(x86)
 1  0x036880f7: push   %ebp               ;...55
 2  0x036880f8: mov    %esp,%ebp          ;...8bec
 3  0x036880fa: sub    $0x38,%esp         ;...83ec38
 4  0x036880fd: jmp    0x0368810a         ;...e9080000 00
 5  0x03688102: xchg   %ax,%ax            ;...6690
 6  0x03688104: test   %eax,0xb78b8100    ;...85050081 8bb7
 7  0x0368810a: mov    0x8(%ecx),%eax     ;...8b4108
 8  0x0368810d: mov    %eax,%esi          ;...8bf0
 9  0x0368810f: inc    %esi               ;...46
10  0x03688110: mov    $0x9a3f03d0,%edi   ;...bfd0033f 9a
11  0x03688115: mov    0x160(%edi),%edi   ;...8bbf6001 0000
12  0x0368811b: mov    %ecx,%edi          ;...8bf9
13  0x0368811d: add    $0x8,%edi          ;...83c708
14* 0x03688120: cmpxchg %esi,(%edi)       ;...0fb137
15  0x03688123: mov    $0x1,%eax          ;...b8010000 00
16  0x03688128: je     0x03688133         ;...0f840500 0000
17  0x0368812e: mov    $0x0,%eax          ;...b8000000 00
18  0x03688133: cmp    $0x0,%eax          ;...83f800
19  0x03688136: je     0x03688104         ;...74cc
20  0x03688138: mov    %esi,%eax          ;...8bc6
21  0x0368813a: mov    %ebp,%esp          ;...8be5
22  0x0368813c: pop    %ebp               ;...5d
```

注意第 14 行中的 `cmpxchg` 指令。前面我们看到编译器给这个指令添加了一个 `lock` 前缀。在没有 SMP（`symmetric multiprocessing`）的情况下，JVM 避免了这种开销，这是静态编译无法做到的。

<br />



# 7 收尾

内存屏障是多线程编程的必要条件。它可以分为不同类型，有显式、隐式之分，也有单向、双向之分。JVM 利用内存屏障实现跨平台的 Java 内存模型。我希望本文能够帮助有经验的 JVM 开发人员更深入地了解他们的代码的工作原理。

<br />



# 8 参考

- [Intel 64 and IA-32 Architectures Software Developer's Manuals](http://www.intel.com/products/processor/manuals/)
- [IA-64 Application Instruction Set Architecture Guide](http://www.csee.umbc.edu/help/architecture/aig.pdf)
- [Java Concurrency in Practice](http://www.amazon.com/Java-Concurrency-Practice-Brian-Goetz/dp/0321349601) by [Brian Goetz](http://www.briangoetz.com/)
- [JSR-133 Cookbook](http://g.oswego.edu/dl/jmm/cookbook.html) by Doug Lea
- Mutual exclusion with [Dekker's Algorithm](http://en.wikipedia.org/wiki/Dekker's_algorithm)
- Examining generated code with [PrintAssembly](http://wikis.sun.com/display/HotSpotInternals/PrintAssembly)
- [The Kenai Project](http://kenai.com/projects/base-hsdis/downloads) - a disassembler plugin
- [hsdis](http://hg.openjdk.java.net/jdk7/hotspot/hotspot/file/tip/src/share/tools/hsdis/) - a disassembler plugin



<br />

**以上！**

<br />