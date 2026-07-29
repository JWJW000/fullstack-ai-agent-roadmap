Spring、Netty、Mybatis等框架的代码中大量运用了Java多线程编程技巧。
# Java内存模型
JVM规范定义了Java内存模型来屏蔽掉各种操作系统、虚拟机实现厂商和硬件的内存访问差异，以确保Java程序在所有操作系统和平台上能够达到一致的内存访问效果。
## 工作内存和主内存
Java内存模型规定所有的变量都存储在主内存中，每个线程都有自己独立的工作内存，工作内存保存了对应该线程使用的变量的主内存副本拷贝。线程对这些变量的操作都在自己的工作内存中进行，不能直接操作主内存和其他工作内存中存储的变量或者变量副本。线程间的变量传递需要通过主内存进行，三者的关系如下图所示。
![[Pasted image 20260728113421.png|697]]

## Java内存操作协议

Java内存模型定义了8种操作来完成主内存和工作内存的变量访问，具体如下。
![[Pasted image 20260728113803.png]]

- read：把一个变量的值从主内存传输到线程的工作内存中，以便随后的load动作使用
- load：把从主内存中读取的变量值载入工作内存的变量副本中。
- use： 把工作内存中一个变量的值传递给Java虚拟机执行引擎。
- assign：把从执行引擎接收到的变量的值赋值给工作内存中的变量
- store：把工作内存中一个变量的值传送到主内存中，以便随后的write操作
- write：工作内存传递过来的变量值放入主内存中。
- lock： 把主内存的一个变量标识为某个线程独占的状态。
- unlock：把主内存中一个处于锁定状态的变量释放出来，被释放的变量才可以被其他线程锁定
## 内存模型三大特性
### 1.原子性

这个概念与事务中的原子性大概一致，表明此操作是不可分割，不可中断的，要么全部执行，要么全部不执行。 Java 内存模型直接保证的原子性操作包括 read、load、use、assign、store、write、lock、unlock 这八个。

### 2.可见性

可见性是指当一个线程修改了共享变量的值，其他线程能够立即得知这个修改。Java 内存模型 是通过在变量修改后将新值同步回主内存，在变量读取前从主内存刷新变量值这种依赖主内存作为传递媒介的方式来实现可见性的，无论是普通变量还是 volatile 变量 都是如此，普通变量与 volatile 变量 的区别是，volatile 的特殊规则保证了新值能立即同步到主内存，以及每次使用前立即从主内存刷新。因此，可以说 volatile 保证了多线程操作时变量的可见性，而普通变量则不能保证这一点。除了 volatile 外，synchronized 也提供了可见性，synchronized 的可见性是由 “对一个变量执行 unlock 操作 之前，必须先把此变量同步回主内存中（执行 store、write 操作）” 这条规则获得。

### 3.有序性

单线程环境下，程序会有序的执行，即:线程内表现为串行语义。但在多线程环境下，由于执行重排，并发执行的正确性会受到影响。在Java中使用volatile和synchronized关键字，可以保证多线程执行的有序性。volatile通过加入内存屏障指令来禁止内存的重排序。synchronized通过加锁，保证同一时刻只有一个线程来执行同步代码。

## volatile 的应用

关键字vloatile是Java提供的最轻量级的同步机制，Java内存模型对volatile专门定义了一些特俗的访问规则。当一个变量被volatile修饰后，它将具备以下两种特性。
- 线程可见性：当一个线程修改了被volatile修饰的变量后，无论是否加锁，其他线程可以立即看到最新的修改，而普通变量却做不到这一点。
- 禁止指令重排序优化：普通的变量仅仅保证在该方法的执行过程中所有依赖赋值结果的地方都能获取正确的结果，而不能保证变量赋值操作的顺序与程序代码的执行顺序一致。下面举例：
```
public class ThreadStopExample {

	private static boolean stop;

	public static void main(String[] args) throws InterruptedException {
		Thread workThread = new Thread(new Runnable() {
			public void run() {
				int i= 0;
				while (!stop) {
					i++;
					try{
						TimeUnit.SECONDS.sleep(1);
					} catch (InterruptedException e) {
						e.printStackTrace();
					}
				}
			}
		});
		workThread.start();
		TimeUnit.SECONDS.sleep(3);
		stop = true;
	}
}
```
预期程序在3秒后停止，但实际会一直运行下去，原因是虚拟机对代码进行了指令重排序和优化，优化后：
```
	if (!stop)
	While(true)
		......
```
workThread 线程 在执行重排序后的代码时，是无法发现 变量 stop 被其它线程修改的，因此无法停止运行。要解决这个问题，只要将 stop 前增加 volatile 修饰符即可。volatile 解决了如下两个问题。第一，主线程对 stop 的修改在 workThread 线程 中可见，也就是说 workThread 线程 立即看到了其他线程对于 stop 变量 的修改。第二，禁止指令重排序，防止因为重排序导致的并发访问逻辑混乱。