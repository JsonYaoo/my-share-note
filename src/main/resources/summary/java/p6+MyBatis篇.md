# 十一、MyBatis篇

### 1.1. MyBatis 是什么？

1. Mybatis 是一个半 ORM（对象关系映射）框架，它内部封装了 JDBC，开发时只需要关注 SQL 语句本身，不需要花费精力去处理加载驱动、创建连接、创建 statement 等繁杂的过程，程序员直接编写原生态 sql，可以严格控制 sql 执行性能，灵活度高。
2. MyBatis 可以使用 XML 或注解来配置和映射原生信息，将 POJO 映射成数据库中的记录，避免了几乎所有的 JDBC 代码和手动设置参数以及获取结果集。

### 1.2. MyBatis 优缺点？

- **优点**：
  1. 基于SQL语句编程，相当灵活，不会对应用程序或者数据库的现有设计造成任何影响，SQL写在 XML 里，解除 sql 与程序代码的耦合，便于统一管理，且提供 XML 标签，支持编写动态 SQL 语句，并可重用。
  2. 与 JDBC 相比，减少了 50% 以上的代码量，消除了 JDBC 大量冗余的代码，不需要手动开关连接。
  3. 很好的与各种数据库兼容（因为 MyBatis 使用 JDBC 来连接数据库，所以只要 JDBC 支持的数据库，MyBatis 都支持）。
  4. 提供映射标签，支持对象与数据库的 ORM 字段关系映射，提供对象关系映射标签，支持对象关系组件维护。
  5. 能够与 Spring 很好的集成。
- **缺点**：
  1. SQL 语句的编写工作量较大，尤其当字段多、关联表多时，对开发人员编写 SQL 语句的功底有一定要求。
  2. SQL 语句依赖于数据库，导致数据库移植性差，不能随意更换数据库。

### 1.3. Mybatis 和 Hibernate 的区别？ 

- **1）概念上**：
  1. Hibernate，是一个开放源代码的对象关系映射框架，它对 JDBC 进行了非常轻量级的对象封装，建立对象与数据库表的映射，是一个全自动的、完全面向对象的持久层框架。
  2. Mybatis，是一个开源对象关系映射框架，原名 Ibatis，2010年由谷歌接管以后更名，是一个半自动化的持久层框架。
- **2）开发速度上**：
  1. Hibernate 开发中，sql 语句已经被封装，直接可以使用，加快系统开发。
  2. Mybatis 属于半自动化，sql需要手工完成，稍微繁琐。
  3. 但是，凡事都不是绝对的，如果对于庞大复杂的系统项目来说，复杂语句较多，hibernate 就不是好方案。
- **3）sql 性能上**：
  1. Hibernate 自动生成sql，有些语句较为繁琐，会多消耗一些性能。
  2. Mybatis 手动编写sql，可以避免不需要的查询，提高系统性能。
- **4）对象管理比对**：
  1. Hibernate，是完整的对象-关系映射的框架，开发工程中，无需过多关注底层实现，只要去管理对象即可。
  2. Mybatis 需要自行管理映射关系，所以，称之为半自动 ORM 映射工具。
     - **ORM**：Object Relational Mapping，对象关系映射，是一种为了解决关系型数据库数据与简单 Java对象（POJO）的映射关系的技术，简单来说就是，ORM 通过使用描述对象和数据库之间映射的元数据，将程序中的对象自动持久化到关系型数据库中。

#### 1、jdbc

```java
@Slf4j
public class UserExecutor {

    public static final String URL = "jdbc:mysql://localhost:3306/muse";
    public static final String USER = "root";
    public static final String PASSWORD = "root";

    public static void main(String[] args) throws Exception {
        //1.加载驱动程序
        Class.forName("com.mysql.jdbc.Driver");
        //2. 获得数据库连接
        Connection conn = DriverManager.getConnection(URL, USER, PASSWORD);
        ResultSet rs = null;
        PreparedStatement ps = null;
        try {
            //3.操作数据库，实现增删改查
            ps = conn.prepareStatement("SELECT name, age FROM tb_user where id = ?");
            ps.setInt(1, 2);
            rs = ps.executeQuery();
            //如果有数据，rs.next()返回true
            while (rs.next()) {
                System.out.println("姓名：" + rs.getString("name") + "，年龄：" + rs.getInt("age"));
            }
        } catch (SQLException e) {
            log.error("error", e);
        } finally {
            close(rs, ps, conn);
        }
    }

    private static void close(ResultSet rs, Statement stmt, Connection connection) {
        try {
            if (rs !=null && !rs.isClosed()) {
                rs.close();
            }
        } catch (SQLException e) {
            log.error("rs.close() error!", e);
        }

        try {
            if (stmt != null && stmt.isClosed()) {
                stmt.close();
            }
        } catch (SQLException e) {
            log.error("stmt.close() error!", e);
        }

        try {
            if (connection != null && connection.isClosed()) {
                connection.close();
            }
        } catch (SQLException e) {
            log.error("connection.close() error!", e);
        }

    }
}
```

#### 2、Hibernate

```java
public class UserExecutor {

    //保存用户的案例
    public static void main(String[] args) {
        Configuration configuration = new Configuration().configure("hibernate.cfg.xml");
        SessionFactory sessionFactory = configuration.buildSessionFactory();
        Session session = null;
        try {
            session = sessionFactory.openSession();
            User user = session.get(User.class, 2L);
            System.out.println("姓名：" + user.getName() + "，年龄：" + user.getAge());
        } finally {
            if (session != null) {
                //7. 释放资源
                session.close();
                sessionFactory.close();
            }
        }
    }
}
```

#### 3、Mybatis

```java
public class MessageExecuter {
    public static void main(String[] args) {
        sqlSession = SqlSessionFactoryUtil.openSqlSession();
        MessageMapper messageMapper = sqlSession.getMapper(MessageMapper.class);
        Message message = messageMapper.getMessageById(2L);
        System.out.println("getMessageById--->" + message);
    }
}
```

### 1.4. MyBatis 核心组件？

1. **SqlSessionFactoryBuilder**：构造器，根据配置信息或代码，来生成 SqlSessionFactory。
2. **SqlSessionFactory**：工厂，用来生成SqlSession。
3. **SqlSession**：会话，可以用来发送 SQL，去执行并返回结果，也可以获取 Mapper 的接口。
4. **SQL Mapper**：它由一个 Java 接口和 XML 文件/注解构成，需要给出对应的 SQL 和映射规则，负责发送 SQL 去执行，并返回结果。

### 1.5. MyBatis 使用？

#### 1、使用步骤

1. **第一步**：配置mybatis-config.xml：

   ```xml
   <configuration>
       <!-- 属性 -->
       <properties resource="sqlmap/mybatis/mysql/jdbc.properties"/>
       
       <!-- 配置 -->
       <settings>
           <!-- PARTIAL是默认值，只会自动映射，没有定义嵌套结果集映射的结果集 -->
           <setting name="autoMappingBehavior" value="PARTIAL"/>
   		<!-- 打印查询语句 -->
           <setting name="logImpl" value="STDOUT_LOGGING"/>
           <!-- 配置驼峰转下划线 数据库中的下划线，转换Java Bean中的驼峰-->
           <setting name="mapUnderscoreToCamelCase" value="true"/>
       </settings>
   
       <!-- 类型别名 -->
       <typeAliases>
           <package name="vo"/>
           <!-- <typeAlias type="vo.User" alias="user"/> -->
       </typeAliases>
   
       <!-- 类型处理器
       <typeHandlers/> -->
   
       <!-- 对象工厂
       <objectFactory type="" />   -->
   
       <!-- 对象包装工厂
       <objectWrapperFactory type="" />    -->
   
       <!-- 对象工厂
       <reflectorFactory type="" />    -->
   
       <!-- 插件
       <plugins>
           <plugin interceptor=""></plugin>
       </plugins> -->
   
       <!-- 配置数据库环境 -->
       <environments default="dev">
           <environment id="dev">
               <transactionManager type="JDBC"/> <!-- 事务管理器 -->
               <dataSource type="POOLED">
                   <property name="driver" value="${driver}"/>
                   <property name="url" value="${url}"/>
                   <property name="username" value="${username}"/>
                   <property name="password" value="${password}"/>
               </dataSource>
           </environment>
       </environments>
   
       <!-- 数据库厂商标识 -->
       <databaseIdProvider type="DB_VENDOR"/>
   
       <!-- 映射器 -->
       <mappers>
           <mapper resource="sqlmap/mybatis/mysql/UserMapper.xml"/>
           <mapper resource="sqlmap/mybatis/mysql/UserContactMapper.xml"/>
           <mapper resource="sqlmap/mybatis/mysql/MessageMapper.xml"/>
           <mapper resource="sqlmap/mybatis/mysql/MessageDetailMapper.xml"/>
       </mappers>
   </configuration>
   ```

2. **第二步**：配置 MessageMapper.xml：

   ```xml
   java id, msgid, status, content, deleted, createtime, update_time...
   ```

