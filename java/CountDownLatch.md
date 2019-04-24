# CountDownLatch
## 介绍
1. CountDownLatch是一个计数器，可以允许一个或者多个线程等待其他线程操作完成之后再操作。
2. 当前线程调用其中`await()`进行等待其他线程执行完毕；『其他线程』的数量是在CountDownLatch初始化的时候传入的`count`。当『其他线程』执行完之后，调用`countDown()`方法，`count`数值变减1。直到`count=0`，则当前线程便可以继续执行。
3. CountDownLatch由于当count变为0之后无法重置，所以CountDownLatch是一次性的。也就是说当“门闩”打开(即count=0)之后，无法再次生效。如果想重置count，请使`CyclicBarrier`。
## 示例
### 示例代码
```java
import java.util.concurrent.CountDownLatch;

class Worker implements Runnable {

  private CountDownLatch startSignal;
  private CountDownLatch doneSignal;

  public Worker(CountDownLatch startSignal, CountDownLatch doneSignal) {
    this.startSignal = startSignal;
    this.doneSignal = doneSignal;
  }

  @Override
  public void run() {
    try {
      startSignal.await(); // a
      System.out.println(Thread.currentThread().getName() + "开始工作.");
      System.out.println(Thread.currentThread().getName() + "工作完成.");
    } catch (InterruptedException e) {
      e.printStackTrace();
    } finally {
      doneSignal.countDown(); // b
    }
  }
}

public class Driver {

  public static void main(String[] args) throws InterruptedException {
    // 标记任务开始的latch
    CountDownLatch startSignal = new CountDownLatch(1);

    // 等待线程数量
    int count = 5;
    // 等待任务全部结束的latch
    CountDownLatch doneSignal = new CountDownLatch(count);
    for (int i = 0; i < count; i++) {
      new Thread(new Worker(startSignal, doneSignal)).start();
    }
    
    System.out.println("等待所有worker就绪");
    Thread.sleep(5000);
    // 在此之前的所有子线程等待，未执行
    startSignal.countDown(); // c
    System.out.println("所有worker准备开始工作");
    System.out.println("等待所有worker完成工作");
    doneSignal.await(); // d
    System.out.println("所有worker工作已完成");
  }
}
```
### 运行结果
等待所有worker就绪  
所有worker准备开始工作  
等待所有worker完成工作  
Thread-0开始工作.  
Thread-1开始工作.  
Thread-1工作完成.  
Thread-2开始工作.  
Thread-0工作完成.  
Thread-4开始工作.  
Thread-3开始工作.  
Thread-3工作完成.  
Thread-4工作完成.  
Thread-2工作完成.  
所有worker工作已完成  
### 结果分析
1. 子线程启动之后执行`run()`方法，但是由于`startSignal.await()`(**注释//a**)的存在，所以子线程需要等待。等待其他线程调用`countDown()`。所以在代码中**注释//c**之前，所有的子线程处于等待状态并没有执行。
2. 主线程执行到**c**，调用`countDown()`之后，startSignal的count变为0，所以子线程开始工作。并且工作完成之后会调用`doneSignal.countDown()`。
3. 主线程调用了`doneSignal.await()`，说明主线程需要等其他5个线程执行完毕。再能继续执行。
4. 所有子线程执行完毕之后，doneSignal的count变为0，所以主线程开始执行。
