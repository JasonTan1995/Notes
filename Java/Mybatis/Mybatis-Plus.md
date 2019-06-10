### Mybatis-Plus

#### 1. 概述

------

`Mybatis-Plus`是一个`Mybatis`的增强工具,在`Mybatis`的基础上只做增强不做改变，为简化开发，提高效率而生。`Mybatis-plus`在处理一些开发比较常用的例如：分页查询，条件查询，CRUD等等都作出了一定的封装，使得开发者不需要写`sql`语句。

[Mybatis官方网站](https://mybatis.plus/)

![](https://mybatis.plus/img/relationship-with-mybatis.png)

#### 2. 特性

------

- **无侵入**：只做增强不做改变，引入它不会对现有工程产生影响，如丝般顺滑
- **损耗小**：启动即会自动注入基本 CURD，性能基本无损耗，直接面向对象操作
- **强大的 CRUD 操作**：内置通用 Mapper、通用 Service，仅仅通过少量配置即可实现单表大部分 CRUD 操作，更有强大的条件构造器，满足各类使用需求
- **支持 Lambda 形式调用**：通过 Lambda 表达式，方便的编写各类查询条件，无需再担心字段写错
- **支持多种数据库**：支持 MySQL、MariaDB、Oracle、DB2、H2、HSQL、SQLite、Postgre、SQLServer2005、SQLServer 等多种数据库
- **支持主键自动生成**：支持多达 4 种主键策略（内含分布式唯一 ID 生成器 - Sequence），可自由配置，完美解决主键问题
- **内置分页插件**：基于 MyBatis 物理分页，开发者无需关心具体操作，配置好插件之后，写分页等同于普通 List 查询
- **内置性能分析插件**：可输出 Sql 语句以及其执行时间，建议开发测试时启用该功能，能快速揪出慢查询
- **内置全局拦截插件**：提供全表 delete 、 update 操作智能分析阻断，也可自定义拦截规则，预防误操作
- **内置 Sql 注入剥离器**：支持 Sql 注入剥离，有效预防 Sql 注入攻击



#### 3. 通用CRUD

------

- 通用CRUD封装`BaseMapper`接口，为 `Mybatis-Plus` 启动时自动解析实体表关系映射转换为 `Mybatis` 内部对象注入容器。
- 泛型 `T` 为任意实体对象
- 参数 `Serializable` 为任意类型主键 `Mybatis-Plus` 不推荐使用复合主键约定每一张表都有自己的唯一 `id` 主键
- 对象 `Wrapper` 为 [条件构造器](https://mybatis.plus/guide/wrapper.html)

```java
/**
 * <p>
  *  Mapper 接口
 * </p>
 *
 * @author Jason Tan
 * @since 2017-08-23
 */
public interface UserMapper extends BaseMapper<User> {

}
```



#### 4. 快速使用

------

##### 添加依赖

我们若要使用`Mybatis-Plus`，只需要在`Maven`工程中引入相关的依赖即可使用。（使用`Spring-Boot作为例子`）

```xml
<dependency>
        <groupId>com.baomidou</groupId>
        <artifactId>mybatis-plus-boot-starter</artifactId>
        <version>3.1.1</version>
    </dependency>
```



##### 配置

添加数据库的相关信息

```yml
# DataSource Config
spring:
  datasource:
    driver-class-name: org.h2.Driver
    schema: classpath:db/schema-h2.sql
    data: classpath:db/data-h2.sql
    url: jdbc:h2:mem:test
    username: root
    password: test
```

在`SpringBoot`启动类中添加`@MapperScan`注解，扫描Mapper(文件夹)

```java
@SpringBootApplication
@MapperScan("com.angular.test.mapper")
public class TestApplication {

    public static void main(String[] args) {
        SpringApplication.run(TestApplication.class, args);
    }

}
```



编写实体类：

```java
@Data
public class User {
    private Long id;
    private String name;
    private Integer age;
    private String email;
}
```

编写`Mapper`

```java
public interface UserMapper extends BaseMapper<User> {

}
```



##### 测试

```java
@RunWith(SpringRunner.class)
@SpringBootTest
public class SampleTest {

    @Autowired
    private UserMapper userMapper;

    @Test
    public void testSelect() {
        System.out.println(("----- selectAll method test ------"));
        List<User> userList = userMapper.selectList(null);
        Assert.assertEquals(5, userList.size());
        userList.forEach(System.out::println);
    }

}
```

通过以上几个简单的步骤，我们就实现了 User 表的 CRUD 功能，甚至连 XML 文件都不用编写！

从以上步骤中，我们可以看到集成`MyBatis-Plus`非常的简单，只需要引入 starter 工程，并配置 mapper 扫描路径即可。



#### 5. 条件构造

实体包装器，用于处理 sql 拼接，排序，实体参数查询等！

> 补充说明： 使用的是数据库字段，不是Java属性!

实体包装器 EntityWrapper 继承 Wrapper.



##### 分页查询+条件查询

```java
 // 1. Create conditional
EntityWrapper<MoocCinemaT> wrapper = new EntityWrapper<>();
Page<MoocCinemaT> page = new Page<>(cinemaQueryVO.getNowPage(), cinemaQueryVO.getPageSize());
if (cinemaQueryVO.getBrandId() != 99) {
  wrapper.eq("brand_id", cinemaQueryVO.getBrandId());
}
if (cinemaQueryVO.getDistrictId() != 99) {
  wrapper.eq("area_id", cinemaQueryVO.getDistrictId());
}
if (cinemaQueryVO.getHallType() != 99) {
  wrapper.like("hall_ids", "%" + cinemaQueryVO.getHallType() + "%");
}

//2. Query Results
List<MoocCinemaT> moocCinemaTS = moocCinemaTMapper.selectPage(page, wrapper);
```

分页查询的主要对象为：`Page`，入参为当前页以及查询的条数，查询后会得出总记录数，页数等。

条件查询的主要对象为：`EntityWrapper`

```javaa
/**
 * <p>
 * 实现分页辅助类
 * </p>
 *
 * @author hubin
 * @Date 2016-03-01
 */
public class Page<T> extends Pagination {

    private static final long serialVersionUID = 1L;

    /**
     * 查询数据列表
     */
    private List<T> records = Collections.emptyList();

    /**
     * 查询参数（ 不会传入到 xml 层，这里是 Controller 层与 service 层传递参数预留 ）
     */
    private Map<String, Object> condition;

    public Page() {
        /* 注意，传入翻页参数 */
    }

    public Page(int current, int size) {
        super(current, size);
    }

    public Page(int current, int size, String orderByField) {
        super(current, size);
        this.setOrderByField(orderByField);
    }

    public Page(int current, int size, String orderByField, boolean isAsc) {
        this(current, size, orderByField);
        this.setAsc(isAsc);
    }

    public List<T> getRecords() {
        return records;
    }

    public Page<T> setRecords(List<T> records) {
        this.records = records;
        return this;
    }

    @Transient
    public Map<String, Object> getCondition() {
        return condition;
    }

    public Page<T> setCondition(Map<String, Object> condition) {
        this.condition = condition;
        return this;
    }

    @Override
    public String toString() {
        StringBuilder pg = new StringBuilder();
        pg.append(" Page:{ [").append(super.toString()).append("], ");
        if (records != null) {
            pg.append("records-size:").append(records.size());
        } else {
            pg.append("records is null");
        }
        return pg.append(" }").toString();
    }

}
```



##### 条件参数

[条件构造器](https://baomidou.gitee.io/mybatis-plus-doc/#/wrapper)

| 查询方式     | 说明                              |
| ------------ | --------------------------------- |
| setSqlSelect | 设置 SELECT 查询字段              |
| where        | WHERE 语句，拼接 + `WHERE 条件`   |
| and          | AND 语句，拼接 + `AND 字段=值`    |
| andNew       | AND 语句，拼接 + `AND (字段=值)`  |
| or           | OR 语句，拼接 + `OR 字段=值`      |
| orNew        | OR 语句，拼接 + `OR (字段=值)`    |
| eq           | 等于=                             |
| allEq        | 基于 map 内容等于=                |
| ne           | 不等于<>                          |
| gt           | 大于>                             |
| ge           | 大于等于>=                        |
| lt           | 小于<                             |
| le           | 小于等于<=                        |
| like         | 模糊查询 LIKE                     |
| notLike      | 模糊查询 NOT LIKE                 |
| in           | IN 查询                           |
| notIn        | NOT IN 查询                       |
| isNull       | NULL 值查询                       |
| isNotNull    | IS NOT NULL                       |
| groupBy      | 分组 GROUP BY                     |
| having       | HAVING 关键词                     |
| orderBy      | 排序 ORDER BY                     |
| orderAsc     | ASC 排序 ORDER BY                 |
| orderDesc    | DESC 排序 ORDER BY                |
| exists       | EXISTS 条件语句                   |
| notExists    | NOT EXISTS 条件语句               |
| between      | BETWEEN 条件语句                  |
| notBetween   | NOT BETWEEN 条件语句              |
| addFilter    | 自由拼接 SQL                      |
| last         | 拼接在最后，例如：last("LIMIT 1") |



#### 6. 一对多关系查询

开发中比较常见的就是一对多关系查询了，这方面呢就只能写`sql`语句了。这方面就是`Mybatis`中的东西了。使用到`resultMap`.

例子：影院中的一套电影对应多场场次。

```xml
<resultMap id="getFilmInfoMap" type="com.stylefeng.guns.api.cinema.vo.FilmInfoVO">
        <result column="film_id" property="filmId"/>
        <result column="film_name" property="filmName"/>
        <result column="film_cats" property="filmCats"/>
        <result column="film_length" property="filmLength"/>
        <result column="actors" property="actors"/>
        <result column="img_address" property="imgAddress"/>
        <result column="film_language" property="filmType"/>
        <collection property="filmFields"
                    javaType="java.util.List" ofType="com.stylefeng.guns.api.cinema.vo.FilmFieldVO">
            <result column="UUID" property="fieldId"/>
            <result column="begin_time" property="beginTime"/>
            <result column="end_time" property="endTime"/>
            <result column="film_language" property="language"/>
            <result column="hall_name" property="hallName"/>
            <result column="price" property="price"/>
        </collection>
    </resultMap>

    <select id="getFilmInfos" parameterType="java.lang.Integer" resultMap="getFilmInfoMap">
        SELECT
        info.film_id,
        info.`film_name`,
        info.`film_length`,
        info.`film_language`,
        info.`film_cats`,
        info.`actors`,
        info.`img_address`,
        f.`UUID`,
        f.`begin_time`,
        f.`end_time`,
        f.`hall_name`,
        f.`price`
        FROM
        hall_film_info_t info
        LEFT JOIN
        field_t f
        ON f.`film_id` = info.`film_id`
        AND f.`cinema_id` = #{cinemaId}
    </select>
```

hall_film_info_t：

```sql
1	2	我不是药神	117	喜剧,剧情	国语2D	程勇,曹斌,吕受益,刘思慧	films/238e2dc36beae55a71cabfc14069fe78236351.jpg
2	3	爱情公寓	123	喜剧,动作,冒险	国语2D	曾小贤,胡一菲,唐悠悠,张伟	films/238e2dc36beae55a71cabfc14069fe78236351.jpg
```

field_t ：

```sql
1	1	2	09:50	11:20	1	一号厅	60
2	1	3	11:50	13:20	2	IMAX厅	60
3	1	3	13:50	15:20	3	飞翔体验厅	60
4	3	2	11:50	13:20	3	7号超大厅	60
5	3	2	11:50	13:20	4	飞翔体验厅	60
6	5	2	11:50	13:20	5	3号VIP厅	60
7	6	2	11:50	13:20	6	5号4D厅	60
```

