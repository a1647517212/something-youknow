# 1 什么是JWT和JWT的组成

## 1.1 什么是JWT

**Json web token (JWT)**, 是为了在网络应用环境间传递声明而执行的一种基于JSON的开放标准（(RFC 7519).该token被设计为紧凑且安全的，特别适用于分布式站点的单点登录（SSO）场景。JWT的声明一般被用来在身份提供者和服务提供者间传递被认证的用户身份信息，以便于从资源服务器获取资源，也可以增加一些额外的其它业务逻辑所必须的声明信息，该token也可直接被用于认证，也可被加密。

## 1.2 JWT有什么好处？

1. **支持跨域访问**: Cookie是不允许垮域访问的，这一点对Token机制是不存在的，前提是传输的用户认证信息通过HTTP头传输.
2. **无状态(也称：服务端可扩展行):**Token机制在服务端不需要存储session信息，因为Token 自身包含了所有登录用户的信息，只需要在客户端的cookie或本地介质存储状态信息.
3. **更适用CDN**: 可以通过内容分发网络请求你服务端的所有资料（如：javascript，HTML,图片等），而你的服务端只要提供API即可.（居于前面两点得出这个更适用于CDN内容分发网络）
4. **去耦**: 不需要绑定到一个特定的身份验证方案。Token可以在任何地方生成，只要在你的API被调用的时候，你可以进行Token生成调用即可.（这个似乎也在继续说前面第一点和第二点的好处。。。）
5. **更适用于移动应用**: 当你的客户端是一个原生平台（iOS, Android，Windows 8等）时，Cookie是不被支持的（你需要通过Cookie容器进行处理），这时采用Token认证机制就会简单得多。
6. **CSRF**:因为不再依赖于Cookie，所以你就不需要考虑对CSRF（跨站请求伪造）的防范。（如果token是用cookie保存，CSRF还是需要考虑,一般建议使用1、在HTTP请求中以参数的形式加入一个服务器端产生的token。或者2.放入http请求头中也就是一次性给所有该类请求加上csrftoken这个HTTP头属性，并把token值放入其中）
   ps:后面会推出一些常见的网络安全的处理
7. **性能**: 一次网络往返时间（通过数据库查询session信息）总比做一次HMACSHA256计算 的Token验证和解析要费时得多.
8. **不需要为登录页面做特殊处理**: 如果你使用Protractor 做功能测试的时候，不再需要为登录页面做特殊处理.（知识面太窄，不是太明白这个的意思）
9. **基于标准化**:你的API可以采用标准化的 JSON Web Token (JWT). 这个标准已经存在多个后端库（.NET, Ruby, Java,Python, PHP）和多家公司的支持（如：Firebase,Google, Microsoft）.

## 1.3 JWT的组成

第一部分我们称它为`头部（header)`,第二部分我们称其为`载荷（payload, 类似于飞机上承载的物品)`，第三部分是`签证（signature)`.

### header

jwt的头部承载两部分信息：

- 声明类型，这里是jwt

- 声明加密的算法 通常直接使用 HMAC SHA256

- 完整的头部就像下面这样的JSON：

  ````.java
  {
    'typ': 'JWT',
    'alg': 'HS256'
  }
  ````


  然后将头部进行base64加密（该加密是可以对称解密的),构成了第一部分

### playload

载荷就是存放有效信息的地方。这个名字像是特指飞机上承载的货品，这些有效信息包含三个部分

- 标准中注册的声明

  ````.java
  标准中注册的声明 (建议但不强制使用)包含字段 ：
    iss: jwt签发者
    sub: jwt所面向的用户
    aud: 接收jwt的一方
    exp: jwt的过期时间，这个过期时间必须要大于签发时间
    nbf: 定义在什么时间之前，该jwt都是不可用的.
    iat: jwt的签发时间
    jti: jwt的唯一身份标识，主要用来作为一次性token,从而回避重放攻击。
  ````

- 公共的声明

  ````.java
  公共的声明可以添加任何的信息，一般添加用户的相关信息或其他业务需要的必要信息.但不建议添加敏感信息，因为该部分在客户端可解密.
  ````

  

- 私有的声明

  ````.java
  私有声明是提供者和消费者所共同定义的声明，一般不建议存放敏感信息，因为base64是对称解密的，意味着该部分信息可以归类为明文信息。
  ````

- 定义一个payload:

  ````.java
  {
    "sub": "1234567890",
    "name": "John Doe",
    "admin": true
  }
  ````

  然后将其进行base64加密，得到Jwt的第二部分。

