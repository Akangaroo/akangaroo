# JWT

## 常见的认证协议

### HTTP Basic Auth

HTTP Basic Auth简单点说明就是每次请求API时都提供用户的username和password，简言之，Basic Auth是配合RESTful API 使用的最简单的认证方式，只需提供用户名密码即可，但由于有把用户名密码暴露给第三方客户端的风险，在生产环境下被使用的越来越少。因此，在开发对外开放的RESTful API时，尽量避免采用HTTP Basic Auth。

### Cookie Auth

Cookie认证机制就是为一次请求认证在服务端创建一个Session对象，同时在客户端的浏览器端创建了一个Cookie对象；通过客户端带上来Cookie对象来与服务器端的session对象匹配来实现状态管理的。默认的，当我们关闭浏览器的时候，cookie会被删除。但可以通过修改cookie 的expire time使cookie在一定时间内有效。



### Oauth

OAuth（开放授权,Open Authorization）是一个开放的授权标准，允许用户让第三方应用访问该用户在某一web服务上存储的私密的资源（如照片，视频，联系人列表），而无需将用户名和密码提供给第三方应用。如网站通过微信、微博登录等，主要用于第三方登录。

OAuth允许用户提供一个令牌，而不是用户名和密码来访问他们存放在特定服务提供者的数据。每一个令牌授权一个特定的第三方系统（例如，视频编辑网站)在特定的时段（例如，接下来的2小时内）内访问特定的资源（例如仅仅是某一相册中的视频）。这样，OAuth让用户可以授权第三方网站访问他们存储在另外服务提供者的某些特定信息，而非所有内容。

基于OAuth的认证机制适用于个人消费者类的互联网产品，如社交类APP等应用，但是不太适合拥有自有认证权限管理的企业应用。

缺点：过重

### Token Auth

使用基于 Token 的身份验证方法，在服务端不需要存储用户的登录记录。大概的流程是这样的：

1. 客户端使用用户名跟密码请求登录
2. 服务端收到请求，去验证用户名与密码
3. 验证成功后，服务端会签发一个 Token，再把这个 Token 发送给客户端
4. 客户端收到 Token 以后可以把它存储起来，比如放在 Cookie 里
5. 客户端每次向服务端请求资源的时候需要带着服务端签发的 Token
6. 服务端收到请求，然后去验证客户端请求里面带着的 Token，如果验证成功，就向客户端返回请
求的数据

相较于前三种方式，Token Auth较为优秀：比第一种方式更安全，比第二种方式更节约服务器资源，比第三种方式更加轻量。

1. 支持跨域访问: Cookie是不允许垮域访问的，这一点对Token机制是不存在的，前提是传输的用户
认证信息通过HTTP头传输.
2. 无状态(也称：服务端可扩展行):Token机制在服务端不需要存储session信息，因为Token 自身包
含了所有登录用户的信息，只需要在客户端的cookie或本地介质存储状态信息.
3. 更适用CDN: 可以通过内容分发网络请求你服务端的所有资料（如：javascript，HTML,图片
等），而你的服务端只要提供API即可.
4. 去耦: 不需要绑定到一个特定的身份验证方案。Token可以在任何地方生成，只要在你的API被调用
的时候，你可以进行Token生成调用即可.
5. 更适用于移动应用: 当你的客户端是一个原生平台（iOS, Android，Windows 10等）时，Cookie
是不被支持的（你需要通过Cookie容器进行处理），这时采用Token认证机制就会简单得多。
6. CSRF:因为不再依赖于Cookie，所以你就不需要考虑对CSRF（跨站请求伪造）的防范。
7. 性能: 一次网络往返时间（通过数据库查询session信息）总比做一次HMACSHA256计算的Token
验证和解析要费时得多.
8. 不需要为登录页面做特殊处理: 如果你使用Protractor 做功能测试的时候，不再需要为登录页面做
特殊处理.
9. 基于标准化:你的API可以采用标准化的 JSON Web Token (JWT). 这个标准已经存在多个后端库
（.NET, Ruby, Java,Python, PHP）和多家公司的支持（如：Firebase,Google, Microsoft）



