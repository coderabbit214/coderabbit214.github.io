# Springboot-任务


# Springboot-任务

## 一.异步任务

1. 主程序注解 @EnableAsync

   ```java
   package com.jsh.task;
   
   import org.springframework.boot.SpringApplication;
   import org.springframework.boot.autoconfigure.SpringBootApplication;
   import org.springframework.scheduling.annotation.EnableAsync;
   
   @SpringBootApplication
   @EnableAsync
   public class SpringbootTaskApplication {
   
       public static void main(String[] args) {
           SpringApplication.run(SpringbootTaskApplication.class, args);
       }
   
   }
   
   ```

2. 需要异步的方法  @Async

   ```java
   package com.jsh.task.service;
   
   import org.springframework.scheduling.annotation.Async;
   import org.springframework.stereotype.Service;
   
   @Service
   public class AsyncService {
       @Async
       public void hello(){
           try {
               Thread.sleep(10000);
           } catch (InterruptedException e) {
               e.printStackTrace();
           }
           System.out.println("处理数据中。。。");
       }
   }
   
   ```

## 二.定时任务

1. 主程序加上注解 @EnableScheduling

2. 需要定时调动的方法 @Scheduled(cron = "0 * * * * MON-SAT")

   > cron 表达式：
   >
   > 六个参数（空格隔开）：秒 分 时 每月的第几天 月 每周第几天
   >
   > 0-7表示周 0和7是周天
   >
   > 可以使用通配符
   >
   > 通配符说明:
   > - “*”表示所有值. 例如:在分的字段上设置，表示每一分钟都会触发。
   > - “?”表示不指定值。使用的场景为不需要关心当前设置这个字段的值。例如:要在每月的10号触发一个操作，但不关心是周几，所以需要周位置的那个字段设置为"?" 具体设置为 0 0 0 10 * ?
   > - “-”表示区间。例如 在小时上设置 "10-12",表示 10,11,12点都会触发。
   > - “,” 表示指定多个值，例如在周字段上设置 "1，3，5" 表示周一，周三和周五触发
   > - “/”用于递增触发。如在秒上面设置"5/15" 表示从5秒开始，每增15秒触发(5,20,35,50)。在月字段上设置'1/3'所示每月1号开始，每隔三天触发一次。一般不写的话，默认递增为基本单位，如1分钟，1秒钟
   > - “L”表示最后的意思。在日字段设置上，表示当月的最后一天(依据当前月份，如果是二月还会依据是否是润年[leap]), 在周字段上表示星期六，相当于"6"或"SAT"。如果在"L"前加上数字，则表示该数据的最后一个。例如在周字段上设置"5L"这样的格式,则表示“本月最后一个星期五"
   > - “W”表示离指定日期的最近那个工作日(周一至周五). 例如在日字段上设置"15W"，表示离每月15号最近的那个工作日触发。如果15号正好是周六，则找最近的周五(14号)触发, 如果15号是周未，则找最近的下周一(16号)触发.如果15号正好在工作日(周一至周五)，则就在该天触发。如果指定格式为 "1W",它则表示每月1号往后最近的工作日触发。如果1号正是周六，则将在3号下周一触发。(注，"W"前只能设置具体的数字,不允许区间"-").
   >   小提示
   >   'L'和 'W'可以一组合使用。如果在日字段上设置"LW",则表示在本月的最后一个工作日触发(一般指发工资 )
   > - “#”序号(表示每月的第几个周几)，例如在周字段上设置"6#3"表示在每月的第三个周六.

   ```java
   package com.jsh.task.service;
   
   import org.springframework.scheduling.annotation.Scheduled;
   import org.springframework.stereotype.Service;
   
   @Service
   public class ScheduledService {
       	 /*
       	    * <ul>
   	        * <li>second</li>
               * <li>minute</li>
               * <li>hour</li>
               * <li>day of month</li>
               * <li>month</li>
               * <li>day of week</li>
               * </ul>
       	  */
       @Scheduled(cron = "0 * * * * MON-SAT")
       public void hello(){
           System.out.println("hello");
       }
   }
   
   ```

## 三.邮件任务

1. application.properties

   ```
   spring.mail.username=1157237955@qq.com
   # 开启服务 申请密码 不是qq密码
   spring.mail.password=xxxxxxxxxxxxxxxxx 
   spring.mail.host=smtp.qq.com
   ```

2. 简单邮箱

   ```java
       @Autowired
       JavaMailSenderImpl mailSender;
       @Test
       void contextLoads() {
               SimpleMailMessage simpleMailMessage = new SimpleMailMessage();
               //邮件设置
           	//标题
               simpleMailMessage.setSubject("通知：今晚开会");
           	//内容
               simpleMailMessage.setText("今晚7：30开会");
               simpleMailMessage.setTo("1157237955@qq.com");
               simpleMailMessage.setFrom("1157237955@qq.com");
               mailSender.send(simpleMailMessage);
       }
   ```

3. 复杂邮箱

   ```java
       @Autowired
       JavaMailSenderImpl mailSender;
       @Test
       void contextLoads2() throws MessagingException {
           //创建一个复杂的消息邮件
           MimeMessage mimeMailMessage =mailSender.createMimeMessage();
           MimeMessageHelper helper = new MimeMessageHelper(mimeMailMessage,true);
           //邮件设置
           helper.setSubject("通知：今晚开会");
           //可以使用html语法 
           helper.setText("<b>今晚7：30开会</b>",true);
           helper.setTo("1157237955@qq.com");
           helper.setFrom("1157237955@qq.com");
           //上传文件
           helper.addAttachment("1.png",new File("E:\\照片\\20.04.13景\\IMG_0490-31.png"));
           mailSender.send(mimeMailMessage);
       }
   ```

   

