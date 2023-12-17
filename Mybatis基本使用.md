
## 数据准备

```sql
CREATE TABLE `user`
(
    `id`       INT(11)      NOT NULL AUTO_INCREMENT,
    `name`     VARCHAR(64)  NOT NULL COMMENT '用户名',
    `password` VARCHAR(256) NOT NULL COMMENT '密码',
    PRIMARY KEY (`id`)
) ENGINE = INNODB
  AUTO_INCREMENT = 1
  DEFAULT CHARSET = utf8mb4 COMMENT = '用户信息表';
```

## 新建项目 & 引入依赖

这里使用 maven 来构建项目，引入基础依赖即可
-> 

- [x] 我
- [ ]   


1. 

```xml
<dependencies>
	<!-- java程序连接MySql所必须的依赖 -->
	<dependency>
	    <groupId>mysql</groupId>
	    <artifactId>mysql-connector-java</artifactId>
	    <version>8.0.28</version>
	</dependency>

	<dependency>
	    <groupId>org.mybatis</groupId>
	    <artifactId>mybatis</artifactId>
	    <version>3.5.6</version>
	</dependency>
	
	<dependency>
	    <groupId>org.projectlombok</groupId>
	    <artifactId>lombok</artifactId>
	    <version>1.18.22</version>
	    <scope>provided</scope>
	</dependency>
	
	<dependency>
	    <groupId>junit</groupId>
	    <artifactId>junit</artifactId>
	    <version>4.13.1</version>
	    <scope>test</scope>
	</dependency>
	
	<dependency>
	    <groupId>com.alibaba.fastjson2</groupId>
	    <artifactId>fastjson2</artifactId>
	    <version>2.0.19</version>
	</dependency>

</dependencies>
```

## Mybatis 配置

在 resource 目录下新建 mybatis-config.xml，这里仅在配置文件中设置了：**sql环境**、**映射器mapper**

### mybatis-confit.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "https://mybatis.org/dtd/mybatis-3-config.dtd">

<configuration>

    <!-- 可以引入外部的配置文件，会进行动态替换 -->
    <properties resource="datasource.properties"></properties>

    <!-- 环境，可以配置多个，default：指定采用哪个环境 -->
    <environments default="dev">

        <environment id="dev">
            <!-- 事务管理器，JDBC类型的事务管理器 -->
            <transactionManager type="JDBC"/>
            <!-- 数据源，池类型的数据源 -->
            <dataSource type="POOLED">
                <property name="driver" value='${datasource.driver}'/>
                <property name="url" value='${datasource.url}'/>
                <property name="username" value='${datasource.username}'/>
                <property name="password" value='${datasource.password}'/>
            </dataSource>
        </environment>

    </environments>

    <mappers>
        <mapper resource="mappers/userMapper.xml"/>
    </mappers>

</configuration>
```

### datasource.properties

```yaml
mysql 8.0 以上用这个驱动
datasource.driver = com.mysql.cj.jdbc.Driver
db的连接属性
datasource.url = jdbc:mysql://localhost:3306/mybatis_study?useUnicode=true&characterEncoding=utf-8
datasource.username = root
datasource.password = 12345678
```

## Java代码

需要 User 的实体类，然后定义相应的 UserMapper 接口及 sql

### User 实体类

```java
@Data
public class User {

    /**
     * 唯一id
     */
    private Integer id;

    /**
     * 用户名
     */
    private String name;

    /**
     * 密码
     */
    private String password;

}
```

### UserMapper 接口

```java
public interface UserMapper {

    /**
     * 根据id查询用户信息
     * @param id
     * @return
     */
    User getUserById(Integer id);

    /**
     * 新增用户
     * @param user
     */
    void addUser(User user);

    /**
     * 修改用户密码
     * @param id
     * @param password
     */
    Integer updateUserPassword(@Param("id") Integer id, @Param("password") String password);

    /**
     * 根据id删除用户信息
     * @param id
     */
    Integer deleteUser(Integer id);
}
```

### sql

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">

<!-- namespace要求全局唯一，所以一般都写对应Mapper接口的路径 -->
<mapper namespace="com.yezi.mybatis.study.dao.UserMapper">
    <resultMap id="BaseResultMap" type="com.yezi.mybatis.study.model.User">
        <result column="id" property="id" jdbcType="INTEGER"/>
        <result column="name" property="name" jdbcType="VARCHAR"/>
        <result column="password" property="password" jdbcType="VARCHAR"/>
    </resultMap>

		<!-- 注意这里的resultMap和resultType区别 -->
    <select id="getUserById" resultMap="BaseResultMap">
        select *
        from user
        where id = #{id}
    </select>

    <insert id="addUser" parameterType="com.yezi.mybatis.study.model.User">
        insert into user(name, password)
        values (#{name}, #{password})
    </insert>

    <update id="updateUserPassword">
        update user
        set password=#{password}
        where id = #{id}
    </update>

    <delete id="deleteUser">
        delete
        from user
        where id = #{id}
    </delete>
    
</mapper>
```

