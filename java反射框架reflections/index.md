# Java反射框架reflections


# Java反射框架reflections

## 前言

​        最近做项目的时候，需要实现一个需求**客户要求获取每个接口的具体访问信息，导出到Excel**，需要获取到swagger中注解的信息，刚开始想了两种实现方式，一种是请求swagger接口获取信息，另一种是通过反射直接获取，最后采用了反射，在网上浏览资料时发现了这个框架，比较简洁。

## 依赖

```xml
        <dependency>
            <groupId>org.reflections</groupId>
            <artifactId>reflections</artifactId>
            <version>0.9.11</version>
        </dependency>
```

## 代码实例

```java
/**
 * @Author: Mr_J
 * @Date: 2021/10/29 6:47 下午
 */
@Component
@Slf4j
public class InterfaceStatisticsInitListener implements ApplicationListener<ContextRefreshedEvent> {

    @Override
    public void onApplicationEvent(ContextRefreshedEvent event) {
        log.info("读取接口层信息开始");
        //构建 反射条件 根据需要添加
        Reflections reflections = new Reflections(
                new ConfigurationBuilder().
                        //指定路径
                        setUrls(ClasspathHelper.forPackage("com.jsh.reflections.controller"))
                        .setScanners(
                                //添加方法参数扫描
                                new MethodParameterScanner()
                                //添加属性注解扫描
                                ,new FieldAnnotationsScanner()
                                //添加子类扫描
                                ,new SubTypesScanner()
                                //添加注解扫描
                                ,new TypeAnnotationsScanner()
                                //添加方法注解扫描
                                ,new MethodAnnotationsScanner())
        );
        //根据注解获取类
        Set<Class<?>> classes = reflections.getTypesAnnotatedWith(Api.class);
        classes.forEach(clazz -> {
            //处理单个类 //反射
            Method[] methods = clazz.getMethods();
            for (Method method : methods) {
                ApiOperation annotation = method.getAnnotation(ApiOperation.class);
                //输出类上注解信息
                if (annotation == null) {
                    continue;
                }
                System.out.println(annotation.value());
                String name = method.getName();
                //输出方法名
                System.out.println(name);
            }
        });

        // 反射出子类
        Set<Class<? extends TestController>> set = reflections.getSubTypesOf( TestController.class ) ;

        // 获取带有特定注解对应的方法
        Set<Method> methods = reflections.getMethodsAnnotatedWith( ApiOperation.class ) ;
        System.out.println("m"+methods);
        // 获取带有特定注解对应的字段
        Set<Field> fields = reflections.getFieldsAnnotatedWith( Autowired.class ) ;
        System.out.println("f"+fields);
        // 获取特定参数对应的方法
        Set<Method> someMethods = reflections.getMethodsMatchParams(long.class, int.class);
        System.out.println("s"+someMethods);
        // 获取特定返回值的方法
        Set<Method> voidMethods = reflections.getMethodsReturn(void.class);
        System.out.println("v"+voidMethods);
        //获取参数带有某个注解的方法
        Set<Method> pathParamMethods =reflections.getMethodsWithAnyParamAnnotated(RequestParam.class);
        System.out.println("p"+pathParamMethods);
    }
}
```


