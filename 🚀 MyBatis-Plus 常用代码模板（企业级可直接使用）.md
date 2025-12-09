# 🚀 MyBatis-Plus 常用代码模板（企业级可直接使用）

------

## 1️⃣ Mapper 基础模板（必须继承 BaseMapper）

```java
@Mapper
public interface UserMapper extends BaseMapper<User> {
}
```

------

## 2️⃣ 单表 CRUD 模板

### ✔ 查询单个

```java
User user = userMapper.selectById(1L);
```

### ✔ 查询全部

```java
List<User> list = userMapper.selectList(null);
```

### ✔ 新增

```java
User user = new User();
user.setName("张三");
user.setAge(20);
userMapper.insert(user);
```

### ✔ 修改

```java
User user = new User();
user.setId(1L);
user.setAge(22);
userMapper.updateById(user);
```

### ✔ 删除

```java
userMapper.deleteById(1L);
```

------

## 3️⃣ 分页查询模板

### ✔ 配置分页插件（一次即可）

```java
@Bean
public MybatisPlusInterceptor mybatisPlusInterceptor() {
    MybatisPlusInterceptor interceptor = new MybatisPlusInterceptor();
    interceptor.addInnerInterceptor(new PaginationInnerInterceptor());
    return interceptor;
}
```

### ✔ 使用分页

```java
Page<User> page = userMapper.selectPage(new Page<>(1, 10), null);
List<User> records = page.getRecords();
long total = page.getTotal();
```

------

## 4️⃣ QueryWrapper 条件查询模板

### ✔ 单条件

```java
QueryWrapper<User> wrapper = new QueryWrapper<>();
wrapper.eq("name", "张三");
userMapper.selectList(wrapper);
```

### ✔ 多条件 + 排序

```java
QueryWrapper<User> wrapper = new QueryWrapper<>();
wrapper.eq("name", "张三")
       .ge("age", 18)
       .orderByDesc("create_time");
```

------

## 5️⃣ LambdaQueryWrapper（推荐写法）

### ✔ 查询

```java
LambdaQueryWrapper<User> wrapper = new LambdaQueryWrapper<>();
wrapper.eq(User::getName, "张三")
       .like(User::getEmail, "@qq.com");

userMapper.selectList(wrapper);
```

### ✔ 条件分页

```java
Page<User> page =
        userMapper.selectPage(new Page<>(1, 10),
        new LambdaQueryWrapper<User>().like(User::getName, "张"));
```

### ✔ 条件更新

```java
LambdaUpdateWrapper<User> wrapper = new LambdaUpdateWrapper<>();
wrapper.eq(User::getName, "张三")
       .set(User::getAge, 30);

userMapper.update(null, wrapper);
```

------

## 6️⃣ 批量操作模板

### ✔ 批量删除

```java
userMapper.deleteBatchIds(Arrays.asList(1,2,3));
```

### ✔ 批量查询

```java
userMapper.selectBatchIds(Arrays.asList(1,2,3));
```

------

## 7️⃣ 自动填充（自动处理 createTime/updateTime）

### ✔ 实体类字段

```java
@TableField(fill = FieldFill.INSERT)
private LocalDateTime createTime;

@TableField(fill = FieldFill.INSERT_UPDATE)
private LocalDateTime updateTime;
```

### ✔ 自定义填充处理器

```java
@Component
public class MyMetaObjectHandler implements MetaObjectHandler {

    @Override
    public void insertFill(MetaObject metaObject) {
        strictInsertFill(metaObject, "createTime", LocalDateTime.class, LocalDateTime.now());
        strictInsertFill(metaObject, "updateTime", LocalDateTime.class, LocalDateTime.now());
    }

    @Override
    public void updateFill(MetaObject metaObject) {
        strictUpdateFill(metaObject, "updateTime", LocalDateTime.class, LocalDateTime.now());
    }
}
```

------

## 8️⃣ 逻辑删除（软删除）

### ✔ 实体类字段

```java
@TableLogic
private Integer deleted;  // 0 未删，1 已删
```

### ✔ yml 配置

```java
mybatis-plus:
  global-config:
    db-config:
      logic-delete-field: deleted
      logic-delete-value: 1
      logic-not-delete-value: 0
```

### ✔ 使用

```java
userMapper.deleteById(1L);
```

------

## 9️⃣ Service 层写法（推荐）

### ✔ 实现类继承 ServiceImpl

```java
@Service
public class UserServiceImpl extends ServiceImpl<UserMapper, User>
        implements UserService {
}
```

### ✔ 使用

```java
@Autowired
private UserService userService;

userService.getById(1L);
userService.list();
userService.save(user);
userService.updateById(user);
userService.removeById(1L);
```

------

# 🎯 一页总结：你必须掌握的 8 大功能

| 功能点        | 必备 API                                      |
| ------------- | --------------------------------------------- |
| CRUD          | selectById / insert / updateById / deleteById |
| 分页          | selectPage                                    |
| QueryWrapper  | eq / like / ge / between                      |
| LambdaWrapper | eq(User::getName…)                            |
| 批量操作      | deleteBatchIds / selectBatchIds               |
| 自动填充      | MetaObjectHandler                             |
| 逻辑删除      | @TableLogic                                   |
| Service API   | ServiceImpl 封装                              |