## JWT简介

### 什么是JWT

 JSON Web Token（JWT）是一个开放的行业标准（RFC 7519），它定义了一种简介的、自包含的协议格式，用于在通信双方传递json对象，传递的信息经过数字签名可以被验证和信任。JWT可以使用HMAC算法或使用RSA的公钥/私钥对来签名，防止被篡改。

官网： `https://jwt.io/`
标准： `https://tools.ietf.org/html/rfc7519`

JWT令牌的优点：
1. jwt基于json，非常方便解析。
2. 可以在令牌中自定义丰富的内容，易扩展。
3. 通过非对称加密算法及数字签名技术，JWT防止篡改，安全性高。
4. 资源服务使用JWT可不依赖认证服务即可完成授权。

JWT令牌的缺点：

1. JWT令牌较长，占存储空间比较大。

### JWT组成

#### 头部(Header)

头部用于描述关于该JWT的最基本的信息，例如其类型（即JWT）以及签名所用的算法（如HS256或RSA）等。这也可以被表示成一个JSON对象。

```json
{
 "alg": "HS256",
 "typ": "JWT"
}
```

typ ：是类型。
alg ：签名的算法，这里使用的算法是HS256算法

我们对头部的json字符串进行BASE64编码（网上有很多在线编码的网站），编码后的字符串如下：

```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9
```

Base64 是一种基于64个可打印字符来表示二进制数据的表示方法。由于2的6次方等于64，所以每6个比特为一个单元，对应某个可打印字符。三个字节有24个比特，对应于4个Base64单元，即3个字节需要用4个可打印字符来表示。JDK 中提供了非常方便的 BASE64Encoder 和 BASE64Decoder ，用它们可以非常方便的完成基于 BASE64 的编码和解码。

#### 负载(Payload)

第二部分是负载，就是存放有效信息的地方。这个名字像是特指飞机上承载的货品，这些有效信息包含三个部分:

1. 标准中注册的声明（建议但不强制使用）

   - iss: jwt签发者
   - sub: jwt所面向的用户
   - aud: 接收jwt的一方
   - exp: jwt的过期时间，这个过期时间必须要大于签发时间nbf: 定义在什么时间之前，该jwt都是不可用的
   - iat: jwt的签发时间
   - jti: jwt的唯一身份标识，主要用来作为一次性token,从而回避重放攻击

2. 公共的声明

   公共的声明可以添加任何的信息，一般添加用户的相关信息或其他业务需要的必要信息.但不建议添加敏感信息，因为该部分在客户端可解密

3. 私有的声明

   私有声明是提供者和消费者所共同定义的声明，一般不建议存放敏感信息，因为base64是对称解密的，意味着该部分信息可以归类为明文信息。

   这个指的就是自定义的claim。比如下面那个举例中的name都属于自定的claim。这些claim跟JWT标准规定的claim区别在于：JWT规定的claim，JWT的接收方在拿到JWT之后，都知道怎么对这些标准的claim进行验证(还不知道是否能够验证)；而private claims不会验证，除非明确告诉接收方要对这些claim进行验证以及规则才行。

   ```json
   {
    "sub": "1234567890",
    "name": "John Doe",
    "iat": 1516239022
   }
   ```

   其中 sub 是标准的声明， name 是自定义的声明（公共的或私有的）
   然后将其进行base64编码，得到Jwt的第二部分

   ```
   eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkphbWVzIiwiYWRtaW4iOnRydWV9
   ```

   提示：声明中不要放一些敏感信息。

#### 签证、签名(signature)

jwt的第三部分是一个签证信息，这个签证信息由三部分组成：

1. header (base64后的)
2. payload (base64后的)
3. secret（盐，一定要保密）

