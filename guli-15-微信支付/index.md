# guli-15-微信支付


# guli-15-微信支付

## 一.后端

### 1.准备

1. 创建service_order模块

2. 配置

   ```properties
   # 服务端口
   server.port=8087
   # 服务名
   spring.application.name=service-order
   #数据库配置
   spring.datasource.url=jdbc:mysql://localhost:3306/guli?serverTimezone=GMT%2B8
   spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver
   spring.datasource.username=root
   spring.datasource.password=123456
   #返回json的全局时间格式
   spring.jackson.date-format=yyyy-MM-dd HH:mm:ss
   spring.jackson.time-zone=GMT+8
   #配置mapper xml文件的路径
   mybatis-plus.mapper-locations=classpath:com/guigu/eduorder/mapper/xml/*.xml
   #mybatis日志
   mybatis-plus.configuration.log-impl=org.apache.ibatis.logging.stdout.StdOutImpl
   # nacos服务地址
   spring.cloud.nacos.discovery.server-addr=127.0.0.1:8848
   ##开启熔断机制
   #feign.hystrix.enabled=true
   ## 设置hystrix超时时间，默认1000ms
   #hystrix.command.default.execution.isolation.thread.timeoutInMilliseconds=3000
   ```

3. 依赖

   ```xml
   <dependencies>
   <!-- 微信支付依赖 -->
       <dependency>
           <groupId>com.github.wxpay</groupId>
           <artifactId>wxpay-sdk</artifactId>
           <version>0.0.3</version>
       </dependency>
       <dependency>
           <groupId>com.alibaba</groupId>
           <artifactId>fastjson</artifactId>
       </dependency>
   </dependencies>
   ```

4. 启动类

   ```java
   package com.guli.eduorder;
   
   @SpringBootApplication
   @ComponentScan(basePackages = "com.guli")
   @MapperScan("com.guli.eduorder.mapper")
   @EnableDiscoveryClient
   @EnableFeignClients
   public class OrdersApplication {
       public static void main(String[] args) {
           SpringApplication.run(OrdersApplication.class,args);
       }
   }
   
   ```

5. 数据库创建表

   ```
   t_order
   t_pay_log
   ```

6. 使用代码生成器创建基本包结构

### 2.接口开发

> 创建订单需要课程信息和用户信息
>
> 需要使用Nacos进行服务间调用
>
> 在公共模块中创建CourseWebVoOrder和UcenterMemberOrder进行数据封装

1. edu/CourseFrontController

   ```java
   //根据课程id查询课程信息
       @PostMapping("getCourseInfoOrder/{id}")
       public CourseWebVoOrder getCourseInfoOrder(@PathVariable String id) {
           CourseWebVo baseCourseInfo = courseService.getBaseCourseInfo(id);
           CourseWebVoOrder courseWebVoOrder = new CourseWebVoOrder();
           BeanUtils.copyProperties(baseCourseInfo,courseWebVoOrder);
           return courseWebVoOrder;
       }
   ```

2. ucenter/UcenterMemberController

   ```java
   //根据用户id获取用户信息
       @PostMapping("getUserInfoOrder/{id}")
       public UcenterMemberOrder getUserInfoOrder(@PathVariable String id) {
           UcenterMember member = memberService.getById(id);
           UcenterMemberOrder memberOrder = new UcenterMemberOrder();
           BeanUtils.copyProperties(member,memberOrder);
           return memberOrder;
       }
   ```

3. 在order模块中注册

   ```java
   package com.guli.eduorder.client;
   
   @Component
   @FeignClient("service-edu")
   public interface EduClient {
       @PostMapping("/eduservice/coursefront/getCourseInfoOrder/{id}")
       CourseWebVoOrder getCourseInfoOrder(@PathVariable("id") String id);
   }
   
   @Component
   @FeignClient("service-ucenter")
   public interface UcenterClient {
       //根据用户id获取用户信息
       @PostMapping("/educenter/member/getUserInfoOrder/{id}")
       UcenterMemberOrder getUserInfoOrder(@PathVariable("id") String id);
   }
   
   ```