3. **第三步**：构建 Message 实体类和 MessageMapper 接口：

   ```java
   public interface MessageMapper { 
       Message getMessageById(Long id); 
       int insert(User user); 
       int delById(Long id);
   }
   ```

#### 2、select | 查询

##### 1）基础类型查询

```java
Message message = messageMapper.getMessageById(2L);

public interface MessageMapper {
    Message getMessageById(Long id); 
}
```

```xml
<select id="getMessageById" parameterType="long" resultMap="messageResult"> 
    select 
    	<include refid="allColumns"/> 
    from tb_message where id = #{id} 
</select>
```

##### 2）Map 类型查询

```java
Map<String, String> paramsMap = new HashMap<>(); 
paramsMap.put("id", "2");
paramsMap.put("msgId", "1001"); 
Message message = messageMapper.getMessageByMap(paramsMap);

public interface MessageMapper {
    Message getMessageByMap(Map<String, String> params); 
}
```

```xml
<select id="getMessageByMap" parameterType="map" resultMap="messageResult"> 
    select 
    	<include refid="allColumns"/> 
    from tb_message where id = #{id} and msg_id = #{msgId} 
</select>
```

##### 3）注解方式传递参数

```java
message = messageMapper.getMessageByAnnotation(2L, "1001");

public interface MessageMapper { 
    Message getMessageByAnnotation(@Param("id") Long id, @Param("msgId") String msgId); 
}
```

```xml
<select id="getMessageByAnnotation" resultMap="messageResult"> 
    select 
    	<include refid="allColumns"/> 
    from tb_message where id = #{id} and msg_id = #{msgId} 
</select>
```

##### 4）Java Bean 方式传递参数

```java
Message param = new Message(); 
param.setId(1L); 
param.setMsgId("1000"); 
message = messageMapper.getMessageByMessage(param);

public interface MessageMapper {
    Message getMessageByMessage(Message message);
}
```

```xml
<select id="getMessageByMessage" parameterType="vo.Message" resultMap="messageResult"> 
     select 
     	<include refid="allColumns"/> 
     from tb_message 
     where id = #{id} 
     and msg_id = #{msgId} 
</select>
```

> 总结：
>
> - 使用 Map 传递参数：
>   => 会导致业务可读性的丧失，后续维护困难，实际工作中应该尽量避免使用这种方式
> - 使用 @Param 注解：
>   => 如果参数 <= 5时，是最佳的传参方式，比 JavaBean 更直观，但是如果参数多，那么会造成接口参数膨胀，可读性和维护性差。
> - 使用 Java Bean 方式：
>   => 当参数个数 > 5时，建议采用这种方式。

#### 3、insert | 插入

##### 1）普通插入 | 主键不回填

```xml
<insert id="insert" parameterType="message" keyProperty="id"> 
    insert into tb_message(<include refid="updateAllColumns"/>) 
    values (#{msgId}, #{status}, #{content}, #{deleted}, #{createTime}) 
</insert>
```

##### 2）主键回填 | useGeneratedKeys="true"

```xml
<insert id="insertAndGetIdBack" parameterType="message" keyProperty="id" useGeneratedKeys="true"> 
    insert into tb_message(<include refid="updateAllColumns"/>) 
    values (#{msgId}, #{status}, #{content}, #{deleted}, #{createTime}) 
</insert>
```

#### 4、update | 更新

```java
messageMapper.updateContentById(1L, "bbbb");
sqlSession.commit();

int updateContentById(@Param("id") Long id, @Param("content") String content);
```

```xml
<update id="updateContentById" parameterType="message"> 
    update tb_message 
    set content=#{content} 
    where id=#{id} 
</update>
```

#### 5、delete | 删除

```java
messageMapper.delById(28L); 
sqlSession.commit();

int delById(@Param("id") Long id);
```

```xml
<delete id="delById" parameterType="long"> 
    delete from tb_message where id = #{id} 
</delete>
```

#### 6、${} 与 #{} | 防止sql注入

1. 简单的说就是，`#{}` 是经过预编译的，属于占位符的作用，参数赋值时只会替换掉占位符，由于 SQL 格式在编译时已经确认，所以无论参数怎么传，都是安全的。
2. 而 `${}` 是未经过预编译的，仅仅是取变量的值，属于字符串 append 添加，可能会追加新的 SQL，是非安全的，存在 SQL 注入的风险。
3. 因此，在编写 mybatis 的映射语句时，尽量采用 `#{}`  的格式。

##### 1）#{} 方式

```xml
<select id="getMessageByMsgId" resultMap="messageResult">
    select 
    	<include refid="allColumns"/> 
    from tb_message where msg_id = #{msgId} 
</select>
```

```java
// 结果输出与结论：所以，#{}采用的是，预编译的方式，去构建查询语句
Preparing: select id, msg_id, status, content, deleted, create_time, update_time from tb_message where msg_id = ? 
==> Parameters: 1001(String)
```

##### 2）${} 方式

```xml
<select id="getMessageByMsgId" resultMap="messageResult"> 
    select 
    	<include refid="allColumns"/> 
    from tb_message where msg_id = ${msgId} 
</select>
```

```java
// 结果输出与结论：所以，${}方式采用的是，值传递的方式，去构建查询语句，存在SQL注入的风险
Preparing: 
select id, msg_id, status, content, deleted, create_time, update_time from tb_message where msg_id = 1001
```

#### 7、结果集处理

##### 1）使用 Map 存储结果集

```java
Map map = messageMapper.getMessageMapById(2L);
```

```xml
<select id="getMessageMapById" resultType="map"> 
    select 
    	<include refid="allColumns"/> 
    from tb_message where id = #{id} 
</select>
```

##### 2）使用 POJO 存储结果集

```java
Message message = messageMapper.getMessageById(2L);
```

```xml
<resultMap id="messageResult" type="vo.Message">
    <id column="id" property="id"/>
    <result column="msg_id" property="msgId"/>
    <result column="status" property="status"/>
    <result column="content" property="content"/>
    <result column="deleted" property="deleted"/>
    <result column="create_time" property="createTime"/>
    <result column="update_time" property="updateTime"/>
</resultMap>

<select id="getMessageById" resultMap="messageResult">
    select
    <include refid="allColumns"/>
    from tb_message where id = #{id}
</select>
```

#### 8、级联查询

##### 1）一对一 | association

```java
Message message = messageMapper.getMessageAndMessageDetailById(2L);

Message getMessageAndMessageDetailById(@Param("id") Long id);
```

```xml
<resultMap id="messageAndDetailResult" type="vo.Message">
    <id column="id" property="id"/>
    <result column="msg_id" property="msgId"/>
    <result column="status" property="status"/>
    <result column="content" property="content"/>
    <result column="deleted" property="deleted"/>
    <result column="create_time" property="createTime"/>
    <result column="update_time" property="updateTime"/>
    <association column="msg_id" property="messageDetail" select="mapper.MessageDetailMapper.getMessageByMsgId"/>
</resultMap>

<select id="getMessageByMsgId" resultMap="msgDetailResult"> 
    select 
    	<include refid="allColumns"/> 
    from tb_message_detail where msg_id = #{msgId} 
</select>
```

##### 2）一对一 | 多参数关联

```java
Message message = messageMapper.getMessageAndMessageDetailById1(2L);

Message getMessageAndMessageDetailById1(@Param("id") Long id);
MessageDetail getMessageByMsgIdAndCreateTime(@Param("msgId") String msgId, @Param("content") String content);
```

```xml
<resultMap id="messageAndDetailResult1" type="vo.Message">
    <id column="id" property="id"/>
    <result column="msg_id" property="msgId"/>
    <result column="status" property="status"/>
    <result column="content" property="content"/>
    <result column="deleted" property="deleted"/>
    <result column="create_time" property="createTime"/>
    <result column="update_time" property="updateTime"/>
    <association column="{msgId=msg_id, content=content}" property="messageDetail"
                 select="mapper.MessageDetailMapper.getMessageByMsgIdAndCreateTime"/>
</resultMap>

<select id="getMessageAndMessageDetailById1" parameterType="long" resultMap="messageAndDetailResult1">
    select
    <include refid="allColumns"/>
    from tb_message where id = #{id}
</select>
```

##### 3）一对多 | collection

```java
User getUserAndContactById(@Param("id") Long id);
```

```xml
<resultMap id="userContactResultMap" type="vo.User">
    <id column="id" property="id"/>
    <result column="name" property="name"/>
    <result column="age" property="age"/>
    <collection column="id" property="userContacts" select="mapper.UserContactMapper.getUserContactByUserId"/>
</resultMap>

<select id="getUserAndContactById" parameterType="long" resultMap="userContactResultMap">
    select id, name, age from tb_user where id = #{id}
</select>
```

#### 9、缓存

##### 1）一级缓存

