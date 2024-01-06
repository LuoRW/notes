# 导学

三更

1.Spring Security官网: https://spring.io/projects/spring-security

2.视频是 BV1mm4y1X7Hc，拿着这个BV号在b站搜，就能搜到课程视频

# 快速入门

## 1. 介绍

springsecurity是安全框架，准确来说是安全管理框架。相比与另外一个安全框架**Shiro**，springsecurity提供了更丰富的功能，社区资源也比Shiro丰富

springsecurity框架用于Web应用的需要进行==认证==和==授权==

**认证：**验证当前访问系统的是不是本系统的用户，并且要确认具体是哪个用户

**授权：**经过认证后判断当前用户是否有权限进行某个操作。

认证和授权也是SpringSecurity作为安全框架的核心功能。

## 2. Boot环境搭建

 打开idea，新建maven项目

![img](img/1689586874143-d1619144-d7ef-4c35-a972-ece35cffb343.jpg)

![img](img/1689586874371-e5e5b354-1723-4de9-9dd3-921c1cc0b8c3.jpg)

maven坐标pom.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.lrw</groupId>
    <artifactId>Security01_QuickStart</artifactId>
    <version>1.0-SNAPSHOT</version>

    <properties>
        <maven.compiler.source>8</maven.compiler.source>
        <maven.compiler.target>8</maven.compiler.target>
    </properties>

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.5.0</version>
    </parent>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
        </dependency>
    </dependencies>

</project>
```

启动类SecurityApplication.java

```java
@SpringBootApplication
public class SecurityApplication {

    public static void main(String[] args) {
        SpringApplication.run(SecurityApplication.class, args);
    }

}
```

测试接口HelloController.java

```java
@RestController
public class HelloController {

    @GetMapping("/hello")
    public String hello() {
        return "hello world!";
    }

}
```

运行SecurityApplication启动类，然后浏览器访问如下，测试SpringBoot工程有没有搭建成功

`http://localhost:8080/hello`

![image-20240105120315242](img/image-20240105120315242.png)

## 3. Boot整合Security

第一步：在pom.xml添加如下

```xml
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-security</artifactId>
</dependency>
```

引入依赖后我们在尝试去访问之前的接口就会自动跳转到一个SpringSecurity的默认登录页面，默认用户名是user，密码会输出在控制台，必须登录之后才能对接口进行访问。

第二步：重新运行SecurityApplication引导类，浏览器访问如下

`http://localhost:8080/hello`

![img](img/1689586875122-342dde8e-e49d-487e-b0de-aff6de710f14.jpg)

访问时会自动被下面的链接拦截

`http://localhost:8080/login`

第三步：在idea控制台查看登录的密码，用户名是user

![img](img/1689586875323-2b6e10a6-de98-4663-91e9-6c8822b2d18c.jpg)

登录之后，就会自动跳到并访问下面的地址

`http://localhost:8080/hello`

第四步：默认的退出登录接口如下

`http://localhost:8080/logout`

![img](img/1689586875526-ca1abcec-fa99-43a0-b611-54e048031fea.jpg)

# 认证

springsecurity的权限管理，是先认证后授权，所以我们先学习认证这一部分。流程图如下，注意下图的jwt指的是 json web token，jwt是登录校验的时候用的技术，可以根据指定的算法进行信息的加密和解密。

![img](img/1689586875709-6eb3d7ed-ee86-4245-9b5f-5bbe49b7144b.jpg)

## 1. springsecurity原理

SpringSecurity的原理其实就是一个过滤器链，内部包含了提供各种功能的过滤器。例如快速入门案例里面使用到的三种过滤器，如下图

![img](img/1689586875966-f65f8796-6a23-4978-bfe1-ee7bc9eb3f8c.jpg)

1. UsernamePasswordAuthenticationFilter：负责处理我们在登陆页面填写了用户名密码后的登陆请求。入门案例的认证工作主要有它负责。
2. ExceptionTranslationFilter：处理过滤器链中抛出的任何AccessDeniedException和AuthenticationException。
3. FilterSecurityInterceptor：负责权限校验的过滤器。

> 注意上图，橙色部分表示认证，黄色部分表示异常处理，红色部分表示授权。

如何查看security提供的过滤器有哪几个，或者叫哪几种，如下

![img](img/1689586876191-670a9f1f-9977-469b-b21d-349356b8588a.jpg)

## 2. 认证流程

我们来详细学一下上面 `1. springsecurity原理`的橙色部分，也就是认证那部分的知识

1. Authentication接口：它的实现类，表示当前访问系统的用户，封装了用户相关信息。
2. AuthenticationManager接口：定义了认证Authentication的方法。
3. UserDetailsService接口：加载用户特定数据的核心接口。里面定义了一个根据用户名查询用户信息的方法。
4. UserDetails接口：提供核心用户信息。通过UserDetailsService根据用户名获取处理的用户信息要封装成UserDetails对象返回。然后将这些信息封装到Authentication对象中。

![img](img/1689586876374-c2b3efaa-da08-48b1-85b9-f862639ddf9d.jpg)

## 3. 自定义security的思路

在 `快速入门`中，我们在Boot项目里面引入了Security依赖，实现了当我们访问某个业务接口时，会被Security的login接口拦截，但是如果我们不想使用Security默认的登录页面，那么怎么办呢，还有springsecurity的校验，我们希望是根据数据库来做校验，那么怎么实现呢。我们需要实现如下

1. 【登录-未实现】
   - 自定义登录接口。用于调用ProviderManager的方法进行认证 如果认证通过生成jwt，然后把用户信息存入redis中。
   - 自定义UserDetailsService接口的实现类。在这个实现类中去查询数据库。

2. 【校验-未实现】
   - 定义Jwt认证过滤器。用于获取token，然后解析token获取其中的userid，还需要从redis中获取用户信息，然后存入SecurityContextHolder。

## 4. 自定义security的实现

### 4.1 项目搭建

新建maven项目

![img](img/1689586876873-4ce8d9db-2180-4708-9b05-2e432a6d5e7b.jpg)

maven坐标pom.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.lrw</groupId>
    <artifactId>Security02_TokenDeom</artifactId>
    <version>1.0-SNAPSHOT</version>

    <properties>
        <maven.compiler.source>8</maven.compiler.source>
        <maven.compiler.target>8</maven.compiler.target>
    </properties>

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.5.0</version>
    </parent>

    <dependencies>

        <!--还要引入这个，不然后面javax.servlet依赖找不到-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <!--springboot整合springsecurity-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-security</artifactId>
        </dependency>

        <!--redis依赖-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-redis</artifactId>
        </dependency>

        <!--fastjson依赖-->
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>fastjson</artifactId>
            <version>1.2.33</version>
        </dependency>

        <!--jwt依赖-->
        <dependency>
            <groupId>io.jsonwebtoken</groupId>
            <artifactId>jjwt</artifactId>
            <version>0.9.0</version>
        </dependency>

        <!--引入MybatisPuls依赖-->
        <dependency>
            <groupId>com.baomidou</groupId>
            <artifactId>mybatis-plus-boot-starter</artifactId>
            <version>3.5.1</version>
        </dependency>

        <!--引入mysql驱动的依赖-->
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
        </dependency>

        <!--引入Lombok依赖，方便实体类开发-->
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
        </dependency>

        <!--引入Junit，用于测试-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
        </dependency>
    </dependencies>

</project>
```

启动类TokenApplication.java

```java
@SpringBootApplication
public class TokenApplication {

    public static void main(String[] args) {
        SpringApplication.run(TokenApplication.class,args);
    }
    
}
```

测试接口HelloController.java

```java
@RestController
public class HelloController {

  	@RequestMapping("/hello")
  	public String hello(){
  			return "欢迎，开始你新的学习旅程吧";
  	}

}
```

### 4.2 工具类相关

FastJsonRedisSerializer.java

```java
/**
 * Redis使用FastJson序列化
 */
public class FastJsonRedisSerializer<T> implements RedisSerializer<T> {

    public static final Charset DEFAULT_CHARSET = Charset.forName("UTF-8");

    private Class<T> clazz;

    static {
        ParserConfig.getGlobalInstance().setAutoTypeSupport(true);
    }

    public FastJsonRedisSerializer(Class<T> clazz) {
        super();
        this.clazz = clazz;
    }

    @Override
    public byte[] serialize(T t) throws SerializationException {
        if (t == null) {
            return new byte[0];
        }
        return JSON.toJSONString(t, SerializerFeature.WriteClassName).getBytes(DEFAULT_CHARSET);
    }

    @Override
    public T deserialize(byte[] bytes) throws SerializationException {
        if (bytes == null || bytes.length <= 0) {
            return null;
        }
        String str = new String(bytes, DEFAULT_CHARSET);

        return JSON.parseObject(str, clazz);
    }


    protected JavaType getJavaType(Class<?> clazz) {
        return TypeFactory.defaultInstance().constructType(clazz);
    }
}
```

RedisConfig.java

```java
@Configuration
public class RedisConfig {

    @Bean
    @SuppressWarnings(value = {"unchecked", "rawtypes"})
    public RedisTemplate<Object, Object> redisTemplate(RedisConnectionFactory connectionFactory) {
        RedisTemplate<Object, Object> template = new RedisTemplate<>();
        template.setConnectionFactory(connectionFactory);

        FastJsonRedisSerializer serializer = new FastJsonRedisSerializer(Object.class);

        // 使用StringRedisSerializer来序列化和反序列化redis的key值
        template.setKeySerializer(new StringRedisSerializer());
        template.setValueSerializer(serializer);

        // Hash的key也采用StringRedisSerializer的序列化方式
        template.setHashKeySerializer(new StringRedisSerializer());
        template.setHashValueSerializer(serializer);

        template.afterPropertiesSet();
        return template;
    }
}
```

RedisCache.java

```java
/**
 * redis工具类
 */
@SuppressWarnings(value = {"unchecked", "rawtypes"})
@Component
public class RedisCache {
    @Autowired
    public RedisTemplate redisTemplate;

    /**
     * 缓存基本的对象，Integer、String、实体类等
     *
     * @param key   缓存的键值
     * @param value 缓存的值
     */
    public <T> void setCacheObject(final String key, final T value) {
        redisTemplate.opsForValue().set(key, value);
    }