4. controller

   - OrderController

   ```java
   package com.guli.eduorder.controller;
   
   @RestController
   @RequestMapping("/eduorder/order")
   @CrossOrigin
   public class OrderController {
       @Autowired
       private OrderService orderService;
   
       //1.生成订单的方法
       @PostMapping("createOrder/{courseId}")
       public R createOrder(@PathVariable String courseId, HttpServletRequest request) {
           //通过前端的请求头信息获取用户id
           String memberIdByJwtToken = JwtUtils.getMemberIdByJwtToken(request);
           //根据用户id和课程id生成订单，返回订单号
           String orderNo = orderService.createOrders(courseId,memberIdByJwtToken);
           return R.ok().data("orderId",orderNo);
       }
   
       //2.根据订单编号查询订单信息
       @GetMapping("getOrderInfo/{orderNo}")
       public R getOrderInfo(@PathVariable String orderNo) {
           QueryWrapper<Order> queryWrapper = new QueryWrapper<>();
           queryWrapper.eq("order_no",orderNo);
           Order order = orderService.getOne(queryWrapper);
           return R.ok().data("item",order);
       }
   }
   
   
   ```

   - PayLogController

   ```java
   package com.guli.eduorder.controller;
   @RestController
   @RequestMapping("/eduorder/paylog")
   @CrossOrigin
   public class PayLogController {
       @Autowired
       private PayLogService payLogService;
   
       //生成微信支付二维码接口
       //参数是订单号
       @GetMapping("createNative/{orderNo}")
       public R createNative(@PathVariable String orderNo) {
           Map map = payLogService.createNatvie(orderNo);
           return R.ok().data("map",map);
       }
   
       //查询订单支付状态
       @GetMapping("queryPayStatus/{orderNo}")
       public R queryPayStatus(@PathVariable String orderNo) {
           Map<String,String> map = payLogService.queryPayStatus(orderNo);
           if (map == null) {
               return R.error().message("支付出错");
           }
           //如果不为空，获取订单状态
           if (map.get("trade_state").equals("SUCCESS")) {//支付成功
               //向支付表中加记录，更新订单表订单状态
               payLogService.updateOrdersStatus(map);
               return R.ok().message("支付成功");
           }
           return R.error().code(25000).message("支付中");
       }
   }
   
   
   ```