MyBatis 默认开启一级缓存，即：同一个 SqlSession 对象，调用同一个 Mapper 的方法时，如果没有声明需要刷新，并且缓存没超时的情况下，一般只执行一次 SQL，其他的查询 SqlSession 都只会取出当前缓存的数据。如下所示：

```java
public class CacheExecuter {
    public static void main(String[] args) {
        SqlSession sqlSession = SqlSessionFactoryUtil.openSqlSession();
        UserMapper userMapper = sqlSession.getMapper(UserMapper.class);

        User user1 = userMapper.getUserById(1L);
        System.out.println("----真实查询-----user1 = " + user1);

        //当使用二级缓存的时候，只有调用了commit方法后才会生效。
        sqlSession.commit();

        User user2 = userMapper.getUserById(1L);
        System.out.println("----缓存查询-----user2 = " + user2);

        /**
         * 开启了新的sqlSession，则无法利用一级缓存。因为一级缓存是sqlSession之间隔离的。
         */
        sqlSession = SqlSessionFactoryUtil.openSqlSession();
        userMapper = sqlSession.getMapper(UserMapper.class);
        User user3 = userMapper.getUserById(1L);
        System.out.println("----真实查询-----user3 = " + user3);
    }
}
```

输出如下：

```java
Created connection 1071097621. Returned connection 1071097621 to pool. Cache Hit Ratio [mapper.UserMapper]: 0.0 Opening JDBC
Connection Checked out connection 1071097621 from pool. Setting autocommit to false on JDBC Connection
[com.mysql.cj.jdbc.ConnectionImpl@3fd7a715] 
==> Preparing: select id, name, age from tbuser where id = ? ==> Parameters: 1(Long) 
<== Columns: id, name, age 
<== Row: 1, muse1, 22 
<== Total: 1 ----真实查询-----user1 = User{id=1, name='muse1', age=22, userContacts=null}
Cache Hit Ratio [mapper.UserMapper]: 0.0 ----缓存查询-----user2 = User{id=1, name='muse1', age=22, userContacts=null} 
Cache Hit Ratio
[mapper.UserMapper]: 0.0 Opening JDBC Connection Created connection 280265505. Setting autocommit to false on JDBC Connection
[com.mysql.cj.jdbc.ConnectionImpl@10b48321] 
==> Preparing: select id, name, age from tbuser where id = ? ==> Parameters: 1(Long) <== Columns: id, name, age 
<== Row: 1, muse1, 22 
<== Total: 1 ----真实查询-----user3 = User{id=1, name='muse1', age=22, userContacts=null}
Process finished with exit code 0
```

##### 2）二级缓存

在 UserMapper.xml 中添加标签，当使用二级缓存的时候，只有调用了 `sqlSession.commit();` 方法后才会生效，且 POJO 必须实现 Serializable 接口。使用方式：

```xml
<mapper namespace="mapper.UserMapper">
	<!-- UserMapper开启二级缓存 -->
	<cache/>
</mapper>
```

##### 3）自定义缓存

可以通过实现 `org.apache.ibatis.cache.Cache` 接口，使用 Redis，Memcache 等缓存机制，来实现自定义缓存。使用方式：

```xml
<mapper namespace="mapper.UserMapper">
	<!-- UserMapper开启二级缓存 -->
	<cache type="com.muse.RedisCache"/>
</mapper>
```

#### 10、动态 SQL

##### 1）if

最常用的判断语句：

```java
public interface UserMapper { 
    User getUserByUser(User user);
}
```

```xml
<select id="getUserByUser" parameterType="vo.User" resultMap="userResultMap"> 
    select id, name, age 
    from tb_user
    where 1=1 
    <if test="id != null"> 
        and id = #{id} 
    </if> 
    <if test="name != null and name != ''">
        and name = #{name}
    </if> 
    <if test="age != null"> 
        and age = #{age} 
    </if> 
</select>
```

##### 2）choose、when、otherwise

相当于 if-if else-else：

```java
User userParam = new User();
userParam.setName("muse1"); 
// userParam.setId(1L); 
userParam.setAge(22);
User user = userMapper.getUserByUser2(userParam);
System.out.println("user = " + user);

public interface UserMapper { 
    User getUserByUser2(User user); 
}
```

```xml
<select id="getUserByUser2" parameterType="vo.User" resultMap="userResultMap">
    select id, name, age
    from tb_user
    where 1=1
    <choose>
        <when test="id != null">
            and id = #{id}
        </when>
        <when test="name != null and name != ''">
            and name = #{name}
        </when>
        <otherwise>
            and age is not null
        </otherwise>
    </choose>
</select>
```

##### 3）where

可以通过标签，来避免去写 where 1=1。如下所示：

```java
User userParam = new User(); 
userParam.setId(1L);
List<User> user = userMapper.getUserByUser3(userParam); System.out.println("user = " + user);

// 指定ID的输出结果：
==> Preparing: select id, name, age from tb_user WHERE id = ? 
==> Parameters: 1(Long) 
<== Columns: id, name, age <== Row: 1, muse1, 22
<== Total: 1 user = [User{id=1, name='muse1', age=22, userContacts=null}]
```

```java
User userParam = new User(); 
// userParam.setId(1L);
List<User> user = userMapper.getUserByUser3(userParam);
System.out.println("user = " + user);

// 不指定ID的输出结果：
==> Preparing: select id, name, age from tb_user
==> Parameters: 
<== Columns: id, name, age 
<== Row: 1, muse1, 22 <== Row: 2, muse2, 24 
<== Total: 2 user = [User{id=1, name='muse1', age=22, userContacts=null}, User{id=2, name='muse2', age=24, userContacts=null}]
```

```xml
<select id="getUserByUser3" parameterType="vo.User" resultMap="userResultMap">
    select
    id, name, age from tb_user
    <where>
        <if test="id != null">
            and id = #{id}
        </if>
    </where>
</select>
```

##### 4）trim

有时候，要去掉一些特殊的 SQL 语法，比如 and、or。则可以使用 trim 标签。

```xml
<select id="getUserByUser4" parameterType="vo.User" resultMap="userResultMap">
    select id, name, age 
    from tb_user 
    <trim prefix="where" prefixOverrides="and">
        and id = #{id} 
    </trim> 
</select>
```

【解释】

- prefix，表示输出前缀语句 where。
- prefixOverrides，表示 where 后面的语句的前缀（第一个）and，要清除掉。

##### 5）set

set 标签会默认把最后一个逗号去掉：

```xml
<update id="updateUserByUser" parameterType="vo.User"> 
    update tb_user
    <set> 
        <if test="name != null and name != ''"> name = #{name}, </if>
        <if test="age != null"> age = #{age}, </if>
    </set> 
    where id = #{id} 
</update>
```

也可以采用 trim 的方式：

```xml
<update id="updateUserByUser" parameterType="vo.User"> 
    update tb_user 
    <trim prefix="set" suffixOverrides=","> 
        <if test="name != null and name != ''"> name = #{name}, </if> 
        <if test="age != null"> age = #{age}, </if> 
    </trim>
    where id = #{id}
</update>
```

##### 6）foreach

foreach 语句，用于循环遍历传入的集合数据：

```java
public interface UserMapper {
    List<User> getUserByIds(@Param("idList") List<Long> idList);
}
```

```xml
<select id="getUserByIds" resultMap="userResultMap">
    select id, name, age 
    from tb_user where id in 
    <foreach collection="idList" index="index" item="id" open="(" separator="," close=")"> 
        #{id} 
    </foreach> 
</select>
```

【解释】

- collection：传递进来的参数名称，可以是数组、List、Set等集合。
- index：当前元素在集合的下标位置。
- item：循环中当前的元素。
- open和close：使用什么符号包装集合元素。
- separator：每个元素的间隔符号。

##### 7）test

test 属性用于条件判断的语句中。

```xml
<select id="getUserByUser3" parameterType="vo.User" resultMap="userResultMap"> 
    select id, name, age 
    from tb_user 
    <where> 
        <if test="id != null"> and id = #{id} </if> 
    </where> 
</select>
```

##### 8）bind

bind 用于优化重复的 SQL 字段模板：

```java
List<User> users = userMapper.getUserByName("muse");

List<User> getUserByName(@Param("name") String name);

// 输出结果：
==> Preparing: select id, name, age from tb_user where name like ? ==> Parameters: %muse%(String) 
<== Columns: id, name, age 
<== Row: 1, muse, 22 
<== Row: 2, muse2, 24 
<== Total: 2
```

```xml
<select id="getUserByName" parameterType="string" resultMap="userResultMap"> 
    <bind name="namePattern" value="'%' + name + '%'"/> 
    select id, name, age from tb_user 
    where name like #{namePattern} 
    <!-- 等于name like concat('%', #{name}, '%') --> 
</select>
```

### 1.6. Java 动态代理？

#### 1、反射

```java
public class Reflaction { 
    public static void main(String[] args) throws Throwable { 
        Class clazz = User.class; 
        User user = (User) clazz.newInstance();
        Method method = clazz.getMethod("setName", String.class); 
        method.invoke(user, "张三");
        System.out.println(user.getName()); // 输出：张三 
    }
}
```

