# Java并发编程

Java 提供了三种创建线程的方法：

- 通过实现 Runnable 接口；
- 通过继承 Thread 类本身；
- 通过 Callable 和 Future 创建线程。

**实例场景：根据行政区名称获取对应的Redis基站数**

1. **没有开线程实现场景**

   ```java
   public class NoThread {
   	public static void main(String[] args) throws InterruptedException, ExecutionException {
   		
   		String[] locationName = { "广州市天河区", "广州市越秀区", "广州市黄埔区", "广州市南沙区", "广州市番禺区" };
   		Map<String, Object> map = new HashMap<String, Object>();
   		JedisCluster jedisCluster = RedisManager.getJedisCluster();
   
   		long startTime =  System.currentTimeMillis();
   		for (int i = 0; i < locationName.length; i++) {
   			Map<String, String> allMap = jedisCluster.hgetAll("region-lacci-" + locationName[i]);
   			map.put(locationName[i], allMap.keySet().size());
   		}
   		long endTime =  System.currentTimeMillis();
   		
   		System.out.println("PhoneCallFtp countLacciPhone 取数花费了"+(endTime-startTime)+"毫秒");
   		System.out.println(map);
   	}
   }
   ```

   运行结果

   ```
   PhoneCallFtp countLacciPhone 取数花费了24683毫秒
   {广州市番禺区=17559, 广州市天河区=19325, 广州市越秀区=8954, 广州市南沙区=3089, 广州市黄埔区=7541}
   ```

2. **通过Runnable 接口实现场景**

   实现类

   ```java
   public class RunableDemo implements Runnable{
   
   	private String threadName;
   	private Thread t;
   	
   	public RunableDemo(String threadName) {
   		this.threadName = threadName;
   	}
   	
   	JedisCluster jedisCluster = RedisManager.getJedisCluster();
   	Map<String, Object> map = new HashMap<String, Object>();
   	
   	public void start () {
   	      System.out.println("Starting " +  threadName );
   	      if (t == null) {
   	         t = new Thread (this, threadName);
   	         t.start ();
   	      }
   	   }
   	
   	@Override
   	public void run() {
   		Map<String, String> allMap = jedisCluster.hgetAll("region-lacci-" + threadName);
   		map.put(threadName, allMap.keySet().size());
   		System.out.println(map);
   	}
   
   }
   ```

   调用类

   ```java
   public class RunableTest{
   	
   	public static void main(String[] args) throws InterruptedException, ExecutionException {
   		String[] locationName = { "广州市天河区", "广州市越秀区", "广州市黄埔区", "广州市南沙区", "广州市番禺区" };
   		long startTime =  System.currentTimeMillis();
   		for (int i = 0; i < locationName.length; i++) {
   			RunableDemo t1 = new RunableDemo(locationName[i]);
   			t1.start();
   			Thread.sleep(50);
   		}
   		long endTime =  System.currentTimeMillis();
   		System.out.println("PhoneCallFtp countLacciPhone 取数花费了"+(endTime-startTime)+"毫秒");
   	}
   }
   ```

   运行结果

   ```
   Starting 广州市天河区
   Starting 广州市越秀区
   Starting 广州市黄埔区
   Starting 广州市南沙区
   Starting 广州市番禺区
   PhoneCallFtp countLacciPhone 取数花费了803毫秒
   {广州市南沙区=3089}
   {广州市黄埔区=7541}
   {广州市越秀区=8954}
   {广州市天河区=19325}
   {广州市番禺区=17559}
   ```

3. **继承 Thread 类实现场景**

   实现类

   ```java
   public class ThreadDemo extends Thread{
   
   	private String threadName;
   	
   	public ThreadDemo(String threadName) {
   		this.threadName = threadName;
   	}
   	
   	JedisCluster jedisCluster = RedisManager.getJedisCluster();
   	Map<String, Object> map = new HashMap<String, Object>();
   	
   	public void run() {
   		Map<String, String> allMap = jedisCluster.hgetAll("region-lacci-" + threadName);
   		map.put(threadName, allMap.keySet().size());
   		System.out.println(map);
   	}
   }
   ```

   调用类

   ```java
   public class ThreadTest {
   	public static void main(String[] args) throws InterruptedException, ExecutionException {
   		String[] locationName = { "广州市天河区", "广州市越秀区", "广州市黄埔区", "广州市南沙区", "广州市番禺区" };
   		long startTime =  System.currentTimeMillis();
   		for (int i = 0; i < locationName.length; i++) {
   			ThreadDemo t1 = new ThreadDemo(locationName[i]);
   			t1.start();
   			Thread.sleep(50);
   		}
   		long endTime =  System.currentTimeMillis();
   		System.out.println("PhoneCallFtp countLacciPhone 取数花费了"+(endTime-startTime)+"毫秒");
   	}
   }
   ```

   运行结果

   ```
   PhoneCallFtp countLacciPhone 取数花费了815毫秒
   {广州市越秀区=8954}
   {广州市黄埔区=7541}
   {广州市南沙区=3089}
   {广州市天河区=19325}
   {广州市番禺区=17559}
   ```

4. **通过 Callable 和 Future 创建线程实现场景**