5. service

   - OrderServiceImpl

   ```java
   package com.guli.eduorder.service.impl;
   
   @Service
   public class OrderServiceImpl extends ServiceImpl<OrderMapper, Order> implements OrderService {
   
       @Autowired
       private EduClient eduClient;
       @Autowired
       private UcenterClient ucenterClient;
       //1.生成订单的方法
       @Override
       public String createOrders(String courseId, String memberIdByJwtToken) {
           //通过远程调用根据课程id获取课程信息
           CourseWebVoOrder courseInfoOrder = eduClient.getCourseInfoOrder(courseId);
           //通过远程调用根据用户id获取用户信息
           UcenterMemberOrder userInfoOrder = ucenterClient.getUserInfoOrder(memberIdByJwtToken);
           //创建order对象，设置数据
           Order order = new Order();
           order.setOrderNo(OrderNoUtil.getOrderNo());//订单号
           order.setCourseId(courseId);//课程id
           order.setCourseTitle(courseInfoOrder.getTitle());//课程标题
           order.setCourseCover(courseInfoOrder.getCover());//课程封面
           order.setTeacherName(courseInfoOrder.getTeacherName());//讲师名称
           order.setTotalFee(courseInfoOrder.getPrice());//课程价格
           order.setMemberId(memberIdByJwtToken);//用户id
           order.setMobile(userInfoOrder.getMobile());//用户昵称
           order.setNickname(userInfoOrder.getNickname());//手机号
           order.setStatus(0);//支付状态，0：未支付，1：已支付
           order.setPayType(1);//支付类型
           //存储
           baseMapper.insert(order);
           return order.getOrderNo();
       }
   }
   
   ```

   - PayLogServiceImpl

   ```java
   package com.guli.eduorder.service.impl;
   
   @Service
   public class PayLogServiceImpl extends ServiceImpl<PayLogMapper, PayLog> implements PayLogService {
   
       @Autowired
       private OrderService orderService;
       //生成二维码
       @Override
       public Map<String, Object> createNatvie(String orderNo) {
           try {
               //1.根据订单号查询订单信息
               QueryWrapper<Order> queryWrapper = new QueryWrapper<>();
               queryWrapper.eq("order_no",orderNo);
               Order order = orderService.getOne(queryWrapper);
   
               //2.使用map设置生成二维码需要参数
               Map m = new HashMap();
               m.put("mch_id", "1558950191");
               m.put("appid", "wx74862e0dfcf69954");
               m.put("nonce_str", WXPayUtil.generateNonceStr());//二维码的编号使用wx的规则
               m.put("body", order.getCourseTitle());//生成二维码中显示的名字
               m.put("out_trade_no", orderNo);//二维码唯一标识
               m.put("total_fee", order.getTotalFee().multiply(new
                       BigDecimal("100")).longValue()+"");//订单的价格
               m.put("spbill_create_ip", "127.0.0.1");//进行支付的域名
               m.put("notify_url",
                       "http://guli.shop/api/order/weixinPay/weixinNotify\n");//回调地址
               m.put("trade_type", "NATIVE");//支付的类型
               //3.发送httpclient请求，传递参数xml格式，微信支付提供的固定地址
               HttpClient client = new HttpClient("https://api.mch.weixin.qq.com/pay/unifiedorder");
               //设置xml格式的参数
               //client设置参数
               client.setXmlParam(WXPayUtil.generateSignedXml(m,
                       "T6m9iK73b0kn9g5v426MKfHQH7X8rKwb"));
               client.setHttps(true);
               //发送请求
               client.post();
   
               //4.得发送请求返回结果
               //使用xml格式返回
               String content = client.getContent();
               //把xml格式转换为map格式
               Map<String,String> resultMap = WXPayUtil.xmlToMap(content);
   
               //最终返回数据封装
               Map map = new HashMap<>();
               map.put("out_trade_no", orderNo);//订单号
               map.put("course_id", order.getCourseId());//课程id
               map.put("total_fee", order.getTotalFee());//价格
               map.put("result_code", resultMap.get("result_code"));//返回二维码操作的状态码
               map.put("code_url", resultMap.get("code_url")); //二维码地址
               return map;
   
           }catch (Exception e) {
               throw new GuliException(20001,"生成二维码失败");
           }
       }
       //根据订单号向腾讯查询订单支付状态
       @Override
       public Map<String, String> queryPayStatus(String orderNo) {
           try {
               //1、封装参数
               Map m = new HashMap<>();
               m.put("appid", "wx74862e0dfcf69954");
               m.put("mch_id", "1558950191");
               m.put("out_trade_no", orderNo);
               m.put("nonce_str", WXPayUtil.generateNonceStr());
               //2、设置请求
               HttpClient client = new
                       HttpClient("https://api.mch.weixin.qq.com/pay/orderquery");
               client.setXmlParam(WXPayUtil.generateSignedXml(m,
                       "T6m9iK73b0kn9g5v426MKfHQH7X8rKwb"));
               client.setHttps(true);
               client.post();
               //3、返回第三方的数据
               String xml = client.getContent();
               //转成Map
               Map<String, String> resultMap = WXPayUtil.xmlToMap(xml);
   
               return resultMap;
           } catch (Exception e) {
               e.printStackTrace();
           }
           return null;
       }
       //修改订单状态，添加支付记录
       @Override
       public void updateOrdersStatus(Map<String, String> map) {
           //获取订单id
           String orderNo = map.get("out_trade_no");
           //根据订单id查询订单信息
           QueryWrapper<Order> wrapper = new QueryWrapper<>();
           wrapper.eq("order_no",orderNo);
           Order order = orderService.getOne(wrapper);
           //更新订单表
           if(order.getStatus().intValue() == 1) return;
           order.setStatus(1);
           orderService.updateById(order);
           //记录支付日志
           PayLog payLog=new PayLog();
           payLog.setOrderNo(order.getOrderNo());//支付订单号
           payLog.setPayTime(new Date());
           payLog.setPayType(1);//支付类型
           payLog.setTotalFee(order.getTotalFee());//总金额(分)
           payLog.setTradeState(map.get("trade_state"));//支付状态
           payLog.setTransactionId(map.get("transaction_id"));
           payLog.setAttr(JSONObject.toJSONString(map));
           baseMapper.insert(payLog);//插入到支付日志表
       }
   }
   
   ```