#### 2、JDK 动态代理

JDK 动态代理，需要提供接口，而 MyBatis 的 Mapper 就是一个接口，它采用的就是 JDK 动态代理。如下所示：

```java
public interface MessageService { 
    void sendMessage(); 
}

public class MessageServiceImpl implements MessageService {
    public void sendMessage() {
        System.out.println("MessageServiceImpl.sendMessage"); 
    } 
}
```

```java
public class JdkProxy<T> implements InvocationHandler {

    T target;

    public T getProxy(T target) {
        this.target = target;

        return (T) Proxy.newProxyInstance(target.getClass().getClassLoader(), target.getClass().getInterfaces(), this);
    }

    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        System.out.println("JDK动态代理拦截开始！");
        Object result =  method.invoke(target, args);
        System.out.println("JDK动态代理拦截结束！");
        return result;
    }
}
```

```java
public class Executer {
    public static void main(String[] args) {
        JdkProxy<MessageService> jdkProxy = new JdkProxy();
        MessageService messageService = jdkProxy.getProxy(new MessageServiceImpl());
        messageService.sendMessage();
    }
}
```

#### 3、CGLIB 动态代理

CGLIB，不需要提供接口，即可实现动态代理，当然，它也可以代理有接口的服务类。如下所示：

```java
public class PlayService {
    public void play() {
        System.out.println("PlayService.play");
    }
}
```

```java
public class CglibProxy<T> implements MethodInterceptor {

    T target;

    public T getProxy(T target) {
        this.target = target;
        Enhancer enhancer = new Enhancer();
        enhancer.setSuperclass(target.getClass());
        enhancer.setCallback(this);
        return (T) enhancer.create();
    }

    public Object intercept(Object o, Method method, Object[] objects, MethodProxy methodProxy) throws Throwable {
        System.out.println("CGLIB动态代理拦截开始!");
        Object result = methodProxy.invokeSuper(o, objects);
        System.out.println("CGLIB动态代理拦截结束!");
        return result;
    }
}
```

```java
public class Executer {
    public static void main(String[] args) {
        CglibProxy<PlayService> cglibProxy = new CglibProxy();
        
        // 代理无接口服务类
        PlayService playService = cglibProxy.getProxy(new PlayService());
        playService.play();

//        CglibProxy<MessageService> cglibProxy1 = new CglibProxy();
//        // 代理有接口服务类
//        MessageService messageService = cglibProxy1.getProxy(new MessageServiceImpl());
//        messageService.sendMessage();
    }
}
```

### 1.7. MyBatis 整体架构？

![1647161327867](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1647161327867.png)

MyBatis 的整体架构分为三层：

1. **API接口层**：
   - 提供给外部使用的接口 API，开发人员通过这些本地 API 来操纵数据库。
   - 接口层一接收到调用请求，就会调用数据处理层来完成具体的数据处理。
2. **数据处理层**：
   - 负责具体的 SQL 查找、SQL 解析、SQL 执行和执行结果映射处理等。
   - 其主要的目的是，根据调用的请求完成一次数据库操作。
3. **基础支撑层**：
   - 负责最基础的功能支撑，包括连接管理、事务管理、配置加载和缓存处理。
   - 这些都是共用的东西，将他们抽取出来作为最基础的组件，为上层的数据处理层提供最基础的支撑。

### 1.8. MyBatis SqlSession 执行流程？

#### 1、获取 Mapper 的动态代理

1. 自定义的 Mapper 接口想要发挥功能，必须有具体的实现类，在 MyBatis 中是通过为 Mapper 每个接口提供一个动态代理。
2. 动态代理类来实现的整个过程主要有四个类：MapperRegistry、MapperProxyFactory、MapperProxy、MapperMethod。
   - **MapperRegistry**：是 Mapper 接口及其对应的代理对象工厂的注册中心。
   - **MapperProxyFactory**：是 MapperProxy 的工厂类，主要方法就是包装了 Java 动态代理 `Proxy.newProxyInstance()` 方法。
   - **MapperProxy**：是一个动态代理类，实现了 `InvocationHandler` 接口，对于代理对象的调用都会被代理到 `InvocationHandler#invoke()` 方法上。
   - **MapperMethod**：包含了具体增删改查方法的实现逻辑。

```java
public class UserExecuter {
    public static void main(String[] args) {
        SqlSession sqlSession = null;
        try {
            sqlSession = SqlSessionFactoryUtil.openSqlSession();
            // 1、获取 Mapper 的动态代理
            UserMapper userMapper = sqlSession.getMapper(UserMapper.class);
            User user = userMapper.getUserById(2L);
            System.out.println("姓名：" + user.getName() + "，年龄：" + user.getAge());
        }
    }
}

public class DefaultSqlSession implements SqlSession {
    @Override
    public <T> T getMapper(Class<T> type) {
        // 2、获取 Mapper 的动态代理
        return configuration.<T>getMapper(type, this);
    }
}

public class Configuration {
    public <T> T getMapper(Class<T> type, SqlSession sqlSession) {
        // 3、获取 Mapper 的动态代理
        return mapperRegistry.getMapper(type, sqlSession);
    }
}

public class MapperRegistry {
    /**
      * 4、加载mybatis-config.xml配置的<mapper>配置，根据指定type，查找对应的MapperProxyFactory对象
      **/
    // eg1: 获得UserMapper的mapperProxyFactory
    final MapperProxyFactory<T> mapperProxyFactory = (MapperProxyFactory<T>) knownMappers.get(type);
    
    /**
      * 5、如果没配置<mapper>，则找不到对应的MapperProxyFactory，抛出BindingException异常
      */
    if (mapperProxyFactory == null) {
        throw new BindingException("Type " + type + " is not known to the MapperRegistry.");
    }
    
    try {
        /** 6、使用该工厂类生成MapperProxy的代理对象 */
        return mapperProxyFactory.newInstance(sqlSession);
    } catch (Exception e) {
        throw new BindingException("Error getting mapper instance. Cause: " + e, e);
    }
}

public class MapperProxyFactory<T> {

    public T newInstance(SqlSession sqlSession) {
        /**
         * 7、创建MapperProxy对象，每次调用都会创建新的MapperProxy对象，MapperProxy implements InvocationHandler
         */
        final MapperProxy<T> mapperProxy = new MapperProxy<T>(sqlSession, mapperInterface, methodCache);
        return newInstance(mapperProxy);
    }
    
   protected T newInstance(MapperProxy<T> mapperProxy) {
        // 8、通过动态代理，创建mapperInterface的代理类对象
        return (T) Proxy.newProxyInstance(mapperInterface.getClassLoader(), new Class[] {mapperInterface}, mapperProxy);
    }
}
```

#### 2、获得 MapperMethod 对象

```java
public class UserExecuter {
    public static void main(String[] args) {
        SqlSession sqlSession = null;
        try {
            sqlSession = SqlSessionFactoryUtil.openSqlSession();
            UserMapper userMapper = sqlSession.getMapper(UserMapper.class);
            // 1、发起方法调用，调用代理对象增强后的方法
            User user = userMapper.getUserById(2L);
            System.out.println("姓名：" + user.getName() + "，年龄：" + user.getAge());
        }
    }
}

public class MapperProxy<T> implements InvocationHandler, Serializable {
    // eg1: UserMapper userMapper = sqlSession.getMapper(UserMapper.class);
    //      User user = userMapper.getUserById(2L); args = {2L}
    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        try {
            /** 2、如果被代理的方法是Object类的方法，如toString()、clone()，则不进行代理 */
            // eg1: method.getDeclaringClass()==interface mapper.UserMapper  由于被代理的方法是UserMapper的getUserById方法，而不是Object的方法，所以返回false
            if (Object.class.equals(method.getDeclaringClass())) {
                return method.invoke(this, args);
            }

            /** 3、如果是接口中的default方法，则调用default方法 */
            else if (isDefaultMethod(method)) { // eg1: 不是default方法，返回false
                return invokeDefaultMethod(proxy, method, args);
            }
        } catch (Throwable t) {
            throw ExceptionUtil.unwrapThrowable(t);
        }
        
        // eg1: method = public abstract vo.User mapper.UserMapper.getUserById(java.lang.Long)
        /** 4、初始化一个MapperMethod并放入缓存中 或者 从缓存中取出之前的MapperMethod */
        final MapperMethod mapperMethod = cachedMapperMethod(method);

        // eg1: sqlSession = DefaultSqlSession@1953  args = {2L}
        /** 99、调用MapperMethod.execute()方法执行SQL语句 */
        return mapperMethod.execute(sqlSession, args);
    }
    
    // eg1: public abstract vo.User mapper.UserMapper.getUserById(java.lang.Long)
    private MapperMethod cachedMapperMethod(Method method) {
        /**
         * 5、在缓存中查找MapperMethod，若没有，则创建MapperMethod对象，并添加到methodCache集合中缓存
         */
        // eg1: 因为methodCache为空，所以mapperMethod等于null
        MapperMethod mapperMethod = methodCache.get(method);
        if (mapperMethod == null) {
            // eg1: 6、构建mapperMethod对象，并维护到缓存methodCache中
            mapperMethod = new MapperMethod(mapperInterface, method, sqlSession.getConfiguration());
            // eg1: method = public abstract vo.User mapper.UserMapper.getUserById(java.lang.Long)
            methodCache.put(method, mapperMethod);
        }
        return mapperMethod;
    }
}

public class MapperMethod {
    // eg1: 7、mapperInterface = interface mapper.UserMapper
    //      method = public abstract vo.User mapper.UserMapper.getUserById(java.lang.Long)
    public MapperMethod(Class<?> mapperInterface, Method method, Configuration config) {
        this.command = new SqlCommand(config, mapperInterface, method);
        this.method = new MethodSignature(config, mapperInterface, method);
    }
}
```

