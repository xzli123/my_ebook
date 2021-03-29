[toc]
# Mybatis和Mybatisplus的对比

```
MyBatis：一种操作数据库的框架，提供一种Mapper类，支持让你用java代码进行增删改查的数据库操作，省去了每次都要手写sql语句的麻烦。但是有一个前提，你得先在xml中写好sql语句，是不是很麻烦？于是有下面的Mybatis Plus
Mybatis Generator：自动为Mybatis生成简单的增删改查sql语句的工具，省去一大票时间，两者配合使用，开发速度快到飞起。
Mybatis Plus：国人团队苞米豆在Mybatis的基础上开发的框架，在Mybatis基础上扩展了许多功能，荣获了2018最受欢迎国产开源软件第5名，当然也有配套的Mybatis Plus Generator
Mybatis Plus Generator：同样为苞米豆开发，比Mybatis Generator更加强大，支持功能更多，自动生成Entity、Mapper、Service、Controller等
总结：
数据库框架：Mybatis Plus > Mybatis
代码生成器：Mybatis Plus Generator > Mybatis Generator


Mybatis-Plus是一个Mybatis的增强工具，它在Mybatis的基础上做了增强，却不做改变。我们在使用Mybatis-Plus之后既可以使用Mybatis-Plus的特有功能，又能够正常使用Mybatis的原生功能。Mybatis-Plus(以下简称MP)是为简化开发、提高开发效率而生，但它也提供了一些很有意思的插件，比如SQL性能监控、乐观锁、执行分析等
```
[Mybatis-Plus官网](https://baomidou.com/)

[tkmybatis VS mybatisplus](https://blog.csdn.net/u013076044/article/details/95376200)

# mybatis-plus主键生成策略说明
- AUTO 数据库ID自增
- INPUT 用户输入ID
- ID_WORKER 全局唯一ID，Long类型的主键
- ID_WORKER_STR 字符串全局唯一ID
- UUID 全局唯一ID，UUID类型的主键
- NONE 该类型为未设置主键类型


# 集成步骤
## pom依赖（注意版本依赖）

```
	<parent>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-parent</artifactId>
		<version>2.2.1.RELEASE</version>
		<relativePath/> <!-- lookup parent from repository -->
	</parent>
	
	<dependencies>
			<!--mybatis-plus配置 -->
		<dependency>
			<groupId>com.baomidou</groupId>
			<artifactId>mybatis-plus-boot-starter</artifactId>
			<version>3.0.7.1</version>
		</dependency>
	</dependencies>	
```
## application.yml配置

```
#mybatis
mybatis-plus:
  # xml
  mapper-locations: classpath:mapper/*Mapper.xml
  # 实体扫描，多个package用逗号或者分号分隔
  type-aliases-package: com.example.activitidemo.model
  global-config:
    #刷新mapper 调试神器
    db-config:
      #主键类型  0:"数据库ID自增", 1:"用户输入ID",2:"全局唯一ID (数字类型唯一ID)", 3:"全局唯一ID UUID";
      id-type: ID_WORKER
      #字段策略 0:"忽略判断",1:"非 NULL 判断"),2:"非空判断"
      field-strategy: NOT_EMPTY
      #驼峰下划线转换
      #column-underline: true
      #数据库大写下划线转换
      #capital-mode: true
      #逻辑删除配置
      logic-delete-value: 1
      logic-not-delete-value: 0
      db-type: mysql
    refresh: true
      #自定义填充策略接口实现
      #meta-object-handler: com.baomidou.springboot.xxx
    #自定义SQL注入器
    #sql-injector: com.baomidou.springboot.xxx
  configuration:
    map-underscore-to-camel-case: true
    cache-enabled: false
```

## 分页配置

在启动项中配置如下内容
```
    /**
     * 分页插件
     */
    @Bean
    public PaginationInterceptor paginationInterceptor() {
        return new PaginationInterceptor();
    }
```


## 编码

### model

```
/**
 * <p>
 *
 * </p>
 *
 * @author xzli
 * @since 2020-09-28
 */
@Data
@EqualsAndHashCode(callSuper = false)
@Accessors(chain = true)
@TableName("user")
public class User extends Model<User> {

    private static final long serialVersionUID=1L;

    /**
     * 主键ID
     */
    @TableId(value = "id", type = IdType.ID_WORKER)
    private Long id;

    /**
     * 姓名
     */
    private String name;

    /**
     * 年龄
     */
    private Integer age;

    /**
     * 邮箱
     */
    private String email;


    @Override
    protected Serializable pkVal() {
        return this.id;
    }

}
```

### mapper.xml

```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.example.activitidemo.mapper.UserMapper">

</mapper>

```
### mapper

```
/**
 * @Author: xzli
 * @Date: 2020/9/27 16:50
 * @Description TODO
 */
public interface UserMapper extends BaseMapper<User> {

}
```
### service

```
/**
 * <p>
 *  服务类
 * </p>
 *
 * @author xzli
 * @since 2020-09-28
 */
public interface UserService extends IService<User> {

}
```
### service.impl

```
/**
 * <p>
 *  服务实现类
 * </p>
 *
 * @author xzli
 * @since 2020-09-28
 */
@Service
public class UserServiceImpl extends ServiceImpl<UserMapper, User> implements UserService {

}
```

### controller
```
/**
 *
 * @author xzli
 * @since 2020-09-28
 */
@RestController
@RequestMapping("/user")
public class UserController {


    @Autowired
    private UserService userService;

    /**
     * user全表查询
     */
    @GetMapping("/list")
    public List<User> list() {
        List<User> list = userService.list();
        return list;
    }


    /**
     * 只查询user表的id和name信息
     */
    @GetMapping("/select")
    public List<User> select() {
        List<User> list = userService.lambdaQuery().select(User::getId, User::getName).list();
        return list;
    }


    /**
     * 查询user表，过滤出name等于某个值的数据
     */
    @GetMapping("/get/{name}")
    public User getOne(@PathVariable String name) {
        QueryWrapper<User> queryWrapper = new QueryWrapper<>();
        queryWrapper.eq("NAME", name);
        queryWrapper.orderByAsc("AGE");
        User user = userService.getOne(queryWrapper);
        return user;
    }

    /**
     * 插入一条数据
     */
    @PostMapping("/add")
    public List<User> add(@RequestBody User user) {
        userService.save(user);
        List<User> list = userService.list();
        return list;
    }

    /**
     * 插入一条数据后返回当前插入数据的id
     */
    @PostMapping("/add/returnId")
    public Long addReturnId(@RequestBody User user) {
        userService.save(user);
        return user.getId();
    }

    /**
     * 根据id删除数据
     */
    @DeleteMapping("/delete/{id}")
    public List<User> del(@PathVariable Long id) {
        userService.removeById(id);
        List<User> list = userService.list();
        return list;
    }

    /**
     * 根据id更新数据
     */
    @PatchMapping("/update")
    public List<User> update(@RequestBody User user) {
//        UpdateWrapper<User> updateWrapper = new UpdateWrapper<>();
//        updateWrapper.eq("NAME", user.getName());
//        userService.update(user, updateWrapper);

        userService.updateById(user);

        List<User> list = userService.list();
        return list;
    }

    /**
     * 分页查询
     * @param page  当前页
     * @param size  每页展示的数据
     * @return
     */
    @GetMapping("/listpage/{page}/{size}")
    public IPage<User> listPage(@PathVariable Integer page, @PathVariable Integer size) {
        IPage<User> iPage = userService.page(new Page<>(page, size));
        return iPage;
    }


    /**
     * 按照查询条件，查询后的结果做分页查询
     * @param page  当前页
     * @param size  每页展示的数据
     * @param name  查询条件
     * @return
     */
    @GetMapping("/listpage/{page}/{size}/{name}")
    public IPage<User> listPageByName(@PathVariable Integer page, @PathVariable Integer size, @PathVariable String name) {

        QueryWrapper<User> queryWrapper = new QueryWrapper<>();
        queryWrapper.eq("NAME", name);

        IPage<User> iPage = userService.page(new Page<>(page, size), queryWrapper);
        return iPage;
    }


}


```