    /**
     * 缓存基本的对象，Integer、String、实体类等
     *
     * @param key      缓存的键值
     * @param value    缓存的值
     * @param timeout  时间
     * @param timeUnit 时间颗粒度
     */
    public <T> void setCacheObject(final String key, final T value, final Integer timeout, final TimeUnit timeUnit) {
        redisTemplate.opsForValue().set(key, value, timeout, timeUnit);
    }

    /**
     * 设置有效时间
     *
     * @param key     Redis键
     * @param timeout 超时时间
     * @return true=设置成功；false=设置失败
     */
    public boolean expire(final String key, final long timeout) {
        return expire(key, timeout, TimeUnit.SECONDS);
    }

    /**
     * 设置有效时间
     *
     * @param key     Redis键
     * @param timeout 超时时间
     * @param unit    时间单位
     * @return true=设置成功；false=设置失败
     */
    public boolean expire(final String key, final long timeout, final TimeUnit unit) {
        return redisTemplate.expire(key, timeout, unit);
    }

    /**
     * 获得缓存的基本对象。
     *
     * @param key 缓存键值
     * @return 缓存键值对应的数据
     */
    public <T> T getCacheObject(final String key) {
        ValueOperations<String, T> operation = redisTemplate.opsForValue();
        return operation.get(key);
    }

    /**
     * 删除单个对象
     *
     * @param key
     */
    public boolean deleteObject(final String key) {
        return redisTemplate.delete(key);
    }

    /**
     * 删除集合对象
     *
     * @param collection 多个对象
     * @return
     */
    public long deleteObject(final Collection collection) {
        return redisTemplate.delete(collection);
    }

    /**
     * 缓存List数据
     *
     * @param key      缓存的键值
     * @param dataList 待缓存的List数据
     * @return 缓存的对象
     */
    public <T> long setCacheList(final String key, final List<T> dataList) {
        Long count = redisTemplate.opsForList().rightPushAll(key, dataList);
        return count == null ? 0 : count;
    }

    /**
     * 获得缓存的list对象
     *
     * @param key 缓存的键值
     * @return 缓存键值对应的数据
     */
    public <T> List<T> getCacheList(final String key) {
        return redisTemplate.opsForList().range(key, 0, -1);
    }

    /**
     * 缓存Set
     *
     * @param key     缓存键值
     * @param dataSet 缓存的数据
     * @return 缓存数据的对象
     */
    public <T> BoundSetOperations<String, T> setCacheSet(final String key, final Set<T> dataSet) {
        BoundSetOperations<String, T> setOperation = redisTemplate.boundSetOps(key);
        Iterator<T> it = dataSet.iterator();
        while (it.hasNext()) {
            setOperation.add(it.next());
        }
        return setOperation;
    }

    /**
     * 获得缓存的set
     *
     * @param key
     * @return
     */
    public <T> Set<T> getCacheSet(final String key) {
        return redisTemplate.opsForSet().members(key);
    }

    /**
     * 缓存Map
     *
     * @param key
     * @param dataMap
     */
    public <T> void setCacheMap(final String key, final Map<String, T> dataMap) {
        if (dataMap != null) {
            redisTemplate.opsForHash().putAll(key, dataMap);
        }
    }

    /**
     * 获得缓存的Map
     *
     * @param key
     * @return
     */
    public <T> Map<String, T> getCacheMap(final String key) {
        return redisTemplate.opsForHash().entries(key);
    }

    /**
     * 往Hash中存入数据
     *
     * @param key   Redis键
     * @param hKey  Hash键
     * @param value 值
     */
    public <T> void setCacheMapValue(final String key, final String hKey, final T value) {
        redisTemplate.opsForHash().put(key, hKey, value);
    }

    /**
     * 获取Hash中的数据
     *
     * @param key  Redis键
     * @param hKey Hash键
     * @return Hash中的对象
     */
    public <T> T getCacheMapValue(final String key, final String hKey) {
        HashOperations<String, String, T> opsForHash = redisTemplate.opsForHash();
        return opsForHash.get(key, hKey);
    }

    /**
     * 删除Hash中的数据
     *
     * @param key
     * @param hkey
     */
    public void delCacheMapValue(final String key, final String hkey) {
        HashOperations hashOperations = redisTemplate.opsForHash();
        hashOperations.delete(key, hkey);
    }

    /**
     * 获取多个Hash中的数据
     *
     * @param key   Redis键
     * @param hKeys Hash键集合
     * @return Hash对象集合
     */
    public <T> List<T> getMultiCacheMapValue(final String key, final Collection<Object> hKeys) {
        return redisTemplate.opsForHash().multiGet(key, hKeys);
    }

    /**
     * 获得缓存的基本对象列表
     *
     * @param pattern 字符串前缀
     * @return 对象列表
     */
    public Collection<String> keys(final String pattern) {
        return redisTemplate.keys(pattern);
    }
}
```

ResponseResult.java

```java
@JsonInclude(JsonInclude.Include.NON_NULL)
public class ResponseResult<T> {
    /**
     * 状态码
     */
    private Integer code;
    /**
     * 提示信息，如果有错误时，前端可以获取该字段进行提示
     */
    private String msg;
    /**
     * 查询到的结果数据，
     */
    private T data;

    public ResponseResult(Integer code, String msg) {
        this.code = code;
        this.msg = msg;
    }

    public ResponseResult(Integer code, T data) {
        this.code = code;
        this.data = data;
    }

    public Integer getCode() {
        return code;
    }

    public void setCode(Integer code) {
        this.code = code;
    }

    public String getMsg() {
        return msg;
    }

    public void setMsg(String msg) {
        this.msg = msg;
    }

    public T getData() {
        return data;
    }

    public void setData(T data) {
        this.data = data;
    }

    public ResponseResult(Integer code, String msg, T data) {
        this.code = code;
        this.msg = msg;
        this.data = data;
    }
}
```

JwtUtil.java

```java
/**
 * JWT工具类
 */
public class JwtUtil {

    //有效期为
    public static final Long JWT_TTL = 60 * 60 * 1000L;// 60 * 60 *1000  一个小时
    //设置秘钥明文, 注意长度必须大于等于6位
    public static final String JWT_KEY = "huanfqc";

    public static String getUUID() {
        String token = UUID.randomUUID().toString().replaceAll("-", "");
        return token;
    }

    /**
     * 生成jtw
     *
     * @param subject token中要存放的数据（json格式）
     * @return
     */
    public static String createJWT(String subject) {
        JwtBuilder builder = getJwtBuilder(subject, null, getUUID());// 设置过期时间
        return builder.compact();
    }

    /**
     * 生成jtw
     *
     * @param subject   token中要存放的数据（json格式）
     * @param ttlMillis token超时时间
     * @return
     */
    public static String createJWT(String subject, Long ttlMillis) {
        JwtBuilder builder = getJwtBuilder(subject, ttlMillis, getUUID());// 设置过期时间
        return builder.compact();
    }

    private static JwtBuilder getJwtBuilder(String subject, Long ttlMillis, String uuid) {
        SignatureAlgorithm signatureAlgorithm = SignatureAlgorithm.HS256;
        SecretKey secretKey = generalKey();
        long nowMillis = System.currentTimeMillis();
        Date now = new Date(nowMillis);
        if (ttlMillis == null) {
            ttlMillis = JwtUtil.JWT_TTL;
        }
        long expMillis = nowMillis + ttlMillis;
        Date expDate = new Date(expMillis);
        return Jwts.builder()
                .setId(uuid)              //唯一的ID
                .setSubject(subject)   // 主题  可以是JSON数据
                .setIssuer("huanf")     // 签发者
                .setIssuedAt(now)      // 签发时间
                .signWith(signatureAlgorithm, secretKey) //使用HS256对称加密算法签名, 第二个参数为秘钥
                .setExpiration(expDate);
    }

    /**
     * 创建token
     *
     * @param id
     * @param subject
     * @param ttlMillis
     * @return
     */
    public static String createJWT(String id, String subject, Long ttlMillis) {
        JwtBuilder builder = getJwtBuilder(subject, ttlMillis, id);// 设置过期时间
        return builder.compact();
    }

    /**
     * 生成加密后的秘钥 secretKey
     *
     * @return
     */
    public static SecretKey generalKey() {
        byte[] encodedKey = Base64.getDecoder().decode(JwtUtil.JWT_KEY);
        SecretKey key = new SecretKeySpec(encodedKey, 0, encodedKey.length, "AES");
        return key;
    }

    /**
     * 解析
     *
     * @param jwt
     * @return
     * @throws Exception
     */
    public static Claims parseJWT(String jwt) throws Exception {
        SecretKey secretKey = generalKey();
        return Jwts.parser()
                .setSigningKey(secretKey)
                .parseClaimsJws(jwt)
                .getBody();
    }

}
```

WebUtils.java

```java
public class WebUtils {
    /**
     * 将字符串渲染到客户端
     *
     * @param response 渲染对象
     * @param string   待渲染的字符串
     * @return null
     */
    public static String renderString(HttpServletResponse response, String string) {
        try {
            response.setStatus(200);
            response.setContentType("application/json");
            response.setCharacterEncoding("utf-8");
            response.getWriter().print(string);
        } catch (IOException e) {
            e.printStackTrace();
        }
        return null;
    }
}
```

User.java

```java
@Data
@AllArgsConstructor
@NoArgsConstructor
@TableName("sys_user")
public class User implements Serializable {
    private static final long serialVersionUID = -40356785423868312L;

    /**
     * 主键
     */
    @TableId
    private Long id;
    /**
     * 用户名
     */
    private String userName;
    /**
     * 昵称
     */
    private String nickName;
    /**
     * 密码
     */
    private String password;
    /**
     * 账号状态（0正常 1停用）
     */
    private String status;
    /**
     * 邮箱
     */
    private String email;
    /**
     * 手机号
     */
    private String phonenumber;
    /**
     * 用户性别（0男，1女，2未知）
     */
    private String sex;
    /**
     * 头像
     */
    private String avatar;
    /**
     * 用户类型（0管理员，1普通用户）
     */
    private String userType;
    /**
     * 创建人的用户id
     */
    private Long createBy;
    /**
     * 创建时间
     */
    private Date createTime;
    /**
     * 更新人
     */
    private Long updateBy;
    /**
     * 更新时间
     */
    private Date updateTime;
    /**
     * 删除标志（0代表未删除，1代表已删除）
     */
    private Integer delFlag;
}
```

上面的代码总体如下图所示

![img](img/1689586877241-1649537f-e9ad-4ae7-993b-7f0996ab3770.jpg)

### 4.3 数据库相关准备

第一步：数据库校验用户。从之前的分析我们可以知道，我们自定义了一个UserDetailsService，让SpringSecurity使用我们的UserDetailsService。我们自己的UserDetailsService可以从数据库中查询用户名和密码。我们先创建一个用户表， 建表语句如下：

> 注意: 要想让用户的密码是明文存储，需要在密码前加{noop}，作用是例如等下在浏览器登陆的时候就可以用huanf作为用户名，112233作为密码来登陆了

```sql
create database if not exists sangeng_security;
use sangeng_security;

