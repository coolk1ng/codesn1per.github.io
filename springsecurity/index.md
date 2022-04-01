# SpringSecurity


### 1.过滤器

**security本质是一个过滤器链**

1. FilterSecurityInterceptor:是一个方法级的权限过滤器,基本位于过滤器链的最底部;

2. ExceptionTranslationFilter:是一个异常过滤器,用来处理在认证授权过程中抛出的异常;

3. UserNamePasswordAuthenticationFilter:对/Login的post请求拦截,校验表单中用户名和密码;

### 2.web权限方案

#### 1.认证

1. 设置登录的用户名和密码

   1. 通过配置文件

      ``` pro
      server.port=5555
      spring.security.user.name=root
      spring.security.user.password=123456
      ```

   2. 通过配置类

      ```java
      @Configuration
      public class SecurityConfig extends WebSecurityConfigurerAdapter {
          @Bean
          PasswordEncoder passwordEncoder(){
              return new BCryptPasswordEncoder();
          }
          @Autowired
          private PasswordEncoder passwordEncoder;
          @Override
          protected void configure(AuthenticationManagerBuilder auth) throws Exception {
              String password = passwordEncoder.encode("123");
              auth.inMemoryAuthentication().withUser("root").password(password).roles("admin");
          }
      }
      
      ```

      

   3. 自定义编写实现类

      ```java
      @Configuration
      public class SecurityConfigTest extends WebSecurityConfigurerAdapter {
          @Autowired
          private UserDetailsService userDetailsService;
          @Bean
          PasswordEncoder passwordEncoder(){
              return new BCryptPasswordEncoder();
          }
      
          @Override
          protected void configure(AuthenticationManagerBuilder auth) throws Exception {
              auth.userDetailsService(userDetailsService).passwordEncoder(passwordEncoder());
          }
      }
      
      ```

      ```java
      @Service("userDetailsService")
      public class MyUserDetailService implements UserDetailsService {
          @Override
          public UserDetails loadUserByUsername(String s) throws UsernameNotFoundException {
              List<GrantedAuthority> role = AuthorityUtils.commaSeparatedStringToAuthorityList("role");
              return new User("root",new BCryptPasswordEncoder().encode("123"),role);
          }
      }
      
      ```

      **通过查询数据库校验**

      1. 导入依赖

         ```xml
          <dependency>
                     <groupId>mysql</groupId>
                     <artifactId>mysql-connector-java</artifactId>
                 </dependency>
                 <dependency>
                     <groupId>org.projectlombok</groupId>
                     <artifactId>lombok</artifactId>
                 </dependency>
                 <dependency>
                     <groupId>com.baomidou</groupId>
                     <artifactId>mybatis-plus-boot-starter</artifactId>
                     <version>3.4.3</version>
                 </dependency>
         ```

      2. 构建实体类

         ```java
         @Data
         @NoArgsConstructor
         @AllArgsConstructor
         public class Users {
             private Integer id;
             private String name;
             private String pwd;
         }
         
         ```

      3. 整合mybatis-plus,继承BaseMapper

         ```java
         @Repository
         public interface UserMapper extends BaseMapper<Users> {
         }
         
         ```

      4. 在MyUserDetailService

         ```java
         @Service("userDetailsService")
         public class MyUserDetailService implements UserDetailsService {
             @Autowired
             private UserMapper userMapper;
             @Override
             public UserDetails loadUserByUsername(String name) throws UsernameNotFoundException {
                 QueryWrapper<Users> wrapper = new QueryWrapper<>();
                 wrapper.eq("name",name);
                 Users user = userMapper.selectOne(wrapper);
                 if (user==null){
                     throw new UsernameNotFoundException("用户名不存在");
                 }
                 List<GrantedAuthority> role = AuthorityUtils.commaSeparatedStringToAuthorityList("role");
                 return new User(user.getName(),new BCryptPasswordEncoder().encode(user.getPwd()),role);
             }
         }
         
         ```

         

      **自定义登录页面和权限**

      1. ```java
             @Override
             protected void configure(HttpSecurity http) throws Exception {
                 http.formLogin() //自定义自己编写的登录页面
                         .loginPage("/login.html") //登录页面设置
                         .loginProcessingUrl("/user/login")  //登录访问路径
                         .defaultSuccessUrl("/test/index").permitAll() //登录成功之后,跳转路劲
                         .and().authorizeRequests()
                         .antMatchers("/","/test/hello","user/login").permitAll() //设置安歇路径可以放行
                         .anyRequest().authenticated()
                         .and().csrf().disable(); //关闭csrf防护
             }
         ```

   