6. 工具类

   - HttpClient

   ```java
   package com.guli.eduorder.utils;
   
   import org.apache.http.Consts;
   import org.apache.http.HttpEntity;
   import org.apache.http.NameValuePair;
   import org.apache.http.client.ClientProtocolException;
   import org.apache.http.client.entity.UrlEncodedFormEntity;
   import org.apache.http.client.methods.*;
   import org.apache.http.conn.ssl.SSLConnectionSocketFactory;
   import org.apache.http.conn.ssl.SSLContextBuilder;
   import org.apache.http.conn.ssl.TrustStrategy;
   import org.apache.http.entity.StringEntity;
   import org.apache.http.impl.client.CloseableHttpClient;
   import org.apache.http.impl.client.HttpClients;
   import org.apache.http.message.BasicNameValuePair;
   import org.apache.http.util.EntityUtils;
   
   import javax.net.ssl.SSLContext;
   import java.io.IOException;
   import java.security.cert.CertificateException;
   import java.security.cert.X509Certificate;
   import java.text.ParseException;
   import java.util.HashMap;
   import java.util.LinkedList;
   import java.util.List;
   import java.util.Map;
   
   /**
    * http请求客户端
    * 
    * @author qy
    * 
    */
   public class HttpClient {
   	private String url;
   	private Map<String, String> param;
   	private int statusCode;
   	private String content;
   	private String xmlParam;
   	private boolean isHttps;
   
   	public boolean isHttps() {
   		return isHttps;
   	}
   
   	public void setHttps(boolean isHttps) {
   		this.isHttps = isHttps;
   	}
   
   	public String getXmlParam() {
   		return xmlParam;
   	}
   
   	public void setXmlParam(String xmlParam) {
   		this.xmlParam = xmlParam;
   	}
   
   	public HttpClient(String url, Map<String, String> param) {
   		this.url = url;
   		this.param = param;
   	}
   
   	public HttpClient(String url) {
   		this.url = url;
   	}
   
   	public void setParameter(Map<String, String> map) {
   		param = map;
   	}
   
   	public void addParameter(String key, String value) {
   		if (param == null)
   			param = new HashMap<String, String>();
   		param.put(key, value);
   	}
   
   	public void post() throws ClientProtocolException, IOException {
   		HttpPost http = new HttpPost(url);
   		setEntity(http);
   		execute(http);
   	}
   
   	public void put() throws ClientProtocolException, IOException {
   		HttpPut http = new HttpPut(url);
   		setEntity(http);
   		execute(http);
   	}
   
   	public void get() throws ClientProtocolException, IOException {
   		if (param != null) {
   			StringBuilder url = new StringBuilder(this.url);
   			boolean isFirst = true;
   			for (String key : param.keySet()) {
   				if (isFirst)
   					url.append("?");
   				else
   					url.append("&");
   				url.append(key).append("=").append(param.get(key));
   			}
   			this.url = url.toString();
   		}
   		HttpGet http = new HttpGet(url);
   		execute(http);
   	}
   
   	/**
   	 * set http post,put param
   	 */
   	private void setEntity(HttpEntityEnclosingRequestBase http) {
   		if (param != null) {
   			List<NameValuePair> nvps = new LinkedList<NameValuePair>();
   			for (String key : param.keySet())
   				nvps.add(new BasicNameValuePair(key, param.get(key))); // 参数
   			http.setEntity(new UrlEncodedFormEntity(nvps, Consts.UTF_8)); // 设置参数
   		}
   		if (xmlParam != null) {
   			http.setEntity(new StringEntity(xmlParam, Consts.UTF_8));
   		}
   	}
   
   	private void execute(HttpUriRequest http) throws ClientProtocolException,
   			IOException {
   		CloseableHttpClient httpClient = null;
   		try {
   			if (isHttps) {
   				SSLContext sslContext = new SSLContextBuilder()
   						.loadTrustMaterial(null, new TrustStrategy() {
   							// 信任所有
   							public boolean isTrusted(X509Certificate[] chain,
   									String authType)
   									throws CertificateException {
   								return true;
   							}
   						}).build();
   				SSLConnectionSocketFactory sslsf = new SSLConnectionSocketFactory(
   						sslContext);
   				httpClient = HttpClients.custom().setSSLSocketFactory(sslsf)
   						.build();
   			} else {
   				httpClient = HttpClients.createDefault();
   			}
   			CloseableHttpResponse response = httpClient.execute(http);
   			try {
   				if (response != null) {
   					if (response.getStatusLine() != null)
   						statusCode = response.getStatusLine().getStatusCode();
   					HttpEntity entity = response.getEntity();
   					// 响应内容
   					content = EntityUtils.toString(entity, Consts.UTF_8);
   				}
   			} finally {
   				response.close();
   			}
   		} catch (Exception e) {
   			e.printStackTrace();
   		} finally {
   			httpClient.close();
   		}
   	}
   
   	public int getStatusCode() {
   		return statusCode;
   	}
   
   	public String getContent() throws ParseException, IOException {
   		return content;
   	}
   
   }
   
   ```

   - OrderNoUtil

   ```java
   package com.guli.eduorder.utils;
   
   public class OrderNoUtil {
   
       /**
        * 获取订单号
        * @return
        */
       public static String getOrderNo() {
           SimpleDateFormat sdf = new SimpleDateFormat("yyyyMMddHHmmss");
           String newDate = sdf.format(new Date());
           String result = "";
           Random random = new Random();
           for (int i = 0; i < 3; i++) {
               result += random.nextInt(10);
           }
           return newDate + result;
       }
   
   }
   
   ```