#### 3、根据 SQL 指令跳转执行语句

```java
public class MapperProxy<T> implements InvocationHandler, Serializable {
    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
      	...
        // eg1: sqlSession = DefaultSqlSession@1953  args = {2L}
        /** 1、调用MapperMethod.execute()方法执行SQL语句 */
        return mapperMethod.execute(sqlSession, args);
    }
}

public class MapperMethod {
    /**
     * 2、MapperMethod采用命令模式运行，根据上下文跳转，它可能跳转到许多方法中。实际上它最后就是通过SqlSession对象去运行对象的SQL。
     */
    // eg1: sqlSession = DefaultSqlSession@1953  args = {2L}
    public Object execute(SqlSession sqlSession, Object[] args) {
        Object result;
        // eg1: command.getType() = SELECT
        switch (command.getType()) {
            case INSERT: {
                Object param = method.convertArgsToSqlCommandParam(args);
                result = rowCountResult(sqlSession.insert(command.getName(), param));
                break;
            }
            case UPDATE: {
                Object param = method.convertArgsToSqlCommandParam(args);
                result = rowCountResult(sqlSession.update(command.getName(), param));
                break;
            }
            case DELETE: {
                Object param = method.convertArgsToSqlCommandParam(args);
                result = rowCountResult(sqlSession.delete(command.getName(), param));
                break;
            }
            case SELECT:
                // eg1: method.returnsVoid() = false  method.hasResultHandler() = false
                if (method.returnsVoid() && method.hasResultHandler()) {
                    executeWithResultHandler(sqlSession, args);
                    result = null;
                } else if (method.returnsMany()) { // eg1: method.returnsMany() = false
                    result = executeForMany(sqlSession, args);
                } else if (method.returnsMap()) { // eg1: method.returnsMap() = false
                    result = executeForMap(sqlSession, args);
                } else if (method.returnsCursor()) { // eg1: method.returnsCursor() = false
                    result = executeForCursor(sqlSession, args);
                } else {
                    // eg1: args = {2L}
                    /** 3、将参数转换为sql语句需要的入参 */
                    Object param = method.convertArgsToSqlCommandParam(args);

                    // eg1: sqlSession=DefaultSqlSession  command.getName()="mapper.UserMapper.getUserById" param={"id":2L, "param1":2L}
                    /** 4、执行sql查询操作 */
                    result = sqlSession.selectOne(command.getName(), param);
                }
                break;
            case FLUSH:
                result = sqlSession.flushStatements();
                break;
            default:
                ...
        }
        return result;
    }
}
```

#### 4、查询前的缓存处理

```java
public class MapperMethod {
    public Object execute(SqlSession sqlSession, Object[] args) {
        ...
        // 1、执行sql查询操作
        result = sqlSession.selectOne(command.getName(), param);
        ...
    }
}

public class DefaultSqlSession implements SqlSession {
    // 2、eg1: statement="mapper.UserMapper.getUserById" parameter={"id":2L, "param1":2L}
    @Override
    public <T> T selectOne(String statement, Object parameter) {
        List<T> list = this.selectList(statement, parameter);
        if (list.size() == 1) {
            return list.get(0);
        } else if (list.size() > 1) {
            throw new TooManyResultsException(
                    "Expected one result (or null) to be returned by selectOne(), but found: " + list.size());
        } else {
            return null;
        }
    }
    
    // 3、eg1: statement="mapper.UserMapper.getUserById" parameter={"id":2L, "param1":2L}
    @Override
    public <E> List<E> selectList(String statement, Object parameter) {
        return this.selectList(statement, parameter, RowBounds.DEFAULT);
    }
    
    // eg1: statement="mapper.UserMapper.getUserById" parameter={"id":2L, "param1":2L}
    @Override
    public <E> List<E> selectList(String statement, Object parameter, RowBounds rowBounds) {
        try {
            // 4、eg1: statement="mapper.UserMapper.getUserById"
            MappedStatement ms = configuration.getMappedStatement(statement);
            // 5、eg1: executor=CachingExecutor
            //      wrapCollection(parameter)=parameter={"id": 2L, "param1", 2L}
            //      rowBounds=RowBounds.DEFAULT=new RowBounds()
            //      Executor.NO_RESULT_HANDLER=null
            return executor.query(ms, wrapCollection(parameter), rowBounds, Executor.NO_RESULT_HANDLER);
        } catch (Exception e) {
            throw ExceptionFactory.wrapException("Error querying database.  Cause: " + e, e);
        } finally {
            ErrorContext.instance().reset();
        }
    }
}

public class CachingExecutor implements Executor {
    // 6、eg1: parameterObject={"id":2L, "param1":2L}
    //      rowBounds=new RowBounds()
    //      resultHandler=null
    @Override
    public <E> List<E> query(MappedStatement ms, Object parameterObject, RowBounds rowBounds, ResultHandler resultHandler) throws SQLException {
        // eg1: parameterObject = {"id":2L, "param1":2L}
        /** 7、获得boundSql对象，承载着sql和对应的参数*/
        BoundSql boundSql = ms.getBoundSql(parameterObject);

        // eg1: parameterObject = {"id":2L, "param1":2L}  rowBounds = new RowBounds()
        /** 8、生成缓存key */
        CacheKey key = createCacheKey(ms, parameterObject, rowBounds, boundSql);

        // eg1: parameterObject = {"id":2L, "param1":2L}  rowBounds = new RowBounds() resultHandler = null
        /** 9、执行查询语句 */
        return query(ms, parameterObject, rowBounds, resultHandler, key, boundSql);
    }
    
    // 10、eg1: parameterObject = {"id": 2L, "param1", 2L}  rowBounds = new RowBounds() resultHandler = null
    @Override
    public <E> List<E> query(MappedStatement ms, Object parameterObject, RowBounds rowBounds,
                             ResultHandler resultHandler, CacheKey key, BoundSql boundSql) throws SQLException {
        Cache cache = ms.getCache();
        // eg1: cache = null
        /** 11、如果在UserMapper.xml配置了<cache/>开启了二级缓存，则cache不为null*/
        if (cache != null) {
            /**
             * 如果flushCacheRequired=true并且缓存中有数据，则先清空缓存
             *
             * <select id="save" parameterType="XXXXXEO" statementType="CALLABLE" flushCache="true" useCache="false">
             *     ……
             * </select>
             * */
            flushCacheIfRequired(ms);

            /** 12、如果useCache=true并且resultHandler=null*/
            if (ms.isUseCache() && resultHandler == null) {
                ensureNoOutParams(ms, parameterObject, boundSql);
                @SuppressWarnings("unchecked")
                List<E> list = (List<E>) tcm.getObject(cache, key);
                if (list == null) {
                    /** 13、执行查询语句 */
                    list = delegate.<E>query(ms, parameterObject, rowBounds, resultHandler, key, boundSql);
                    /** 14、以cacheKey为主键，将结果维护到缓存中 */
                    tcm.putObject(cache, key, list); // issue #578 and #116
                }
                return list;
            }
        }
        // 15、如果没有开启二级缓存，则走这里
        // eg1: delegate = SimpleExecutor(BaseExecutor) parameterObject = {"id": 2L, "param1", 2L}  rowBounds = new RowBounds() resultHandler = null
        return delegate.<E>query(ms, parameterObject, rowBounds, resultHandler, key, boundSql);
    }
}

public abstract class BaseExecutor implements Executor {
    // 16、eg1: parameter = {"id": 2L, "param1", 2L}  rowBounds = new RowBounds() resultHandler = null
    @SuppressWarnings("unchecked")
    @Override
    public <E> List<E> query(MappedStatement ms, Object parameter, RowBounds rowBounds, ResultHandler resultHandler,
                             CacheKey key, BoundSql boundSql) throws SQLException {
        ErrorContext.instance().resource(ms.getResource()).activity("executing a query").object(ms.getId());
        // 17、eg1: closed = false
        if (closed) {
            throw new ExecutorException("Executor was closed.");
        }

        // eg1: queryStack = 0  ms.isFlushCacheRequired() = false
        /** 18、如果配置了flushCacheRequired=true并且queryStack=0（没有正在执行的查询操作），则会执行清空缓存操作*/
        if (queryStack == 0 && ms.isFlushCacheRequired()) {
            clearLocalCache();
        }

        List<E> list;
        try {
            /** 19、记录正在执行查询操作的任务数*/
            queryStack++; // eg1: queryStack=1

            // eg1: resultHandler=null localCache.getObject(key)=null
            /** 20、localCache维护一级缓存，试图从一级缓存中获取结果数据，如果有数据，则返回结果；如果没有数据，再执行queryFromDatabase */
            list = resultHandler == null ? (List<E>) localCache.getObject(key) : null;
            // eg1: list = null
            if (list != null) {
                /** 21、如果是执行存储过程 */
                handleLocallyCachedOutputParameters(ms, key, parameter, boundSql);
            } else {
                // eg1: parameter = {"id": 2L, "param1", 2L}  rowBounds = new RowBounds() resultHandler = null
                list = queryFromDatabase(ms, parameter, rowBounds, resultHandler, key, boundSql);
            }
        } finally {
            queryStack--;
        }
        if (queryStack == 0) {
            /** 22、延迟加载处理 */
            for (DeferredLoad deferredLoad : deferredLoads) {
                deferredLoad.load();
            }
            // issue #601
            deferredLoads.clear();

            // eg1: configuration.getLocalCacheScope()=SESSION
            /** 23、如果设置了<setting name="localCacheScope" value="STATEMENT"/>，则会每次执行完清空缓存。即：使得一级缓存失效 */
            if (configuration.getLocalCacheScope() == LocalCacheScope.STATEMENT) {
                // issue #482
                clearLocalCache();
            }
        }
        return list;
    }
    
    @Override
    public void clearLocalCache() {
        if (!closed) {
            localCache.clear();
            localOutputParameterCache.clear();
        }
    }
}
```

