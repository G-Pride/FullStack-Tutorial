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

2. **通过Runnable 接口+线程池实现场景**

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
   	
   	private static ExecutorService service = Executors.newFixedThreadPool(6);
   	
   	public static void main(String[] args) throws InterruptedException, ExecutionException {
   		String[] locationName = { "广州市天河区", "广州市越秀区", "广州市黄埔区", "广州市南沙区", "广州市番禺区" };
   		long startTime =  System.currentTimeMillis();
   		for (int i = 0; i < locationName.length; i++) {
   			RunableDemo t = new RunableDemo(locationName[i]);
   			service.execute(t);
               /**
   			 * 采用sumbit提交
   			 * Future future = service.submit(t);
   			 * if(future.get()==null){
   			 * //如果Future's get返回null，任务完成
                  		System.out.println(locationName[i]+"任务完成");
               	}
   			 */
   		}
   		long endTime =  System.currentTimeMillis();
   		service.shutdown();
   		System.out.println("PhoneCallFtp countLacciPhone 取数花费了"+(endTime-startTime)+"毫秒");
   	}
   }
   ```

   运行结果

   ```
   PhoneCallFtp countLacciPhone 取数花费了543毫秒
   {广州市南沙区=3089}
   {广州市黄埔区=7541}
   {广州市越秀区=8954}
   {广州市天河区=19325}
   {广州市番禺区=17559}
   ```

3. **继承 Thread 类+线程池实现场景**

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
   	
   	private static ExecutorService service = Executors.newFixedThreadPool(6);
   	
   	public static void main(String[] args) throws InterruptedException, ExecutionException {
   		String[] locationName = { "广州市天河区", "广州市越秀区", "广州市黄埔区", "广州市南沙区", "广州市番禺区" };
   		long startTime =  System.currentTimeMillis();
   		for (int i = 0; i < locationName.length; i++) {
   			ThreadDemo t = new ThreadDemo(locationName[i]);
   			service.execute(t);
               /**
   			 * 采用sumbit提交
   			 * Future future = service.submit(t);
   			 * if(future.get()==null){
   			 * //如果Future's get返回null，任务完成
                  		System.out.println(locationName[i]+"任务完成");
               	}
   			 */
   		}
   		long endTime =  System.currentTimeMillis();
   		service.shutdown();
   		System.out.println("PhoneCallFtp countLacciPhone 取数花费了"+(endTime-startTime)+"毫秒");
   	}
   }
   ```

   运行结果

   ```
   PhoneCallFtp countLacciPhone 取数花费了537毫秒
   {广州市南沙区=3089}
   {广州市黄埔区=7541}
   {广州市番禺区=17559}
   {广州市越秀区=8954}
   {广州市天河区=19325}
   ```

4. **通过 Callable 和 Future 创建线程实现场景**

   实现类

   ```java
   public class CallableDemo  implements Callable{
   
   	private String threadName;
   	
   	public CallableDemo(String threadName) {
   		this.threadName = threadName;
   	}
   	
   	JedisCluster jedisCluster = RedisManager.getJedisCluster();
   	Map<String, Object> map = new HashMap<String, Object>();
   	
   	@Override
   	public Object call() throws Exception {
   		Map<String, String> allMap = jedisCluster.hgetAll("region-lacci-" + threadName);
   		map.put(threadName, allMap.keySet().size());
   		return map;
   	}
   
   }
   ```

   调用类

   ```java
   public class CallableTest {
   
   	private static ExecutorService service = Executors.newFixedThreadPool(6);
   
   	public static void main(String[] args) throws InterruptedException, ExecutionException {
   		
   		String[] locationName = { "广州市天河区", "广州市越秀区", "广州市黄埔区", "广州市南沙区", "广州市番禺区" };
   		List<Future<Map<String, Object>>> futureList = new ArrayList<Future<Map<String, Object>>>();
   		
   		long startTime = System.currentTimeMillis();
   		for (int i = 0; i < locationName.length; i++) {
   			CallableDemo cd = new CallableDemo(locationName[i]);
   			Future<Map<String, Object>> future = service.submit(cd);
   			futureList.add(future);
   		}
   		
   		for(Future<Map<String, Object>> future:futureList) {
   			//future没完成返回就一直循环直到isDone为true
   			while(!future.isDone()); 
   			//直到future全部isDone，才执行下面的代码
   			Map<String,Object> futureMap = future.get();
   			System.out.println("Future 完成："+future.get());
   		}
   		
   		long endTime = System.currentTimeMillis();
   		
   		System.out.println("PhoneCallFtp countLacciPhone 取数花费了" + (endTime - startTime) + "毫秒");
   		service.shutdown();
   	}
   }
   ```

   运行结果

   ```
   Future 完成：{广州市天河区=19325}
   Future 完成：{广州市越秀区=8954}
   Future 完成：{广州市黄埔区=7541}
   Future 完成：{广州市南沙区=3089}
   Future 完成：{广州市番禺区=17559}
   PhoneCallFtp countLacciPhone 取数花费了11681毫秒
   ```

## **总结：**

- Thread、Runable适合不需要返回值的情况，Callable 适合需要返回值的情况。
- 优先选择线程池提交线程，没用线程池则需要Thread.sleep来保证线程有足够的时间调用run方法。
- 不需要获取线程执行状态的直接用execute提交，需要返回线程执行状态的用submit提交。
- 不需要返回值的比需要返回值的耗时更短。