## 二.前端

> 安装插件qriously

1. 在course/_id页面点击购买

   - 调用生成订单方法
   - 传递课程id
   - 得到订单号
   - 带着订单号跳转/orders/_oid

2. orders/_oid

   - 异步:根据订单编号查询订单信息，传递订单号，得到订单信息，在页面展示
   - 点击支付，带着订单号跳转到/pay/_pid

3. pay/_pid

   - 异步:根据订单编号调用生成微信二维码

   - 每隔三秒查询订单状态，判断情况

     ```html
     <template>
       <div class="cart py-container">
         <!--主内容-->
         <div class="checkout py-container pay">
           <div class="checkout-tit">
             <h4 class="fl tit-txt">
               <span class="success-icon"></span
               ><span class="success-info"
                 >订单提交成功，请您及时付款！订单号：{{ payObj.out_trade_no }}</span
               >
             </h4>
             <span class="fr"
               ><em class="sui-lead">应付金额：</em
               ><em class="orange money">￥{{ payObj.total_fee }}</em></span
             >
             <div class="clearfix"></div>
           </div>
           <div class="checkout-steps">
             <div class="fl weixin">微信支付</div>
             <div class="fl sao">
               <p class="red">请使用微信扫一扫。</p>
               <div class="fl code">
                 <!-- <img id="qrious" src="~/assets/img/erweima.png" alt=""> -->
                 <!-- <qriously value="weixin://wxpay/bizpayurl?pr=R7tnDpZ" :size="338"/> -->
                 <qriously :value="payObj.code_url" :size="338" />
                 <div class="saosao">
                   <p>请使用微信扫一扫</p>
                   <p>扫描二维码支付</p>
                 </div>
               </div>
             </div>
             <div class="clearfix"></div>
             <!-- <p><a href="pay.html" target="_blank">> 其他支付方式</a></p> -->
           </div>
         </div>
       </div>
     </template>
     <script>
     import ordersApi from "@/api/orders";
     export default {
       asyncData({ params, error }) {
         return ordersApi.createNative(params.pid).then((response) => {
           return {
             payObj: response.data.data.map,
           };
         });
       },
       data() {
         return {
           timer1: "",
         };
       },
       //每隔三秒调用一次查询订单状态的方法
       mounted() {
         //页面渲染之后执行
         this.timer1 = setInterval(() => {
           this.queryOrderStatus(this.payObj.out_trade_no);
         }, 3000);
       },
       methods: {
         queryOrderStatus(orderNo) {
           ordersApi.queryPayStatus(orderNo).then((response) => {
             if (response.data.success) {
               //支付成功，清除定时器
               clearInterval(this.timer1);
               //提示
               this.$message({
                 type: "success",
                 message: "支付成功!",
               });
               //跳转回到课程详情页面
               this.$router.push({ path: "/course/" + this.payObj.course_id });
             }
           });
         },
       },
     };
     </script>
     
     ```

   - 响应器拦截“支付中”消息request

     ```js
     import axios from 'axios'
     import cookie from 'js-cookie';
     import { MessageBox, Message } from 'element-ui';
     // 创建axios实例
     const service = axios.create({
       baseURL: 'http://localhost:9001', // api的base_url
       timeout: 20000 // 请求超时时间
     })
     //3.编写拦截器
     service.interceptors.request.use(
       config => {
         //debugger
         //判断1是否有名为'guli_token'的cookie
         if (cookie.get('guli_token')) {
           config.headers['token'] = cookie.get('guli_token');
         }
         return config
       },
       err => {
         return Promise.reject(err);
       }
     )
     // http response 拦截器
     service.interceptors.response.use(
       response => {
         //debugger
         if (response.data.code == 28004) {
           console.log("response.data.resultCode是28004")
           // 返回 错误代码-1 清除ticket信息并跳转到登录页面
           //debugger
           window.location.href = "/login"
           return
         } else {
           if (response.data.code !== 20000) {
             //25000：订单支付中，不做任何提示
             if (response.data.code != 25000) {
               Message({
                 message: response.data.message || 'error',
                 type: 'error',
                 duration: 5 * 1000
               })
             }
           } else {
             return response;
           }
         }
       },
       error => {
         return Promise.reject(error.response) // 返回接口返回的错误信息
       });
     export default service
     ```