这个部分需要base64加密后的header和base64加密后的payload使用.连接组成的字符串，然后通过header中声明的加密方式进行加盐secret组合加密，然后就构成了jwt的第三部分

```
cThIIoDvwdueQB468K5xDc5633seEFoqwxjF_xSJyQQ
```

将这三部分用.连接成一个完整的字符串,构成了最终的jwt

```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiaWF0IjoxNTE2MjM5MDIyfQ.cThIIoDvwdueQB468K5xDc5633seEFoqwxjF_xSJyQQ
```

![image-20210217133527406](SpringSecurity Oauth2整合JWT.assets/image-20210217133527406.png)

注意： secret 是保存在服务器端的， jwt 的签发生成也是在服务器端的， secret 就是用来进行 jwt的签发和 jwt 的验证，所以，它就是你服务端的私钥，在任何场景都不应该流露出去。一旦客户端得知这个 secret , 那就意味着客户端是可以自我签发 jwt 了。



## JJWT简介

### 什么是JJWT

JJWT是一个提供端到端的JWT创建和验证的Java库。永远免费和开源(Apache License，版本2.0)，JJW很容易使用和理解。它被设计成一个以建筑为中心的流畅界面，隐藏了它的大部分复杂性。

规范官网：`https://jwt.io/`



### 快速入门

导入依赖

```xml
    <dependencies>
        <dependency>
            <groupId>io.jsonwebtoken</groupId>
            <artifactId>jjwt</artifactId>
            <version>0.9.1</version>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>
```



#### token的创建

创建测试类JwtTest，用于生成token

```java
    @Test
    public void testJWT() {
        // 创建一个JwtBuilder对象
        JwtBuilder jwtBuilder = Jwts.builder()
                // 唯一id{"id": "888"}
                .setId("888")
                // 主体{"sub": "jack"}
                .setSubject("jack")
                // 签发时间 {"iat": "..."}
                .setIssuedAt(new Date())
                // 签名手段，参数1：算法，参数2：盐
                .signWith(SignatureAlgorithm.HS256, "xxxxx");
        // 获取jwt的token
        String token = jwtBuilder.compact();
        System.out.println(token);

        // 三个部分的base64解密
        System.out.println("======================");
        String[] split = token.split("\\.");
        System.out.println(Base64Codec.BASE64.decodeToString(split[0]));
        System.out.println(Base64Codec.BASE64.decodeToString(split[1]));
        // 第三部分会乱码，因为解析不了
        System.out.println(Base64Codec.BASE64.decodeToString(split[2]));
    }
```

![image-20210217134305375](SpringSecurity Oauth2整合JWT.assets/image-20210217134305375.png)



#### token的验证解析

我们刚才已经创建了token ，在web应用中这个操作是由服务端进行然后发给客户端，客户端在下次向服务端发送请求时需要携带这个token（这就好像是拿着一张门票一样），那服务端接到这个token应该解析出token中的信息（例如用户id）,根据这些信息查询数据库返回相应的结果。

```java
    @Test
    public void testToken() {
        String token = "eyJhbGciOiJIUzI1NiJ9.eyJqdGkiOiI4ODgiLCJzdWIiOiJqYWNrIiwiaWF0IjoxNjEyNjE3NTY1fQ .hZ7p_fMbrEccF2r8pxP1rlWSQBTR7sBREkLFaovJoTI";
        // 解析token获取负载中的声明对象
        Claims claims = (Claims) Jwts.parser()
                .setSigningKey("xxxx")
                .parse(token)
                .getBody();

        System.out.println(claims.getId());
        System.out.println(claims.getSubject());
        System.out.println(claims.getIssuedAt());
    }
```

![image-20210217134730283](SpringSecurity Oauth2整合JWT.assets/image-20210217134730283.png)

#### token的过期校验

有很多时候，我们并不希望签发的token是永久生效的，所以我们可以为token添加一个过期时间。原因：从服务器发出的token，服务器自己并不做记录，就存在一个弊端就是，服务端无法主动控制某token的立刻失效。