- 这个指的就是自定义的claim。比如前面那个结构举例中的admin和name都属于自定的claim。这些claim跟JWT标准规定的claim区别在于：JWT规定的claim，JWT的接收方在拿到JWT之后，都知道怎么对这些标准的claim进行验证(还不知道是否能够验证)；而private claims不会验证，除非明确告诉接收方要对这些claim进行验证以及规则才行。

### signature

jwt的第三部分是一个签证信息，这个签证信息由三部分组成：

**header (base64后的)**
**payload (base64后的)**
**secret**
这个部分需要base64加密后的header和base64加密后的payload使用.连接组成的字符串，然后通过header中声明的加密方式进行**加盐secret**组合加密，然后就构成了jwt的第三部分。

````.java
var encodedString = base64UrlEncode(header) + '.' + base64UrlEncode(payload);

var signature = HMACSHA256(encodedString, 'secret'); // TJVA95OrM7E2cBab30RMHrHDcEfxjoYZgeFONFh7HgQ

将这三部分用.连接成一个完整的字符串,构成了最终的jwt:
````


注意：secret是保存在服务器端的，jwt的签发生成也是在服务器端的，secret就是用来进行jwt的签发和jwt的验证，所以，它就是你服务端的私钥，在任何场景都不应该流露出去。一旦客户端得知这个secret, 那就意味着客户端是可以自我签发jwt了。

# 2. 在Java中如何使用JWT

JWT如何应用（目前就这两种适用、当然还可以将token添加到cookie中，但是不建议使用）
**在请求地址中添加token并验证**
CSRF攻击之所以能够成功，是因为攻击者可以伪造用户的请求，该请求中所有的用户验证信息都存在于Cookie中，因此攻击者可以在不知道这些验证信息的情况下直接利用用户自己的Cookie来通过安全验证。由此可知，抵御CSRF攻击的关键在于：在请求中放入攻击者所不能伪造的信息，并且该信息不存在于Cookie之中。鉴于此，系统开发者可以在HTTP请求中以参数的形式加入一个随机产生的token，并在服务器端建立一个拦截器来验证这个token，如果请求中没有token或者token内容不正确，则认为可能是CSRF攻击而拒绝该请求。

**在HTTP头中自定义属性并验证**
自定义属性的方法也是使用token并进行验证，和前一种方法不同的是，这里并不是把token以参数的形式置于HTTP请求之中，而是把它放到HTTP头中自定义的属性里。通过XMLHttpRequest这个类，可以一次性给所有该类请求加上csrftoken这个HTTP头属性，并把token值放入其中。这样解决了前一种方法在请求中加入token的不便，同时，通过这个类请求的地址不会被记录到浏览器的地址栏，也不用担心token会通过Referer泄露到其他网站。

## **2.1 Java的JJWT实现JWT**

创建jwt:
本文主要使用的是jjwt-0.6.0.jar