CREATE TABLE `sys_user` (
  `id` BIGINT(20) NOT NULL AUTO_INCREMENT COMMENT '主键',
  `user_name` VARCHAR(64) NOT NULL DEFAULT 'NULL' COMMENT '用户名',
  `nick_name` VARCHAR(64) NOT NULL DEFAULT 'NULL' COMMENT '昵称',
  `password` VARCHAR(64) NOT NULL DEFAULT 'NULL' COMMENT '密码',
  `status` CHAR(1) DEFAULT '0' COMMENT '账号状态（0正常 1停用）',
  `email` VARCHAR(64) DEFAULT NULL COMMENT '邮箱',
  `phonenumber` VARCHAR(32) DEFAULT NULL COMMENT '手机号',
  `sex` CHAR(1) DEFAULT NULL COMMENT '用户性别（0男，1女，2未知）',
  `avatar` VARCHAR(128) DEFAULT NULL COMMENT '头像',
  `user_type` CHAR(1) NOT NULL DEFAULT '1' COMMENT '用户类型（0管理员，1普通用户）',
  `create_by` BIGINT(20) DEFAULT NULL COMMENT '创建人的用户id',
  `create_time` DATETIME DEFAULT NULL COMMENT '创建时间',
  `update_by` BIGINT(20) DEFAULT NULL COMMENT '更新人',
  `update_time` DATETIME DEFAULT NULL COMMENT '更新时间',
  `del_flag` INT(11) DEFAULT '0' COMMENT '删除标志（0代表未删除，1代表已删除）',
  PRIMARY KEY (`id`)
) ENGINE=INNODB AUTO_INCREMENT=2 DEFAULT CHARSET=utf8mb4 COMMENT='用户表';

insert into sys_user values (1,'admin','管理员','{noop}123456','0',DEFAULT,DEFAULT,DEFAULT,DEFAULT,'0',DEFAULT,DEFAULT,DEFAULT,DEFAULT,DEFAULT);
insert into sys_user values (2,'huanf','涣沷a靑惷','{noop}112233','0',DEFAULT,DEFAULT,DEFAULT,DEFAULT,'1',DEFAULT,DEFAULT,DEFAULT,DEFAULT,DEFAULT);
```

第二步：在pom.xml添加如下

```xml
<!--引入MybatisPuls依赖-->
<dependency>
	<groupId>com.baomidou</groupId>
	<artifactId>mybatis-plus-boot-starter</artifactId>
	<version>3.4.3</version>
</dependency>

<!--引入mysql驱动的依赖-->
<dependency>
	<groupId>mysql</groupId>
	<artifactId>mysql-connector-java</artifactId>
</dependency>
```

第三步：application.yml

```yml
spring:
  datasource:
    url: jdbc:mysql://localhost:3306/sangeng_security?characterEncoding=utf-8&serverTimezone=UTC
    username: root
    password: 123456
    driver-class-name: com.mysql.cj.jdbc.Driver
  # 如果不配置redis的连接信息，默认就是本地的6379端口
  redis:
    host: 127.0.0.1
    port: 6379
mybatis-plus:
  # 配置MenuMapper.xml文件的路径
  # 也可以不写，因为默认就是在类加载路径(resouces)下的mapper目录的任意层级的后缀为xml的文件，都会被扫描到
  mapper-locations: classpath*:/mapper/**/*.xml
```

第四步：mapper.UserMapper.java 接口

```java
public interface UserMapper extends BaseMapper<User> {
}
```

第五步：指定扫描mapper包

```java
@MapperScan("com.lrw.mapper")
```

第六步：在pom.xml添加如下

```xml
<!--引入Junit，用于测试-->
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-test</artifactId>
</dependency>
```

第七步：MapperTest.java类。测试mybatis-plus是否正常

```java
@SpringBootTest
public class UserMapperTest {

    @Autowired
    private UserMapper userMapper;

    @Test
    public void testUserMapper(){
        //查询所有用户
        List<User> users = userMapper.selectList(null);
        System.out.println(users);
    }

}
```

第八步：运行MapperTest类的testUserMapper方法，看是否能查到数据库的所有用户。到此，可以确定数据库是没问题的，环境到此就准备好了

![img](img/1689586877662-856e4f5a-794b-49f2-87ae-2f07a99b4f7a.jpg)

### 4.4 自定义security的认证实现

==从数据库查询用户名和密码==

上面我们已经把准备工作做好了，包括搭建、代码、数据库。接下来我们会实现让security在认证的时候，根据我们数据库的用户和密码进行认证，也就是被security拦截业务接口，出现登录页面之后，我们需要通过输入数据库里的用户和密码来登录，而不是使用security默认的用户和密码进行登录

思路：只需要新建一个实现类，在这个实现类里面实现Security官方的UserDetailsService接口，然后重写里面的loadUserByUsername方法

注意：重写好loadUserByUsername方法之后，我们需要把拿到 '数据库与用户输入的数据' 进行比对的结果，也就是user对象这个结果封装成能被 'Security官方的UserDetailsService接口' 接收的类型，例如可以封装成我们下面写的LoginUser类型。然后才能伪装好数据，给Security官方的认证机制去对比user对象与数据库的结果是否匹配。Security官方的认证机制会拿LoginUser类的方法数据(数据库拿，不再用默认的)，跟我们封装过去的user对象进行匹配，要使匹配一致，就证明认证通过，也就是用户在浏览器页面输入的用户名和密码能被Security认证通过，就不再拦截该用户去访问我们的业务接口

第一步：在domain目录新建LoginUser类，作为UserDetails接口(Security官方提供的接口)的实现类

```java
@Data
@NoArgsConstructor
public class LoginUser implements UserDetails {

    private User user;

    public LoginUser(User user, List<String> permissions) {
        this.user = user;
        this.permissions = permissions;
    }

    @Override
    public Collection<? extends GrantedAuthority> getAuthorities() {
        return null;
    }

    @Override
    public String getPassword() {
        return user.getPassword();
    }

    @Override
    public String getUsername() {
        return user.getUserName();
    }

    @Override
    public boolean isAccountNonExpired() {
        return true;
    }

    @Override
    public boolean isAccountNonLocked() {
        return true;
    }

    @Override
    public boolean isCredentialsNonExpired() {
        return true;
    }

    @Override
    public boolean isEnabled() {
        return true;
    }
}
```

第二步：MyUserDetailServiceImpl 类

```java
@Service
public class MyUserDetailServiceImpl implements UserDetailsService {

    @Autowired
    private UserMapper userMapper;

    @Override
    public UserDetails loadUserByUsername(String userName) throws UsernameNotFoundException {
        //查询用户信息。我们写的userMapper接口里面是空的，所以调用的是mybatis-plus提供的方法
        LambdaQueryWrapper<User> queryWrapper = new LambdaQueryWrapper<>();
        //eq方法表示等值匹配，第一个参数是数据库的用户名，第二个参数是我们传进来的用户名，这两个参数进行比较是否相等
        queryWrapper.eq(User::getUserName, userName);
        User user = userMapper.selectOne(queryWrapper);
        //如果用户传进来的用户名，但是数据库没有这个用户名，就会导致我们是查不到的情况，那么就进行下面的判断。避免程序安全问题
        if (Objects.isNull(user)) { //判断user对象是否为空。当在数据库没有查到数据时，user就会为空，也就会进入这个判断
            throw new RuntimeException("用户名不存在！");
        }

        //把查询到的user结果，封装成UserDetails类型，然后返回。
        //但是由于UserDetails是个接口，所以我们先需要在domino目录新建LoginUser类，作为UserDetails的实现类，再写下面那行
        return new LoginUser(user);
    }
}

```

第三步：测试。运行TokenApplication引导类，浏览器输入如下，然后我们输入一下登录的用户名和密码，看是不是根据数据库来进行认证

![img](img/1689586877812-d005ecd5-979d-4cae-b441-807e6530d14e.jpg)

![img](img/1689586877949-6f588c18-8c36-4e85-b8d5-97abb1b5ffc0.jpg)

### 4.5 密码加密校验问题

上面我们实现了自定义Security的认证机制，让Security根据数据库的数据，来认证用户输入的数据是否正确。但是当时存在一个问题，就是我们在数据库存入用户表的时候，插入的huanf用户的密码是 {noop}112233，为什么用112233不行呢

原因：SpringSecurity默认使用的PasswordEncoder要求数据库中的密码格式为：{加密方式}密码。对应的就是{noop}112233，实际表示的是112233

但是我们在数据库直接暴露112233为密码，会造成安全问题，所以我们需要把加密后的1234的密文当作密码，此时用户在浏览器登录时输入1234，我们如何确保用户能够登录进去呢，答案是SpringSecurity默认的密码校验，替换为SpringSecurity为我们提供的BCryptPasswordEncoder

我们只需要使用把BCryptPasswordEncoder对象注入Spring容器中，SpringSecurity就会使用该PasswordEncoder来进行密码校验。我们可以定义一个SpringSecurity的配置类，SpringSecurity要求这个配置类要继承WebSecurityConfigurerAdapter。

#### 4.5.1 加密测试

第一步：在config目录新建 SecurityConfig 类，配置PasswordEncoder。作用是根据原文，生成一个密文

```java
@Configuration
@EnableGlobalMethodSecurity(prePostEnabled = true)
public class SecurityConfig extends WebSecurityConfigurerAdapter {

    /**
     * 把BCryptPasswordEncoder对象注入Spring容器中，SpringSecurity就会使用该PasswordEncoder来进行密码校验
     */
    @Bean
    public PasswordEncoder passwordEncoder(){
        return new BCryptPasswordEncoder();
    }
}
```

第二步：测试。在MyTest类，添加如下，然后运行 BCryptPasswordEncoderTest方法

```java
@SpringBootTest
public class MyTest {

    @Autowired
    private PasswordEncoder passwordEncoder;