## 三.完善

order

```java
  //3.根据课程id和用户id查询订单表中订单状态
    @GetMapping("isBuyCourse/{courseId}/{memberId}")
    public Boolean isBuyCourse (@PathVariable String courseId,@PathVariable String memberId) {
        QueryWrapper<Order> queryWrapper = new QueryWrapper<>();
        queryWrapper.eq("course_id",courseId);
        queryWrapper.eq("member_id",memberId);
        queryWrapper.eq("status",1);
        int count = orderService.count(queryWrapper);
        if (count > 0){
            return true;
        } else {
            return false;
        }
    }
```

在edu中调用（补充课程详情方法）

```java
//2 课程详情的方法
    @GetMapping("getFrontCourseInfo/{courseId}")
    public R getFrontCourseInfo(@PathVariable String courseId, HttpServletRequest request) {
        //根据课程id，编写sql语句查询课程信息
        CourseWebVo courseWebVo = courseService.getBaseCourseInfo(courseId);

        //根据课程id查询章节和小节
        List<ChapterVo> chapterVideoList = chapterService.getChapterById(courseId);

        //根据课程id和用户id查询课程是否可观看（已支付）
        String memberId = JwtUtils.getMemberIdByJwtToken(request);
        Boolean buyCourse = ordersClient.isBuyCourse(courseId, memberId);
        return R.ok().data("courseWebVo",courseWebVo).data("chapterVideoList",chapterVideoList).data("isBuy",buyCourse);
    }
```