```java
    @Test
    void testJWTHasExpire() {
        // 过期时间，这里是1分钟后的时间长整型
        long exp = System.currentTimeMillis() + 60 * 1000;

        JwtBuilder jwtBuilder = Jwts.builder()
                // 唯一id{"id": "888"}
                .setId("888")
                // 接受的用户 {"sub": "jack"}
                .setSubject("jack")
                // 签发时间 {"iat": "..."}
                .setIssuedAt(new Date())
                // 签发算法及秘钥
                .signWith(SignatureAlgorithm.HS256, "xxxxx")
                .setExpiration(new Date(exp));

        String token = jwtBuilder.compact();
        System.out.println(token);
        
    }
```

当未过期时可以正常读取，当过期时会引发io.jsonwebtoken.ExpiredJwtException异常。

#### 自定义claims

刚才的例子只是存储了id和subject两个信息，如果你想存储更多的信息（例如角色）可以定义自定
义claims

```java
    @Test
    public void testJWTEnhancer() {

        JwtBuilder jwtBuilder = Jwts.builder()
                // 唯一id{"id": "888"}
                .setId("888")
                // 接受的用户 {"sub": "jack"}
                .setSubject("jack")
                // 签发时间 {"iat": "..."}
                .setIssuedAt(new Date())
                // 签发算法及秘钥
                .signWith(SignatureAlgorithm.HS256, "xxxxx")
                .claim("name", "rose");

        String token = jwtBuilder.compact();
        System.out.println(token);
    }
```

自定义claims解析

```java
    @Test
    public void testTokenEnhancer() {
        String token = "eyJhbGciOiJIUzI1NiJ9.eyJqdGkiOiI4ODgiLCJzdWIiOiJqYWNrIiwiaWF0IjoxNjEyNjE4NTczLCJuYW1lIjoicm9zZSJ9.DaTDkCTFugpCy7GSgP0I6noaLrM45gX_LHmQC960v2k";


        Claims claims = (Claims) Jwts.parser()
                .setSigningKey("xxxx")
                .parse(token)
                .getBody();

        System.out.println(claims.getId());
        System.out.println(claims.getSubject());
        System.out.println(claims.getIssuedAt());
        System.out.println(claims.get("name"));
    }
```

![image-20210217135750914](SpringSecurity Oauth2整合JWT.assets/image-20210217135750914.png)



# SpringSecurity Oauth2 整合 JWT

之前Spring Security Oauth2的完整代码进行修改

## 整合JWT

使用Jwt存储token的配置

```java
@Configuration
public class JwtTokenStoreConfig {


    @Bean
    public TokenStore jwtTokenStore() {
        return new JwtTokenStore(jwtAccessTokenConverter());
    }

    @Bean
    public JwtAccessTokenConverter jwtAccessTokenConverter() {
        JwtAccessTokenConverter jwtAccessTokenConverter = new JwtAccessTokenConverter();
        // 设置jwt秘钥
        jwtAccessTokenConverter.setSigningKey("test_key");
        return jwtAccessTokenConverter;
    }
}
```

在认证服务器配置中指定令牌的存储策略为JWT

```java
@Configuration
@EnableAuthorizationServer
public class AuthorizationServerConfig extends AuthorizationServerConfigurerAdapter {

    @Autowired
    private PasswordEncoder passwordEncoder;

    @Autowired
    private AuthenticationManager authenticationManager;

    @Autowired
    private UserService userService;

    @Autowired
    @Qualifier("jwtTokenStore")
    private TokenStore tokenStore;

    @Autowired
    private JwtAccessTokenConverter jwtAccessTokenConverter;



    @Override
    public void configure(AuthorizationServerEndpointsConfigurer endpoints) throws Exception {

        endpoints.authenticationManager(authenticationManager)
                .userDetailsService(userService)
                // 配置存储令牌策略
                .tokenStore(tokenStore)
                .accessTokenConverter(jwtAccessTokenConverter);
    }
    
        @Override
    public void configure(ClientDetailsServiceConfigurer clients) throws Exception {
        clients.inMemory()
                // 配置client_id
                .withClient("client")
                // 配置client-secret
                .secret(passwordEncoder.encode("1234"))
                // 配置redirect_uri，用于授权成功后跳转
                .redirectUris("http://www.baidu.com")
                // 配置申请的权限范围
                .scopes("all")
                // 配置grant_type，表示授权类型
                .authorizedGrantTypes("authorization_code", "password");

    }
}
```

