# MyBatis-Plus 是如何提升效率的



> **MyBatis 是一个通过 XML 或注解将 Java 对象与 SQL 语句自动关联（映射）的持久层框架**



## 演变过程

### 1. 传统JDBC方法

1. 准备好数据库，新建**Maven 项目**，引入驱动

   ```xml
   <dependencies>
       <dependency>
           <groupId>mysql</groupId>
           <artifactId>mysql-connector-java</artifactId>
           <version>8.0.33</version>
       </dependency>
   </dependencies>
   ```

2. 编写JDBC代码

   ```java
   import java.sql.*;
   
   public class JdbcDemo {
       public static void main(String[], args) {
           // 数据库信息
           String url = "jdbc:mysql://localhost:3306/test_db";
           String user = "root";
           String password = "123456";
   
           try(Connection conn = DriverManager.getConnection(url, user, password);
               PreparedStatement pstmt = conn.prepareStatement("SELECT * FROM user");
               ResultSet rs = pstmt.executeQuery()) {
   
                   System.out.println("连接成功！查询结果如下：");
                   while (rs.next()) {
                       System.out.println("姓名：" + rs.getString("name") + "，年龄：" + rs.getInt("age"));
                   }
               } catch (SQLException e) {
                   e.printStackTrace();
               }
       }
   }
   ```

   1. **准备账号**：定义数据库地址、用户名和密码。
   2. **拨号连接**：用 `DriverManager` 建立 `Connection`（物理通道）。
   3. **准备指令**：把 SQL 语句交给 `PreparedStatement`（准备发车）。
   4. **执行任务**：发出指令，拿到 `ResultSet`（带回数据包裹）。
   5. **手动拆箱**：用 `while(rs.next())` 把包裹里的数据一行行取出来。
   6. **自动挂断**：`try-with-resources` 任务结束自动断开，防止占线



### 2. MyBatis

1. 配置`mybatis-config.xml`管理数据库连接环境

   > 代替 JDBC 的 `url`、`user`、`password` 配置

   ```xml
   <configuration>
       <environments default="development">
           <environment id="development">
               <transactionManager type="JDBC"/>
               <!-- 数据库连接池，不用每次都重新开管道 -->
               <dataSource type="POOLED">
                   <property name="driver" value="com.mysql.cj.jdbc.Driver"/>
                   <property name="url" value="jdbc:mysql://localhost:3306/test_db"/>
                   <property name="username" value="root"/>
                   <property name="password" value="123456"/>
               </dataSource>
           </environment>
       </environments>
       
       <!-- 告诉 MyBatis 去哪里找具体的 SQL 映射文件 -->
       <mappers>
           <mapper resource="com/example/mapper/UserMapper.xml"/>
       </mappers>
   </configuration>
   ```

2. 写DO层

   > 代替 JDBC `while` 循环里的手动变量接收，作为一个“容器”

   ```java
   public class User {
       private Integer id;
       private String name;
       private Integer age;
   
       // 要么就用Lombok
       public String getName() { return name; }
       public void setName(String name) { this.name = name; }
   }
   ```

3. 写Mapper层

   > Interface + XML

   1. **接口（Interface）**

      ```java
      public interface UserMapper {
          // 定义一个方法名，不需要写实现类
          User findUserById(Integer id);
      }
      ```

   2. **XML 映射文件 (SQL)**

      > 代替 JDBC 的 `pstmt.executeQuery()` 和 `rs.next()`

      ```xml
      <mapper namespace="com.example.mapper.UserMapper">
          <!-- id 必须和接口方法名一致 -->
          <!-- resultType 告诉 MyBatis：查出来的结果直接装进 User 对象 -->
          <select id="findUserById" resultType="com.example.User">
              SELECT * FROM user WHERE id = #{id}
          </select>
      </mapper>
      ```

      

### 3. MyBatis-Plus

1. 在 `pom.xml` 中把原生 MyBatis 换成 [MyBatis-Plus Starter](https://baomidou.com/getting-started/)

2. 写DO

   ```java
   @TableName("user")
   public class UserDO {
       @TableId(type = IdType.AUTO)
       private Long id;
       private String name;
       private Integer age;
       // Getter/Setter 略
   }
   ```

3. 写Mapper

   > ```
   > 继承了 BaseMapper<UserDO>，你就拥有了插入、删除、修改、全表查询等 17 个方法
   > ```

   ```java
   public interface UserMapper extends BaseMapper<UserDO> {
       
   }
   ```

4. 写Service

   ```java
   // 1. 定义接口继承 IService
   public interface UserService extends IService<UserDO> { }
   
   // 2. 编写实现类继承 ServiceImpl
   @Service
   public class UserServiceImpl extends ServiceImpl<UserMapper, UserDO> implements UserService {
       // 自动关联了 UserMapper，自带批量保存等高级功能
   }
   ```

   

