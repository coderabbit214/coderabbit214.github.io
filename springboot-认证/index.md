# Springboot-认证


# Springboot-权限控制

1. mvn

   ```xml
           <dependency>
               <groupId>org.springframework.boot</groupId>
               <artifactId>spring-boot-starter-security</artifactId>
           </dependency>
   ```

2. 编写security配置类

```java
package com.jsh.security.config;


import org.springframework.security.config.annotation.authentication.builders.AuthenticationManagerBuilder;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.annotation.web.configuration.EnableWebSecurity;
import org.springframework.security.config.annotation.web.configuration.WebSecurityConfigurerAdapter;
import org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder;

@EnableWebSecurity
public class MySecurityConfig extends WebSecurityConfigurerAdapter {
    @Override
    protected void configure(HttpSecurity http) throws Exception {
//        super.configure(http);
        //定制请求授权的规则
        http.authorizeRequests().antMatchers("/").permitAll()
                .antMatchers("/level1/**").hasAnyRole("VIP1")
                .antMatchers("/level2/**").hasAnyRole("VIP2")
                .antMatchers("/level3/**").hasAnyRole("VIP3");
        //开启自动配置的登录功能
        /**
        <h2 align="center">游客您好，如果想查看武林秘籍 <a th:href="@{/userlogin}">请登录</a></h2>
        */
        http.formLogin()
                .usernameParameter("user").passwordParameter("pwd")//定制参数name
                .loginPage("/userlogin");//定制登陆页面
        /**
        		验证请求
        		<form th:action="@{/userlogin}" method="post">
					用户名:<input name="user"/><br>
					密码:<input name="pwd"><br/>
					<input type="checkbox" name="remeber">记住我<br/>
					<input type="submit" value="登陆">
				</form>	
        */
        //1.成功："/login"
        //2.失败："/login?error"

        //开启自动配置的注销功能
        http.logout().logoutSuccessUrl("/");//注销成功后来到首页
        //1.访问/logout表示用户注销 ，清空session
        //2.注销成功 返回/login?logout 页面
        /**
        <form th:action="@{/logout}" method="post">
			<input type="submit" value="注销">
		</form>
        */

        //开启记住我功能
        http.rememberMe().rememberMeParameter("remeber");
        //登陆成功后，将cookie发送给浏览器保存， 注销后会删除
    }
    //定义认证规则
    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
//        super.configure(auth);
        auth.inMemoryAuthentication()
                .passwordEncoder(new BCryptPasswordEncoder()).withUser("张三").password(new BCryptPasswordEncoder().encode("123456")).roles("VIP1","VIP2")
                .and()
                .passwordEncoder(new BCryptPasswordEncoder()).withUser("李四").password(new BCryptPasswordEncoder().encode("123456")).roles("VIP3","VIP2");
    }
}

```


