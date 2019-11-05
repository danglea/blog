---
title: 密码加密和微服务鉴权JWT
date: 2019-10-18 15:05:46
tags: [springSecurity,安全框架]
---

***简介：*** Spring  Security是一个能够为基于Spring的企业应用系统提供声明式的安全访问控制解决方案的安全框架。它提供了一组可以在Spring应用上下文中配置的Bean，充分利用了Spring IoC，DI（控制反转Inversion of Control  ,DI:Dependency Injection  依赖注入）和AOP（面向切面编程）功能，为应用系统提供声明式的安全访问控制功能，减少了为企业系统安全控制编写大量重复代码的工作。 
<!--more-->
##1.BCrypt密码加密

###### （1）工程中引入jar

```java
<dependency> 
<groupId>org.springframework.boot</groupId> 
<artifactId>spring‐boot‐starter‐security</artifactId> 
</dependency>
```

###### （2）添加配置类

我们在添加了spring security依赖后，所有的地址都被spring security所控制了，我们目 
前只是需要用到BCrypt密码加密的部分，所以我们要添加一个配置类，配置为所有地址 
都可以匿名访问。

```java
/*** 安全配置类 */
@Configuration 
@EnableWebSecurity 
public class WebSecurityConfig extends WebSecurityConfigurerAdapter{
    @Override 
    protected void configure(HttpSecurity http) throws Exception { 
        http 
            .authorizeRequests() //表示开始需要的权限认证
            .antMatchers("/**").permitAll() //拦截路径和需要的权限			                           .anyRequest().authenticated()  //任何请求和授权                               
            .and().csrf().disable();//固定写法，拦截csrf
    }
}
```

修改工程的Application, 配置bean 

```java
@Bean 
public BCryptPasswordEncoder bcryptPasswordEncoder(){ 
    return new BCryptPasswordEncoder();
}
```

###### （3）密码加密

```java
@Autowired
BCryptPasswordEncoder encoder; 
public void add(Admin admin) { 
    //主键值
    admin.setId(idWorker.nextId()+""); 
    //密码加密
    String newpassword = encoder.encode(admin.getPassword());
    //加密后的密码 
    admin.setPassword(newpassword); 
    //保存到数据库
    adminDao.save(admin);
}
```

###### （4）登录时密码校验

```java
/*** 根据登陆名和密码查询 
* @param loginname
* @param password 
* @return */ 
public Admin findByLoginnameAndPassword(String loginname, String password){
    Admin admin = adminDao.findByLoginname(loginname); 
    if( admin!=null && encoder.matches(password,admin.getPassword())) 
    {
        return admin; //登陆成功
    }else{
        return null;//登陆失败
    } 
}
```

数据库中的密码为以下形式 

```java
$2a$10$a/EYRjdKwQ6zjr0/HJ6RR.rcA1dwv1ys7Uso1xShUaBWlIWTyJl5S
```

##  2.基于**JWT**的**Token**认证机制实现 

​    JSON Web Token（JWT）是一个非常轻巧的规范。这个规范允许我们使用JWT在用 
户和服务器之间传递安全可靠的信息。 

###### （1）JWT组成

​	一个JWT实际上就是一个字符串，它由三部分组成，头部、载荷与签名。
​	jwt的第三部分是一个签证信息，这个签证信息由三部分组成： 

```java
header(base64后)
payload(base64后)
secret(加盐)
```

​	这个部分需要base64加密后的header和base64加密后的payload使用.连接组成的字符 
串，然后通过header中声明的加密方式进行加盐secret组合加密，然后就构成了jwt的第 
三部分。 

###### （2） JJWT快速入门

```java
<dependency> 
    <groupId>io.jsonwebtoken</groupId>
    <artifactId>jjwt</artifactId> 
    <version>0.6.0</version>
</dependency>
```

创建类CreateJwtTest，用于生成token 

```java
public class CreateJwtTest {
    public static void main(String[] args) {
        //为了方便测试，我们将过期时间设置为1分钟 
        long now = System.currentTimeMillis();
        //当前时间 
        long exp = now + 1000*60;        //过期时间为1分钟 
        JwtBuilder builder= Jwts.builder().setId("888") //载荷  jwts类名
            .setSubject("小白") //个人信息
            .setIssuedAt(new Date()) //设置登陆时间
            .signWith(SignatureAlgorithm.HS256,"itcast")//加盐 
             .setExpiration(new Date(exp))//设置过期时间
            .claim("roles","admin") //自定义属性
            .claim("logo","logo.png");
        System.out.println( builder.compact() );
    } 
}
```

token解析

```java
public class ParseJwtTest {
    public static void main(String[] args) {
        String token="eyJhbGciOiJIUzI1NiJ9.eyJqdGkiOiI4ODgiLCJzdWIiOiLlsI_nmb0iLCJpYXQiO jE1MjM0MTM0NTh9.gq0J‐cOM_qCNqU_s‐d_IrRytaNenesPmqAIhQpYXHZk"; 
        Claims claims = Jwts.parser().setSigningKey("itcast")
            .parseClaimsJws(token).getBody();
        System.out.println("id:"+claims.getId()); 
        System.out.println("subject:"+claims.getSubject()); 
        System.out.println("IssuedAt:"+claims.getIssuedAt()); 
        System.out.println("roles:"+claims.get("roles")); 
        System.out.println("logo:"+claims.get("logo"));
        SimpleDateFormat sdf=new SimpleDateFormat("yyyy‐MM‐dd hh:mm:ss"); 
        System.out.println("签发时间:"+sdf.format(claims.getIssuedAt()));
        System.out.println("过期时 间:"+sdf.format(claims.getExpiration())); 
        System.out.println("当前时间:"+sdf.format(new Date()) );
    }
}
```

综上所述，自己可以编写一个工具类。

###### （3）使用拦截器方式实现**token**鉴权 

创建拦截器JWtFilter

```java
@Component
public class JwtFilter extends HandlerInterceptorAdapter { 
    @Autowired private JwtUtil jwtUtil;
    @Override 
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        System.out.println("经过了拦截器"); 
        final String authHeader = request.getHeader("Authorization");
        if (authHeader != null && authHeader.startsWith("Bearer ")) {
            final String token = authHeader.substring(7); // The part after "Bearer " 
            Claims claims = jwtUtil.parseJWT(token);
            if (claims != null) { 
                if("admin".equals(claims.get("roles"))){
                //如果是管理员
                request.setAttribute("admin_claims", claims); 
            }if("user".equals(claims.get("roles"))){
                //如果是用户 
                request.setAttribute("user_claims", claims); 
            }
                                } 
        }
        return true; 
    } 
}
```

配置拦截器

```java
@Configuration 
public class ApplicationConfig extends WebMvcConfigurationSupport { 
    @Autowired 
    private JwtFilter jwtFilter; 
    @Override
    public void addInterceptors(InterceptorRegistry registry) {        
        registry.addInterceptor(jwtFilter).
        addPathPatterns("/**").
        excludePathPatterns("/**/login");
                                                              }
}
```

我们通过session就可以判断用户的权限，同时也可以防止伪造请求。