#### 5、执行 DB 查询操作

Mapper执行的过程是通过 Executor、StatementHandler、ParameterHandler 和 ResultHandler 来完成数据库操作和结果返回的：

1. **Executor**：
   - 1）代表执行器，由它来调度 StatementHandler、ParameterHandler、ResultHandler 等来执行对应的SQL。
   - 2）执行器 Executor 是一个真正执行执行 Java 和数据库交互的类，一共有 3 种执行器，可以在MyBatis 的配置文件中，设置 defaultExecutorType 属性来进行选择：
     1. **SIMPLE**：org.apache.ibatis.executor.SimpleExecutor，简易执行器，默认执行器。
     2. **REUSE**：org.apache.ibatis.executor.ReuseExecutor，是一种执行器重用预处理语句。
     3. **BATCH**：org.apache.ibatis.executor.BatchExecutor，执行器重用语句和批量更新，它是针对批量专用批量专用的执行器。
2. **StatementHandler**：
   - 1）作用是使用数据库的 Statement（PreparedStatement）执行操作，起到承上启下的作用，专门处理数据库会话的，创建 StatementHandler 的过程在 Configuration 中。
   - 2）MyBatis 是使用来委派模式，把具体的 StatementHandler 类型隐藏起来，通过 `RoutingStatementHandler` 来统一管理，一共用三种具体的StatementHandler类型：
     SimpleHandler、PreparedStatementHandler 和 CallableStatementHandler。
   - 3）在 Executor 的具体执行逻辑中，主要关注 `StatementHandler#prepared` 和 `StatementHandler#parameterize` 两个方法。
3. **ParameterHandler**：
   - 1）用于SQL对参数的处理。
   - 2）MyBatis 是通过 ParameterHandler 对预编译的语句进行参数设置的。
   - 3）MyBatis 为 ParameterHandler 提供了一个实现类 DefaultParameterHandler，具体执行过程是：
     1. 从 parameterObject 对象中取参数。
     2. 然后使用 typeHandler 进行参数处理。
     3. 而 typeHandler 也是在 MyBatis 初始化时，注册在 Configuration 里面的，需要时可以直接拿来用。
4. **ResultHandler**：
   - 1）进行最后数据集（ResultSet）的封装返回处理。
   - 2）MyBatis 为我们提供了一个 DefaultResultSetHandler 类，在默认情况下，都是通过这个类进行处理的。
   - 3）这个类 JAVASSIST 或者 CGLIB 作为延迟加载，然后通过 typeHandler 和 ObjectFactory 进行组装结果再返回。

=> 以 SimpleExecutor 来看一下 Executor 的**具体执行逻辑**：

1. 根据 Configuration 来构建 StatementHandler。
2. 然后使用 `prepareStatement` 方法，对 SQL 编译并对参数进行初始化。
3. 在 `prepareStatement` 方法中，调用了 `StatementHandler#prepared` 进行了预编译和基础设置。
4. 然后通过 `StatementHandler#parameterize` 来设置参数并执行。
5. 包装好的 Statement 通过 StatementHandler 来执行，并把结果传递给 resultHandler。

```java
public abstract class BaseExecutor implements Executor {
    @SuppressWarnings("unchecked")
    @Override
    public <E> List<E> query(MappedStatement ms, Object parameter, RowBounds rowBounds, ResultHandler resultHandler,
                             CacheKey key, BoundSql boundSql) throws SQLException {
        ...
        // 1、eg1: parameter = {"id": 2L, "param1", 2L}  rowBounds = new RowBounds() resultHandler = null
        list = queryFromDatabase(ms, parameter, rowBounds, resultHandler, key, boundSql);
        ...
    }
    
    // 2、eg1: parameter = {"id": 2L, "param1", 2L}  rowBounds = new RowBounds() resultHandler = null
    private <E> List<E> queryFromDatabase(MappedStatement ms, Object parameter, RowBounds rowBounds,
                                          ResultHandler resultHandler, CacheKey key, BoundSql boundSql) throws SQLException {
        List<E> list;
        // 3、eg1: key = -445449180:-48278933:mapper.UserMapper.getUserById:0:2147483647:select id, name, age from tb_user where id = ?:2:dev
        localCache.putObject(key, EXECUTION_PLACEHOLDER);
        try {
            // 4、eg1: SimpleExecutor.doQuery parameter = {"id": 2L, "param1", 2L}  rowBounds = new RowBounds() resultHandler = null
            list = doQuery(ms, parameter, rowBounds, resultHandler, boundSql);
        } finally {
            localCache.removeObject(key);
        }
        /** 99、将查询结果放到一级缓存中，如果同一session中有相同查询操作，则可以直接从缓存中获取结果*/
        localCache.putObject(key, list);

        // eg1: ms.getStatementType() = PREPARED
        if (ms.getStatementType() == StatementType.CALLABLE) {
            localOutputParameterCache.putObject(key, parameter);
        }
        return list;
    }
}

public class SimpleExecutor extends BaseExecutor {
    // 5、eg1: parameter = {"id": 2L, "param1", 2L}  rowBounds = new RowBounds() resultHandler = null
    @Override
    public <E> List<E> doQuery(MappedStatement ms, Object parameter, RowBounds rowBounds, ResultHandler resultHandler,
                               BoundSql boundSql) throws SQLException {
        Statement stmt = null;
        try {
            Configuration configuration = ms.getConfiguration();
            /** 6、根据Configuration来构建StatementHandler */
            StatementHandler handler = configuration.newStatementHandler(wrapper, ms, parameter, rowBounds,
                    resultHandler, boundSql);

            // eg1: handler=RoutingStatementHandler
            /** 7、然后使用prepareStatement方法，对SQL进行预编译并设置入参 */
            stmt = prepareStatement(handler, ms.getStatementLog());

            // eg1: handler=RoutingStatementHandler parameter = {"id": 2L, "param1", 2L}  rowBounds = new RowBounds() resultHandler = null
            /** 21、开始执行真正的查询操作。将包装好的Statement通过StatementHandler来执行，并把结果传递给resultHandler */
            return handler.<E>query(stmt, resultHandler);
        } finally {
            closeStatement(stmt);
        }
    }
    
    /**
     * 使用prepareStatement方法，对SQL编译并设置入参
     *
     * @param handler
     * @param statementLog
     * @return
     * @throws SQLException
     */
    // 8、eg1: handler=RoutingStatementHandler
    private Statement prepareStatement(StatementHandler handler, Log statementLog) throws SQLException {
        Statement stmt;

        /** 9、获得Connection实例 */
        Connection connection = getConnection(statementLog);

        // eg1: handler=RoutingStatementHandler
        /** 10、第一步：调用了StatementHandler的prepared进行了【sql的预编译】 */
        stmt = handler.prepare(connection, transaction.getTimeout());

        /** 14、第二步：通过PreparedStatementHandler的parameterize来给【sql设置入参】 */
        handler.parameterize(stmt);

        // 20、eg1: 返回org.apache.ibatis.logging.jdbc.PreparedStatementLogger@2e570ded
        return stmt;
    }
}

public class RoutingStatementHandler implements StatementHandler {
    // 11、eg1: delegate=PreparedStatementHandler
    @Override
    public Statement prepare(Connection connection, Integer transactionTimeout) throws SQLException {
        return delegate.prepare(connection, transactionTimeout);
    }
    
    // 22、eg1: delegate = PreparedStatementHandler  resultHandler = null
    @Override
    public <E> List<E> query(Statement statement, ResultHandler resultHandler) throws SQLException {
        return delegate.<E>query(statement, resultHandler);
    }
    
    // 15、第二步：通过PreparedStatementHandler的parameterize来给【sql设置入参】
    @Override
    public void parameterize(Statement statement) throws SQLException {
        delegate.parameterize(statement);
    }
}

public abstract class BaseStatementHandler implements StatementHandler {
    // eg1: delegate=PreparedStatementHandler
    /**
     * 12、执行预编译语句
     */
    @Override
    public Statement prepare(Connection connection, Integer transactionTimeout) throws SQLException {
        ErrorContext.instance().sql(boundSql.getSql());
        Statement statement = null;
        try {
            // eg1: 13、调用PreparedStatementHandler的instantiateStatement
            statement = instantiateStatement(connection);
            setStatementTimeout(statement, transactionTimeout);
            setFetchSize(statement);
            return statement;
        } catch (SQLException e) {
            closeStatement(statement);
            throw e;
        } catch (Exception e) {
            closeStatement(statement);
            throw new ExecutorException("Error preparing statement.  Cause: " + e, e);
        }
    }
}

public class PreparedStatementHandler extends BaseStatementHandler {
    // 16、eg1: org.apache.ibatis.logging.jdbc.PreparedStatementLogger@2e570ded
    @Override
    public void parameterize(Statement statement) throws SQLException {
        parameterHandler.setParameters((PreparedStatement) statement);
    }
    
    // 23、eg1: delegate = PreparedStatementHandler  resultHandler = null
    @Override
    public <E> List<E> query(Statement statement, ResultHandler resultHandler) throws SQLException {
        /** 24、最终还是使用JDBC去进行数据操作 */
        PreparedStatement ps = (PreparedStatement) statement;

        /** 25、执行查询操作 */
        ps.execute();

        // eg1: 封装结果集 resultSetHandler=DefaultResultSetHandler
        /** 26、将结果集进行封装 */
        return resultSetHandler.handleResultSets(ps);
    }
}

public class DefaultParameterHandler implements ParameterHandler {
    // eg1: org.apache.ibatis.logging.jdbc.PreparedStatementLogger@2e570ded
    /**
     * 17、针对预处理语句，设置入参
     */
    @Override
    public void setParameters(PreparedStatement ps) {
        ErrorContext.instance().activity("setting parameters").object(mappedStatement.getParameterMap().getId());

        // 18、eg1: parameterMappings[0] = ParameterMapping{property='id', mode=IN, javaType=class java.lang.Long, jdbcType=null, numericScale=null, resultMapId='null', jdbcTypeName='null', expression='null'}
        List<ParameterMapping> parameterMappings = boundSql.getParameterMappings();
        if (parameterMappings != null) {
            ...
            // eg1: typeHandler=BaseTypeHandler
            /** 19、针对预处理语句，设置入参 */
            typeHandler.setParameter(ps, i + 1, value, jdbcType);
            ...
        }
    }
}
```

