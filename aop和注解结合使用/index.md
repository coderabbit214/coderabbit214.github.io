# AOP 和 注解 结合使用


# AOP 和 注解 结合使用

> aop的切面可以定义为注解，用打印接口访问时间举例

1. 注解

   ```java
   @Target({ElementType.METHOD})
   @Retention(RetentionPolicy.RUNTIME)
   public @interface CountTime {
   }
   ```

2. aop

   ```java
   @Component
   @Aspect
   @Slf4j
   public class CountTimeAspect {
       //定义切入点（也可以直接写在下边）
       @Pointcut("@annotation(com.coderabbit.utildemo.annotation.CountTime)")
       public void countTime(){
   
       }
   
       @Around("countTime()")
       public Object doAround(ProceedingJoinPoint joinPoint) {
           Object object = null;
           try {
               long beginTime = System.currentTimeMillis();
               object = joinPoint.proceed();
               String serviceName = joinPoint.getSignature().getName();
               String className = joinPoint.getSignature().getDeclaringTypeName();
               log.info("\nController Name:{}\nService Name:{}\n响应耗时{}",className,serviceName,System.currentTimeMillis() - beginTime +"ms");
           } catch (Throwable throwable) {
               throwable.printStackTrace();
           }
           return object;
       }
   }
   ```

   

