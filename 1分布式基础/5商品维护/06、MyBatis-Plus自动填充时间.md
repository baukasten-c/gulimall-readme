# MyBatis-Plus自动填充时间

1、在实体类的时间变量上添加注解填充字段

```java
@TableField(fill = FieldFill.INSERT)
private Date createTime;
@TableField(fill = FieldFill.INSERT_UPDATE)
private Date updateTime;
```

2、在 config 包下添加 MyMetaObjectHandler 类

```java
@Component
public class MyMetaObjectHandler implements MetaObjectHandler {
    @Override
    public void insertFill(MetaObject metaObject) {
        this.strictInsertFill(metaObject, "createTime", () -> LocalDateTime.now(), LocalDateTime.class);
        this.strictUpdateFill(metaObject, "updateTime", () -> LocalDateTime.now(), LocalDateTime.class);
    }

    @Override
    public void updateFill(MetaObject metaObject) {
        this.strictUpdateFill(metaObject, "updateTime", () -> LocalDateTime.now(), LocalDateTime.class);
    }
}
```

3、重启进行测试，发现新插入的数据并没有自动填充时间

4、发现是类型不匹配，数据库中使用 datetime，MyMetaObjectHandler 中也使用的 LocalDateTime，而实体类中使用的 Date，修改

```java
@TableField(fill = FieldFill.INSERT)
private LocalDateTime createTime;
@TableField(fill = FieldFill.INSERT_UPDATE)
private LocalDateTime updateTime;
```

5、再次重启，可以自动填充时间