#### 6、针对 ResultSet 结果集转换为 POJO

```java
public class PreparedStatementHandler extends BaseStatementHandler {
    @Override
    public <E> List<E> query(Statement statement, ResultHandler resultHandler) throws SQLException {
        ...
        // eg1: 封装结果集 resultSetHandler=DefaultResultSetHandler
        /** 1、将结果集进行封装 */
        return resultSetHandler.handleResultSets(ps);
    }
}

public class DefaultResultSetHandler implements ResultSetHandler {
    
    // 4、eg1: 执行到这里
    private ResultSetWrapper getFirstResultSet(Statement stmt) throws SQLException {
        // eg1: rs != null
        /** 5、通过JDBC获得结果集ResultSet */
        ResultSet rs = stmt.getResultSet();
        while (rs == null) {
            if (stmt.getMoreResults()) {
                rs = stmt.getResultSet();
            } else {
                /**
                 * 6、getUpdateCount()==-1,既不是结果集,又不是更新计数了.说明没的返回了。
                 * 如果getUpdateCount()>=0,则说明当前指针是更新计数(0的时候有可能是DDL指令)。
                 * 无论是返回结果集或是更新计数,那么则可能还继续有其它返回。
                 * 只有在当前指指针getResultSet()==null && getUpdateCount()==-1才说明没有再多的返回。
                 */
                if (stmt.getUpdateCount() == -1) {
                    // no more results. Must be no resultset
                    break;
                }
            }
        }
        // eg1: rs不为空，则将结果集封装到ResultSetWrapper中
        /** 7、将结果集ResultSet封装到ResultSetWrapper实例中 */
        return rs != null ? new ResultSetWrapper(rs, configuration) : null;
    }
    
    /**
     * 2、处理数据库操作的结果集
     */
    // eg1: 执行到这里
    @Override
    public List<Object> handleResultSets(Statement stmt) throws SQLException {
        ErrorContext.instance().activity("handling results").object(mappedStatement.getId());

        final List<Object> multipleResults = new ArrayList<>();
        int resultSetCount = 0;

        /** 3、首先：获得执行后的结果集，并封装到ResultSetWrapper */
        ResultSetWrapper rsw = getFirstResultSet(stmt);

        /** 8、其次：如果rsw != null && resultMapCount < 1，则抛异常ExecutorException */
        List<ResultMap> resultMaps = mappedStatement.getResultMaps();
        int resultMapCount = resultMaps.size(); // eg1: resultMapCount = 1
        validateResultMapsCount(rsw, resultMapCount);

        // eg1: rsw不为空 resultMapCount=1 resultSetCount=0
        /** 9、第三步：处理结果集 */
        while (rsw != null && resultMapCount > resultSetCount) {
            // eg1: ResultMap resultMap=resultMaps.get(0);
            ResultMap resultMap = resultMaps.get(resultSetCount);

            /** 10、处理结果集, 存储在multipleResults中 */
            handleResultSet(rsw, resultMap, multipleResults, null);

            // 33、eg1: rsw=null
            rsw = getNextResultSet(stmt);

            cleanUpAfterHandlingResultSet();
            resultSetCount++; // eg1: 自增后resultSetCount=1
        }

        String[] resultSets = mappedStatement.getResultSets();
        // eg1: 34、resultSets = null
        if (resultSets != null) {
            while (rsw != null && resultSetCount < resultSets.length) {
                ResultMapping parentMapping = nextResultMaps.get(resultSets[resultSetCount]);
                if (parentMapping != null) {
                    String nestedResultMapId = parentMapping.getNestedResultMapId();
                    ResultMap resultMap = configuration.getResultMap(nestedResultMapId);
                    handleResultSet(rsw, resultMap, null, parentMapping);
                }
                rsw = getNextResultSet(stmt);
                cleanUpAfterHandlingResultSet();
                resultSetCount++;
            }
        }

        // eg1: multipleResults.get(0).get(0) = User{id=2, name='muse2', age=24, userContacts=null}
        /** 99、返回结果 */
        return collapseSingleResultList(multipleResults);
    }
    
    // eg1: parentMapping = null
    /**
     * 11、处理结果集
     */
    private void handleResultSet(ResultSetWrapper rsw, ResultMap resultMap, List<Object> multipleResults,
                                 ResultMapping parentMapping) throws SQLException {
        try {
            // eg1: parentMapping = null
            if (parentMapping != null) {
                handleRowValues(rsw, resultMap, null, RowBounds.DEFAULT, parentMapping);
            } else {
                // eg1: resultHandler = null
                if (resultHandler == null) {
                    // eg1: objectFactory = DefaultObjectFactory defaultResultHandler里面包含了一个空集合的ArrayList实例
                    /** 12、初始化ResultHandler实例，用于解析查询结果并存储于该实例对象中 */
                    DefaultResultHandler defaultResultHandler = new DefaultResultHandler(objectFactory);
                    /** 13、解析行数据 */
                    handleRowValues(rsw, resultMap, defaultResultHandler, rowBounds, null);
                    multipleResults.add(defaultResultHandler.getResultList());
                } else {
                    handleRowValues(rsw, resultMap, resultHandler, rowBounds, null);
                }
            }
        } finally {
            // eg1：
            /** 关闭ResultSet */
            closeResultSet(rsw.getResultSet());
        }
    }
    
    // 14、eg1: parentMapping = null
    public void handleRowValues(ResultSetWrapper rsw, ResultMap resultMap, ResultHandler<?> resultHandler,
                                RowBounds rowBounds, ResultMapping parentMapping) throws SQLException {
        // eg1: resultMap.hasNestedResultMaps()=false
        /** 15、是否是聚合Nested类型的结果集 */
        if (resultMap.hasNestedResultMaps()) {
            ensureNoRowBounds();
            checkResultHandler();
            handleRowValuesForNestedResultMap(rsw, resultMap, resultHandler, rowBounds, parentMapping);
        } else {
            // 16、eg1: parentMapping = null
            handleRowValuesForSimpleResultMap(rsw, resultMap, resultHandler, rowBounds, parentMapping);
        }
    }
    
    // 17、eg1: parentMapping = null
    private void handleRowValuesForSimpleResultMap(ResultSetWrapper rsw, ResultMap resultMap,
                                                   ResultHandler<?> resultHandler, RowBounds rowBounds,
                                                   ResultMapping parentMapping) throws SQLException {
        DefaultResultContext<Object> resultContext = new DefaultResultContext<>();

        // eg1: skipRows里面没做什么事情
        /** 18、将指针移动到rowBounds.getOffset()指定的行号，即：略过（skip）offset之前的行 */
        skipRows(rsw.getResultSet(), rowBounds);

        // eg1: shouldProcessMoreRows(resultContext, rowBounds) = true    rsw.getResultSet().next() = true
        while (shouldProcessMoreRows(resultContext, rowBounds) && rsw.getResultSet().next()) {
            /** 23、解析结果集中的鉴别器<discriminate/> */
            ResultMap discriminatedResultMap = resolveDiscriminatedResultMap(rsw.getResultSet(), resultMap, null);

            /** 24、将数据库操作结果保存到POJO并返回 */
            Object rowValue = getRowValue(rsw, discriminatedResultMap);

            // eg1: rowValue=User{id=2, name='muse2', age=24, userContacts=null}  parentMapping = null
            /** 32、存储POJO对象到DefaultResultHandler中 */
            storeObject(resultHandler, resultContext, rowValue, parentMapping, rsw.getResultSet());
        }
    }
    
    // eg1:
    /**
     * 19、将指针移动到rowBounds.getOffset()指定的行号，即：略过（skip）offset之前的行
     *
     * @param rs
     * @param rowBounds
     * @throws SQLException
     */
    private void skipRows(ResultSet rs, RowBounds rowBounds) throws SQLException {
        // 20、eg1: rs.getType() = 1003 = ResultSet.TYPE_FORWARD_ONLY
        /**
         * ResultSet.TYPE_FORWARD_ONLY          结果集的游标只能向下滚动
         * ResultSet.TYPE_SCROLL_INSENSITIVE    结果集的游标可以上下移动，当数据库变化时，当前结果集不变。
         * ResultSet.TYPE_SCROLL_SENSITIVE      返回可滚动的结果集，当数据库变化时，当前结果集同步改变
         */
        if (rs.getType() != ResultSet.TYPE_FORWARD_ONLY) {
            /** rowBounds.getOffset()不为0 */
            if (rowBounds.getOffset() != RowBounds.NO_ROW_OFFSET) {
                /** 21、将指针移动到此ResultSet对象的给定行编号rowBounds.getOffset()。 */
                rs.absolute(rowBounds.getOffset());
            }
        } else {
            // eg1: rowBounds.getOffset() = 0
            for (int i = 0; i < rowBounds.getOffset(); i++) {
                /** 22、将指针移动到此ResultSet对象的给定行编号rowBounds.getOffset()。 */
                rs.next();
            }
        }
    }
    
    /**
     * 25、将数据库操作结果保存到POJO并返回
     */
    private Object getRowValue(ResultSetWrapper rsw, ResultMap resultMap) throws SQLException {
        final ResultLoaderMap lazyLoader = new ResultLoaderMap();
        /** 26、创建空的结果对象 */
        Object rowValue = createResultObject(rsw, resultMap, lazyLoader, null);

        // eg1: rowValue=User{id=null, name='null', age=null, userContacts=null}   hasTypeHandlerForResultObject(rsw, resultMap.getType())=false
        if (rowValue != null && !hasTypeHandlerForResultObject(rsw, resultMap.getType())) {
            /** 27、创建rowValue的metaObject */
            final MetaObject metaObject = configuration.newMetaObject(rowValue);

            // eg1: foundValues = useConstructorMappings = false
            boolean foundValues = this.useConstructorMappings;

            // eg1: shouldApplyAutomaticMappings(resultMap, false) = true
            /** 28、是否应用自动映射 */
            if (shouldApplyAutomaticMappings(resultMap, false)) {
                // eg1: applyAutomaticMappings(rsw, resultMap, metaObject, null)=true
                /**
                 * 29、将查询出来的值赋值给metaObject中的POJO对象
                 */
                foundValues = applyAutomaticMappings(rsw, resultMap, metaObject, null) || foundValues; // eg1: foundValues=true
            }

            // eg1: foundValues=true
            foundValues = applyPropertyMappings(rsw, resultMap, metaObject, lazyLoader, null) || foundValues;

            // eg1: lazyLoader.size()=0   foundValues=true
            foundValues = lazyLoader.size() > 0 || foundValues;

            // 31、eg1: foundValues=true  configuration.isReturnInstanceForEmptyRow()=false
            /** configuration.isReturnInstanceForEmptyRow() 当返回行的所有列都是空时，MyBatis默认返回null。当开启这个设置时，MyBatis会返回一个空实例。*/
            rowValue = (foundValues || configuration.isReturnInstanceForEmptyRow()) ? rowValue : null;
        }
        return rowValue; // eg1: rowValue=User{id=2, name='muse2', age=24, userContacts=null}
    }
    
    // eg1: columnPrefix=null
    private boolean applyAutomaticMappings(ResultSetWrapper rsw, ResultMap resultMap, MetaObject metaObject,
                                           String columnPrefix) throws SQLException {
        /** 30、创建自动映射 */
        List<UnMappedColumnAutoMapping> autoMapping = createAutomaticMappings(rsw, resultMap, metaObject, columnPrefix);
        boolean foundValues = false;

        // eg1: autoMapping={UnMappedColumnAutoMapping("id", "id", LongTypeHandler@2397, false),
        //                   UnMappedColumnAutoMapping("name", "name", StringTypeHandler@2418, false),
        //                   UnMappedColumnAutoMapping("age", "age", IntegerTypeHandler@2433, false)}
        if (autoMapping.size() > 0) {
            for (UnMappedColumnAutoMapping mapping : autoMapping) {
                // eg1: mapping.column="id"      mapping.typeHandler=LongTypeHandler       value=2L
                // eg1: mapping.column="name"    mapping.typeHandler=StringTypeHandler     value="muse2"
                // eg1: mapping.column="age"     mapping.typeHandler=IntegerTypeHandler    value=24
                final Object value = mapping.typeHandler.getResult(rsw.getResultSet(), mapping.column);
                if (value != null) {
                    // eg1: foundValues = true
                    // eg1: foundValues = true
                    // eg1: foundValues = true
                    foundValues = true;
                }
                // eg1: configuration.isCallSettersOnNulls()=false  mapping.primitive=false
                // eg1: configuration.isCallSettersOnNulls()=false  mapping.primitive=false
                // eg1: configuration.isCallSettersOnNulls()=false  mapping.primitive=false
                if (value != null || (configuration.isCallSettersOnNulls() && !mapping.primitive)) {
                    // eg1: mapping.property="id"  value=2L
                    // eg1: mapping.property="name"  value="muse2"
                    // eg1: mapping.property="age"  value=24
                    metaObject.setValue(mapping.property, value);
                }
            }
        }
        return foundValues; // eg1: 返回true
    }
}
```

