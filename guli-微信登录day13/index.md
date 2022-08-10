# guli-微信登录(day13)


# guli-微信登录(day13)

## 一.微信扫描登录

1. [注册开发者资质](https://open.weixin.qq.com/)

   - 申请网站应用名称
   - 需要域名地址

2. service_ucenter/utils创建类 配置微信id，密匙和域名地址

   ```java
   package com.guli.educenter.utils;
   
   @Component
   public class ConstantWxUtils implements InitializingBean {
   
       @Value("${wx.open.app_id}")
       private String appId;
   
       @Value("${wx.open.app_secret}")
       private String appSecret;
   
       @Value("${wx.open.redirect_url}")
       private String redirectUrl;
   
       public static String WX_OPEN_APP_ID;
       public static String WX_OPEN_APP_SECRET;
       public static String WX_OPEN_REDIRECT_URL;
   
       @Override
       public void afterPropertiesSet() throws Exception {
           WX_OPEN_APP_ID = appId;
           WX_OPEN_APP_SECRET = appSecret;
           WX_OPEN_REDIRECT_URL = redirectUrl;
       }
   }
   ```

3. 创建WxApiController

   > 因为需要直接请求网址不是返回json所以注释变为@Controller
   >
   > return "redirect:"+url; 重定向

   ```java
   package com.guli.educenter.controller;
   
   @CrossOrigin
   @RequestMapping("/api/ucenter/wx")
   @Controller
   public class WxApiController {
       //1.生成微信扫描二维码
       @GetMapping("login")
       public String getWxCode() {
           //固定地址 拼接字符串
   //        String url = "https://open.weixin.qq.com/connect/qrconnect?appid=";
           String baseUrl = "https://open.weixin.qq.com/connect/qrconnect" +
                   "?appid=%s" +
                   "&redirect_uri=%s" +
                   "&response_type=code" +
                   "&scope=snsapi_login" +
                   "&state=%s" +
                   "#wechat_redirect";
           //对redirect_uri进行编码
           String redirectUrl = ConstantWxUtils.WX_OPEN_REDIRECT_URL;
           try {
               redirectUrl = URLEncoder.encode(redirectUrl, "utf-8");
           } catch (UnsupportedEncodingException e) {
               e.printStackTrace();
           }
           //设置%s中的值
           String url = String.format(
                   baseUrl,
                   ConstantWxUtils.WX_OPEN_APP_ID,
                   redirectUrl,
                   "guli");
           //请求微信地址
           return "redirect:"+url;
       }
   }
   
   ```

## 二.扫描后获取扫描人信息

1. controller

   > 需要使用HttpClientUtils工具类模拟浏览器去请求腾讯数据
   >
   > ```java
   > package com.guli.educenter.utils;
   > 
   > import org.apache.commons.io.IOUtils;
   > import org.apache.commons.lang.StringUtils;
   > import org.apache.http.Consts;
   > import org.apache.http.HttpEntity;
   > import org.apache.http.HttpResponse;
   > import org.apache.http.NameValuePair;
   > import org.apache.http.client.HttpClient;
   > import org.apache.http.client.config.RequestConfig;
   > import org.apache.http.client.config.RequestConfig.Builder;
   > import org.apache.http.client.entity.UrlEncodedFormEntity;
   > import org.apache.http.client.methods.HttpGet;
   > import org.apache.http.client.methods.HttpPost;
   > import org.apache.http.conn.ConnectTimeoutException;
   > import org.apache.http.conn.ssl.SSLConnectionSocketFactory;
   > import org.apache.http.conn.ssl.SSLContextBuilder;
   > import org.apache.http.conn.ssl.TrustStrategy;
   > import org.apache.http.conn.ssl.X509HostnameVerifier;
   > import org.apache.http.entity.ContentType;
   > import org.apache.http.entity.StringEntity;
   > import org.apache.http.impl.client.CloseableHttpClient;
   > import org.apache.http.impl.client.HttpClients;
   > import org.apache.http.impl.conn.PoolingHttpClientConnectionManager;
   > import org.apache.http.message.BasicNameValuePair;
   > 
   > import javax.net.ssl.SSLContext;
   > import javax.net.ssl.SSLException;
   > import javax.net.ssl.SSLSession;
   > import javax.net.ssl.SSLSocket;
   > import java.io.IOException;
   > import java.net.SocketTimeoutException;
   > import java.security.GeneralSecurityException;
   > import java.security.cert.CertificateException;
   > import java.security.cert.X509Certificate;
   > import java.util.ArrayList;
   > import java.util.List;
   > import java.util.Map;
   > import java.util.Map.Entry;
   > import java.util.Set;
   > 
   > /**
   >  *  依赖的jar包有：commons-lang-2.6.jar、httpclient-4.3.2.jar、httpcore-4.3.1.jar、commons-io-2.4.jar
   >  * @author zhaoyb
   >  *
   >  */
   > public class HttpClientUtils {
   > 
   > 	public static final int connTimeout=10000;
   > 	public static final int readTimeout=10000;
   > 	public static final String charset="UTF-8";
   > 	private static HttpClient client = null;
   > 
   > 	static {
   > 		PoolingHttpClientConnectionManager cm = new PoolingHttpClientConnectionManager();
   > 		cm.setMaxTotal(128);
   > 		cm.setDefaultMaxPerRoute(128);
   > 		client = HttpClients.custom().setConnectionManager(cm).build();
   > 	}
   > 
   > 	public static String postParameters(String url, String parameterStr) throws ConnectTimeoutException, SocketTimeoutException, Exception{
   > 		return post(url,parameterStr,"application/x-www-form-urlencoded",charset,connTimeout,readTimeout);
   > 	}
   > 
   > 	public static String postParameters(String url, String parameterStr,String charset, Integer connTimeout, Integer readTimeout) throws ConnectTimeoutException, SocketTimeoutException, Exception{
   > 		return post(url,parameterStr,"application/x-www-form-urlencoded",charset,connTimeout,readTimeout);
   > 	}
   > 
   > 	public static String postParameters(String url, Map<String, String> params) throws ConnectTimeoutException,
   > 			SocketTimeoutException, Exception {
   > 		return postForm(url, params, null, connTimeout, readTimeout);
   > 	}
   > 
   > 	public static String postParameters(String url, Map<String, String> params, Integer connTimeout,Integer readTimeout) throws ConnectTimeoutException,
   > 			SocketTimeoutException, Exception {
   > 		return postForm(url, params, null, connTimeout, readTimeout);
   > 	}
   > 
   > 	public static String get(String url) throws Exception {
   > 		return get(url, charset, null, null);
   > 	}
   > 
   > 	public static String get(String url, String charset) throws Exception {
   > 		return get(url, charset, connTimeout, readTimeout);
   > 	}
   > 
   > 	/**
   > 	 * 发送一个 Post 请求, 使用指定的字符集编码.
   > 	 *
   > 	 * @param url
   > 	 * @param body RequestBody
   > 	 * @param mimeType 例如 application/xml "application/x-www-form-urlencoded" a=1&b=2&c=3
   > 	 * @param charset 编码
   > 	 * @param connTimeout 建立链接超时时间,毫秒.
   > 	 * @param readTimeout 响应超时时间,毫秒.
   > 	 * @return ResponseBody, 使用指定的字符集编码.
   > 	 * @throws ConnectTimeoutException 建立链接超时异常
   > 	 * @throws SocketTimeoutException  响应超时
   > 	 * @throws Exception
   > 	 */
   > 	public static String post(String url, String body, String mimeType,String charset, Integer connTimeout, Integer readTimeout)
   > 			throws ConnectTimeoutException, SocketTimeoutException, Exception {
   > 		HttpClient client = null;
   > 		HttpPost post = new HttpPost(url);
   > 		String result = "";
   > 		try {
   > 			if (StringUtils.isNotBlank(body)) {
   > 				HttpEntity entity = new StringEntity(body, ContentType.create(mimeType, charset));
   > 				post.setEntity(entity);
   > 			}
   > 			// 设置参数
   > 			Builder customReqConf = RequestConfig.custom();
   > 			if (connTimeout != null) {
   > 				customReqConf.setConnectTimeout(connTimeout);
   > 			}
   > 			if (readTimeout != null) {
   > 				customReqConf.setSocketTimeout(readTimeout);
   > 			}
   > 			post.setConfig(customReqConf.build());
   > 
   > 			HttpResponse res;
   > 			if (url.startsWith("https")) {
   > 				// 执行 Https 请求.
   > 				client = createSSLInsecureClient();
   > 				res = client.execute(post);
   > 			} else {
   > 				// 执行 Http 请求.
   > 				client = HttpClientUtils.client;
   > 				res = client.execute(post);
   > 			}
   > 			result = IOUtils.toString(res.getEntity().getContent(), charset);
   > 		} finally {
   > 			post.releaseConnection();
   > 			if (url.startsWith("https") && client != null&& client instanceof CloseableHttpClient) {
   > 				((CloseableHttpClient) client).close();
   > 			}
   > 		}
   > 		return result;
   > 	}
   > 
   > 
   > 	/**
   > 	 * 提交form表单
   > 	 *
   > 	 * @param url
   > 	 * @param params
   > 	 * @param connTimeout
   > 	 * @param readTimeout
   > 	 * @return
   > 	 * @throws ConnectTimeoutException
   > 	 * @throws SocketTimeoutException
   > 	 * @throws Exception
   > 	 */
   > 	public static String postForm(String url, Map<String, String> params, Map<String, String> headers, Integer connTimeout,Integer readTimeout) throws ConnectTimeoutException,
   > 			SocketTimeoutException, Exception {
   > 
   > 		HttpClient client = null;
   > 		HttpPost post = new HttpPost(url);
   > 		try {
   > 			if (params != null && !params.isEmpty()) {
   > 				List<NameValuePair> formParams = new ArrayList<NameValuePair>();
   > 				Set<Entry<String, String>> entrySet = params.entrySet();
   > 				for (Entry<String, String> entry : entrySet) {
   > 					formParams.add(new BasicNameValuePair(entry.getKey(), entry.getValue()));
   > 				}
   > 				UrlEncodedFormEntity entity = new UrlEncodedFormEntity(formParams, Consts.UTF_8);
   > 				post.setEntity(entity);
   > 			}
   > 
   > 			if (headers != null && !headers.isEmpty()) {
   > 				for (Entry<String, String> entry : headers.entrySet()) {
   > 					post.addHeader(entry.getKey(), entry.getValue());
   > 				}
   > 			}
   > 			// 设置参数
   > 			Builder customReqConf = RequestConfig.custom();
   > 			if (connTimeout != null) {
   > 				customReqConf.setConnectTimeout(connTimeout);
   > 			}
   > 			if (readTimeout != null) {
   > 				customReqConf.setSocketTimeout(readTimeout);
   > 			}
   > 			post.setConfig(customReqConf.build());
   > 			HttpResponse res = null;
   > 			if (url.startsWith("https")) {
   > 				// 执行 Https 请求.
   > 				client = createSSLInsecureClient();
   > 				res = client.execute(post);
   > 			} else {
   > 				// 执行 Http 请求.
   > 				client = HttpClientUtils.client;
   > 				res = client.execute(post);
   > 			}
   > 			return IOUtils.toString(res.getEntity().getContent(), "UTF-8");
   > 		} finally {
   > 			post.releaseConnection();
   > 			if (url.startsWith("https") && client != null
   > 					&& client instanceof CloseableHttpClient) {
   > 				((CloseableHttpClient) client).close();
   > 			}
   > 		}
   > 	}
   > 
   > 
   > 
   > 
   > 	/**
   > 	 * 发送一个 GET 请求
   > 	 *
   > 	 * @param url
   > 	 * @param charset
   > 	 * @param connTimeout  建立链接超时时间,毫秒.
   > 	 * @param readTimeout  响应超时时间,毫秒.
   > 	 * @return
   > 	 * @throws ConnectTimeoutException   建立链接超时
   > 	 * @throws SocketTimeoutException   响应超时
   > 	 * @throws Exception
   > 	 */
   > 	public static String get(String url, String charset, Integer connTimeout,Integer readTimeout)
   > 			throws ConnectTimeoutException,SocketTimeoutException, Exception {
   > 
   > 		HttpClient client = null;
   > 		HttpGet get = new HttpGet(url);
   > 		String result = "";
   > 		try {
   > 			// 设置参数
   > 			Builder customReqConf = RequestConfig.custom();
   > 			if (connTimeout != null) {
   > 				customReqConf.setConnectTimeout(connTimeout);
   > 			}
   > 			if (readTimeout != null) {
   > 				customReqConf.setSocketTimeout(readTimeout);
   > 			}
   > 			get.setConfig(customReqConf.build());
   > 
   > 			HttpResponse res = null;
   > 
   > 			if (url.startsWith("https")) {
   > 				// 执行 Https 请求.
   > 				client = createSSLInsecureClient();
   > 				res = client.execute(get);
   > 			} else {
   > 				// 执行 Http 请求.
   > 				client = HttpClientUtils.client;
   > 				res = client.execute(get);
   > 			}
   > 
   > 			result = IOUtils.toString(res.getEntity().getContent(), charset);
   > 		} finally {
   > 			get.releaseConnection();
   > 			if (url.startsWith("https") && client != null && client instanceof CloseableHttpClient) {
   > 				((CloseableHttpClient) client).close();
   > 			}
   > 		}
   > 		return result;
   > 	}
   > 
   > 
   > 	/**
   > 	 * 从 response 里获取 charset
   > 	 *
   > 	 * @param ressponse
   > 	 * @return
   > 	 */
   > 	@SuppressWarnings("unused")
   > 	private static String getCharsetFromResponse(HttpResponse ressponse) {
   > 		// Content-Type:text/html; charset=GBK
   > 		if (ressponse.getEntity() != null  && ressponse.getEntity().getContentType() != null && ressponse.getEntity().getContentType().getValue() != null) {
   > 			String contentType = ressponse.getEntity().getContentType().getValue();
   > 			if (contentType.contains("charset=")) {
   > 				return contentType.substring(contentType.indexOf("charset=") + 8);
   > 			}
   > 		}
   > 		return null;
   > 	}
   > 
   > 
   > 
   > 	/**
   > 	 * 创建 SSL连接
   > 	 * @return
   > 	 * @throws GeneralSecurityException
   > 	 */
   > 	private static CloseableHttpClient createSSLInsecureClient() throws GeneralSecurityException {
   > 		try {
   > 			SSLContext sslContext = new SSLContextBuilder().loadTrustMaterial(null, new TrustStrategy() {
   > 				public boolean isTrusted(X509Certificate[] chain,String authType) throws CertificateException {
   > 					return true;
   > 				}
   > 			}).build();
   > 
   > 			SSLConnectionSocketFactory sslsf = new SSLConnectionSocketFactory(sslContext, new X509HostnameVerifier() {
   > 
   > 				@Override
   > 				public boolean verify(String arg0, SSLSession arg1) {
   > 					return true;
   > 				}
   > 
   > 				@Override
   > 				public void verify(String host, SSLSocket ssl)
   > 						throws IOException {
   > 				}
   > 
   > 				@Override
   > 				public void verify(String host, X509Certificate cert)
   > 						throws SSLException {
   > 				}
   > 
   > 				@Override
   > 				public void verify(String host, String[] cns,
   > 								   String[] subjectAlts) throws SSLException {
   > 				}
   > 
   > 			});
   > 
   > 			return HttpClients.custom().setSSLSocketFactory(sslsf).build();
   > 
   > 		} catch (GeneralSecurityException e) {
   > 			throw e;
   > 		}
   > 	}
   > 
   > 	public static void main(String[] args) {
   > 		try {
   > 			String str= post("https://localhost:443/ssl/test.shtml","name=12&page=34","application/x-www-form-urlencoded", "UTF-8", 10000, 10000);
   > 			//String str= get("https://localhost:443/ssl/test.shtml?name=12&page=34","GBK");
   >             /*Map<String,String> map = new HashMap<String,String>();
   >             map.put("name", "111");
   >             map.put("page", "222");
   >             String str= postForm("https://localhost:443/ssl/test.shtml",map,null, 10000, 10000);*/
   > 			System.out.println(str);
   > 		} catch (ConnectTimeoutException e) {
   > 			// TODO Auto-generated catch block
   > 			e.printStackTrace();
   > 		} catch (SocketTimeoutException e) {
   > 			// TODO Auto-generated catch block
   > 			e.printStackTrace();
   > 		} catch (Exception e) {
   > 			// TODO Auto-generated catch block
   > 			e.printStackTrace();
   > 		}
   > 	}
   > 
   > }
   > ```
   >
   >  //1.获取到code值，临时票据，类似于验证码
   >
   > //2.拿着code请求 微信固定的地址，得到两个值 accsess_token 和openid
   >
   > baseAccessTokenUrl =
   >                     "https://api.weixin.qq.com/sns/oauth2/access_token" +
   >                             "?appid=%s" +
   >                             "&secret=%s" +
   >                             "&code=%s" +
   >                             "&grant_type=authorization_code";
   >
   > //3.使用access_token和openid，再去请求微信提供的固定的地址，获取到扫码人的信息
   >
   > baseUserInfoUrl = "https://api.weixin.qq.com/sns/userinfo" +
   >                         "?access_token=%s" +
   >                         "&openid=%s";
   >
   >  
   >
   >  
   >
   > 避免因分布式导致cookie丢失 使用jwt生成token传到前端 前端通过token进行数据查询

   ```java
   package com.guli.educenter.controller;
   
   @CrossOrigin
   @RequestMapping("/api/ucenter/wx")
   @Controller
   public class WxApiController {
       @Autowired
       private UcenterMemberService memberService;
       //2.获取扫描人信息，添加数据
       @GetMapping("callback")
       public String callback(String code,String state) {
           try {
               //1.获取到code值，临时票据，类似于验证码
               //2.拿着code请求 微信固定的地址，得到两个值 accsess_token 和openid
               String baseAccessTokenUrl =
                       "https://api.weixin.qq.com/sns/oauth2/access_token" +
                               "?appid=%s" +
                               "&secret=%s" +
                               "&code=%s" +
                               "&grant_type=authorization_code";
               String accessTokenUrl = String.format(
                       baseAccessTokenUrl,
                       ConstantWxUtils.WX_OPEN_APP_ID,
                       ConstantWxUtils.WX_OPEN_APP_SECRET,
                       code
               );
               //使用httpclient发送请求，得到返回结果
               //{
               //"access_token":"37_xWo_q06bqu32N8KFks79IpE77A0_QrZl6GIMNIDQW-uu1Lvtv-LHbTum8j-fiTjUFO0VlVj2viv8sZsggx6QvKNcDj3YDoSQ9PtJrriq628",
               //"expires_in":7200,
               //"refresh_token":"37_hTRAxcOJOKi8aDYEDUVonwpl5K8Xn0HL_82WEfvi0d4KGqjiF9B0IEd6O9XNG9pBC-aRd5B9zBuhB5GFOLCEjbuLaCWuK-wT1ZuNaLXb6Pg",
               //"openid":"o3_SC56gLo5jX3MDKK33y_CR-u5E",
               //"scope":"snsapi_login",
               //"unionid":"oWgGz1DbXmO9nZNN5RNixRZ0yshM"
               //}
               String accessTokenInfo = HttpClientUtils.get(accessTokenUrl);
               //需要从 accessTokenInfo 字符串中获取access_token，openid
               //解决：吧accessTokenInfo字符串转换成map集合，根据map的key获取对应值
               //使用josn转换工具 gson
               Gson gson = new Gson();
               HashMap mapAccessToken = gson.fromJson(accessTokenInfo, HashMap.class);
               String access_token = (String) mapAccessToken.get("access_token");
               String openid = (String) mapAccessToken.get("openid");
   
               //把扫码人信息加到数据库
               //判断数据库中有没有相同的微信信息
               UcenterMember member = memberService.getOpenIdMember(openid);
               if (member == null) {//没有相同的微信数据，进行添加
   
                   //3.使用access_token和openid，再去请求微信提供的固定的地址，获取到扫码人的信息
                   String baseUserInfoUrl = "https://api.weixin.qq.com/sns/userinfo" +
                           "?access_token=%s" +
                           "&openid=%s";
                   //拼接两个参数
                   String userInfoUrl = String.format(
                           baseUserInfoUrl,
                           access_token,
                           openid
                   );
                   //{
                   // "openid":"o3_SC56gLo5jX3MDKK33y_CR-u5E",
                   // "nickname":"熙熙攘攘",
                   // "sex":1,
                   // "language":"zh_CN",
                   // "city":"Linfen",
                   // "province":"Shanxi",
                   // "country":"CN",
                   // "headimgurl":"https:\/\/thirdwx.qlogo.cn\/mmopen\/vi_32\/RDQPia8q0N5C5n3ygkkfUrKhHS78xTr59VBAjLsexEC0BMRa7lbK2LhG9A5AA4ptFh3Fx0iclvHPWEMt25DLiaLeQ\/132",
                   // "privilege":[],
                   // "unionid":"oWgGz1DbXmO9nZNN5RNixRZ0yshM"
                   // }
                   String userInfo = HttpClientUtils.get(userInfoUrl);
                   HashMap userInfoMap = gson.fromJson(userInfo, HashMap.class);
                   String nickname = (String) userInfoMap.get("nickname");
                   String headimgurl = (String) userInfoMap.get("headimgurl");
   
                   member = new UcenterMember();
                   member.setOpenid(openid);
                   member.setNickname(nickname);
                   member.setAvatar(headimgurl);
                   memberService.save(member);
               }
               //使用jwt根据member对象生成token字符串
               String jwtToken = JwtUtils.getJwtToken(member.getId(), member.getNickname());
               //最后：返回首页面，通过路径传递token字符串
               return "redirect:http://localhost:3000?token="+jwtToken;
           } catch (Exception e) {
               throw new GuliException(20001,"登陆失败");
           }
       }
       //1.生成微信扫描二维码
       @GetMapping("login")
       public String getWxCode() {
           //固定地址 拼接字符串
   //        String url = "https://open.weixin.qq.com/connect/qrconnect?appid=";
           String baseUrl = "https://open.weixin.qq.com/connect/qrconnect" +
                   "?appid=%s" +
                   "&redirect_uri=%s" +
                   "&response_type=code" +
                   "&scope=snsapi_login" +
                   "&state=%s" +
                   "#wechat_redirect";
           //对redirect_uri进行编码
           String redirectUrl = ConstantWxUtils.WX_OPEN_REDIRECT_URL;
           try {
               redirectUrl = URLEncoder.encode(redirectUrl, "utf-8");
           } catch (UnsupportedEncodingException e) {
               e.printStackTrace();
           }
           //设置%s中的值
           String url = String.format(
                   baseUrl,
                   ConstantWxUtils.WX_OPEN_APP_ID,
                   redirectUrl,
                   "guli");
           //请求微信地址
           return "redirect:"+url;
       }
   }
   
   ```

## 三.前端数据显示

> 避免因分布式导致cookie丢失 使用jwt生成token传到前端 前端通过token进行数据查询

default.vue

```js
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
    //获取路径中的值
    this.token = this.$route.query.token;
    if (this.token) {
      this.wxLogin();
    }
    this.showInfo();
  },
  methods: {
    //微信登录显示的方法
    wxLogin() {
      //把token值放到cookie里面
      cookie.set('guli_token',this.token,{domain: 'localhost'})
      //调用接口，根据token值获取用户信息
      loginApi.getLoginUserInfo()
        .then(response => {
           cookie.set('guli_ucenter',this.loginInfo,{domain: 'localhost'})
        })
    },
    //从cookie获取用户信息
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
```