#### 2.授权

**基于角色或权限进行访问控制**

* 第一个方法:hasAuthority():如果当前主体具有指定的权限,则返回true,否则返回false;

1. 在配置类设置当前访问地址有哪些权限

   ````java
    .antMatchers("/test/index").hasAnyAuthority("admin") //当前登录用户只有admin才能访问这个路径
   ````

2. 在UserDetailsService,把返回User对象设置权限

   ```java
    List<GrantedAuthority> role = AuthorityUtils.commaSeparatedStringToAuthorityList("admin");
   
   ```

* 第二个方法:hasAnyAuthority():如果当前的主体有任何提供的权限,则返回true,否则返回false;

  ```java
  .antMatchers("/test/index").hasAnyAuthority("admin","manage")
  ```

* 第三个方法:hasRole():给定角色进行访问

  1. 

     ```java
     .antMatchers("/test/index").hasRole("sale")
     ```

  2. UserDetailService,把返回对象设置角色

     ```java
      List<GrantedAuthority> role = AuthorityUtils.commaSeparatedStringToAuthorityList("admin,ROLE_sale");
     ```

  

* 第四个方法:hasAnyRole():具备任何一个角色就可以访问



#### 3. 自定义403页面

```java
http.exceptionHandling().accessDeniedPage("/unauth.html");
```

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
<h1>权限不够</h1>
</body>
</html>
```



####  4. 什么是CSRF

​		CSRF:跨站请求伪造;通过伪造用户请求访问受信任站点的非法请求;

​		跨域: 只要网络协议,ip地址,端口中任何一个不同就是跨域请求;

​		客户端与服务器进行交互时,由于协议本身是无状态协议,所以引入了cookie进行记录客户端身份;在cookie中会存放session id用来识别客户端身份;在跨域的情况下,seesion id可能被第三方恶意劫持,通过这个session id向服务器发起请求时,服务端会认为这个请求是合法的;

#### 5. SpringSecurity中的CSRF

​		从SpringSecurity4开始CSRF防护默认开启;默认会拦截请求;进行CSRF处理;CSRF为了保证不是其他第三方网站访问,要求访问时携带参数名为_csrf值为token的内容,如果token和服务端的token匹配成功,则正常访问;

# Oauth2认证

## 1. Oauth2简介

第三方认证技术方案最主要解决认证协议的通用标准问题,因为要实现跨系统认证,各系统之间要遵循一定的接口协议.

Oauth协议为用户资源的授权提供了安全的,开放而简易的标准.同时任何第三方都可以使用Oauth认证服务,任何服务提供商都可以实现自身的Oauth认证服务,因而Oauth是开放的.业界提供了Oauth的多种实现如PHP,JavaScript,Java等各种语言的开发包.

### 1.1 角色

**客户端**

本身不存储资源,需要通过资源拥有者的授权去请求资源服务器的资源.

**资源拥有者**

通常为用户,也可以是应用程序,即该资源的拥有者.

**授权服务器(也称认证服务器)**

用来对资源拥有的身份进行认证,对访问资源进行授权.客户端要访问资源需要通过认证服务器由资源拥有者授权后可访问.

**资源服务器**

存储资源的服务器,比如,网站用户管理服务器存储了网站的用户信息,网站相册服务器存储了用户的相册信息,微信的资源服务存储了微信的用户信息等.客户端最终访问资源服务器获取资源信息.

### 1.2. 常用术语

* 客户凭证: 仅用于授权码授权类型,用于交换获取访问令牌和刷新令牌
* 访问令牌: 用于嗲表一个用户或服务器直接去访问受保护的资源
* 刷新令牌: 用于去授权服务器获取一个刷新访问令牌
* BearerToken: 不管谁拿到Token都可以访问资源.
* Proof of Prssession Token: 可以校验client是否对Token有明确的拥有权.

### 1.3. 特点

**优点: **

* 更安全,客户端不需要接触用户密码,服务器端更易集中保护
* 广泛传播并被持续采用
* 短寿命和封装的token
* 资源服务器和授权服务器的解耦
* 集中式授权,简化客户端
* HTTP/JSON友好,易于请求和传递token
* 考虑多种客户端架构场景
* 客户可以具有不同的信任级别

**缺点**

* 协议框架太宽泛,造成各种实现的兼容性和互操作性差
* 不是一个认证协议,本身并不能告诉你任何用户信息

## 2. 授权模式

### 2.1 授权码模式(Authorization Code)

![image-20210829221743207](../../static/image-20210829221743207.png)

### 2.2 简化授权模式

![image-20210829221809041](../../static/image-20210829221809041.png)

### 2.3 密码模式

### 2.4 客户端模式

![image-20210829222019351](../../static/image-20210829222019351.png)

### 2.5 刷新令牌

![image-20210829222049642](../../static/image-20210829222049642.png)

# Spring Security Oauth2

## 1.1 授权服务器

![image-20210829222318609](../../static/image-20210829222318609.png)

* Authorize Endpoint: 授权端点,进行授权
* Token Endpoint: 令牌端点,经过授权拿到对应的Token
* Introspection Endpoint: 校验端点,校验Token的合法性
* Revocation Endpoint: 撤销端点,撤销授权

## 1.2 流程

1. 用户访问,此时没有token.Oauth2RestTemplate会报错.这个报错信息会被Oauth2ClientContextFilter捕获并重新定向到认证服务器
2. 认证服务器通过Authorization Endpoint进行授权,并通过AuthorizationServerTokenServices生成授权码并返回给客户端
3. 客户端拿到授权码去认证服务器通过Token Endpoint调用AuthorizationServerTokenServices生成Token并返回给客户端
4. 客户端拿到Token去资源服务器访问资源,一般会通过Oauth2AuthenticationManager调用ResourcesServerTokenServices进行校验.校验通过可以获取资源



# JWT

## 1. 简介

### 1.1 什么是JWT

JSON Web Token(JWT) 是一个开放的行业标准,他定义了一种简介的,自包含的协议格式,用于在通信双方传递json对象,传递的信息经过数字签名可以被验证和信任.JWT可以使用HMAC算法或使用RSA的公钥/私钥来签名,防止被篡改.

JWT令牌的优点:

1. JWT基于json,非常方便解析.
2. 可以在令牌中自定义丰富的内容,易拓展.
3. 通过非对称加密算法及签名技术,JWT防止篡改,安全性高.
4. 资源服务使用JWT可不依赖认证服务即可完成授权.

缺点: 

1. JWT令牌较长,占存储空间比较大.

### 1.2 JWT组成

一个JWT实际上就是一个字符串,它由三部分组成,头部,负载与签名.

#### 1.2.1 头部(header)

头部用于描述关于该JWT的最基本的信息,例如其类型以及签名使用的算法等.这也可以被表示成一个JSON对象.

```json
{
  "alg": "HS256"
  "typ": "JWT"
}
```

* typ: 是类型
* alg: 签名的算法,这里使用的算法是HS256算法

#### 1.2.2 负载(Payload)

第二部分师傅在,就是存放有效信息的地方.这个名字像是特指飞机上承载的货品,这些有效信息包含三个部分: 

* 标准中注册的声明(建议但不强制使用)

```
iss: jwt签发者
sub: jwt所面向的用户
aud: 接受jwt的一方
exp: jwt的过期时间,这个过期时间必须要大于签发时间
nbf: 定义在什么时间之前,该jwt都是不可用的
iat: jwt的签发时间
jti: jwt的唯一身份标识,主要用来作为一次性token,从而回避重放攻击.
```

* 公共的声明

  公共的声明可以添加任何的信息,一般添加用户的相关信息或其他业务需要的必要信息.但不建议添加敏感信息,因为该部分在客户端可解密

* 私有的声明

  私有的声明是提供者和消费者所共同定义的声明,一般不建议存放敏感信息,因为base64是对称解密的,意味着该部分信息可以归类为明文信息.

    

  这个指的就是自定义的claim.比如下面这个举例中的name都属于自定义的claim.这些claim跟jwt标准规定的claim区别在于: jwt规定的claim.jwt的结合搜放在拿到jwt之后,都知道怎么对这些标准的claim进行验证;而private claim不会验证,除非明确告诉接收方要对这些claim进行验证以及规则才行.

  ```
  {
  "sub": "1234567890"
  "name": "John Doe"
  "iat": 1516239022
  }
  ```

  其中sub是标准的声明,name是自定义的声明(公共的或私有的)

  然后将其进行base64编码,得到jwt的第二部分

  **提示: 声明中不要放一些敏感信息**

#### 1.2.3 签名,签名(signature)

jwt的第三部分是一个签证信息,这个签证信息由三部分组成:

1. header(base64后的)
2. payload(base64后的)
3. secret(盐)

这个部分需要base64加密后的header和base64加密后的payload使用,连接组成的字符串啊,然后通过header中的声明的加密方式进行加盐secret组合加密,然后就构成jwt的第三部分;

将这三部分用.连接成一个完整的字符串,构成了最终的jwt;

**注意: secret是保存在服务器端的,jwt的签发生成也是在服务器端的,secret就是用来进行jwt的签发和jwt的验证,所以,他就是你服务器端的私钥,在任何场景都不应该流露出去.一旦客户端得知这个secret,那就意味着客户端是可以自我签发jwt了**

## 2. JJWT简介

### 2.1 什么事JJWT

JJWT是一个提供端到端的JWT创建和验证的java库.永远免费和开源,JJWT很容易使用和理解.他被设计成一个以建筑为中心的流畅界面,隐藏了他的大部分复杂性;

### 2.2 token创建

导入依赖

```xml
				<dependency>
            <groupId>io.jsonwebtoken</groupId>
            <artifactId>jjwt</artifactId>
            <version>0.9.1</version>
        </dependency>