    @Test
    public void BCryptPasswordEncoderTest(){
        //模拟用户输入的密码
        String encode1 = passwordEncoder.encode("123456");
        //再模拟一次用户输入的密码
        String encode2 = passwordEncoder.encode("123456");
        //虽然这两次的密码都是一样的，但是加密后是不一样的。每次运行，对同一原文都会有不同的加密结果
        //原因:会添加随机的盐，加密结果=盐+原文+加密。达到每次加密后的密文都不相同的效果
        System.out.println(encode1);
        System.out.println(encode2);
    }
}
```

![img](img/1689586878080-d2e1f168-a4c4-4ff7-8e53-a7bc7dfb1788.jpg)

#### 4.5.2 校验测试

第一步(已做可跳过)：在 SecurityConfig 类，配置PasswordEncoder。作用是根据原文，生成一个密文

```java
@Configuration
@EnableGlobalMethodSecurity(prePostEnabled = true)
public class SecurityConfig extends WebSecurityConfigurerAdapter {

    /**
     * 把BCryptPasswordEncoder对象注入Spring容器中，SpringSecurity就会使用该PasswordEncoder来进行密码校验
     */
    @Bean
    public PasswordEncoder passwordEncoder(){
        return new BCryptPasswordEncoder();
    }
}
```

第二步：测试。在MyTest类，添加如下，然后运行 yyTestBCryptPasswordEncoder 方法

```java
@Test
public void yyTestBCryptPasswordEncoder() {

  //模拟用户输入了1234(第一个参数)，然后我们去跟数据库的密文进行比较(第二个参数)
  boolean result = passwordEncoder.matches("1234", "$2a$10$zOitKu6UNk.b/iPFTtIj2u80sH/dfJI9vFr57qhDGteuXj/Wl8uSy");

  //看一下比对结果
  System.out.println(result);

}
```

#### 4.5.3 security密码加密

第一步(已做可跳过)：在 SecurityConfig 类，配置PasswordEncoder。作用是根据原文，生成一个密文

```java
@Configuration
@EnableGlobalMethodSecurity(prePostEnabled = true)
public class SecurityConfig extends WebSecurityConfigurerAdapter {

    /**
     * 把BCryptPasswordEncoder对象注入Spring容器中，SpringSecurity就会使用该PasswordEncoder来进行密码校验
     */
    @Bean
    public PasswordEncoder passwordEncoder(){
        return new BCryptPasswordEncoder();
    }
}
```

第二步：修改数据库表密码字段为密文。

第三步：登录测试。

### 4.6 jwt工具类实现token生成与解析

刚刚我们在测试类学习了密码的加密和校验，我们下面会先学如何使用jwt工具类实现token的生成与解析

首先是在pom.xml添加如下(已做可跳过)

```xml
<!--jwt依赖-->
<dependency>
	<groupId>io.jsonwebtoken</groupId>
	<artifactId>jjwt</artifactId>
	<version>0.9.0</version>
</dependency>
```

#### 4.6.1 token生成测试

```java
@Test
public void createJWTTest() {
		String jwt = JwtUtil.createJWT("123456");
		System.out.println(jwt);
}
```

![image-20240105143912828](img/image-20240105143912828.png)

#### 4.6.2 token解析测试

```java
@Test
public void parseJWTTest() throws Exception {
  	String jwt = "eyJhbGciOiJIUzI1NiJ9.eyJqdGkiOiJjMmViMjVlODllZGU0YTQwODhiYjJjMGIzYWUyYzFkMyIsInN1YiI6IjEyMzQ1NiIsImlzcyI6Imh1YW5mIiwiaWF0IjoxNzA0NDM2NjkxLCJleHAiOjE3MDQ0NDAyOTF9.-Hxziaq8rneM4b4PItpv3vv6aD3P8YlxUSa5mEzAUZo";
  	Claims claims = JwtUtil.parseJWT(jwt);
  	String subject = claims.getSubject();
  	System.out.println(subject);
}
```

![image-20240105144202118](img/image-20240105144202118.png)

### 4.7 登录接口

- 我们需要自定义登陆接口，也就是在controller目录新建LoginController类，在controller方法里面去调用service接口，在service接口实现AuthenticationManager去进行用户的认证，注意，我们定义的controller方法要让SpringSecurity对这个接口放行(如果不放行的话，会被SpringSecurity拦截)，让用户访问这个接口的时候不用登录也能访问。

- 在service接口中我们通过AuthenticationManager的authenticate方法来进行用户认证，所以需要在SecurityConfig中配置把AuthenticationManager注入容器

- 认证成功的话要生成一个jwt，放入响应中返回。并且为了让用户下回请求时能通过jwt识别出具体的是哪个用户，我们需要把用户信息存入redis，可以把用户id作为key。

SecurityConfig.java添加如下配置

```java
@Bean
@Override
public AuthenticationManager authenticationManagerBean() throws Exception {
  return super.authenticationManagerBean();
}

@Override
protected void configure(HttpSecurity http) throws Exception {
  http
    //由于是前后端分离项目，所以要关闭csrf
    .csrf().disable()
    //由于是前后端分离项目，所以session是失效的，我们就不通过Session获取SecurityContext
    .sessionManagement().sessionCreationPolicy(SessionCreationPolicy.STATELESS)
    .and()
    //指定让spring security放行登录接口的规则
    .authorizeRequests()
    // 对于登录接口 anonymous表示允许匿名访问
    .antMatchers("/user/login").anonymous()
    // 除上面外的所有请求全部需要鉴权认证
    .anyRequest().authenticated();
}
```

第三步：在service目录新建 LoginService 接口，写入如下

```java
public interface ILoginService {
    ResponseResult login(User user);
}
```

第四步：在service目录新建 impl.LoginServiceImpl 类，写入如下

```java
@Service
public class LoginServiceImpl implements ILoginService {

    @Autowired
    //先在SecurityConfig，使用@Bean注解重写官方的authenticationManagerBean类，然后这里才能注入成功
    private AuthenticationManager authenticationManager;

    @Autowired
    //RedisCache是我们在utils目录写好的类
    private RedisCache redisCache;

    @Override
    public ResponseResult login(User user) {
        //用户在登录页面输入的用户名和密码
        UsernamePasswordAuthenticationToken authenticationToken = new UsernamePasswordAuthenticationToken(user.getUserName(), user.getPassword());

        //获取AuthenticationManager的authenticate方法来进行用户认证（调用到myUserDetailServiceImpl的方法查数据库）
        Authentication authenticate = authenticationManager.authenticate(authenticationToken);

        //判断上面那行的authenticate是否为null，如果是则认证没通过，就抛出异常
        if (Objects.isNull(authenticate)) {
            throw new RuntimeException("登录失败");
        }

        //如果认证通过，就使用userid生成一个jwt，然后把jwt存入ResponseResult后返回
        LoginUser loginUser = (LoginUser) authenticate.getPrincipal();
        String userId = loginUser.getUser().getId().toString();
        String jwt = JwtUtil.createJWT(userId);

        //把完整的用户信息存入redis，其中userid作为key，注意存入redis的时候加了前缀 login:
        Map<String, String> map = new HashMap<>();
        map.put("token", jwt);
        redisCache.setCacheObject("login:" + userId, loginUser);
        return new ResponseResult(200, "登录成功", map);
    }

}
```

第五步：在controller目录新建 LoginController 类，写入如下

```java
@RestController
@RequestMapping("/user")
public class LoginController {

    @Autowired
    private ILoginService loginService;

    @PostMapping("/login")
    public ResponseResult login(@RequestBody User user) {
        return loginService.login(user);
    }

}
```

第六步：测试。打开postman，发送下面的POST请求

![img](img/1689586879423-4efbfef9-4611-4cf8-9e29-1e0e52dfc14d.jpg)

### 4.8 认证过滤器

要实现Jwt认证过滤器，我们需要获取token，然后解析token获取其中的userId，还需要从redis中获取用户信息，然后存入SecurityContextHolder

为什么要有redis参与：是为了防止过了很久之后，浏览器没有关闭，拿着token也能访问，这样不安全

认证过滤器的作用是什么：上面我们实现登录接口的时，当某个用户登录之后，该用户就会有一个token值，我们可以通过认证过滤器，由于有token值，并且token值认证通过，也就是证明是这个用户的token值，那么该用户访问我们的业务接口时，就不会被Security拦截。==简单理解作用就是登录过的用户可以访问我们的业务接口，拿到对应的资源==

第一步：定义过滤器。JwtAuthenticationTokenFilter.java

```java
@Component
public class JwtAuthenticationTokenFilter extends OncePerRequestFilter {

    @Autowired
    private RedisCache redisCache;

    @Override
    protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response, FilterChain filterChain) throws ServletException, IOException {
        //获取token，指定你要获取的请求头叫什么
        String token = request.getHeader("token");
        //判空，不一定所有的请求头中都有token，所以上面那行的token可能为空
        if (!StringUtils.hasText(token)) {
            //如果请求没有携带token，那么就不需要解析token，不需要获取用户信息，直接放行就可以，后续security会进行认证判断
            filterChain.doFilter(request, response);
            //return之后，就不会走下面那些代码
            return;
        }
        //解析token
        String userId;
        try {
            Claims claims = JwtUtil.parseJWT(token);
            userId = claims.getSubject();
        } catch (Exception e) {
            e.printStackTrace();
            throw new RuntimeException("token非法");
        }
        //从redis中获取用户信息
        String redisKey = "login:" + userId;
        LoginUser loginUser = redisCache.getCacheObject(redisKey);
        //判断获取到的用户信息是否为空，因为redis里面可能并不存在这个用户信息，例如缓存过期了
        if (Objects.isNull(loginUser)) {
            throw new RuntimeException("用户未登录");
        }

        //把最终的LoginUser用户信息，通过setAuthentication方法，存入SecurityContextHolder
        UsernamePasswordAuthenticationToken authenticationToken =
                //第一个参数是LoginUser用户信息，第二个参数是凭证(null)，第三个参数是权限信息(null)
                new UsernamePasswordAuthenticationToken(loginUser, null, null); // 这个方法内部设置了“已认证”状态
        SecurityContextHolder.getContext().setAuthentication(authenticationToken);

        //全部做完之后，就放行
        filterChain.doFilter(request, response);
    }
}
```

第二步: 修改SecurityConfig类为如下，配置JwtAuthenticationTokenFilter

```java
@Configuration
@EnableGlobalMethodSecurity(prePostEnabled = true)
public class SecurityConfig extends WebSecurityConfigurerAdapter {