````.java
/**

   * 创建jwt
   * @param id
   * @param subject
   * @param ttlMillis 过期的时间长度
   * @return
   * @throws Exception
     /
public String createJWT(String id, String subject, long ttlMillis) throws Exception {

     SignatureAlgorithm signatureAlgorithm = SignatureAlgorithm.HS256; //指定签名的时候使用的签名算法，也就是header那部分，jjwt已经将这部分内容封装好了。
     long nowMillis = System.currentTimeMillis();//生成JWT的时间
     Date now = new Date(nowMillis);
     Map<String,Object> claims = new HashMap<String,Object>();//创建payload的私有声明（根据特定的业务需要添加，如果要拿这个做验证，一般是需要和jwt的接收方提前沟通好验证方式的）
     claims.put("uid", "DSSFAWDWADAS...");
     claims.put("user_name", "admin");
     claims.put("nick_name","DASDA121");
     SecretKey key = generalKey();//生成签名的时候使用的秘钥secret,这个方法本地封装了的，一般可以从本地配置文件中读取，切记这个秘钥不能外露哦。它就是你服务端的私钥，在任何场景都不应该流露出去。一旦客户端得知这个secret, 那就意味着客户端是可以自我签发jwt了。
     //下面就是在为payload添加各种标准声明和私有声明了
     JwtBuilder builder = Jwts.builder() //这里其实就是new一个JwtBuilder，设置jwt的body
             .setClaims(claims)          //如果有私有声明，一定要先设置这个自己创建的私有的声明，这个是给builder的claim赋值，一旦写在标准的声明赋值之后，就是覆盖了那些标准的声明的
             .setId(id)                  //设置jti(JWT ID)：是JWT的唯一标识，根据业务需要，这个可以设置为一个不重复的值，主要用来作为一次性token,从而回避重放攻击。
             .setIssuedAt(now)           //iat: jwt的签发时间
             .setSubject(subject)        //sub(Subject)：代表这个JWT的主体，即它的所有人，这个是一个json格式的字符串，可以存放什么userid，roldid之类的，作为什么用户的唯一标志。
             .signWith(signatureAlgorithm, key);//设置签名使用的签名算法和签名使用的秘钥
     if (ttlMillis >= 0) {
         long expMillis = nowMillis + ttlMillis;
         Date exp = new Date(expMillis);
         builder.setExpiration(exp);     //设置过期时间
     }
     return builder.compact();           //就开始压缩为xxxxxxxxxxxxxx.xxxxxxxxxxxxxxx.xxxxxxxxxxxxx这样的jwt
     //打印了一哈哈确实是下面的这个样子
     //eyJhbGciOiJIUzI1NiJ9.eyJ1aWQiOiJEU1NGQVdEV0FEQVMuLi4iLCJzdWIiOiIiLCJ1c2VyX25hbWUiOiJhZG1pbiIsIm5pY2tfbmFtZSI6IkRBU0RBMTIxIiwiZXhwIjoxNTE3ODI4MDE4LCJpYXQiOjE1MTc4Mjc5NTgsImp0aSI6Imp3dCJ9.xjIvBbdPbEMBMurmwW6IzBkS3MPwicbqQa2Y5hjHSyo
         }
         
         

解析jwt
/**

   * 解密jwt
   * @param jwt
   * @return
   * @throws Exception
     /
         public Claims parseJWT(String jwt) throws Exception{
     SecretKey key = generalKey();  //签名秘钥，和生成的签名的秘钥一模一样
     Claims claims = Jwts.parser()  //得到DefaultJwtParser
        .setSigningKey(key)         //设置签名的秘钥
        .parseClaimsJws(jwt).getBody();//设置需要解析的jwt
     return claims;
         }

上面代码中两次用到的generalKey()方法
/**

   * 由字符串生成加密key
   * @return
     /
         public SecretKey generalKey(){
     String stringKey = Constant.JWT_SECRET;//本地配置文件中加密的密文7786df7fc3a34e26a61c034d5ec8245d
     byte[] encodedKey = Base64.decodeBase64(stringKey);//本地的密码解码[B@152f6e2
     System.out.println(encodedKey);//[B@152f6e2
     System.out.println(Base64.encodeBase64URLSafeString(encodedKey));//7786df7fc3a34e26a61c034d5ec8245d
     SecretKey key = new SecretKeySpec(encodedKey, 0, encodedKey.length, "AES");// 根据给定的字节数组使用AES加密算法构造一个密钥，使用 encodedKey中的始于且包含 0 到前 leng 个字节这是当然是所有。（后面的文章中马上回推出讲解Java加密和解密的一些算法）
     return key;
         }



main方法测试
    public static void main(String[] args) throws Exception {
        JwtUtil util=   new JwtUtil();
        String ab=util.createJWT("jwt", "{id:100,name:xiaohong}", 60000);
        System.out.println(ab);
        
String jwt="eyJhbGciOiJIUzI1NiJ9.eyJ1aWQiOiJEU1NGQVdEV0FEQVMuLi4iLCJzdWIiOiJ7aWQ6MTAwLG5hbWU6eGlhb2hvbmd9IiwidXNlcl9uYW1lIjoiYWRtaW4iLCJuaWNrX25hbWUiOiJEQVNEQTEyMSIsImV4cCI6MTUxNzgzNTEwOSwiaWF0IjoxNTE3ODM1MDQ5LCJqdGkiOiJqd3QifQ.G_ovXAVTlB4WcyD693VxRRjOxa4W5Z-fklOp_iHj3Fg";
        Claims c=util.parseJWT(jwt);//注意：如果jwt已经过期了，这里会抛出jwt过期异常。
        System.out.println(c.getId());//jwt
        System.out.println(c.getIssuedAt());//Mon Feb 05 20:50:49 CST 2018
        System.out.println(c.getSubject());//{id:100,name:xiaohong}
        System.out.println(c.getIssuer());//null
        System.out.println(c.get("uid", String.class));//DSSFAWDWADAS...
    }
````



------------------------------------------------
版权声明：本文为CSDN博主「等有一天成了大神。。。」的原创文章，遵循 CC 4.0 BY-SA 版权协议，转载请附上原文出处链接及本声明。
原文链接：https://blog.csdn.net/qq_37636695/article/details/79265711