### 1.9. Mybatis 是如何进行分页的？分页插件的原理是什么？

1. Mybatis 使用 RowBounds 对象进行分页，它是针对 ResultSet 结果集执行的**内存分页**，而非物理分页，可以在 sql 内直接书写带有物理分页的参数，来完成物理分页功能，也可以使用分页插件来完成物理分页。
2. 分页插件的基本原理是，使用 Mybatis 提供的插件接口，实现自定义插件，在插件的拦截方法内，拦截待执行的 sql，然后重写 sql，根据 dialect 方言，添加对应的物理分页语句和物理分页参数。
3. 比如，`select * from student`，拦截 sql 后重写为 `select t.* from (select * from student) t limit 0, 10`，从而完成插件分页的功能。

### 2.0. Mybatis 插件运行原理，以及如何编写一个插件？

1. Mybatis 仅可以编写针对 ParameterHandler、ResultSetHandler、StatementHandler、Executor 这4种接口的插件。
2. Mybatis 使用 JDK 动态代理，为需要拦截的接口生成代理对象，以实现接口方法拦截功能，每当执行这 4 种接口对象的方法时，就会进入拦截方法，具体就是 `InvocationHandler#invoke()` 方法，拦截那些指定需要拦截的方法。
3. 具体做法为，实现 `Mybatis#Interceptor#intercept()` 方法，然后在给插件编写注解，指定要拦截哪一个接口的哪些方法，最后在配置文件中，配置所编写的插件即可。