    @Autowired
    private JwtAuthenticationTokenFilter jwtAuthenticationTokenFilter;

    /**
     * 把BCryptPasswordEncoder对象注入Spring容器中，SpringSecurity就会使用该PasswordEncoder来进行密码校验
     */
    @Bean
    public PasswordEncoder passwordEncoder(){
        return new BCryptPasswordEncoder();
    }

    @Bean
    @Override
    public AuthenticationManager authenticationManagerBean() throws Exception {
        return super.authenticationManagerBean();
    }

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http
                //由于是前后端分离项目，所以要关闭csrf
                .csrf().disable()
                //由于是前后端分离项目，所以session是失效的，我们就不通过Session获取SecurityContext
                .sessionManagement().sessionCreationPolicy(SessionCreationPolicy.STATELESS)
                .and()
                //指定让spring security放行登录接口的规则
                .authorizeRequests()
                // 对于登录接口 anonymous表示允许匿名访问
                .antMatchers("/user/login").anonymous()
                // 除上面外的所有请求全部需要鉴权认证
                .anyRequest().authenticated();

        // 配置认证过滤器，第一个参数是上面注入的我们在filter目录写好的类，第二个参数表示你想添加到哪个过滤器之前
        http.addFilterBefore(jwtAuthenticationTokenFilter, UsernamePasswordAuthenticationFilter.class);

    }
```

第三步：登录。打开postman，发送如下的POST请求，作用是先登录一个用户，拿到用户对应的token值

![img](img/1689586879536-90627f28-3804-49bc-977f-1c2bb436d337.jpg)

第四步：测试。继续在你的postman，发送如下GET请求，作用是拿着刚刚的token值，去访问我们的业务接口，看会不会被Security拦截，如果不会拦截，那么就说明认证过滤器生效了，使用场景就是简单理解就是登录过的用户可以访问我们的业务接口，拿到对应的资源

注意，由于token值我们是存在redis，所以是有默认过期时间的。注意在请求头那里，key要写token，value要写你复制的token值，然后点击发送请求。这个token值实际上就是使用jwt工具类把112233密码加密后的密文，不信你翻一下前面笔记看当时112233的密文，长得是不是跟现在的token值格式一样

![img](img/1689586879663-eeb5cf30-1538-4c1b-9a3a-cffa3fb5dab2.jpg)

### 4.9 退出登录

实现起来也比较简单，只需要定义一个登出接口，然后获取SecurityContextHolder中的认证信息，删除redis中对应的数据即可

第一步：LoginService接口中增加登出方法

```java
public interface ILoginService {
    ResponseResult login(User user);

    ResponseResult logout();
}
```

第二步：LoginServiceImpl实现类中增加登出方法

```java
@Override
public ResponseResult logout() {
  // 获取我们在JwtAuthenticationTokenFilter类写的SecurityContextHolder对象中的用户id
  Authentication authentication = SecurityContextHolder.getContext().getAuthentication();
  LoginUser loginUser = (LoginUser) authentication.getPrincipal();
  String userId = String.valueOf(loginUser.getUser().getId());
  //根据用户id，删除redis中的token值，注意我们的key是被 login: 拼接过的，所以下面写完整key的时候要带上 longin:
  redisCache.deleteObject("login:" + userId);
  return new ResponseResult(200, "注销成功");
}
```

第三步：LoginController类增加退出登录接口

```java
@RequestMapping("/logout")
public ResponseResult logout() {
  return loginService.logout();
}
```

第四步：登录。打开postman，发送如下的POST请求，作用是先登录一个用户，获取用户对应的token值

![img](img/1689586879944-1d3e001a-5aa9-40d8-9f85-7e6cb3044d0a.jpg)

第五步：测试。继续在你的postman，发送如下GET请求，作用是拿着刚刚的token值，去访问我们的业务接口，看在有登录状态的情况下，能不能访问

![img](img/1689586880085-7cb8d9ea-0f01-47bf-bbb9-3108502a6617.jpg)

第六步: 登出。继续在你的postman，发送如下GET请求，作用是退出登录，然后去访问我们的业务接口，看在没有登录状态的情况下，能不能访

![img](img/1689586880218-b56a9bcc-159f-4a60-89f3-4c01b72d8d00.jpg)

![img](img/1689586880345-a33ab474-8505-4996-8be4-19a45ceca458.jpg)



# 授权

## 1. 权限系统的作用

为什么要设计权限系统 ?

例如要设计一个图书管理系统，普通学生账号的权限不能使用书籍编辑、删除的功能，普通学生能使用的功能仅仅是浏览页面，但是，如果是图书管理员用户，那么就能使用所有权限。简单理解就是我们需要不同的用户使用不同的功能，这就是权限系统要实现的效果

虽然前端也可以去判断用户的权限来选择是否显示某些功能的页面或组件，但是不安全，因为如果有人知道了对应功能的接口地址就可以不通过前端，直接去发送请求来实现相关功能操作，所以我们还需要在后端进行用户权限的判断，判断当前用户是否有相应的权限，必须具有所需权限才能进行相应的操作

==前端防君子，后端防小人==

## 2. 授权的基本流程

在SpringSecurity中，会使用默认的FilterSecurityInterceptor来进行权限校验。在FilterSecurityInterceptor中会从SecurityContextHolder获取其中的Authentication，然后获取其中的权限信息。当前用户是否拥有访问当前资源所需的权限

![img](img/1689586875966-f65f8796-6a23-4978-bfe1-ee7bc9eb3f8c-170444026439145.jpg)

所以我们在项目中只需要把当前登录用户的权限信息也存入Authentication，然后设置我们的资源所需要的权限即可

## 3. 自定义访问路径的权限

SpringSecurity为我们提供了基于注解的权限控制方案，这也是我们项目中主要采用的方式。我们可以使用注解去指定访问对应的资源所需的权限

第一步：在SecurityConfig配置类添加如下，作用是开启相关配置

```java
@EnableGlobalMethodSecurity(prePostEnabled = true)
```

第二步：开启了相关配置之后，就能使用@PreAuthorize等注解了。在HelloController类的hello方法，添加如下注解，其中test表示自定义权限的名字

```java
@PreAuthorize("hasAuthority('test')")
```

## 4. 带权限访问的实现

权限信息：有特殊含义的字符串

我们前面在登录时，会调用到MyUserDetailServiceImpl类的loadUserByUsername方法，当时我们写loadUserByUsername方法时，只写了查询用户数据信息的代码，还差查询用户权限信息的代码。在登录完之后，因为携带了token，所以需要在JwtAuthenticationTokenFilter类添加 '获取权限信息封装到Authentication中' 的代码，添加到UsernamePasswordAuthenticationToken的第三个参数里面，我们当时第三个参数传的是null。

第一步：给HelloController类的 /hello 路径添加了权限限制，只有用户具有叫test的权限，才能访问这个路径

![img](img/1689586880749-e59a3647-9678-47c6-b1f1-21b06474d0eb.jpg)

第二步：MyUserDetailServiceImpl的loadUserByUsername方法增加查询用户权限信息的代码

```java
@Service
public class MyUserDetailServiceImpl implements UserDetailsService {

    @Autowired
    private UserMapper userMapper;
    @Autowired
    private MenuMapper menuMapper;

    @Override
    public UserDetails loadUserByUsername(String userName) throws UsernameNotFoundException {
        //查询用户信息。我们写的userMapper接口里面是空的，所以调用的是mybatis-plus提供的方法
        LambdaQueryWrapper<User> queryWrapper = new LambdaQueryWrapper<>();
        //eq方法表示等值匹配，第一个参数是数据库的用户名，第二个参数是我们传进来的用户名，这两个参数进行比较是否相等
        queryWrapper.eq(User::getUserName, userName);
        User user = userMapper.selectOne(queryWrapper);
        //如果用户传进来的用户名，但是数据库没有这个用户名，就会导致我们是查不到的情况，那么就进行下面的判断。避免程序安全问题
        if (Objects.isNull(user)) { //判断user对象是否为空。当在数据库没有查到数据时，user就会为空，也就会进入这个判断
            throw new RuntimeException("用户名不存在！");
        }

        //由于我们自定义了3个权限，所以用List集合存储。注意权限实际就是'有特殊含义的字符串'，所以下面的三个字符串就是自定义的
        //下面那行就是我们的权限集合，等下还要在LoginUser类做权限集合的转换
        List<String> list = new ArrayList<>(Arrays.asList("test","adminAuth","huanfAuth"));

        //把查询到的user结果，封装成UserDetails类型，然后返回。
        //但是由于UserDetails是个接口，所以我们先需要在domino目录新建LoginUser类，作为UserDetails的实现类，再写下面那行
        return new LoginUser(user, list); //这里传了第二个参数，表示的是权限信息
    }
}
```

![image-20240105155159975](img/image-20240105155159975.png)

第三步：封装权限信息。把LoginUser类修改为如下，主要是增加了把用户权限字符串的集合，转换封装在authorities变量里面

```java
@Data
@NoArgsConstructor
public class LoginUser implements UserDetails {

    private User user;

    private List<String> permissions;

    public LoginUser(User user, List<String> permissions) {
        this.user = user;
        this.permissions = permissions;
    }

    //我们把这个List写到外面这里了，注意成员变量名一定要是authorities，不然会出现奇奇怪怪的问题
    @JSONField(serialize = false) //这个注解的作用是不让下面那行的成员变量序列化存入redis，避免redis不支持而报异常
    private List<SimpleGrantedAuthority> authorities;

    @Override
    public Collection<? extends GrantedAuthority> getAuthorities() {
        //当authorities集合为空，就说明是第一次，就需要转换，当不为空就说明不是第一次，就不需要转换直接返回
        if (authorities != null) {
            return authorities;
        }

        /*// 第一种权限集合的转换写法如下，传统的方式
        for (String permission : permissions) {
            SimpleGrantedAuthority simpleGrantedAuthority = new SimpleGrantedAuthority(permission);
            authorities.add(simpleGrantedAuthority);
        }*/

        // 第二种权限集合的转换写法如下，函数式编程 + stream流 的方式，双引号表示方法引用
        authorities = permissions.stream().map(SimpleGrantedAuthority::new).collect(Collectors.toList());

        return authorities;
    }

    @Override
    public String getPassword() {
        return user.getPassword();
    }