注意查询标签中 resultMap 和 resultType 的区别：
- resultMap：可以自定义实体类和db记录的映射规则，一般用于**复杂对象**，例如上面自定义的 BaseResultMap，将 user bean 的属性与 db 中 user 记录的属性进行映射
- resultType：由 mybatis 来进行实体类与 db 记录的映射，这就要求实体类的属性名称要与 db 记录的属性名称一致，不太符合实际场景，所以仅当返回对象是简单类型时，如果 Integer、String，可以使用，否则优先使用 resultMap
---

## 测试

在 `test` 文件夹下新建 `UserMapper` 的测试类，这里就简单验证 CRUD

```java
public class UserMapperTest {

    private UserMapper userMapper;

    private SqlSession sqlSession;

    @Before
    public void setUp() throws IOException {
        InputStream inputStream = Resources.getResourceAsStream("mybatis-config.xml");
        SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);
        sqlSession = sqlSessionFactory.openSession();
        userMapper = sqlSession.getMapper(UserMapper.class);

    }

    @Test
    public void test_getUserById() {
        User user = userMapper.getUserById(1);
        System.out.println(JSON.toJSON(user));
    }

    @Test
    public void test_addUser() {
        User user = new User();
        user.setName("李四");
        user.setPassword("123123");
        userMapper.addUser(user);
        sqlSession.commit();
    }

    @Test
    public void test_updateUserPassword() {
        System.out.println("affected rows: " + userMapper.updateUserPassword(1, "11111111"));
        sqlSession.commit();
    }

    @Test
    public void test_deleteUser() {
        System.out.println("affected rows: " + userMapper.deleteUser(1));
        sqlSession.commit();
    }
}
```

## 总结

以上就完成了 Mybatis 的独立使用：

1. 引入依赖，创建配置文件
    
2. 创建相应的 Mapper 接口及 sql
    
3. 创建 SqlSessionFactory，并以此拿到 SqlSession
    
4. 通过 SqlSession 拿到前面创建的 Mapper 接口实例（Mybatis 通过动态代理的方式生成实例，不需要手动编写实现类）
    
5. 最后使用 Mapper 的方法操作 db，若是 insert、update 等事务操作，需要最后调用 sqlSession.commit() 来提交事务
    

---

# Spring 整合 Mybatis

在现在服务基本都是 Spring 的背景下，Mybatis 也基本上不会独立使用，都是结合 Spring 或 SpringBoot 使用，本节就在上面的基础上，引入 Spring 框架

## 配置 Maven依赖

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.yezi</groupId>
    <artifactId>mybatis-study</artifactId>
    <version>1.0-SNAPSHOT</version>

    <properties>
        <spring.core.version>5.3.22</spring.core.version>
        <spring.dao.version>5.3.22</spring.dao.version>
        <spring.test.version>5.3.22</spring.test.version>
        <mysql.connector.version>8.0.28</mysql.connector.version>
        <mybatis.version>3.5.6</mybatis.version>
        <lombok.version>1.18.22</lombok.version>
        <junit.version>4.13.1</junit.version>
        <fastjson.version>2.0.19</fastjson.version>
        <druid.version>1.2.8</druid.version>
        <mybatis.spring.version>2.0.6</mybatis.spring.version>

        <maven.compiler.source>11</maven.compiler.source>
        <maven.compiler.target>11</maven.compiler.target>
    </properties>

    <dependencies>

        <!-- spring核心依赖：core、context、beans -->
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-core</artifactId>
            <version>${spring.core.version}</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-context</artifactId>
            <version>${spring.core.version}</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-beans</artifactId>
            <version>${spring.core.version}</version>
        </dependency>

        <!-- spring dao相关依赖 -->
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-jdbc</artifactId>
            <version>${spring.core.version}</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-tx</artifactId>
            <version>${spring.core.version}</version>
        </dependency>

        <!-- spring test -->
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-test</artifactId>
            <version>${spring.test.version}</version>
            <scope>test</scope>
        </dependency>

        <!-- 连接mysql -->
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>${mysql.connector.version}</version>
        </dependency>

        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>druid</artifactId>
            <version>${druid.version}</version>
        </dependency>

        <dependency>
            <groupId>org.mybatis</groupId>
            <artifactId>mybatis</artifactId>
            <version>${mybatis.version}</version>
        </dependency>

        <dependency>
            <groupId>org.mybatis</groupId>
            <artifactId>mybatis-spring</artifactId>
            <version>${mybatis.spring.version}</version>
        </dependency>

        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <version>${lombok.version}</version>
            <scope>provided</scope>
        </dependency>

        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>${junit.version}</version>
            <scope>test</scope>
        </dependency>

        <dependency>
            <groupId>com.alibaba.fastjson2</groupId>
            <artifactId>fastjson2</artifactId>
            <version>${fastjson.version}</version>
        </dependency>

    </dependencies>

</project>
```

[简单使用的Demo](https://github.com/climb0/mybatis-study/tree/main)