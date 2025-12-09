# 📄 **MyBatis-Plus 常用动态 SQL 模板.md**

~~~java
# MyBatis-Plus 常用动态 SQL 模板

MyBatis-Plus（MP）通过 `QueryWrapper`、`LambdaQueryWrapper`、`UpdateWrapper` 等封装，
实现了 MyBatis 的动态 SQL（不再需要写 `<if>`、`<choose>`、`<where>` 这些 XML 标签）。

本文件收录最常用的动态 SQL 模板，可直接复制到项目中使用。

---

# 1. 基本查询（eq / ne / gt / lt）

```java
LambdaQueryWrapper<User> qw = new LambdaQueryWrapper<>();
qw.eq(name != null, User::getName, name)
  .ne(status != null, User::getStatus, status)
  .gt(ageMin != null, User::getAge, ageMin)
  .lt(ageMax != null, User::getAge, ageMax);

List<User> list = userMapper.selectList(qw);
~~~

------

# 2. 模糊查询（like / likeLeft / likeRight）

```java
qw.like(keyword != null, User::getName, keyword);
qw.likeLeft(email != null, User::getEmail, email);
qw.likeRight(phone != null, User::getPhone, phone);
```

------

# 3. in / notIn 查询

```java
qw.in(ids != null && !ids.isEmpty(), User::getId, ids);
qw.notIn(blockList != null, User::getId, blockList);
```

------

# 4. between 查询

```java
qw.between(start != null && end != null, User::getCreateTime, start, end);
```

------

# 5. 排序：orderByAsc / orderByDesc

```java
qw.orderByDesc(User::getUpdateTime)
  .orderByAsc(User::getAge);
```

------

# 6. 分页查询（推荐）

```java
Page<User> page = new Page<>(pageNum, pageSize);
Page<User> result = userMapper.selectPage(page, qw);
```

------

# 7. choose 逻辑（用 Java if 代替）

原 MyBatis:

```java
<choose>
    <when test="name != null">AND name = #{name}</when>
    <when test="phone != null">AND phone = #{phone}</when>
    <otherwise>AND status = 1</otherwise>
</choose>
```

MyBatis-Plus:

```java
if (name != null) {
    qw.eq(User::getName, name);
} else if (phone != null) {
    qw.eq(User::getPhone, phone);
} else {
    qw.eq(User::getStatus, 1);
}
```

------

# 8. update 动态 SQL（set 自动去掉多余逗号）

```java
LambdaUpdateWrapper<User> uw = new LambdaUpdateWrapper<>();
uw.set(name != null, User::getName, name)
  .set(age != null, User::getAge, age)
  .set(status != null, User::getStatus, status)
  .eq(User::getId, id);

userMapper.update(null, uw);
```

------

# 9. 逻辑删除（常用）

实体类加字段：

```java
@TableLogic
private Integer isDeleted;
```

查询自动加上 `is_deleted = 0`，无需手写。

------

# 10. select 指定字段

```java
qw.select(User::getId, User::getName, User::getAge);
```

------

# 11. wrapper 嵌套查询（and / or / nested）

```java
qw.and(w -> w.eq(User::getStatus, 1)
             .gt(User::getAge, 18))
  .or(w -> w.like(User::getName, keyword));
```

效果类似：

```java
AND (status = 1 AND age > 18)
OR (name LIKE '%xx%')
```

------

# 12. exists 查询

```java
qw.exists("select 1 from order where user_id = user.id");
```

------

# 13. 自定义 SQL + Wrapper（XML + MP 联动）

XML：

```java
<select id="queryUser" resultType="User">
    SELECT * FROM user ${ew.customSqlSegment}
</select>
```

Java：

```java
LambdaQueryWrapper<User> qw = new LambdaQueryWrapper<>();
qw.eq(User::getStatus, 1)
  .orderByDesc(User::getCreateTime);

List<User> users = userMapper.queryUser(qw);
```

------

# 14. 统计 count / sum / max / min

```java
Integer count = userMapper.selectCount(qw);

qw.select(SqlFunc.max(User::getAge));
User oldest = userMapper.selectOne(qw);
```

------

# 15. 常用查询模板汇总

## （1）根据条件动态查询

```java
public List<User> query(UserQueryDTO dto) {
    LambdaQueryWrapper<User> qw = new LambdaQueryWrapper<>();
    qw.like(dto.getName() != null, User::getName, dto.getName())
      .eq(dto.getStatus() != null, User::getStatus, dto.getStatus())
      .between(dto.getStart() != null && dto.getEnd() != null, 
               User::getCreateTime, dto.getStart(), dto.getEnd());
    return userMapper.selectList(qw);
}
```

## （2）更新

```java
public void update(User user) {
    LambdaUpdateWrapper<User> uw = new LambdaUpdateWrapper<>();
    uw.eq(User::getId, user.getId())
      .set(user.getName() != null, User::getName, user.getName())
      .set(user.getAge() != null, User::getAge, user.getAge());
    userMapper.update(null, uw);
}
```

## （3）分页查询

```java
Page<User> page = new Page<>(pageNum, pageSize);
return userMapper.selectPage(page, qw);
```

------

# ✔️ 总结（最重要的一句话）

**在 MyBatis-Plus 中：所有动态 SQL 都通过 Wrapper 条件判断来实现，不再需要 XML 里的 `<if>` / `<choose>` / `<where>` 等标签。**

```java
eq(condition, field, value)
set(condition, field, value)
like(condition, field, value)
in(condition, field, value)
```

condition 为 true 就拼 SQL，为 false 就跳过。