    @Override
    public String getUsername() {
        return user.getUserName();
    }

    @Override
    public boolean isAccountNonExpired() {
        return true;
    }

    @Override
    public boolean isAccountNonLocked() {
        return true;
    }

    @Override
    public boolean isCredentialsNonExpired() {
        return true;
    }

    @Override
    public boolean isEnabled() {
        return true;
    }
}
```

![img](img/1689586880999-d3a4edd2-5767-42fd-8487-1b1824cfd8d7.jpg)

第四步：把JwtAuthenticationTokenFilter类修改为如下，主要是补充了前面没写的第三个参数，写成第三步封装好的权限信息

```java
@Component
public class JwtAuthenticationTokenFilter extends OncePerRequestFilter {

    @Autowired
    private RedisCache redisCache;

    @Override
    protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response, FilterChain filterChain) throws ServletException, IOException {
        //从请求头中获取token
        String token = request.getHeader("token");
        //判空，不一定所有的请求头中都有token，所以上面那行的token可能为空
        if (!StringUtils.hasText(token)) {
            //如果请求没有携带token，那么就不需要解析token，不需要获取用户信息，直接放行就可以，后续security会进行认证判断
            filterChain.doFilter(request, response);
            //return之后，就不会走下面那些代码
            return;
        }
        //解析token
        String userId;
        try {
            Claims claims = JwtUtil.parseJWT(token);
            userId = claims.getSubject();
        } catch (Exception e) {
            e.printStackTrace();
            throw new RuntimeException("token非法");
        }
        //从redis中获取用户信息
        String redisKey = "login:" + userId;
        LoginUser loginUser = redisCache.getCacheObject(redisKey);
        //判断获取到的用户信息是否为空，因为redis里面可能并不存在这个用户信息，例如缓存过期了
        if (Objects.isNull(loginUser)) {
            throw new RuntimeException("用户未登录");
        }

        //把最终的LoginUser用户信息，通过setAuthentication方法，存入SecurityContextHolder
        UsernamePasswordAuthenticationToken authenticationToken =
                //第一个参数是LoginUser用户信息，第二个参数是凭证(null)，第三个参数是权限信息(null)
                new UsernamePasswordAuthenticationToken(loginUser, null, loginUser.getAuthorities()); // 这个方法内部设置了“已认证”状态
        SecurityContextHolder.getContext().setAuthentication(authenticationToken);

        //全部做完之后，就放行
        filterChain.doFilter(request, response);
    }
}
```

![img](img/1689586881137-5d58d0e8-9b86-4e44-8cec-4c6231753e05.jpg)

第五步：测试。打开postman，发送如下的POST请求，作用是先登录一个用户，这样就能生成这个用户对应的token值

![img](img/1689586881275-9ce613bb-a5ed-4826-a36b-fe657078e5bc.jpg)

第六步：测试。继续在你的postman，发送如下GET请求，作用是拿着刚刚的token值，去访问我们的业务接口，看在有登录状态的情况下，能不能访问

![img](img/1689586881448-84e3cd7a-4088-4380-8872-f3d7cdd9f7f9.jpg)

## 5. RBAC权限模型

### 5.1 介绍

刚刚我们实现了只有当用户具备某种权限，才能访问我们的某个业务接口。但是存在一个问题，我们在给用户设置权限的时候，是写死的，在真正的开发中，我们是需要从数据库查询权限信息，下面就来学习如何从数据库查询权限信息，然后封装给用户。这个功能需要先准备好数据库和java代码，所以，下面的 '授权-RBAC权限模型' 都是在围绕这个功能进行学习，直到实现这个功能

![img](img/1689586881992-b5b33df1-e277-4ad8-9952-d4dc73d8147d.jpg)

**RBAC权限模型 (Role-Based Access Control)** ，是权限系统用到的经典模型，基于角色的权限控制。该模型由以下五个主要组成部分构成:

1. 用户：在系统中代表具体个体的实体，可以是人员、程序或其他实体。用户需要访问系统资源
2. 角色：角色是权限的集合，用于定义一组相似权限的集合。角色可以被赋予给用户，从而授予用户相应的权限
3. 权限：权限表示系统中具体的操作或功能，例如读取、写入、执行等。每个权限定义了对系统资源的访问规则
4. 用户-角色映射：用户-角色映射用于表示用户与角色之间的关系。通过为用户分配适当的角色，用户可以获得与角色相关联的权限
5. 角色-权限映射：角色-权限映射表示角色与权限之间的关系。每个角色都被分配了一组权限，这些权限决定了角色可执行的操作

截止目前，我们数据库只有1张表， sys_user 用户表，下面我们会新增4张表，分别是权限表(每条数据是单个'粒度细的权限')、角色表(每条数据是多个'粒度细的权限')、角色表与权限表的中间表、用户表与角色表的中间表。总共5张表，组成了RBAC模型，中间表的作用是维护两张表的多对多关系

### 5.2 数据库表的创建

```sql
create database if not exists sangeng_security;
use sangeng_security;
CREATE TABLE `sys_menu` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT,
  `menu_name` varchar(64) NOT NULL DEFAULT 'NULL' COMMENT '菜单名',
  `path` varchar(200) DEFAULT NULL COMMENT '路由地址',
  `component` varchar(255) DEFAULT NULL COMMENT '组件路径',
  `visible` char(1) DEFAULT '0' COMMENT '菜单状态（0显示 1隐藏）',
  `status` char(1) DEFAULT '0' COMMENT '菜单状态（0正常 1停用）',
  `perms` varchar(100) DEFAULT NULL COMMENT '权限标识',
  `icon` varchar(100) DEFAULT '#' COMMENT '菜单图标',
  `create_by` bigint(20) DEFAULT NULL,
  `create_time` datetime DEFAULT NULL,
  `update_by` bigint(20) DEFAULT NULL,
  `update_time` datetime DEFAULT NULL,
  `del_flag` int(11) DEFAULT '0' COMMENT '是否删除（0未删除 1已删除）',
  `remark` varchar(500) DEFAULT NULL COMMENT '备注',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=2 DEFAULT CHARSET=utf8mb4 COMMENT='权限表';

CREATE TABLE `sys_role` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT,
  `name` varchar(128) DEFAULT NULL,
  `role_key` varchar(100) DEFAULT NULL COMMENT '角色权限字符串',
  `status` char(1) DEFAULT '0' COMMENT '角色状态（0正常 1停用）',
  `del_flag` int(1) DEFAULT '0' COMMENT 'del_flag',
  `create_by` bigint(200) DEFAULT NULL,
  `create_time` datetime DEFAULT NULL,
  `update_by` bigint(200) DEFAULT NULL,
  `update_time` datetime DEFAULT NULL,
  `remark` varchar(500) DEFAULT NULL COMMENT '备注',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=3 DEFAULT CHARSET=utf8mb4 COMMENT='角色表';

CREATE TABLE `sys_role_menu` (
  `role_id` bigint(200) NOT NULL AUTO_INCREMENT COMMENT '角色ID',
  `menu_id` bigint(200) NOT NULL DEFAULT '0' COMMENT '菜单id',
  PRIMARY KEY (`role_id`,`menu_id`)
) ENGINE=InnoDB AUTO_INCREMENT=2 DEFAULT CHARSET=utf8mb4;

