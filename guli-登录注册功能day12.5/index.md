# guli-登录注册功能（day12.5）


# guli-登录注册功能（day12.5）

## 一.单点登录三种方式介绍

![](https://edu-guli-0214.oss-cn-beijing.aliyuncs.com/%E7%AC%94%E8%AE%B0/02%20%E5%8D%95%E7%82%B9%E7%99%BB%E5%BD%95%E4%B8%89%E7%A7%8D%E6%96%B9%E5%BC%8F%E4%BB%8B%E7%BB%8D.png)

## 二.JWT

- token是按照一定规则生成字符串，包含用户信息
- 规则不一定
- 一般采用通用规则，JWT

> JWT就是给我们规定好了规则，使用jwt规则生成字符串，包含用户信息
>
> jwt生成字符串包含三部分
>
> 1. jwt头信息
> 2. 有效载荷 包含主体信息
> 3. 签名哈希 防伪标志

### 使用

1. 在common引入依赖

   common_utils

   ```xml
   <dependencies>
   <!-- JWT -->
       <dependency>
           <groupId>io.jsonwebtoken</groupId>
           <artifactId>jjwt</artifactId>
       </dependency>
   </dependencies>
   ```

2. 复制jwt工具类

   ```java
   package com.guli.commonutils;
   
   import io.jsonwebtoken.Claims;
   import io.jsonwebtoken.Jws;
   import io.jsonwebtoken.Jwts;
   import io.jsonwebtoken.SignatureAlgorithm;
   import org.springframework.http.server.reactive.ServerHttpRequest;
   import org.springframework.util.StringUtils;
   
   import javax.servlet.http.HttpServletRequest;
   import java.util.Date;
   
   /**
    * @author helen
    * @since 2019/10/16
    */
   public class JwtUtils {
   
       //token过期时间
       public static final long EXPIRE = 1000 * 60 * 60 * 24;
       //密钥
       public static final String APP_SECRET = "ukc8BDbRigUDaY6pZFfWus2jSH";
       //生成token字符串的方法
       public static String getJwtToken(String id, String nickname){
   
           String JwtToken = Jwts.builder()
                   //设置头信息
                   .setHeaderParam("typ", "JWT")
                   .setHeaderParam("alg", "HS256")
                   //设置过期时间
                   .setSubject("guli-user")
                   .setIssuedAt(new Date())
                   .setExpiration(new Date(System.currentTimeMillis() + EXPIRE))
                   //设置token主体部分
                   .claim("id", id)
                   .claim("nickname", nickname)
                   //设置编码
                   .signWith(SignatureAlgorithm.HS256, APP_SECRET)
                   .compact();
   
           return JwtToken;
       }
   
       /**
        * 判断token是否存在与有效
        * @param jwtToken
        * @return
        */
       public static boolean checkToken(String jwtToken) {
           if(StringUtils.isEmpty(jwtToken)) return false;
           try {
               Jwts.parser().setSigningKey(APP_SECRET).parseClaimsJws(jwtToken);
           } catch (Exception e) {
               e.printStackTrace();
               return false;
           }
           return true;
       }
   
       /**
        * 判断token是否存在与有效
        * @param request
        * @return
        */
       public static boolean checkToken(HttpServletRequest request) {
           try {
               String jwtToken = request.getHeader("token");
               if(StringUtils.isEmpty(jwtToken)) return false;
               Jwts.parser().setSigningKey(APP_SECRET).parseClaimsJws(jwtToken);
           } catch (Exception e) {
               e.printStackTrace();
               return false;
           }
           return true;
       }
   
       /**
        * 根据token获取会员id
        * @param request
        * @return
        */
       public static String getMemberIdByJwtToken(HttpServletRequest request) {
           String jwtToken = request.getHeader("token");
           if(StringUtils.isEmpty(jwtToken)) return "";
           Jws<Claims> claimsJws = Jwts.parser().setSigningKey(APP_SECRET).parseClaimsJws(jwtToken);
           Claims claims = claimsJws.getBody();
           return (String)claims.get("id");
       }
   }
   ```

## 三.阿里云短信服务

### 1.创建service_msm模块

### 2.创建包结构，controller，service，启动类，配置文件

### 3.阿里云控制台

1. 开通短信服务
2. 申请签名管理和模板管理

## 四.阿里云短信发送

1. 引入依赖

   ```xml
   <dependencies>
   <!-- 阿里云短信依赖 -->
       <!-- json转换工具 -->
       <dependency>
           <groupId>com.alibaba</groupId>
           <artifactId>fastjson</artifactId>
       </dependency>
       <!-- 阿里云短信依赖核心库 -->
       <dependency>
           <groupId>com.aliyun</groupId>
           <artifactId>aliyun-java-sdk-core</artifactId>
       </dependency>
   </dependencies>
   ```

2. controller

   > - 使用redis存储五分钟验证码
   > - 使用RandomUtil工具类生成验证码

   ```java
   package com.guli.msmservice.controller;
   
   @RestController
   @RequestMapping("/edumsm/msm")
   @CrossOrigin
   public class MsmController {
   
       @Autowired
       private MsmService msmService;
   
       @Autowired
       private RedisTemplate<String,String> redisTemplate;
   
       //发送短信的方法
       @GetMapping("send/{phone}")
       public R sendMsm(@PathVariable("phone") String phone) {
           //从redis中获取验证码，如果获取到直接返回
           String code = redisTemplate.opsForValue().get(phone);
           if (!StringUtils.isEmpty(code)) {
               return R.ok();
           }
           //如果redis获取不到，进行阿里云发送
           //生成随机值，传递阿里云进行发送
           code = RandomUtil.getSixBitRandom();
           Map<String,Object> param = new HashMap<>();
           param.put("code",code);
           //调用service进行短信发送
           boolean isSend = msmService.send(param,phone);
           if (isSend) {
               //发送成功，把发送成功方法放在redis中
               //设置有效时间
               redisTemplate.opsForValue().set(phone,code,5, TimeUnit.MINUTES);
               return R.ok();
           } else {
               return R.error().message("发送短信错误");
           }
   
       }
   }
   
   ```

3. service

   > 有新方法

   ```java
   package com.guli.msmservice.service.impl;
   
   @Service
   public class MsmServiceImpl implements MsmService {
       //发送短信的方法
       @Override
       public boolean send(Map<String, Object> param, String phone) {
           if(StringUtils.isEmpty(phone)) return false;
           DefaultProfile profile = DefaultProfile.getProfile("default","key","secret");
           IAcsClient client = new DefaultAcsClient(profile);
           //设置相关固定参数
           CommonRequest request = new CommonRequest();
           request.setMethod(MethodType.POST);
           request.setDomain("dysmsapi.aliyuncs.com");
           request.setVersion("2017-05-25");
           request.setAction("SendSms");
   
           //设置发送相关参数
           //手机号
           request.putQueryParameter("PhoneNumbers",phone);
           //签名名称
           request.putQueryParameter("SignName","我的鼓励在线教育网站");
           //模板code
           request.putQueryParameter("TemplateCode","SMS_203672814");
           //设置验证码 需要转换为JSON
           request.putQueryParameter("TemplateParam", JSONObject.toJSONString(param));
   
           //发送
           try {
               CommonResponse commonResponse = client.getCommonResponse(request);
               boolean success = commonResponse.getHttpResponse().isSuccess();
               return success;
   
           } catch (ClientException e) {
               e.printStackTrace();
               return false;
           }
       }
   }
   
   ```

## 五.登录注册接口

1. 创建service_ucenter模块

2. 数据库创建用户表，使用代码生成器生成三层架构

3. 创建包结构,启动类，配置文件

4. controller

   > ```java
   > package com.guli.educenter.entity.vo;
   > 
   > import io.swagger.annotations.ApiModelProperty;
   > import lombok.Data;
   > 
   > @Data
   > public class RegisterVo {
   >     @ApiModelProperty(value = "昵称")
   >     private String nickname;
   >     @ApiModelProperty(value = "手机号")
   >     private String mobile;
   >     @ApiModelProperty(value = "密码")
   >     private String password;
   >     @ApiModelProperty(value = "验证码")
   >     private String code;
   > }
   > 
   > ```

   ```java
   package com.guli.educenter.controller;
   @RestController
   @RequestMapping("/educenter/member")
   @CrossOrigin
   public class UcenterMemberController {
       @Autowired
       private UcenterMemberService memberService;
   
       //登录
       @PostMapping("login")
       public R loginUser(@RequestBody UcenterMember member) {
           //返回token
           String token = memberService.login(member);
           return R.ok().data("token",token);
       }
       //注册
       @PostMapping("register")
       public R registerUser(@RequestBody RegisterVo registerVo) {
           memberService.register(registerVo);
           return R.ok();
       }
       //根据token获取用户信息
       @GetMapping("getMemberInfo")
       public R getMemberInfo(HttpServletRequest request) {
           //调用jwt ， 获取用户id
           String memberId = JwtUtils.getMemberIdByJwtToken(request);
           UcenterMember ugetMemberInfo = memberService.getById(memberId);
           return R.ok().data("ugetMemberInfo",ugetMemberInfo);
       }
   }
   ```

5. service

   > 密码进行MD5加密 
   >
   > ```java
   > package com.guli.commonutils;
   > 
   > import java.security.MessageDigest;
   > import java.security.NoSuchAlgorithmException;
   > 
   > 
   > public final class MD5 {
   > 
   >     public static String encrypt(String strSrc) {
   >         try {
   >             char hexChars[] = { '0', '1', '2', '3', '4', '5', '6', '7', '8',
   >                     '9', 'a', 'b', 'c', 'd', 'e', 'f' };
   >             byte[] bytes = strSrc.getBytes();
   >             MessageDigest md = MessageDigest.getInstance("MD5");
   >             md.update(bytes);
   >             bytes = md.digest();
   >             int j = bytes.length;
   >             char[] chars = new char[j * 2];
   >             int k = 0;
   >             for (int i = 0; i < bytes.length; i++) {
   >                 byte b = bytes[i];
   >                 chars[k++] = hexChars[b >>> 4 & 0xf];
   >                 chars[k++] = hexChars[b & 0xf];
   >             }
   >             return new String(chars);
   >         } catch (NoSuchAlgorithmException e) {
   >             e.printStackTrace();
   >             throw new RuntimeException("MD5加密出错！！+" + e);
   >         }
   >     }
   > 
   >     public static void main(String[] args) {
   >         System.out.println(MD5.encrypt("111111"));
   >     }
   > 
   > }
   > 
   > ```
   >
   > 

   ```java
   package com.guli.educenter.service.impl;
   
   @Service
   public class UcenterMemberServiceImpl extends ServiceImpl<UcenterMemberMapper, UcenterMember> implements UcenterMemberService {
   
       @Autowired
       private RedisTemplate<String,String> redisTemplate;
       //登录
       @Override
       public String login(UcenterMember member) {
           //获取登录手机号和密码
           String mobile = member.getMobile();
           String password = member.getPassword();
   
           //判空
           if(StringUtils.isEmpty(mobile) || StringUtils.isEmpty(password)) {
               throw new GuliException(20001,"登陆失败");
           }
   
           //判断手机号是否正确
           QueryWrapper<UcenterMember> queryWrapper = new QueryWrapper<>();
           queryWrapper.eq("mobile",mobile);
           UcenterMember mobileMember = baseMapper.selectOne(queryWrapper);
           if (mobileMember == null) {
               throw new GuliException(20001,"手机号不存在");
           }
   
           //判断密码
           //存储到数据库中的密码需要加密
           //把输入的密码先进行加密 再 进行比较
           //加密方式 MD5
           if (!MD5.encrypt(password).equals(mobileMember.getPassword())) {
               throw new GuliException(20001,"账号或密码错误");
           }
   
           //判断用户是否禁用
           if (mobileMember.getIsDisabled()) {
               throw new GuliException(20001,"用户已禁用");
           }
   
           //登录成功
           //生成token字符串，使用jwt工具类
           String jwtToken = JwtUtils.getJwtToken(mobileMember.getId(), mobileMember.getNickname());
           return jwtToken;
       }
   
       //注册
       @Override
       public void register(RegisterVo registerVo) {
           //获取注册的数据
           String code = registerVo.getCode();
           String mobile = registerVo.getMobile();
           String nickname = registerVo.getNickname();
           String password = registerVo.getPassword();
   
           //非空判断
           if (StringUtils.isEmpty(code) || StringUtils.isEmpty(mobile) || StringUtils.isEmpty(nickname) || StringUtils.isEmpty(password)) {
               throw new GuliException(20001,"注册失败");
           }
   
           //判断手机验证码是否正确
           //获取redis中的验证码
           String redisCode = redisTemplate.opsForValue().get(mobile);
           //判断
           if (redisCode == null) {
               throw new GuliException(20001,"手机号或验证码错误");
           }
           if (!code.equals(redisCode)) {
               throw new GuliException(20001,"手机号或验证码错误");
           }
           //判断手机号是否重复
           QueryWrapper<UcenterMember> wrapper = new QueryWrapper<>();
           wrapper.eq("mobile",mobile);
           Integer integer = baseMapper.selectCount(wrapper);
           if (integer > 0){
               throw new GuliException(20001,"已注册");
           }
           //将数据添加到数据库中
           UcenterMember member = new UcenterMember();
           member.setMobile(mobile);
           member.setNickname(nickname);
           member.setPassword(MD5.encrypt(password));
           member.setIsDisabled(false);
           member.setAvatar("http://thirdwx.qlogo.cn/mmopen/vi_32/DYAIOgq83epMicP9UT6mVjYWdno0OJZkOXiajG0sllJTbGJ9DYiceej2XvbDSGCK8LCF7jv1PuG2uoYlePWic9XO8A/132");
           baseMapper.insert(member);
       }
   }
   ```

## 六.前端页面整合

1. 安装插件

   - npm install element-ui
   - npm install vue-qriously
   -  npm install js-cookie

2. 在nuxt环境中安装插件

   plugins/nuxt-swiper-plugin.js

   ```js
   import Vue from 'vue'
   import VueAwesomeSwiper from 'vue-awesome-swiper/dist/ssr'
   import VueQriously from 'vue-qriously'
   import ElementUI from 'element-ui' //element-ui的全部组件
   import 'element-ui/lib/theme-chalk/index.css'//element-ui的css
   Vue.use(ElementUI) //使用elementUI
   Vue.use(VueQriously)
   Vue.use(VueAwesomeSwiper)
   ```

3. 在layouts文件夹创建登录注册页面布局

   layouts/sign.vue

   ```html
   <template>
   <div class="sign">
   <!--标题-->
   <div class="logo">
   <img src="~/assets/img/logo.png" alt="logo">
   </div>
   <!--表单-->
   <nuxt/>
   </div>
   </template>
   ```

4. 在default.vue中修改登录注册超链接地址

5. 创建登录注册页面

   register.vue

   ```html
   <template>
     <div class="main">
       <div class="title">
         <a href="/login">登录</a>
         <span>·</span>
         <a class="active" href="/register">注册</a>
       </div>
       <div class="sign-up-container">
         <el-form ref="userForm" :model="params">
           <el-form-item
             class="input-prepend restyle"
             prop="nickname"
             :rules="[
               {
                 required: true,
                 message: '请输入你的昵称',
                 trigger: 'blur',
               },
             ]"
           >
             <div>
               <el-input
                 type="text"
                 placeholder="你的昵称"
                 v-model="params.nickname"
               />
               <i class="iconfont icon-user" />
             </div>
           </el-form-item>
           <el-form-item
             class="input-prepend restyle no-radius"
             prop="mobile"
             :rules="[
               { required: true, message: '请输入手机号码', trigger: 'blur' },
               { validator: checkPhone, trigger: 'blur' },
             ]"
           >
             <div>
               <el-input
                 type="text"
                 placeholder="手机号"
                 v-model="params.mobile"
               />
               <i class="iconfont icon-phone" />
             </div>
           </el-form-item>
           <el-form-item
             class="input-prepend restyle no-radius"
             prop="code"
             :rules="[
               { required: true, message: '请输入验证码', trigger: 'blur' },
             ]"
           >
             <div
               style="width: 100%; display: block; float: left; position: relative"
             >
               <el-input type="text" placeholder="验证码" v-model="params.code" />
               <i class="iconfont icon-phone" />
             </div>
             <div
               class="btn"
               style="position: absolute; right: 0; top: 6px; width: 40%"
             >
               <a
                 href="javascript:"
                 type="button"
                 @click="getCodeFun()"
                 :value="codeTest"
                 style="border: none; background-color: none"
                 >{{ codeTest }}</a
               >
             </div>
           </el-form-item>
           <el-form-item
             class="input-prepend"
             prop="password"
             :rules="[{ required: true, message: '请输入密码', trigger: 'blur' }]"
           >
             <div>
               <el-input
                 type="password"
                 placeholder="设置密码"
                 v-model="params.password"
               />
               <i class="iconfont icon-password" />
             </div>
           </el-form-item>
           <div class="btn">
             <input
               type="button"
               class="sign-up-button"
               value="注册"
               @click="submitRegister()"
             />
           </div>
           <p class="sign-up-msg">
             点击 “注册” 即表示您同意并愿意遵守简书
             <br />
             <a target="_blank" href="http://www.jianshu.com/p/c44d171298ce"
               >用户协议</a
             >
             和
             <a target="_blank" href="http://www.jianshu.com/p/2ov8x3">隐私政策</a>
           </p>
         </el-form>
         <!-- 更多注册方式 -->
         <div class="more-sign">
           <h6>社交帐号直接注册</h6>
           <ul>
             <li>
               <a
                 id="weixin"
                 class="weixin"
                 target="_blank"
                 href="http://huaan.free.idcfengye.com/api/ucenter/wx/login"
                 ><i class="iconfont icon-weixin"
               /></a>
             </li>
             <li>
               <a id="qq" class="qq" target="_blank" href="#"
                 ><i class="iconfont icon-qq"
               /></a>
             </li>
           </ul>
         </div>
       </div>
     </div>
   </template>
   <script>
   import "~/assets/css/sign.css";
   import "~/assets/css/iconfont.css";
   import registerApi from "@/api/register";
   export default {
     layout: "sign",
     data() {
       return {
         params: {
           mobile: "",
           code: "",
           nickname: "",
           password: "",
         },
         sending: true, //是否发送验证码
         second: 60, //倒计时间
         codeTest: "获取验证码",
       };
     },
     methods: {
       //发送验证码
       getCodeFun() {
         //debugger
         // prop 换成你想监听的prop字段
         this.$refs.userForm.validateField("mobile", (errMsg) => {
           if (errMsg == "") {
             registerApi.sendCode(this.params.mobile).then((res) => {
               this.sending = false;
               this.timeDown();
             });
           }
         });
       },
       //倒计时
       timeDown() {
         let result = setInterval(() => {
           --this.second;
           this.codeTest = this.second;
           if (this.second < 1) {
             clearInterval(result);
             this.sending = true;
             //this.disabled = false;
             this.second = 60;
             this.codeTest = "获取验证码";
           }
         }, 1000);
       },
       //注册提交的方法
       submitRegister() {
             registerApi.registerMember(this.params).then((response) => {
               //提示注册成功
               this.$message({
                 type: "success",
                 message: "注册成功",
               });
               this.$router.push({ path: "/login" });
             });
       },
       checkPhone(rule, value, callback) {
         //debugger
         if (!/^1[34578]\d{9}$/.test(value)) {
           return callback(new Error("手机号码格式不正确"));
         }
         return callback();
       },
     },
   };
   </script>
   
   ```

   login.vue

   > 使用cookie存储用户数据

   ```html
   <template>
     <div class="main">
       <div class="title">
         <a class="active" href="/login">登录</a>
         <span>·</span>
         <a href="/register">注册</a>
       </div>
   
       <div class="sign-up-container">
         <el-form ref="userForm" :model="user">
           <el-form-item
             class="input-prepend restyle"
             prop="mobile"
             :rules="[
               { required: true, message: '请输入手机号码', trigger: 'blur' },
               { validator: checkPhone, trigger: 'blur' },
             ]"
           >
             <div>
               <el-input type="text" placeholder="手机号" v-model="user.mobile" />
               <i class="iconfont icon-phone" />
             </div>
           </el-form-item>
   
           <el-form-item
             class="input-prepend"
             prop="password"
             :rules="[{ required: true, message: '请输入密码', trigger: 'blur' }]"
           >
             <div>
               <el-input
                 type="password"
                 placeholder="密码"
                 v-model="user.password"
               />
               <i class="iconfont icon-password" />
             </div>
           </el-form-item>
   
           <div class="btn">
             <input
               type="button"
               class="sign-in-button"
               value="登录"
               @click="submitLogin()"
             />
           </div>
         </el-form>
         <!-- 更多登录方式 -->
         <div class="more-sign">
           <h6>社交帐号登录</h6>
           <ul>
             <li>
               <a
                 id="weixin"
                 class="weixin"
                 target="_blank"
                 href="http://qy.free.idcfengye.com/api/ucenter/weixinLogin/login"
                 ><i class="iconfont icon-weixin"
               /></a>
             </li>
             <li>
               <a id="qq" class="qq" target="_blank" href="#"
                 ><i class="iconfont icon-qq"
               /></a>
             </li>
           </ul>
         </div>
       </div>
     </div>
   </template>
   
   <script>
   import "~/assets/css/sign.css";
   import "~/assets/css/iconfont.css";
   import cookie from "js-cookie";
   import loginApi from "@/api/login";
   export default {
     layout: "sign",
   
     data() {
       return {
         user: {
           mobile: "",
           password: "",
         },
         //用户信息
         loginInfo: {},
       };
     },
   
     methods: {
       //登陆方法
       submitLogin(){
         //1.调用接口进行登录，返回token字符串
         loginApi.submitLogin(this.user).then(response=>{
           //2.获取token字符串，放在cookie中
           //cookie.set(key,value,作用范围)
           cookie.set("guli_token",response.data.data.token,{domain:'localhost'})
           //4.调用接口 获得用户信息 首页面显示
           loginApi.getLoginUserInfo().then(response=>{
             cookie.set("guli_ucenter",response.data.data.ugetMemberInfo,{domain:'localhost'})
             window.location.href = '/'
           })
         })
       },
       checkPhone(rule, value, callback) {
         //debugger
         if (!/^1[34578]\d{9}$/.test(value)) {
           return callback(new Error("手机号码格式不正确"));
         }
         return callback();
       },
     },
   };
   </script>
   <style>
   .el-form-item__error {
     z-index: 9999999;
   }
   </style>
   ```

   

6. 创建api 定义接口方法

   register.js

   ```js
   import request from '@/utils/request'
   export default {
       //根据手机号码发送短信
       sendCode(phone) {
           return request({
               url: `/edumsm/msm/send/${phone}`,
               method: 'get'
           })
       },
       //用户注册
       registerMember(formItem) {
           return request({
               url: `/educenter/member/register`,
               method: 'post',
               data: formItem
           })
       }
   }
   ```

   login.js

   ```js
   import request from '@/utils/request'
   export default {
       //登陆方法
       submitLogin(userInfo) {
           return request({
               url: `/educenter/member/login`,
               method: 'post',
               data:userInfo
           })
       },
       //用户信息
       getLoginUserInfo() {
           return request({
               url: `/educenter/member/getMemberInfo`,
               method: 'get',
           })
       }
   }
   ```

7. request添加拦截器

   > 添加头信息

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

8. 首页改造 default.vue

   > - 判断登陆状态进行分别显示
   > - 添加登录信息和退出方法

   ```html
   <template>
     <div class="in-wrap">
       <!-- 公共头引入 -->
       <header id="header">
         <section class="container">
           <h1 id="logo">
             <a href="#" title="谷粒学院">
               <img src="~/assets/img/logo.png" width="100%" alt="谷粒学院" />
             </a>
           </h1>
           <div class="h-r-nsl">
             <ul class="nav">
               <router-link to="/" tag="li" active-class="current" exact>
                 <a>首页</a>
               </router-link>
               <router-link to="/course" tag="li" active-class="current">
                 <a>课程</a>
               </router-link>
               <router-link to="/teacher" tag="li" active-class="current">
                 <a>名师</a>
               </router-link>
               <router-link to="/article" tag="li" active-class="current">
                 <a>文章</a>
               </router-link>
               <router-link to="/qa" tag="li" active-class="current">
                 <a>问答</a>
               </router-link>
             </ul>
             <!-- / nav -->
             <ul class="h-r-login">
               <li v-if="!loginInfo.id" id="no-login">
                 <a href="/login" title="登录">
                   <em class="icon18 login-icon">&nbsp;</em>
                   <span class="vam ml5">登录</span>
                 </a>              |
                 <a href="/register" title="注册">
                   <span class="vam ml5">注册</span>
                 </a>
               </li>
               <li v-if="loginInfo.id" id="is-login-one" class="mr10">
                 <a id="headerMsgCountId" href="#" title="消息">
                   <em class="icon18 news-icon">&nbsp;</em>
                 </a>
                 <q class="red-point" style="display: none">&nbsp;</q>
               </li>
               <li v-if="loginInfo.id" id="is-login-two" class="h-r-user">
                 <a href="/ucenter" title>
                   <img
                     :src="loginInfo.avatar"
                     width="30"
                     height="30"
                     class="vam picImg"
                     alt
                   />
                   <span id="userName" class="vam disIb">{{
                     loginInfo.nickname
                   }}</span>
                 </a>
                 <a
                   href="javascript:void(0);"
                   title="退出"
                   @click="logout()"
                   class="ml5"
                   >退 出</a
                 >
               </li>
               <!-- /未登录显示第1 li；登录后显示第2，3 li -->
             </ul>
             <aside class="h-r-search">
               <form action="#" method="post">
                 <label class="h-r-s-box">
                   <input
                     type="text"
                     placeholder="输入你想学的课程"
                     name="queryCourse.courseName"
                     value
                   />
                   <button type="submit" class="s-btn">
                     <em class="icon18">&nbsp;</em>
                   </button>
                 </label>
               </form>
             </aside>
           </div>
           <aside class="mw-nav-btn">
             <div class="mw-nav-icon"></div>
           </aside>
           <div class="clear"></div>
         </section>
       </header>
       <!-- /公共头引入 -->
   
       <nuxt />
   
       <!-- 公共底引入 -->
       <footer id="footer">
         <section class="container">
           <div class>
             <h4 class="hLh30">
               <span class="fsize18 f-fM c-999">友情链接</span>
             </h4>
             <ul class="of flink-list">
               <li>
                 <a href="http://www.atguigu.com/" title="尚硅谷" target="_blank"
                   >尚硅谷</a
                 >
               </li>
             </ul>
             <div class="clear"></div>
           </div>
           <div class="b-foot">
             <section class="fl col-7">
               <section class="mr20">
                 <section class="b-f-link">
                   <a href="#" title="关于我们" target="_blank">关于我们</a>|
                   <a href="#" title="联系我们" target="_blank">联系我们</a>|
                   <a href="#" title="帮助中心" target="_blank">帮助中心</a>|
                   <a href="#" title="资源下载" target="_blank">资源下载</a>|
                   <span>服务热线：010-56253825(北京) 0755-85293825(深圳)</span>
                   <span>Email：info@atguigu.com</span>
                 </section>
                 <section class="b-f-link mt10">
                   <span>©2018课程版权均归谷粒学院所有 京ICP备17055252号</span>
                 </section>
               </section>
             </section>
             <aside class="fl col-3 tac mt15">
               <section class="gf-tx">
                 <span>
                   <img src="~/assets/img/wx-icon.png" alt />
                 </span>
               </section>
               <section class="gf-tx">
                 <span>
                   <img src="~/assets/img/wb-icon.png" alt />
                 </span>
               </section>
             </aside>
             <div class="clear"></div>
           </div>
         </section>
       </footer>
       <!-- /公共底引入 -->
     </div>
   </template>
   <script>
   import "~/assets/css/reset.css";
   import "~/assets/css/theme.css";
   import "~/assets/css/global.css";
   import "~/assets/css/web.css";
   import cookie from "js-cookie";
   export default {
     data() {
       return {
         token: "",
         loginInfo: {
           id: "",
           age: "",
           avatar: "",
           mobile: "",
           nickname: "",
           sex: "",
         },
       };
     },
     created() {
       this.showInfo();
     },
     methods: {
       showInfo() {
         //debugger
         var jsonStr = cookie.get("guli_ucenter");
         //alert(jsonStr)
         if (jsonStr) {
           this.loginInfo = JSON.parse(jsonStr);
         }
       },
       logout() {
         //debugger
         cookie.set("guli_ucenter", "", { domain: "localhost" });
         cookie.set("guli_token", "", { domain: "localhost" });
         //跳转页面
         window.location.href = "/";
       },
     },
   };
   </script>
   ```

   

