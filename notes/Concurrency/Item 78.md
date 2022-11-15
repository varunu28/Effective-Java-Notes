# Item 78: Synchronize access to shared mutable data

Proper synchronization ensures two things:
 - No thread will observe the synchronized object in an inconsistent state(Partially modified by another thread).
 - Each thread entering a synchronized block will see effects of all previous modifications on the object.

Atomic operations such as reading an integer value needs to be synchronized as the thread might not see an inconsistent state with synchronization but there is no guarantee that it will see the most updated state from previous modifications.

**Never use `Thread.stop()`**. Right way should be a flag which should decide when a thread should stop. If it is set true by an external thread, then the thread should terminate it's operations. But this requires proper use of synchronization else we will end up with `liveness failures` where the `backgroundThread` below never terminates as it is not able to see the updated value of `stopRequested` due to lack of synchronization.
```
public class StopThread {

  private static boolean stopRequested;

  public static void main(String[] args) throws InterruptedException {
    Thread backgroundThread = new Thread(() -> {
      int i = 0;
      while (!stopRequested) {
        i++;
      }
    });
    backgroundThread.start();

    TimeUnit.SECONDS.sleep(1);
    stopRequested = true;
  }
}
```

Using proper synchronization, this issue can be fixed by setting and accessing the `stopRequested` in a synchronized block.
```
public class StopThread {

  private static boolean stopRequested;

  private static synchronized boolean isStopRequested() {
    return stopRequested;
  }

  private static synchronized void requestStop() {
    stopRequested = true;
  }

  public static void main(String[] args) throws InterruptedException {
    Thread thread = new Thread(() -> {
      int i = 0;
      while (!isStopRequested()) {
        i++;
      }
    });
    thread.start();

    TimeUnit.SECONDS.sleep(1);
    requestStop();
  }
}
```
We need to ensure that we don't only synchronize the update but also the read operation. The update to `stopRequested` flag is an atomic operation, so we are using synchronization primarily for communicating changes done by one thread to another thread. We can achieve this in less-verbose way using `volatile` keyword that will ensure that the thread has memory visibility over changes done by other threads.
```
public class StopThread {

  private static volatile boolean stopRequested;

  public static void main(String[] args) throws InterruptedException {
    Thread thread = new Thread(() -> {
      int i = 0;
      while (!stopRequested) {
        i++;
      }
    });
    thread.start();

    TimeUnit.SECONDS.sleep(1);
    stopRequested = true;
  }
}
```
Be careful of using `volatile` as it doesn't guarantees atomicity but just memory visibility. So non-atomic operations such as incrementing an integer as `count++` will result in incorrect results. This is because the increment operation is not atomic but comprises of read-write-modify pattern. So a thread can end up overwriting modifications done by another thread. 

To overcome these issues for non-atomic operations, always prefer to use atomic version of primitives exposed as part of `java.util.concurrent.atomic` such as `AtomicInteger` that guarantees atomicity. 