使用密码模式进行测试

![image-20210217142009047](SpringSecurity Oauth2整合JWT.assets/image-20210217142009047.png)

将拿到的令牌去jwt.io解析



![image-20210217142208458](SpringSecurity Oauth2整合JWT.assets/image-20210217142208458.png)





## 扩展JWT中存储的内容

有时候我们需要扩展JWT中存储的内容，这里我们在JWT中扩展一个 key为enhance，value为enhance info 的数据

继承TokenEnhancer实现一个JWT内容增强器

```java
@Component
public class JwtTokenEnhancer implements TokenEnhancer {
    @Override
    public OAuth2AccessToken enhance(OAuth2AccessToken oAuth2AccessToken, OAuth2Authentication oAuth2Authentication) {
        Map<String, Object> info = new HashMap<>();
        info.put("enhance", "enhanceInfo");
        ((DefaultOAuth2AccessToken)oAuth2AccessToken).setAdditionalInformation(info);
        return oAuth2AccessToken;
    }
}
```

在认证服务器中配置一下

```java
    @Autowired
    private JwtTokenEnhancer jwtTokenEnhancer;

    @Override
    public void configure(AuthorizationServerEndpointsConfigurer endpoints) throws Exception {
        TokenEnhancerChain tokenEnhancerChain = new TokenEnhancerChain();
        List<TokenEnhancer> delegates = new ArrayList<>();
        // 配置JWT的内容增强器
        delegates.add(jwtTokenEnhancer);
        delegates.add(jwtAccessTokenConverter);
        tokenEnhancerChain.setTokenEnhancers(delegates);

        endpoints.authenticationManager(authenticationManager)
                .userDetailsService(userService)
                // 配置存储令牌策略
                .tokenStore(tokenStore)
                .accessTokenConverter(jwtAccessTokenConverter)
                .tokenEnhancer(tokenEnhancerChain);
    }
```

用密码模式进行测试，拿token去jwt.io解析一下，可以发现，确实有扩展的信息。

![image-20210217143349210](SpringSecurity Oauth2整合JWT.assets/image-20210217143349210.png)





## 刷新令牌

在Spring Cloud Security 中使用oauth2时，如果令牌失效了，可以使用刷新令牌通过refresh_token的授权模式再次获取access_token。只需修改认证服务器的配置，添加refresh_token的授权模式即可。

```java
    @Override
    public void configure(ClientDetailsServiceConfigurer clients) throws Exception {
        clients.inMemory()
                // 配置client_id
                .withClient("client")
                // 配置client-secret
                .secret(passwordEncoder.encode("1234"))
                // 配置访问token的有效期
                .accessTokenValiditySeconds(30)
                // 配置刷新token的有效期
                .refreshTokenValiditySeconds(864000)
                // 配置redirect_uri，用于授权成功后跳转
                .redirectUris("http://www.baidu.com")
                // 配置申请的权限范围
                .scopes("all")
                // 配置grant_type，表示授权类型
                // authorization_code: 授权码模式
                // password: 密码模式
                // refresh_token: 刷新令牌
                .authorizedGrantTypes("authorization_code", "password", "refresh_token");
    }
```

通过refresh_token获取新的令牌

![image-20210217144456356](SpringSecurity Oauth2整合JWT.assets/image-20210217144456356.png)