```



```java
	public void jwtBuilderTest() {
        JwtBuilder builder = Jwts.builder()
                //设置id {"jtl":"555"}
                .setId("555")
                //签发用户
                .setSubject("CodeSniper")
                //签发时间
                .setIssuedAt(new Date())
                .signWith(SignatureAlgorithm.HS256,"xxxx");

        //生成token
        String token = builder.compact();
        System.out.println(token);
        String[] split = token.split("\\.");

        //头部
        System.out.println(Base64Codec.BASE64.decodeToString(split[0]));
        //负载
        System.out.println(Base64Codec.BASE64.decodeToString(split[1]));
        //签名,乱码
        System.out.println(Base64Codec.BASE64.decodeToString(split[2]));
    }
```



### 2.3 解析token

```java
public void parseToken(){
	    String token="eyJhbGciOiJIUzI1NiJ9.eyJqdGkiOiI1NTUiLCJzdWIiOiJDb2RlU25pcGVyIiwiaWF0IjoxNjMwMjE1NTkxfQ.Ztnwgh8USc1iEPez7RmHGcrH_6RLFKE60TKAzM1wOTw";
	    //获取claims(荷载)
        Claims claims = Jwts.parser()
                .setSigningKey("xxxx")
                .parseClaimsJws(token)
                .getBody();
        System.out.println("jti:"+claims.getId());
        System.out.println("sub:"+claims.getSubject());
        System.out.println("iat:"+claims.getIssuedAt());
```



### 2.4 token过期校验

```java
  //创建token (过期时间)
    @Test
    public void jwtBuilderTestHasExp() {
        long now = System.currentTimeMillis();
        long exp = now + 60 * 1000;
        JwtBuilder builder = Jwts.builder()
                //设置id {"jtl":"555"}
                .setId("555")
                //签发用户
                .setSubject("CodeSniper")
                //签发时间
                .setIssuedAt(new Date())
                .signWith(SignatureAlgorithm.HS256, "xxxx")
                .setExpiration(new Date(exp));

        //生成token
        String token = builder.compact();
        System.out.println(token);
        String[] split = token.split("\\.");

        //头部
        System.out.println(Base64Codec.BASE64.decodeToString(split[0]));
        //负载
        System.out.println(Base64Codec.BASE64.decodeToString(split[1]));
        //签名,乱码
        System.out.println(Base64Codec.BASE64.decodeToString(split[2]));
    }

    //解析token(过期时间)
    @Test
    public void parseTokenHasExp() {
        String token = "eyJhbGciOiJIUzI1NiJ9.eyJqdGkiOiI1NTUiLCJzdWIiOiJDb2RlU25pcGVyIiwiaWF0IjoxNjMwMjE1NTkxfQ.Ztnwgh8USc1iEPez7RmHGcrH_6RLFKE60TKAzM1wOTw";
        //获取claims(荷载)
        Claims claims = Jwts.parser()
                .setSigningKey("xxxx")
                .parseClaimsJws(token)
                .getBody();
        System.out.println("jti:" + claims.getId());
        System.out.println("sub:" + claims.getSubject());
        System.out.println("iat:" + claims.getIssuedAt());
        System.out.println("当前时间"+new Date());
        SimpleDateFormat format = new SimpleDateFormat("yyyy-MM-dd");
        System.out.println("过期时间"+format.format(claims.getExpiration()));
    }
```

### 2.4 自定义声明

参数是map

```java
.claim("logo","xxx.jpg")
```

获取

```java
claims.get("logo")
```



