# SpringBoot常用


# SpringBoot常用

## 配置

### 数据库

```properties
spring.datasource.url=jdbc:mysql://localhost:3306/wolf2w?characterEncoding=utf8&useSSL=false&serverTimezone=UTC&rewriteBatchedStatements=true
spring.datasource.username=root
spring.datasource.password=123456
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver
```

### mybatis日志

```properties
logging.level.com.jsh.trip.mapper=trace
```



## 统一JSON返回

```java
import lombok.Data;

import java.util.HashMap;
import java.util.Map;

//统一返回结果的类
@Data
public class R {
    private Boolean success;

    private Integer code;

    private String msg;

    private Map<String, Object> data = new HashMap<String, Object>();

    //构造方法私有化
    private R(){}

    //链式编程
    //成功静态方法
    public static R ok(){
        R r = new R();
        r.setSuccess(true);
        r.setCode(ResultCode.SUCCESS);
        r.setMsg("成功");
        return r;
    }

    //失败静态方法
    public static R error(){
        R r = new R();
        r.setSuccess(false);
        r.setCode(ResultCode.ERROR);
        r.setMsg("失败");
        return r;
    }

    public R success(Boolean success){
        this.setSuccess(success);
        return this;
    }
    public R msg(String message){
        this.setMsg(message);
        return this;
    }
    public R code(Integer code){
        this.setCode(code);
        return this;
    }
    public R data(String key, Object value){
        this.data.put(key, value);
        return this;
    }
    public R data(Map<String, Object> map) {
        this.setData(map);
        return this;
    }
}

```

### 返回code

```java
public interface ResultCode {
    public static Integer SUCCESS = 200;//成功
    public static Integer ERROR = 500;//失败
}
```

## 异常处理

### 自定义运行时异常

> 与统一异常处理相结合,编程过程中使用断言方式，如果不符合预期就抛出异常，统一异常处理后返回给前端

```java
@Setter
@Getter
@AllArgsConstructor
@NoArgsConstructor
public class UtilException extends RuntimeException{
    private String msg;
}
```

### 统一异常处理

```java
@ControllerAdvice
public class CommonExceptionAdvice {

    @ExceptionHandler(UtilException.class)
    @ResponseBody
    public R UtilExceptionhandler(UtilException e, HttpServletResponse resp){
        e.printStackTrace();
        resp.setContentType("application/json;charset=UTF-8");
        return R.error().msg(e.getMsg());
    }

    @ExceptionHandler(RuntimeException.class)
    @ResponseBody
    public R runtimeExceptionHandler(RuntimeException e, HttpServletResponse resp){
        e.printStackTrace();
        resp.setContentType("application/json;charset=UTF-8");
        return R.error();
    }

}
```

## 跨域问题

> 域名，端口号，协议一个不相同就会产生

### 单一解决

```java
@CrossOrigin
```

### 全局解决

```java
@Configuration
public class MvcConfig implements WebMvcConfigurer {
    @Bean
    public WebMvcConfigurer corsConfigurer() {
        return new WebMvcConfigurer() {
            @Override
            //重写父类提供的跨域请求处理的接口
            public void addCorsMappings(CorsRegistry registry) {
                //添加映射路径
                registry.addMapping("/**")
                        //放行哪些原始域
                        .allowedOrigins("*")
                        //是否发送Cookie信息
                        .allowCredentials(true)
                        //放行哪些原始域(请求方式)
                        .allowedMethods("GET", "POST", "PUT", "DELETE", "OPTIONS")
                        //放行哪些原始域(头部信息)
                        .allowedHeaders("*")
                        //暴露哪些头部信息（因为跨域访问默认不能获取全部头部信息）
                        .exposedHeaders("Header1", "Header2");
            }
        };
    }
}
```

## 单点登录

