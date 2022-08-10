# springboot自定义异步线程池


# springboot自定义异步线程池

## 自定义线程池

```java
@Configuration
@EnableAsync
public class SyncConfiguration{
    @Bean(name = "asyncPoolTaskExecutor")
    public ThreadPoolTaskExecutor executor() {
        ThreadPoolTaskExecutor taskExecutor = new ThreadPoolTaskExecutor();
        //核心线程数
        taskExecutor.setCorePoolSize(1);
        //线程池维护线程的最大数量,只有在缓冲队列满了之后才会申请超过核心线程数的线程
        taskExecutor.setMaxPoolSize(2);
        //缓存队列
        taskExecutor.setQueueCapacity(2);
        //许的空闲时间,当超过了核心线程出之外的线程在空闲时间到达之后会被销毁
        taskExecutor.setKeepAliveSeconds(200);
        //异步方法内部线程名称
        taskExecutor.setThreadNamePrefix("async-");
        /**
         * 当线程池的任务缓存队列已满并且线程池中的线程数目达到maximumPoolSize，如果还有任务到来就会采取任务拒绝策略
         * 通常有以下四种策略：
         * ThreadPoolExecutor.AbortPolicy:丢弃任务并抛出RejectedExecutionException异常。
         * ThreadPoolExecutor.DiscardPolicy：也是丢弃任务，但是不抛出异常。
         * ThreadPoolExecutor.DiscardOldestPolicy：丢弃队列最前面的任务，然后重新尝试执行任务（重复此过程）
         * ThreadPoolExecutor.CallerRunsPolicy：直接主线程执行
         */
        taskExecutor.setRejectedExecutionHandler(new ThreadPoolExecutor.AbortPolicy());
        taskExecutor.initialize();
        return taskExecutor;
    }
}
```

## 使用

> 在注解中添加bean名称

```java
@Async("asyncPoolTaskExecutor")
public void doTask1() throws InterruptedException {
    long t1 = System.currentTimeMillis();
    Thread.sleep(2000);
    long t2 = System.currentTimeMillis();
    log.info("task1 cost {} ms" , t2-t1);
}
```

## 设置为Async默认线程池

> 继承AsyncConfigurer，重写getAsyncExecutor方法

```java
@Configuration
@EnableAsync
public class SyncConfiguration implements AsyncConfigurer{
    private final Logger log = LoggerFactory.getLogger(SyncConfiguration.class);

    @Bean(name = "asyncPoolTaskExecutor")
    public ThreadPoolTaskExecutor executor() {
        ThreadPoolTaskExecutor taskExecutor = new ThreadPoolTaskExecutor();
        //核心线程数
        taskExecutor.setCorePoolSize(1);
        //线程池维护线程的最大数量,只有在缓冲队列满了之后才会申请超过核心线程数的线程
        taskExecutor.setMaxPoolSize(2);
        //缓存队列
        taskExecutor.setQueueCapacity(2);
        //许的空闲时间,当超过了核心线程出之外的线程在空闲时间到达之后会被销毁
        taskExecutor.setKeepAliveSeconds(200);
        //异步方法内部线程名称
        taskExecutor.setThreadNamePrefix("async-");
        /**
         * 当线程池的任务缓存队列已满并且线程池中的线程数目达到maximumPoolSize，如果还有任务到来就会采取任务拒绝策略
         * 通常有以下四种策略：
         * ThreadPoolExecutor.AbortPolicy:丢弃任务并抛出RejectedExecutionException异常。
         * ThreadPoolExecutor.DiscardPolicy：也是丢弃任务，但是不抛出异常。
         * ThreadPoolExecutor.DiscardOldestPolicy：丢弃队列最前面的任务，然后重新尝试执行任务（重复此过程）
         * ThreadPoolExecutor.CallerRunsPolicy：直接主线程执行
         */
        taskExecutor.setRejectedExecutionHandler(new ThreadPoolExecutor.AbortPolicy());
        taskExecutor.initialize();
        return taskExecutor;
    }


    /**
     * 指定默认线程池
     */
    @Override
    public Executor getAsyncExecutor() {
        return executor();
    }

    @Override
    public AsyncUncaughtExceptionHandler getAsyncUncaughtExceptionHandler() {
        return (ex, method, params) ->
                log.error("线程池执行任务发送未知错误,执行方法：{}",method.getName(),ex);
    }
}
```



## 备注

- 如果使用AbortPolicy处理策略，可以增加异常处理器，捕获RejectedExecutionException异常处理