前端course/_id

```html
<template>
  <div id="aCoursesList" class="bg-fa of">
    <!-- /课程详情 开始 -->
    <section class="container">
      <section class="path-wrap txtOf hLh30">
        <a href="#" title class="c-999 fsize14">首页</a>
        \
        <a href="#" title class="c-999 fsize14">{{courseWebVo.subjectLevelOne}}</a>
        \
        <span class="c-333 fsize14">{{courseWebVo.subjectLevelTwo}}</span>
      </section>
      <div>
        <article class="c-v-pic-wrap" style="height: 357px;">
          <section class="p-h-video-box" id="videoPlay">
            <img height="357px" :src="courseWebVo.cover" :alt="courseWebVo.title" class="dis c-v-pic">
          </section>
        </article>
        <aside class="c-attr-wrap">
          <section class="ml20 mr15">
            <h2 class="hLh30 txtOf mt15">
              <span class="c-fff fsize24">{{courseWebVo.title}}</span>
            </h2>
            <section class="c-attr-jg">
              <span class="c-fff">价格：</span>
              <b class="c-yellow" style="font-size:24px;">￥{{courseWebVo.price}}</b>
            </section>
            <section class="c-attr-mt c-attr-undis">
              <span class="c-fff fsize14">主讲： {{courseWebVo.teacherName}}&nbsp;&nbsp;&nbsp;</span>
            </section>
            <section class="c-attr-mt of">
              <span class="ml10 vam">
                <em class="icon18 scIcon"></em>
                <a class="c-fff vam" title="收藏" href="#" >收藏</a>
              </span>
            </section>
            <section  v-if="isbuy || Number(courseWebVo.price) === 0" class="c-attr-mt">
              <a href="#" title="立即观看" class="comm-btn c-btn-3">立即观看</a>
            </section>
            <section  v-else class="c-attr-mt">
              <a @click="createOrders()" href="#" title="立即购买" class="comm-btn c-btn-3">立即购买</a>
            </section>
          </section>
        </aside>
        <aside class="thr-attr-box">
          <ol class="thr-attr-ol">
            <li>
              <p>&nbsp;</p>
              <aside>
                <span class="c-fff f-fM">购买数</span>
                <br>
                <h6 class="c-fff f-fM mt10">{{courseWebVo.buyCount}}</h6>
              </aside>
            </li>
            <li>
              <p>&nbsp;</p>
              <aside>
                <span class="c-fff f-fM">课时数</span>
                <br>
                <h6 class="c-fff f-fM mt10">20</h6>
              </aside>
            </li>
            <li>
              <p>&nbsp;</p>
              <aside>
                <span class="c-fff f-fM">浏览数</span>
                <br>
                <h6 class="c-fff f-fM mt10">501</h6>
              </aside>
            </li>
          </ol>
        </aside>
        <div class="clear"></div>
      </div>
      <!-- /课程封面介绍 -->
      <div class="mt20 c-infor-box">
        <article class="fl col-7">
          <section class="mr30">
            <div class="i-box">
              <div>
                <section id="c-i-tabTitle" class="c-infor-tabTitle c-tab-title">
                  <a name="c-i" class="current" title="课程详情">课程详情</a>
                </section>
              </div>
              <article class="ml10 mr10 pt20">
                <div>
                  <h6 class="c-i-content c-infor-title">
                    <span>课程介绍</span>
                  </h6>
                  <div class="course-txt-body-wrap">
                    <section class="course-txt-body">
                      <p v-html="courseWebVo.description">{{courseWebVo.description}}</p>
                    </section>
                  </div>
                </div>
                <!-- /课程介绍 -->
                <div class="mt50">
                  <h6 class="c-g-content c-infor-title">
                    <span>课程大纲</span>
                  </h6>
                  <section class="mt20">
                    <div class="lh-menu-wrap">
                      <menu id="lh-menu" class="lh-menu mt10 mr10">
                        <ul>
                          <!-- 文件目录 -->
                          <li class="lh-menu-stair" v-for="chapter in chapterVideoList" :key="chapter.id">
                            <a href="javascript: void(0)" :title="chapter.title" class="current-1">
                              <em class="lh-menu-i-1 icon18 mr10"></em>{{chapter.title}}
                            </a>

                            <ol class="lh-menu-ol" style="display: block;">
                              <li class="lh-menu-second ml30" v-for="video in chapter.children" :key="video.id">
                                <a :href="'/player/'+video.videoSourceId" target="_blank">
                                  <span class="fr">
                                    <i class="free-icon vam mr10">免费试听</i>
                                  </span>
                                  <em class="lh-menu-i-2 icon16 mr5">&nbsp;</em>{{video.title}}
                                </a>
                              </li>
                              
                            </ol>

                          </li>
                        </ul>
                      </menu>
                    </div>
                  </section>
                </div>
                <!-- /课程大纲 -->
              </article>
            </div>
          </section>
        </article>
        <aside class="fl col-3">
          <div class="i-box">
            <div>
              <section class="c-infor-tabTitle c-tab-title">
                <a title href="javascript:void(0)">主讲讲师</a>
              </section>
              <section class="stud-act-list">
                <ul style="height: auto;">
                  <li>
                    <div class="u-face">
                      <a href="#">
                        <img :src="courseWebVo.avatar" width="50" height="50" alt>
                      </a>
                    </div>
                    <section class="hLh30 txtOf">
                      <a class="c-333 fsize16 fl" href="#">{{courseWebVo.teacherName}}</a>
                    </section>
                    <section class="hLh20 txtOf">
                      <span class="c-999">{{courseWebVo.intro}}</span>
                    </section>
                  </li>
                </ul>
              </section>
            </div>
          </div>
        </aside>
        <div class="clear"></div>
      </div>
    </section>
    <!-- /课程详情 结束 -->
  </div>
</template>

<script>
import courseApi from '@/api/course'
import ordersApi from '@/api/orders'
export default {
   asyncData({ params, error }) {
     return {courseId: params.id}
   },
   data() {
     return {
        courseWebVo: {},
        chapterVideoList: [],
        isbuy: false,
     }
   },
   created() {//在页面渲染之前执行
      this.initCourseInfo()
   },
   methods:{
     //查询课程详情信息
     initCourseInfo() {
        courseApi.getCourseInfo(this.courseId)
          .then(response => {
            this.courseWebVo=response.data.data.courseWebVo,
            this.chapterVideoList=response.data.data.chapterVideoList,
            this.isbuy=response.data.data.isBuy
        })
     },
     //生成订单
     createOrders() {
       ordersApi.createOrders(this.courseId)
        .then(response => {
          //获取返回订单号
          //生成订单之后，跳转订单显示页面
          this.$router.push({path:'/orders/'+response.data.data.orderId})
        })
     }
   }
};
</script>

```