![截屏2021-08-16 19.51.03](https://images-jsh.oss-cn-beijing.aliyuncs.com/img/%E6%88%AA%E5%B1%8F2021-08-16%2019.51.03.png)

### cookie+redis

#### 登录

> 前端请求
>
> ```js
> $("#_j_login_form").ajaxSubmit({
>                 url:domainUrl +"/users/login",
>                 type:"POST",
>                 success:function (data) {
>                     if(data.code == 200){
>                         var map = data.data;
>                         var token = map.token;  //后续后端获取当前登录用户信息
>                         var user = map.user;  //前端页面需要显示用户信息
>                         //localStorage  客户端技术可以在浏览器窗口存储数据, 数据操作是永久
>                         //cookie 客户端技术可以在浏览器窗口存储数据, 特点有时效性
>                         //参数1:cookie的key值, 参数2: cookie的value值, 参数3: 有效时间, 单位天
>                         Cookies.set('user', JSON.stringify(user), { expires: 1/48,path:'/'});
>                         Cookies.set('token', token, { expires: 1/48,path:'/'});
>                         //document.referrer 上一个请求路径
>                         var url = document.referrer ? document.referrer : "/";
>                         if(url.indexOf("regist.html") > -1 || url.indexOf("login.html") > -1){
>                             url = "/";
>                         }
>                         window.location.href = url
>                     }else{
>                         popup(data.msg);
>                     }
>                 }
>             })
> ```

1. controller

   ```java
       @PostMapping("/login")
       public R login(String username,String password){
         //判断用户名密码
           Userinfo userinfo = userinfoService.login(username, password);
         //存储token
           String token = userinfoRedisService.setToken(userinfo);
           return R.ok().data("user",userinfo).data("token",token);
       }
   ```

2. userinfoService

   ```java
   		public Userinfo login(String username, String password) {
           QueryWrapper<Userinfo> queryWrapper = new QueryWrapper<>();
           queryWrapper.eq("phone",username);
           queryWrapper.eq("password",password);
           Userinfo userinfo = baseMapper.selectOne(queryWrapper);
           if (userinfo == null) {
             //统一异常返回处理
               throw new UtilException("用户名或密码错误");
           }
           return userinfo;
       }
   ```

3. userinfoRedisService

   ```java
       public String setToken(Userinfo userinfo) {
         //生成token
           String token = UUID.randomUUID().toString().replaceAll("-", "");
         //生成key
           String key = RedisKeys.USER_LOGIN_TOKEN.join(token);
         //用户信息作为value
           String value = JSON.toJSONString(userinfo);
         //存储到redis中
           template.opsForValue().set(key, value, RedisKeys.USER_LOGIN_TOKEN.getTime(), TimeUnit.SECONDS);
           return token;
       }
   ```
   
4. 生成key的类

   ```java
   @Getter
   public enum  RedisKeys {
   
       REGIST_VERIFY_CODE("regist_verify_code", Consts.VERIFY_CODE_TIME*60L),
       USER_LOGIN_TOKEN("user_login_token", Consts.USER_INFO_TOKEN_TIME*60L),
       STRATEGY_STATIS_VO("user_login_token", -1L);
   
       private String prefix;
       private Long time;//有效时间
   
       private RedisKeys(String prefix, Long time){
           this.prefix = prefix;
           this.time = time;
       }
   
       //拼接一个key
       public String join(String... values){
           StringBuffer sb = new StringBuffer();
           sb.append(this.prefix);
           for (String value : values) {
               sb.append(":");
               sb.append(value);
           }
           return sb.toString();
       }
   
   }
   ```

#### 资源认证(访问时判断是否登录)

> 前端进行请求时都会带上token，否则就是未登录

1. 业务处理

   编写自定义注解，如果有这个注解则表示需要登录才能访问，也就是需要token正确

   ```java
   @Target(ElementType.METHOD)
   @Retention(RetentionPolicy.RUNTIME)
   public @interface RequireLogin {
   }
   ```

2. 拦截器CheckLoginInterceptor

   ```java
   public class CheckLoginInterceptor implements HandlerInterceptor {
   
       @Resource
       private IUserinfoRedisService userinfoRedisService;
   
       @Override
       public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
           //1.刷新用户redis存储的时间 2.获取用户信息
           String token = request.getHeader("token");
           Userinfo userinfo = userinfoRedisService.getUserByToken(token);
           //动态资源（controller） 这种情况都是handler是handlerMethod的实例
           if (!(handler instanceof HandlerMethod)) {
               return true;
           }
           HandlerMethod handlerMethod = (HandlerMethod) handler;
           RequireLogin methodAnnotation = handlerMethod.getMethodAnnotation(RequireLogin.class);
           if (methodAnnotation == null) {
               return true;
           }
           if (userinfo == null) {
               throw new LoginException("请先登录!");
           }
           return true;
       }
   }
   
   ```

3. userinfoRedisService

   ```java
       public Userinfo getUserByToken(String token) {
           if (!StringUtils.hasLength(token)) {
               return null;
           }
           String key = RedisKeys.USER_LOGIN_TOKEN.join(token);
           Boolean aBoolean = template.hasKey(key);
           if (aBoolean) {
               String s = template.opsForValue().get(key);
               Userinfo userinfo = JSON.parseObject(s, Userinfo.class);
             //延时
               template.expire(key,RedisKeys.USER_LOGIN_TOKEN.getTime(), TimeUnit.SECONDS);
               return userinfo;
           }
           return null;
       }
   ```

4. 注册配置拦截器

   ```java
   @Configuration
   public class MvcConfig implements WebMvcConfigurer {
   
     //注册拦截器
       @Bean
       public CheckLoginInterceptor checkLoginInterceptor(){
           return new CheckLoginInterceptor();
       }
   
     //配置拦截器
       @Override
       public void addInterceptors(InterceptorRegistry registry) {
           registry.addInterceptor(checkLoginInterceptor())
             			//拦截
                   .addPathPatterns("/**")
             			//放行
                   .excludePathPatterns("/users/**");
       }
   
   }
   ```

5. 测试

   ```java
   @RestController
   public class TestController {
       @GetMapping("/test")
       @RequireLogin
       public R test(){
           return R.ok().msg("进入成功");
       }
   }
   ```



## 自动参数注入HandlerMethodArgumentResolver

目的举例：实现用户对controller请求时自动获取到当前登录人的信息(避免每次去请求redis获取)

1. 编写具体的ArgumentResolver 实现HandlerMethodArgumentResolver

   ```java
   public class UserInfoArgumentResolver implements HandlerMethodArgumentResolver {
   
       @Resource
       private IUserinfoRedisService userinfoRedisService;
       /**
        * 判断满足条件的参数
        * 返回值 true和false
        * 如果返回值为true则进行参数的处理
        */
       @Override
       public boolean supportsParameter(MethodParameter methodParameter) {
           return methodParameter.getParameterType() == Userinfo.class;
       }
   		/**
        * 对参数进行处理 并返回
        */
       @Override
       public Object resolveArgument(MethodParameter methodParameter, ModelAndViewContainer modelAndViewContainer, NativeWebRequest nativeWebRequest, WebDataBinderFactory webDataBinderFactory) throws Exception {
           HttpServletRequest nativeRequest = nativeWebRequest.getNativeRequest(HttpServletRequest.class);
           String token = nativeRequest.getHeader("token");
           return userinfoRedisService.getUserByToken(token);
       }
   }
   ```

2. 配置使其生效    此时就可以使用

   ```java
   @Configuration
   public class MvcConfig implements WebMvcConfigurer {
   
     ...
       
       @Bean
       public UserInfoArgumentResolver userInfoArgumentResolver(){
           return new UserInfoArgumentResolver();
       }
   
       @Override
       public void addArgumentResolvers(List<HandlerMethodArgumentResolver> resolvers) {
           resolvers.add(userInfoArgumentResolver());
       }
   
     ...
     
   }
   ```

3. 为了判断是否需要参数的自动注入 （使用注解方式判断）

   1. 新建注解

      ```java
      @Target(ElementType.METHOD)
      @Retention(RetentionPolicy.RUNTIME)
      public @interface RequireLogin {
      }
      ```

   2. 修改HandlerMethodArgumentResolver

      > 修改supportsParameter 方法  

      ```java
      public class UserInfoArgumentResolver implements HandlerMethodArgumentResolver {
      
          @Resource
          private IUserinfoRedisService userinfoRedisService;
          /**
           * 用于指定当前解析器解析的参互类型
           */
          @Override
          public boolean supportsParameter(MethodParameter methodParameter) {
              return methodParameter.getParameterType() == Userinfo.class && methodParameter.hasParameterAnnotation(UserParam.class);
          }
      
          @Override
          public Object resolveArgument(MethodParameter methodParameter, ModelAndViewContainer modelAndViewContainer, NativeWebRequest nativeWebRequest, WebDataBinderFactory webDataBinderFactory) throws Exception {
              HttpServletRequest nativeRequest = nativeWebRequest.getNativeRequest(HttpServletRequest.class);
              String token = nativeRequest.getHeader("token");
              return userinfoRedisService.getUserByToken(token);
          }
      ```

4. 使用

   

   ![WeChat9a0864173964a33dbb3188aa419b911a](https://images-jsh.oss-cn-beijing.aliyuncs.com/img/WeChat9a0864173964a33dbb3188aa419b911a.png)





## 使用springboot容器生命周期进行redis初始化 ApplicationListener

> ContextRefreshedEvent 泛型决定监听器执行的时机

```java
@Component
@Slf4j
public class RedisDataInitListener implements ApplicationListener<ContextRefreshedEvent> {

    @Resource
    private IStrategyService strategyService;

    @Resource
    private IStrategyStatisVORedisService strategyStatisVORedisService;

    @Override
    public void onApplicationEvent(ContextRefreshedEvent contextRefreshedEvent) {
        log.info("RedisStrategy初始化开始");
        List<Strategy> strategyList = strategyService.list();
        for (Strategy strategy : strategyList) {
            if (strategyStatisVORedisService.isVoExist(strategy.getId())) {
                continue;
            }
            StrategyStatisVO vo = new StrategyStatisVO();
            BeanUtils.copyProperties(strategy,vo);
            vo.setStrategyId(strategy.getId());
            strategyStatisVORedisService.setStrategyVo(vo);
        }
        log.info("RedisStrategy初始化结束");
    }
}
```



## Redis开发模式

```powershell
# 启动
redis-server

# 带配置文件启动
redis-server ./redis.conf 

# 配置后台启动，且端口是 1123
redis-server ./redis.conf --daemonize yes --port 1123
```

### 开发顺序

![e04cc49e914fec3ad91efab04464138b](https://images-jsh.oss-cn-beijing.aliyuncs.com/img/e04cc49e914fec3ad91efab04464138b.jpg)

### 目录

![截屏2021-08-26 18.53.36](https://images-jsh.oss-cn-beijing.aliyuncs.com/img/%E6%88%AA%E5%B1%8F2021-08-26%2018.53.36.png)

### 配置

1. pom

   ```xml
           <dependency>
               <groupId>org.springframework.boot</groupId>
               <artifactId>spring-boot-starter-data-redis</artifactId>
           </dependency>
   ```

2. properties

   ```properties
   spring.redis.host=127.0.0.1
   spring.redis.port=6379
   ```


### 使用

1. 预热(使用springboot生命周期监听，springboot启动完毕后执行)

   ```java
   @Component
   @Slf4j
   public class RedisDataInitListener implements ApplicationListener<ContextRefreshedEvent> {
   
       @Resource
       private IStrategyService strategyService;
   
       @Resource
       private IStrategyStatisVORedisService strategyStatisVORedisService;
   
       @Override
       public void onApplicationEvent(ContextRefreshedEvent contextRefreshedEvent) {
           log.info("RedisStrategy初始化开始");
           List<Strategy> strategyList = strategyService.list();
           for (Strategy strategy : strategyList) {
               if (strategyStatisVORedisService.isVoExist(strategy.getId())) {
                   continue;
               }
               StrategyStatisVO vo = new StrategyStatisVO();
               BeanUtils.copyProperties(strategy,vo);
               vo.setStrategyId(strategy.getId());
               strategyStatisVORedisService.setStrategyVo(vo);
           }
           log.info("RedisStrategy初始化结束");
       }
   }
   
   ```

2. 中间使用

   ```java
   		@Resource
   		private StringRedisTemplate template;
   
       @Override
       public void savePhoneCode(String phone, String code) {
           String key = RedisKeys.REGIST_VERIFY_CODE.join(phone);
           template.opsForValue().set(key, code, RedisKeys.REGIST_VERIFY_CODE.getTime(), TimeUnit.SECONDS);
       }
   
       @Override
       public String getPhoneCode(String phone) {
           String key = RedisKeys.REGIST_VERIFY_CODE.join(phone);
           return template.opsForValue().get(key);
       }
   ```

3. 落地(持久化) 采用定时方法

   ```java
   @Component
   @Slf4j
   public class RedisDataPersistenceJob {
   
       @Resource
       private IStrategyStatisVORedisService strategyStatisVORedisService;
   
       @Resource
       private IStrategyService strategyService;
   
       @Scheduled(cron = "0 0 2 * * ?")
       public void doWork() {
           //redis持久化 ， 将redis中的vo对象拿出 ， 存储到mysql
           log.info("redis----->>mysql vo对象持久化开始");
           List<StrategyStatisVO> strategyStatisVOList = strategyStatisVORedisService.queryVoByPattern("*");
   
           for (StrategyStatisVO vo : strategyStatisVOList) {
               strategyService.updateStatisVO(vo);
           }
           log.info("redis----->>mysql vo对象持久化结束");
       }
   }
   
   ```

### key的设计

```java
@Getter
public enum  RedisKeys {

    REGIST_VERIFY_CODE("regist_verify_code", Consts.VERIFY_CODE_TIME*60L),
    USER_LOGIN_TOKEN("user_login_token", Consts.USER_INFO_TOKEN_TIME*60L),
    STRATEGY_STATIS_VO("startegy_statis_vo", -1L),
    USER_STRATEGY_FAVOR("user_startegy_favor", -1L),
    STRATEGY_THUMB("startegy_thumb",-1L);

    private String prefix;
    private Long time;//有效时间

    private RedisKeys(String prefix, Long time){
        this.prefix = prefix;
        this.time = time;
    }

    //拼接一个key
    public String join(String... values){
        StringBuffer sb = new StringBuffer();
        sb.append(this.prefix);
        for (String value : values) {
            sb.append(":");
            sb.append(value);
        }
        return sb.toString();
    }

}
```

## Mongodb开发模式

进入data和log所在目录命令行运行

> --dbpath  指定为刚才创建好的data目录 
>
> --logpath  指定log存放位置 
>
> --logappend  mongo在后台运行

```
sudo mongod --dbpath data/db --logpath log/mongod.log --logappend
```

> 预热和持久化 参考redis

### 目录

![截屏2021-08-27 19.43.40](https://images-jsh.oss-cn-beijing.aliyuncs.com/img/%E6%88%AA%E5%B1%8F2021-08-27%2019.43.40.png)

### 配置

1. pom

   ```xml
           <dependency>
               <groupId>org.springframework.boot</groupId>
               <artifactId>spring-boot-starter-data-mongodb</artifactId>
           </dependency>
   ```

2. properties

   ```properties
   spring.data.mongodb.uri=mongodb://127.0.0.1:27017/wolf2w
   logging.level.org.springframework.data.mongodb.core=debug
   ```

### 使用

> 两种方式
>
> ```java
>  		//自己编写
> 		@Resource
>     private StrategyCommnetRepository strategyCommnetRepository;
> 							
> 		//系统定义
>     @Resource
>     private MongoTemplate template;
> ```

1. 定义实体类

   ```java
   //例子
   
   import org.springframework.data.annotation.Id;
   import org.springframework.data.mongodb.core.mapping.Document;
   /**
    * 攻略评论
    */
   @Setter
   @Getter
   @Document("strategy_comment")
   @ToString
   public class StrategyComment implements Serializable {
       @Id
       private String id;  //id
       private Long strategyId;  //攻略(明细)id
       private String strategyTitle; //攻略标题
       private Long userId;    //用户id
       private String nickname;  //用户名
       private String city;
       private int level;
       private String headImgUrl;     //头像
       private Date createTime;    //创建时间
       private String content;      //评论内容
       private int thumbupnum;     //点赞数
       private List<String> thumbuplist = new ArrayList<>();
   }
   ```

2. Repository

   > 声明Repository接口
   >
   > 1. 继承MongoRepository接口
   > 2. 传入泛型<实体对象类型,主键类型>

   ```java
   // 例子
   public interface StrategyCommnetRepository extends MongoRepository<StrategyComment,String> {
   
   }
   ```

3. service

   ```java
   @Service
   public class StrategyComnetServiceImpl implements IStrategyCommentService {
   
       @Resource
       private StrategyCommnetRepository strategyCommnetRepository;
   
       @Resource
       private MongoTemplate template;
   
       @Override
       public void save(StrategyComment comment) {
           comment.setId(null);
           strategyCommnetRepository.save(comment);
       }
   
       @Override
       public Page<StrategyComment> queryPage(StrategyCommentQuery qo) {
           //查询对象
           Query query = new Query();
           if (qo.getStrategyId() != null) {
               //用于设置查询条件
               query.addCriteria(Criteria.where("strategyId").is(qo.getStrategyId()));
           }
           //查询总条数
           long count = template.count(query, StrategyComment.class);
           if (count == 0) {
               return Page.empty();
           }
           //设置分页条件 页数从0开始计算
           PageRequest pageRequest = PageRequest.of(qo.getCurrentPage() - 1, qo.getPageSize());
           query.with(pageRequest);
           //查询
           List<StrategyComment> strategyComments = template.find(query, StrategyComment.class);
           //组装page
           return new PageImpl<>(strategyComments, pageRequest, count);
       }
   }
   ```

## elasticsearch开发模式

### 目录

![截屏2021-08-27 20.04.26](https://images-jsh.oss-cn-beijing.aliyuncs.com/img/%E6%88%AA%E5%B1%8F2021-08-27%2020.04.26.png)

### 配置

1. pom

   ```xml
           <dependency>
               <groupId>org.springframework.boot</groupId>
               <artifactId>spring-boot-starter-data-elasticsearch</artifactId>
           </dependency>
   ```

2. properties

   ```properties
   spring.data.elasticsearch.cluster-name=elasticsearch
   spring.data.elasticsearch.cluster-nodes=localhost:9300
   spring.data.elasticsearch.repositories.enabled=true
   ```

### 使用

#### 准备阶段

1. 实体类

   ```java
   import org.springframework.data.annotation.Id;
   import org.springframework.data.elasticsearch.annotations.Document;
   import org.springframework.data.elasticsearch.annotations.Field;
   import org.springframework.data.elasticsearch.annotations.FieldType;
   /**
    * 攻略搜索对象
    */
   @Getter
   @Setter
   @Document(indexName="wolf2w_strategy",type="strategy")
   public class StrategyEs implements Serializable {
       public static final String INDEX_NAME = "wolf2w_strategy";
       public static final String TYPE_NAME = "strategy";
       //@Field 每个文档的字段配置（类型、是否分词、是否存储、分词器 ）
       @Id
       @Field(store=true, index = false,type = FieldType.Long)
       private Long id;  //id
       @Field(index=true,analyzer="ik_max_word",store=true,searchAnalyzer="ik_max_word",type = FieldType.Text)
       private String title;  //攻略标题
       @Field(index=true,analyzer="ik_max_word",store=true,searchAnalyzer="ik_max_word",type = FieldType.Text)
       private String subTitle;  //攻略标题
       @Field(index=true,analyzer="ik_max_word",store=true,searchAnalyzer="ik_max_word",type = FieldType.Text)
       private String summary; //攻略简介
   }
   ```

2. Repository

   ```java
   public interface StrategyEsRepository extends ElasticsearchRepository<StrategyEs, Long> {
   }
   ```

3. service

   ```java
   @Service
   public class StrategyEsService implements IStrategyEsService {
   		//自定义
       @Autowired
       private StrategyEsRepository repository;
   		//系统1定义
       @Resource
       private ElasticsearchTemplate template;
   
       @Override
       public void save(StrategyEs es) {
           repository.save(es);
       }
   }
   ```

#### 预热(初始化)

```java
		//初始化服务
    @GetMapping("/dataInit")
    public String dataInit() {
        //攻略
        List<Strategy> sts = strategyService.list();
        for (Strategy st : sts) {
            StrategyEs es = new StrategyEs();
            BeanUtils.copyProperties(st, es);
            strategyEsService.save(es);
        }

        List<Travel> travels = travelService.list();
        for (Travel travel : travels) {
            TravelEs es = new TravelEs();
            BeanUtils.copyProperties(travel, es);
            travelEsService.save(es);
        }

        List<Destination> destinations = destinationService.list();
        for (Destination dest : destinations) {
            DestinationEs es = new DestinationEs();
            BeanUtils.copyProperties(dest, es);
            destinationEsService.save(es);
        }

        List<Userinfo> users = userinfoService.list();
        for (Userinfo user : users) {
            UserInfoEs es = new UserInfoEs();
            BeanUtils.copyProperties(user, es);
            userInfoEsService.save(es);
        }
        return "ok";
    }
```

#### 中间查询使用

> 用高亮检索举例

1. controller

   ```java
       @GetMapping("/q")
       public R search(@ModelAttribute("qo") SearchQueryObject qo) throws UnsupportedEncodingException {
         //qo中有两个关键值 type(查询类型，攻略，游记，城市，用户，所有)和keyword
           qo.setKeyword(URLDecoder.decode(qo.getKeyword(), "utf8"));
           switch (qo.getType()) {
               case SearchQueryObject.TYPE_DEST:
                   return this.searchDest(qo);
               case SearchQueryObject.TYPE_STRATEGY:
                   return this.searchStartegy(qo);
               case SearchQueryObject.TYPE_USER:
                   return this.searchUser(qo);
               case SearchQueryObject.TYPE_TRAVEL:
                   return this.searchTravel(qo);
               default:
                   return this.searchAll(qo);
           }
       }
   
       private R searchAll(SearchQueryObject qo) {
           SearchResultVo vo = new SearchResultVo();
           Page<Userinfo> userinfoPage = searchService.searchWithHighLight(UserInfoEs.INDEX_NAME, UserInfoEs.TYPE_NAME, Userinfo.class, qo, "info", "city");
           Page<Strategy> strategyPage  = searchService.searchWithHighLight(StrategyEs.INDEX_NAME, StrategyEs.TYPE_NAME, Strategy.class, qo, "title", "subTitle", "summary");
           Page<Travel> travelPage = searchService.searchWithHighLight(TravelEs.INDEX_NAME, TravelEs.TYPE_NAME, Travel.class, qo, "title", "summary");
           for (Travel travel : travelPage) {
               Userinfo userinfo = userinfoService.getById(travel.getAuthorId());
               travel.setAuthor(userinfo);
           }
           Page<Destination> destinationPage = searchService.searchWithHighLight(DestinationEs.INDEX_NAME, DestinationEs.TYPE_NAME, Destination.class, qo, "name", "info", "summary");
           vo.setStrategys(strategyPage.getContent());
           vo.setTravels(travelPage.getContent());
           vo.setUsers(userinfoPage.getContent());
           vo.setDests(destinationPage.getContent());
           return R.ok().data("result", vo).data("qo", qo);
       }
   
       private R searchTravel(SearchQueryObject qo) {
           Page<Travel> page = searchService.searchWithHighLight(TravelEs.INDEX_NAME, TravelEs.TYPE_NAME, Travel.class, qo, "title", "summary");
           return R.ok().data("page", page).data("qo", qo);
       }
   
       private R searchUser(SearchQueryObject qo) {
           Page<Userinfo> page = searchService.searchWithHighLight(UserInfoEs.INDEX_NAME, UserInfoEs.TYPE_NAME, Userinfo.class, qo, "info", "city");
           return R.ok().data("page", page).data("qo", qo);
       }
   
       private R searchStartegy(SearchQueryObject qo) {
           Page<Strategy> page = searchService.searchWithHighLight(StrategyEs.INDEX_NAME, StrategyEs.TYPE_NAME, Strategy.class, qo, "title", "subTitle", "summary");
           return R.ok().data("page", page).data("qo", qo);
       }
   
       private R searchDest(SearchQueryObject qo) {
           Destination destination = destinationService.queryByName(qo.getKeyword());
           SearchResultVo searchResultVo = null;
           if (destination != null) {
               List<Strategy> strategyList = strategyService.queryByDestId(destination.getId());
               List<Travel> travelList = travelService.queryByDestId(destination.getId());
               List<Userinfo> userinfoList = userinfoService.queryByCity(destination.getName());
               searchResultVo = new SearchResultVo();
               searchResultVo.setStrategys(strategyList);
               searchResultVo.setTravels(travelList);
               searchResultVo.setUsers(userinfoList);
               searchResultVo.setTotal((long) (strategyList.size() + travelList.size() + userinfoList.size() + 1));
           }
           return R.ok().data("qo", qo).data("dest", destination).data("result", searchResultVo);
       }
   ```

2. service

   ```java
   @Service
   public class SearchServiceImpl implements ISearchService {
   
       @Autowired
       private ElasticsearchTemplate template;
   
       @Autowired
       private IStrategyService strategyService;
   
       @Autowired
       private ITravelService travelService;
   
       @Autowired
       private IDestinationService destinationService;
   
       @Autowired
       private IUserinfoService userinfoService;
   
       @Override
       public <T> Page<T> searchWithHighLight(String index, String type, Class<T> clz, SearchQueryObject qo, String... fields) {
           NativeSearchQueryBuilder searchQuery = new NativeSearchQueryBuilder();
           //指定索引和类型
           searchQuery.withIndices(index).withTypes(type);
           //指定查询条件和要查询的列
           searchQuery.withQuery(QueryBuilders.multiMatchQuery(qo.getKeyword(), fields));
           //分页查询条件
           Pageable pageable = PageRequest.of(qo.getCurrentPage() - 1, qo.getPageSize());
           searchQuery.withPageable(pageable);
           //高亮检索条件
           String preTags = "<span style='color:red;'>";
           String postTags = "</span>";
           HighlightBuilder.Field[] fs = new HighlightBuilder.Field[fields.length];
           for (int i = 0; i < fs.length; i++) {
               fs[i] = new HighlightBuilder.Field(fields[i]).preTags(preTags).postTags(postTags);
           }
           searchQuery.withHighlightFields(fs);
   
           return template.queryForPage(searchQuery.build(), clz, new SearchResultMapper() {
               @SneakyThrows
               @Override
               public <T> AggregatedPage<T> mapResults(SearchResponse response, Class<T> clz, Pageable pageable) {
                   SearchHits hits = response.getHits();
                   SearchHit[] searchHits = hits.getHits();
                   List<T> list = new ArrayList<>();
                   for (SearchHit searchHit : searchHits) {
                       T t = mapSearchHit(searchHit, clz);
                       Map<String, String> map = highLightFieldsCopy(searchHit.getHighlightFields(), fields);
                       BeanUtils.copyProperties(t, map);
                       list.add(t);
                   }
                   System.out.println(list);
                   AggregatedPageImpl<T> ts = new AggregatedPageImpl<>(list, pageable, response.getHits().getTotalHits());
                   return ts;
               }
   
               private Map<String, String> highLightFieldsCopy(Map<String, HighlightField> map, String... fields) {
                   Map<String, String> mm = new HashMap<>();
                   for (String field : fields) {
                       //到highLight中找到title/subtitle/summarg
                       HighlightField hf = map.get(field);
                       if (hf != null) {
                           Text[] fragments = hf.fragments();
                           String str = "";
                           for (Text fragment : fragments) {
                               str += fragment;
                           }
                           mm.put(field, str);
                       }
                   }
                   return mm;
               }
   
               @Override
               public <T> T mapSearchHit(SearchHit searchHit, Class<T> clz) {
                   //根据具体查询的索引和类型，找到一个hit的id
                   String id = searchHit.getSourceAsMap().get("id").toString();
                   T t = null;
                   if (clz == Userinfo.class) {
                       t = (T) userinfoService.getById(Long.parseLong(id));
                   } else if (clz == Strategy.class) {
                       t = (T) strategyService.getById(Long.parseLong(id));
                   } else if (clz == Travel.class) {
                       t = (T) travelService.getById(Long.parseLong(id));
                   } else if (clz == Destination.class) {
                       t = (T) destinationService.getById(Long.parseLong(id));
                   }
                   return t;
               }
           });
       }
   }
   ```

#### 中间添加使用(elasticsearch写慢，需要使用消息组件)

![截屏2021-08-27 20.23.05](https://images-jsh.oss-cn-beijing.aliyuncs.com/img/%E6%88%AA%E5%B1%8F2021-08-27%2020.23.05.png)





## mysql优化

### 对于查询需要计算的操作

查询的数据生成一个新表，然后用定时任务进行更新(添加statis_time字段)，之后每次查询只取最新时间的

举例：![截屏2021-08-27 20.33.51](https://images-jsh.oss-cn-beijing.aliyuncs.com/img/%E6%88%AA%E5%B1%8F2021-08-27%2020.33.51.png)

1. 存储

```java
@Component
@Slf4j
public class StrategyRankStatisJob {

    @Resource
    private IStrategyService strategyService;

    @Resource
    private IStrategyRankService strategyRankService;

    @Scheduled(cron = "0 0 2 * * ?")
    public void doWork() {
        List<Strategy> hotList = queryData(null, "viewnum+replynum");
        List<Strategy> abroadList = queryData(1, "thumbsupnum+replynum");
        List<Strategy> chinaList = queryData(0, "thumbsupnum+replynum");
        Date date = new Date();
        List<StrategyRank> ranks = new ArrayList<>();
        ranks.addAll(parseData(hotList, date, StrategyRank.TYPE_HOT));
        ranks.addAll(parseData(abroadList, date, StrategyRank.TYPE_ABROAD));
        ranks.addAll(parseData(chinaList, date, StrategyRank.TYPE_CHINA));
        strategyRankService.saveBatch(ranks);

    }

    private List<Strategy> queryData(Integer isabroad, String columnSql) {
        QueryWrapper<Strategy> wrapper = new QueryWrapper<>();
        wrapper.eq(isabroad != null, "isabroad", isabroad)
                .orderByDesc(columnSql).last("limit 10");
        return strategyService.list(wrapper);
    }

    private List<StrategyRank> parseData(List<Strategy> list, Date date, Integer type) {
        List<StrategyRank> strategyRankList = new ArrayList<>();
        for (Strategy strategy : list) {
            StrategyRank strategyRank = new StrategyRank();
            strategyRank.setDestId(strategy.getDestId());
            strategyRank.setDestName(strategy.getDestName());
            strategyRank.setStrategyId(strategy.getId());
            strategyRank.setStrategyTitle(strategy.getTitle());
            strategyRank.setStatisTime(date);
            if (type.equals(StrategyRank.TYPE_HOT)) {
                strategyRank.setType(StrategyRank.TYPE_HOT);
                strategyRank.setStatisnum((long) (strategy.getViewnum() + strategy.getReplynum()));
            } else if (type.equals(StrategyRank.TYPE_ABROAD)) {
                strategyRank.setType(StrategyRank.TYPE_ABROAD);
                strategyRank.setStatisnum((long) (strategy.getThumbsupnum() + strategy.getReplynum()));
            } else if (type.equals(StrategyRank.TYPE_CHINA)) {
                strategyRank.setType(StrategyRank.TYPE_CHINA);
                strategyRank.setStatisnum((long) (strategy.getThumbsupnum() + strategy.getReplynum()));
            }
            strategyRankList.add(strategyRank);
        }

        return strategyRankList;
    }
}

```

2. 查询

```java
 @Override
    public List<StrategyRank> qyeryBytype(Integer type) {
        QueryWrapper<StrategyRank> strategyRankQueryWrapper = new QueryWrapper<>();

        strategyRankQueryWrapper.eq("type",type)
                .inSql("statis_time","select max(statis_time) from strategy_rank")
                .orderByDesc("statisnum")
                .last("limit 10");
        return baseMapper.selectList(strategyRankQueryWrapper);
    }
```