CREATE TABLE `sys_user_role` (
  `user_id` bigint(200) NOT NULL AUTO_INCREMENT COMMENT '用户id',
  `role_id` bigint(200) NOT NULL DEFAULT '0' COMMENT '角色id',
  PRIMARY KEY (`user_id`,`role_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

insert into sys_user_role values (2,1);
insert into sys_role values
(1,'经理','ceo',0,0,default,default,default,default,default),
(2,'程序员','coder',0,0,default,default,default,default,default);
insert into sys_role_menu values (1,1),(1,2);
insert into sys_menu values
(1,'部门管理','dept','system/dept/index',0,0,'system:dept:list','#',default,default,default,default,default,default),
(2,'测试','test','system/test/index',0,0,'system:test:list','#',default,default,default,default,default,default)
```

查询用户权限信息

```sql
# 通过用户id去查询这个用户具有的权限列表。也就是根据userid查询perms，并且限制条件为role和menu都必须正常状态么也就是等于0
SELECT 
	DISTINCT m.`perms`
FROM
	sys_user_role ur
	LEFT JOIN `sys_role` r ON ur.`role_id` = r.`id`
	LEFT JOIN `sys_role_menu` rm ON ur.`role_id` = rm.`role_id`
	LEFT JOIN `sys_menu` m ON m.`id` = rm.`menu_id`
WHERE
	user_id = 2
	AND r.`status` = 0
	AND m.`status` = 0
```

![img](img/1689586882259-bcd7580b-bd5f-475d-b49a-17c05dbc18ce.jpg)

> 注意: 可以查到两条数据，这两条数据就是等下数据库返回给我们的权限字符串

### 5.3 查询数据库的权限信息

Menu 实体类

```java
/**
 * 权限表(也叫菜单表)的实体类
 */
@TableName(value="sys_menu") //指定表名，避免等下mybatisplus的影响
@Data
@AllArgsConstructor
@NoArgsConstructor
@JsonInclude(JsonInclude.Include.NON_NULL)
//Serializable是官方提供的，作用是将对象转化为字节序列
public class Menu implements Serializable {
    private static final long serialVersionUID = -54979041104113736L;

    @TableId
    private Long id;
    /**
     * 菜单名
     */
    private String menuName;
    /**
     * 路由地址
     */
    private String path;
    /**
     * 组件路径
     */
    private String component;
    /**
     * 菜单状态（0显示 1隐藏）
     */
    private String visible;
    /**
     * 菜单状态（0正常 1停用）
     */
    private String status;
    /**
     * 权限标识
     */
    private String perms;
    /**
     * 菜单图标
     */
    private String icon;

    private Long createBy;

    private Date createTime;

    private Long updateBy;

    private Date updateTime;
    /**
     * 是否删除（0未删除 1已删除）
     */
    private Integer delFlag;
    /**
     * 备注
     */
    private String remark;
}
```

MenuMapper 接口

```java
public interface MenuMapper extends BaseMapper<Menu> {
    //由于是多表联查，mybatisplus的BaseMapper接口没有提供，我们需要自定义方法，所以需要创建对应的mapper文件，定义对应的sql语句
    List<String> selectPermsByUserId(Long id);
}
```

MenuMapper.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd" >
<mapper namespace="com.lrw.mapper.MenuMapper">

    <select id="selectPermsByUserId" resultType="java.lang.String">
        SELECT
            DISTINCT m.`perms`
        FROM
            sys_user_role ur
                LEFT JOIN `sys_role` r ON ur.`role_id` = r.`id`
                LEFT JOIN `sys_role_menu` rm ON ur.`role_id` = rm.`role_id`
                LEFT JOIN `sys_menu` m ON m.`id` = rm.`menu_id`
        WHERE
            user_id = #{userid}
          AND r.`status` = 0
          AND m.`status` = 0
    </select>

</mapper>
```

测试。检查mybatismlus能不能拿到数据库的权限字符串。

```java
@Autowired
private MenuMapper menuMapper;

@Test
public void testSelectPermsByUserId(){
	//L表示Long类型
	List<String> list = menuMapper.selectPermsByUserId(2L);
	System.out.println(list);
}
```

![img](img/1689586882642-52ea49c5-7654-43f4-88e3-c6b111b821c9.jpg)

### 5.4 RBAC权限模型的实现

不要把RBAC模型想得很难，其实难的话只是数据库表的设计和SQL语句的编写，需要5张表。数据库设计好之后就很简单了，使用mybatis-plus去查询数据库表的权限字符串(例如我们的权限字符串是放在sys_menu表)，然后把你查到的数据去替换死数据就好了。我们只剩最后一步，就是替换死数据，如下

```java
@Service
public class MyUserDetailServiceImpl implements UserDetailsService {

    @Autowired
    private UserMapper userMapper;
    @Autowired
    private MenuMapper menuMapper;

    @Override
    public UserDetails loadUserByUsername(String userName) throws UsernameNotFoundException {
        //查询用户信息。我们写的userMapper接口里面是空的，所以调用的是mybatis-plus提供的方法
        LambdaQueryWrapper<User> queryWrapper = new LambdaQueryWrapper<>();
        //eq方法表示等值匹配，第一个参数是数据库的用户名，第二个参数是我们传进来的用户名，这两个参数进行比较是否相等
        queryWrapper.eq(User::getUserName, userName);
        User user = userMapper.selectOne(queryWrapper);
        //如果用户传进来的用户名，但是数据库没有这个用户名，就会导致我们是查不到的情况，那么就进行下面的判断。避免程序安全问题
        if (Objects.isNull(user)) { //判断user对象是否为空。当在数据库没有查到数据时，user就会为空，也就会进入这个判断
            throw new RuntimeException("用户名不存在！");
        }

        //由于我们自定义了3个权限，所以用List集合存储。注意权限实际就是'有特殊含义的字符串'，所以下面的三个字符串就是自定义的
        //下面那行就是我们的权限集合，等下还要在LoginUser类做权限集合的转换
        // List<String> list = new ArrayList<>(Arrays.asList("test","adminAuth","huanfAuth"));
        List<String> list = menuMapper.selectPermsByUserId(user.getId());

        //把查询到的user结果，封装成UserDetails类型，然后返回。
        //但是由于UserDetails是个接口，所以我们先需要在domino目录新建LoginUser类，作为UserDetails的实现类，再写下面那行
        return new LoginUser(user, list); //这里传了第二个参数，表示的是权限信息
    }
}
```

![img](img/1689586882778-1b974f30-c8e3-4593-a081-330a8405effa.jpg)

hello接口添加权限，测试

## 6.权限校验的方法

学的是HelloController类的 `@PreAuthorize`注解 的三个方法

我们前面都是使用@PreAuthorize注解，然后在在其中使用的是hasAuthority方法进行校验。SpringSecurity还为我们提供了其它方法例如: hasAnyAuthority，hasRole，hasAnyRole等

### 6.1 hasAuthority方法

hasAuthority方法：执行到了SecurityExpressionRoot的hasAuthority，内部其实是调用authentication的getAuthorities方法获取用户的权限列表。然后判断我们存入的方法参数数据在权限列表中。hasAnyAuthority方法可以传入多个权限，只有用户有其中任意一个权限都可以访问对应资源

hasAuthority方法的执行流程如下图，图比较多，请从上往下查看

![img](img/1689586885764-72155704-ba7a-4d17-8feb-3cd474fd32a1.jpg)

![img](img/1689586885926-9de01534-ab7d-4153-89b3-7f087f560dfa.jpg)

![img](img/1689586886073-9487ef1e-1085-4d96-a9fd-43adca4b540c.jpg)

![img](img/1689586886239-534ab5b9-7023-47d0-9b79-89bb0dec9758.jpg)

![img](img/1689586886382-9e028350-b4d3-4aba-a9e6-227590646b47.jpg)

### 6.2 hasAnyAuthority方法

hasAnyAuthority方法的执行流程跟上面的hasAuthority方法是一样的，只是传参不同。所以重点演示传参，执行流程是真的一模一样，把上面的hasAuthority方法的执行流程认真看一次就行了。hasAuthority方法只能传入一个参数，也就是一个权限字符串。hasAnyAuthority方法可以传入多个参数，也就是多个权限字符串，只要用户具有其中任意一个权限就能访问指定业务接口

```java
@PreAuthorize("hasAnyAuthority('zidingyi','huanf','system:dept:list')"); //传入3个自定义权限
```

### 6.3 hasRole方法

```java
@PreAuthorize("hasRole('system:dept:list')") //只能传一个权限字符串，多传会报红线
```

首先是执行流程，如下几张图，从上往下看

![img](img/1689586886881-a743d733-8c7b-46d7-99b9-140df7301898.jpg)

![img](img/1689586887015-870697dc-000b-40ac-88df-054c4600011c.jpg)

![img](img/1689586887160-8d38aca6-643c-4104-a7b9-b6d8df5db58d.jpg)

![img](img/1689586887309-ea19e33c-7576-4f27-a056-5f18eb76cabb.jpg)

### 6.4 hasAnyRole方法

执行流程跟刚刚的hasRole方法是一模一样的，只是传参不同。把上面的hasRole方法的执行流程认真看一次就行了。hasRole方法只能传入一个参数，也就是一个权限字符串。hasAnyRole方法可以传入多个参数，也就是多个权限字符串，只要用户具有其中任意一个权限就能访问指定业务接口

```java
@PreAuthorize("hasAnyRole('zidingyi','huanf','system:dept:list')")
```

### 6.5 自定义权限校验的方法

在上面的源码中，我们知道security校验权限的PreAuthorize注解，其实就是获取用户权限，然后跟业务接口的权限进行比较，最后返回一个布尔类型。自定义一个权限校验方法的话，就需要新建一个类，在类里面定义一个方法，按照前面学习的三种方法的定义格式，然后返回值是布尔类型。

第一步：在 expression中新建MyExpressionRoot 类

```java
/**
 * 自定义权限校验的方法
 */
@Component("myEX")
public class MyExpressionRoot {

    //自定义权限校验的方法
    public boolean myHasAuthority(String authority){

        //获取用户具有的权限字符串，有可能用户具有多个权限字符串，所以获取后是一个集合
        Authentication authentication = SecurityContextHolder.getContext().getAuthentication();
        //LoginUser是我们在domain目录写好的实体类
        LoginUser loginUser = (LoginUser) authentication.getPrincipal();
        List<String> permissions = loginUser.getPermissions();

        //判断用户权限集合中，是否存在跟业务接口(业务接口的权限字符串会作为authority形参传进来)一样的权限
        return permissions.contains(authority);
    }

}
```

在SPEL表达式来获取容器中bean的名字。修改HelloController类为如下

```java
@RestController
public class HelloController {

    @RequestMapping("/hello")
    // @PreAuthorize("hasAuthority('system:test:list')")
    @PreAuthorize("@myEX.myHasAuthority('system:dept:list')")
    public String hello() {
        return "欢迎，开始你新的学习旅程吧";
    }

}
```

### 6.6 基于配置的权限控制

前面学习的权限控制是基于@PreAuthorize注解来完成的，如何使用配置的方式，也就是在配置类当中，来实现权限控制，如下

第一步：在HelloController类添加如下，作用是新增一个接口

```java
//基于配置的权限控制
@RequestMapping("/configAuth")
public ResponseResult xx(){
	return new ResponseResult(200,"访问成功");
}
```

第二步：把SecurityConfig类修改为如下，主要就是添加权限控制相关的配置

```java
@Configuration
@EnableGlobalMethodSecurity(prePostEnabled = true)
public class SecurityConfig extends WebSecurityConfigurerAdapter {

    @Autowired
    private JwtAuthenticationTokenFilter jwtAuthenticationTokenFilter;
    @Autowired
    private AuthenticationEntryPointImpl authenticationEntryPoint;
    @Autowired
    private AccessDeniedHandlerImpl accessDeniedHandler;

    /**
     * 把BCryptPasswordEncoder对象注入Spring容器中，SpringSecurity就会使用该PasswordEncoder来进行密码校验
     */
    @Bean
    public PasswordEncoder passwordEncoder(){
        return new BCryptPasswordEncoder();
    }

    @Bean
    @Override
    public AuthenticationManager authenticationManagerBean() throws Exception {
        return super.authenticationManagerBean();
    }

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http
                //由于是前后端分离项目，所以要关闭csrf
                .csrf().disable()
                //由于是前后端分离项目，所以session是失效的，我们就不通过Session获取SecurityContext
                .sessionManagement().sessionCreationPolicy(SessionCreationPolicy.STATELESS)
                .and()
                //指定让spring security放行登录接口的规则
                .authorizeRequests()
                
                // 对于登录接口 anonymous表示允许匿名访问
                .antMatchers("/user/login").anonymous()

                //基于配置的的权限控制。指定接口的地址，为HelloController类里面的/configAuth接口，指定权限为system:dept:list
                .antMatchers("/configAuth").hasAuthority("system:dept:list")
                //上一行的hasAuthority方法就是security官方提供的4种权限控制的方法之一
                
                // 除上面外的所有请求全部需要鉴权认证
                .anyRequest().authenticated();

        // 配置认证过滤器，第一个参数是上面注入的我们在filter目录写好的类，第二个参数表示你想添加到哪个过滤器之前
        http.addFilterBefore(jwtAuthenticationTokenFilter, UsernamePasswordAuthenticationFilter.class);

        // 配置异常处理器
        http.exceptionHandling()
                //配置认证失败的处理器
                .authenticationEntryPoint(authenticationEntryPoint)
                //配置授权失败的处理器
                .accessDeniedHandler(accessDeniedHandler);


        // 设置security运行跨域访问
        http.cors();

    }

}
```

![img](img/1689586888192-b61e1da0-dff2-41c6-8bb1-384ea1c7fb3f.jpg)

# 自定义异常处理

上面的我们学习了 '认证' 和 '授权'，实现了基本的权限管理，然后也学习了从数据库获取授权的 '授权-RBAC权限模型'，实现了从数据库获取用户具备的权限字符串。到此，我们完整地实现了权限管理的功能，但是，当认证或授权出现报错时，我们希望响应回来的json数据有实体类的code、msg、data这三个字段，怎么实现呢

我们需要学习Spring Security的异常处理机制，就可以在认证失败或者是授权失败的情况下也能和我们的接口一样返回相同结构的json，这样可以让前端能对响应进行统一的处理

![img](img/1689586875966-f65f8796-6a23-4978-bfe1-ee7bc9eb3f8c-17045062965369.jpg)

在SpringSecurity中，如果我们在认证或者授权的过程中出现了异常会被ExceptionTranslationFilter捕获到，如上图。在ExceptionTranslationFilter中会去判断是认证失败还是授权失败出现的异常，其中有如下两种情况

一、如果是==认证过程中==出现的异常会被封装成AuthenticationException然后调用AuthenticationEntryPoint对象的方法去进行异常处理。

二、如果是==授权过程中==出现的异常会被封装成AccessDeniedException然后调用AccessDeniedHandler对象的方法去进行异常处理。

**总结：如果我们需要自定义异常处理，我们只需要创建AuthenticationEntryPoint和AccessDeniedHandler的实现类对象，然后配置给SpringSecurity即可**

1. 认证异常

handler包下新建AuthenticationEntryPointImpl类

```java
/**
 * <p>
 * 自定义认证异常处理器
 * </p>
 *
 * @author 'luoriwang'
 * @since 2023-12-28
 */
@Component
public class AuthenticationEntryPointImpl implements AuthenticationEntryPoint {

    @Override
    public void commence(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse, AuthenticationException e) throws IOException, ServletException {
        ResponseResult result = new ResponseResult(HttpStatus.UNAUTHORIZED.value(), "用户认证失败，请重新登录");
        String jsonString = JSON.toJSONString(result);
        WebUtils.renderString(httpServletResponse, jsonString);
    }

}
```

2. 授权异常

handler包下新建AccessDeniedHandlerImpl类

```java
/**
 * <p>
 * 自定义授权异常处理器
 * </p>
 *
 * @author 'luoriwang'
 * @since 2023-12-28
 */
@Component
public class AccessDeniedHandlerImpl implements AccessDeniedHandler {

    @Override
    public void handle(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse, AccessDeniedException e) throws IOException, ServletException {
        ResponseResult result = new ResponseResult(HttpStatus.FORBIDDEN.value(), "您没有权限进行访问");
        String jsonString = JSON.toJSONString(result);
        WebUtils.renderString(httpServletResponse, jsonString);
    }

}
```

3. SecurityConfig配置异常处理类

```java
@Configuration
@EnableGlobalMethodSecurity(prePostEnabled = true)
public class SecurityConfig extends WebSecurityConfigurerAdapter {

    @Autowired
    private JwtAuthenticationTokenFilter jwtAuthenticationTokenFilter;
    @Autowired
    private AuthenticationEntryPointImpl authenticationEntryPoint;
    @Autowired
    private AccessDeniedHandlerImpl accessDeniedHandler;

    /**
     * 把BCryptPasswordEncoder对象注入Spring容器中，SpringSecurity就会使用该PasswordEncoder来进行密码校验
     */
    @Bean
    public PasswordEncoder passwordEncoder(){
        return new BCryptPasswordEncoder();
    }

    @Bean
    @Override
    public AuthenticationManager authenticationManagerBean() throws Exception {
        return super.authenticationManagerBean();
    }

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http
                //由于是前后端分离项目，所以要关闭csrf
                .csrf().disable()
                //由于是前后端分离项目，所以session是失效的，我们就不通过Session获取SecurityContext
                .sessionManagement().sessionCreationPolicy(SessionCreationPolicy.STATELESS)
                .and()
                //指定让spring security放行登录接口的规则
                .authorizeRequests()
                // 对于登录接口 anonymous表示允许匿名访问
                .antMatchers("/user/login").anonymous()
                // 除上面外的所有请求全部需要鉴权认证
                .anyRequest().authenticated();

        // 配置认证过滤器，第一个参数是上面注入的我们在filter目录写好的类，第二个参数表示你想添加到哪个过滤器之前
        http.addFilterBefore(jwtAuthenticationTokenFilter, UsernamePasswordAuthenticationFilter.class);

        // 配置异常处理器
        http.exceptionHandling()
                //配置认证失败的处理器
                .authenticationEntryPoint(authenticationEntryPoint)
                //配置授权失败的处理器
                .accessDeniedHandler(accessDeniedHandler);

    }

}
```

![img](img/1689586883655-e9ede9ef-d688-414a-bdc3-fa40d65e558f.jpg)

4. 测试。

认证异常

![img](img/1689586883864-79b413be-194a-4dbb-9cb4-cb40eab574bc.jpg)

授权异常

![img](img/1689586884156-95254bdb-a6fe-4539-9a29-1c979071db50.jpg)

# 跨域

## 1. 跨域的后端解决

由于我们的SpringSecurity负责所有请求和资源的管理，当请求经过SpringSecurity时，如果SpringSecurity不允许跨域，那么也是会被拦截，所以下面我们将学习并解决跨域问题。前面我们在测试时，是在postman测试，因此没有出现跨域问题的情况，postman只是负责发请求跟浏览器没关系

浏览器出于安全的考虑，使用 XMLHttpRequest 对象发起HTTP请求时必须遵守同源策略，否则就是跨域的HTTP请求，默认情况下是被禁止的。 同源策略要求源相同才能正常进行通信，即协议、域名、端口号都完全一致。 前后端分离项目，前端项目和后端项目一般都不是同源的，所以肯定会存在跨域请求的问题

我们要实现如下两个需求 

1、开启SpringBoot的允许跨域访问

2、开启SpringSecurity的允许跨域访问

第一步：开启SpringBoot的允许跨域访问。在 config 目录新建 CorsConfig 类

```java
/**
 * <p>
 * 允许跨域
 * </p>
 *
 * @author 'luoriwang'
 * @since 2023-12-30
 */
