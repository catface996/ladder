- ## 线程状态枚举值（java.lang.Thread.State）
	- ### NEW
		- ```java
		          /**
		           * Thread state for a thread which has not yet started.
		           */
		          NEW,
		  ```
	- ### RUNABLE
		- /**
		           * Thread state for a runnable thread.  A thread in the runnable
		           * state is executing in the Java virtual machine but it may
		           * be waiting for other resources from the operating system
		           * such as processor.
		           */
		          RUNNABLE,
	- ### BLOCKED
		- ```java
		          /**
		           * Thread state for a thread blocked waiting for a monitor lock.
		           * A thread in the blocked state is waiting for a monitor lock
		           * to enter a synchronized block/method or
		           * reenter a synchronized block/method after calling
		           * {@link Object#wait() Object.wait}.
		           */
		          BLOCKED,
		  ```
	- ### *WAITING
		- ```java
		          /**
		           * Thread state for a waiting thread.
		           * A thread is in the waiting state due to calling one of the
		           * following methods:
		           * <ul>
		           *   <li>{@link Object#wait() Object.wait} with no timeout</li>
		           *   <li>{@link #join() Thread.join} with no timeout</li>
		           *   <li>{@link LockSupport#park() LockSupport.park}</li>
		           * </ul>
		           *
		           * <p>A thread in the waiting state is waiting for another thread to
		           * perform a particular action.
		           *
		           * For example, a thread that has called <tt>Object.wait()</tt>
		           * on an object is waiting for another thread to call
		           * <tt>Object.notify()</tt> or <tt>Object.notifyAll()</tt> on
		           * that object. A thread that has called <tt>Thread.join()</tt>
		           * is waiting for a specified thread to terminate.
		           */
		          WAITING,
		  ```
	- ### TIMED_WAITING
		- ```java
		          /**
		           * Thread state for a waiting thread with a specified waiting time.
		           * A thread is in the timed waiting state due to calling one of
		           * the following methods with a specified positive waiting time:
		           * <ul>
		           *   <li>{@link #sleep Thread.sleep}</li>
		           *   <li>{@link Object#wait(long) Object.wait} with timeout</li>
		           *   <li>{@link #join(long) Thread.join} with timeout</li>
		           *   <li>{@link LockSupport#parkNanos LockSupport.parkNanos}</li>
		           *   <li>{@link LockSupport#parkUntil LockSupport.parkUntil}</li>
		           * </ul>
		           */
		          TIMED_WAITING,
		  ```
	- ### TERMINATED
		- ```java
		          /**
		           * Thread state for a terminated thread.
		           * The thread has completed execution.
		           */
		          TERMINATED;
		  ```
- ## 线程标记状态的字段及获取方法
	- ```
	  /* Java thread status for tools,
	   * initialized to indicate thread 'not yet started'
	   */
	  
	  private volatile int threadStatus = 0;
	  ```
	- ```
	  */**
	  ** * Returns the state of this thread.
	  ** * This method is designed for use in monitoring of the system state,
	  ** * not for synchronization control.
	  ** *
	  ** * ****@return ****this thread's state.
	  ** * ****@since ****1.5
	  ** */
	  *public State getState() {
	      // get current thread state
	      return sun.misc.VM.*toThreadState*(threadStatus);
	  }
	  ```
- ![](https://pic3.zhimg.com/80/v2-1bfe043a055ab2e0dfd0fcef42961466_1440w.webp)
-
- 参考资料
	- [深入理解Java线程状态](https://zhuanlan.zhihu.com/p/82769548)