@Configuration
public class CorsConfig implements WebMvcConfigurer {

    //重写spring提供的WebMvcConfigurer接口的addCorsMappings方法
    @Override
    public void addCorsMappings(CorsRegistry registry) {
        // 设置允许跨域的路径
        registry.addMapping("/**")
                // 设置允许跨域请求的域名
                .allowedOriginPatterns("*")
                // 是否允许cookie
                .allowCredentials(true)
                // 设置允许的请求方式
                .allowedMethods("GET", "POST", "DELETE", "PUT")
                // 设置允许的header属性
                .allowedHeaders("*")
                // 跨域允许时间
                .maxAge(3600);
    }

}
```

第二步：开启SpringSecurity的允许跨域访问。在 SecurityConfig 中配置。

```java
@Configuration
@EnableGlobalMethodSecurity(prePostEnabled = true)
public class SecurityConfig extends WebSecurityConfigurerAdapter {

    @Autowired
    private JwtAuthenticationTokenFilter jwtAuthenticationTokenFilter;
    @Autowired
    private AuthenticationEntryPointImpl authenticationEntryPoint;
    @Autowired
    private AccessDeniedHandlerImpl accessDeniedHandler;

    /**
     * 把BCryptPasswordEncoder对象注入Spring容器中，SpringSecurity就会使用该PasswordEncoder来进行密码校验
     */
    @Bean
    public PasswordEncoder passwordEncoder(){
        return new BCryptPasswordEncoder();
    }

    @Bean
    @Override
    public AuthenticationManager authenticationManagerBean() throws Exception {
        return super.authenticationManagerBean();
    }

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http
                //由于是前后端分离项目，所以要关闭csrf
                .csrf().disable()
                //由于是前后端分离项目，所以session是失效的，我们就不通过Session获取SecurityContext
                .sessionManagement().sessionCreationPolicy(SessionCreationPolicy.STATELESS)
                .and()
                //指定让spring security放行登录接口的规则
                .authorizeRequests()
                // 对于登录接口 anonymous表示允许匿名访问
                .antMatchers("/user/login").anonymous()
                // 除上面外的所有请求全部需要鉴权认证
                .anyRequest().authenticated();

        // 配置认证过滤器，第一个参数是上面注入的我们在filter目录写好的类，第二个参数表示你想添加到哪个过滤器之前
        http.addFilterBefore(jwtAuthenticationTokenFilter, UsernamePasswordAuthenticationFilter.class);

        // 配置异常处理器
        http.exceptionHandling()
                //配置认证失败的处理器
                .authenticationEntryPoint(authenticationEntryPoint)
                //配置授权失败的处理器
                .accessDeniedHandler(accessDeniedHandler);


        // 设置security运行跨域访问
        http.cors();

    }

}
```

![img](img/1689586884287-d2400ff5-dae4-4129-b2ba-94081a4d3b2c.jpg)

# 防护CSRF攻击

在SecurityConfig类里面的configure方法里面，有一个配置如下，我们上面都没有去学习，下面就来了解一下

```java
http..csrf().disable(); //关闭csrf，可防护csrf攻击。如果不关闭的话
```

CSRF是指跨站请求伪造（Cross-site request forgery），是web常见的攻击之一，如图

![img](img/1689586888852-2edfebdb-9d54-4bce-8501-6f698d2e29ab.jpg)

详细看这篇博客: https://blog.csdn.net/freeking101/article/details/86537087

> 防护：SpringSecurity去防止CSRF攻击的方式就是通过csrf_token。后端会生成一个csrf_token，前端发起请求的时候需要携带这个csrf_token，后端会有过滤器进行校验，如果没有携带或者是伪造的就不允许访问

我们可以发现CSRF攻击依靠的是cookie中所携带的认证信息。但是在前后端分离的项目中我们的认证信息其实是token，而token并不是存储中cookie中，并且需要前端代码去把token设置到请求头中才可以，所以CSRF攻击也就不用担心了，前后端分离的项目，在配置类关闭csrf就能防范csrf攻击

















