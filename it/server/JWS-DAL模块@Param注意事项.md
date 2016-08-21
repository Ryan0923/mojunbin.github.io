# 背景

@Param经常作为虚拟属性，配合DAL中的缓存模块使用。

使用缓存模块时，经常会在dal/caches.xml编写取数据的SQL逻辑遇到它，因为要获取满足业务条件
的数据，经常需要跨多个DDL，又或者需要在DDL添加一些虚拟属性。

但JWS的DAL缓存模块，是面向ORM实现的，一个缓存结构只能有一个DDL作为条件，可以通过@Param弥补这种限制。

这里具体对@Param用法不细讲，主要讨论**@Param和IN**操作结合时的一些注意事项。

# 数据信息

为了测试验证，我造了一些数据信息。

目的：希望查找level=1 or level=2 的User。即：select * from user where level in(1,2)，从测试数据上看，会有3条记录。

PS：为什么不直接【level=1 or level=2 】？，有可能再出现level=3,4...的需求，打算把条件做成IN，于是就掉坑了。。。

## DDL User

```
@Table(name = "user")
public class User {
    @Id
    @GeneratedValue(generationType = GenerationType.Auto)
    @Column(name = "id", type = DbType.BigInt)
    private Long id;
    @Column(name = "name", type = DbType.Varchar)
    private String name;
    @Column(name = "level", type = DbType.TinyInt)
    private Integer level;
    @Param
    private String levels;
}
```

## User测试数据

ID|NAME|LEVEL
---|---|---
1|A|1
2|B|2
3|C|1

## @参数节点位置

@参数在sql节点:
```
<cache name="user_l_levels_sql_" class="ddl.User" prefix="user_l_levels_sql_" enabled ="true"  comment ="@参数在sql节点">
    <sql>select * from user u WHERE u.level in( @levels ) ORDER BY level</sql>
    <key>@levels</key>
    <params></params>
    <cache-item>id,name,level</cache-item>
    <offset>0</offset>
    <count>1</count>
</cache>
```

@参数在params节点:

```
<cache name="user_l_levels_params_" class="ddl.User" prefix="user_l_levels_params_" enabled ="true"  comment ="@参数在params节点">
    <sql>select * from user u WHERE u.level in( ? ) ORDER BY level</sql>
    <key>@levels</key>
    <params>@levels</params>
    <cache-item>id,name,level</cache-item>
    <offset>0</offset>
    <count>1</count>
</cache>
```

## 客户端调用

```
@Test
public void testLevel() {
    String levels = "1,2";
    //select * from user u WHERE u.level in(1,2) ORDER BY level;
    List<User> list1 = UserDao.listUserByLevels(UserDao.USER_L_LEVELS_SQL_, levels);
    //select * from user u WHERE u.level in('1,2') ORDER BY level;
    List<User> list2 = UserDao.listUserByLevels(UserDao.USER_L_LEVELS_PARAMS_, levels);
}
```

如果清晰知道DAL模块对@参数在不同位置如何处理，恭喜你！**点击右上角的"X"，不要浪费时间了**。。


# 分析过程

要验证整个流程，少不了源码分析和跑demo验证。

关键代码：

```
jws.dal.cache.Listcache#paramFieldsHandle
jws.dal.DbBase#paramsPrepare
```

关键流程：

```
JWS如何初始化listcache，看其运行时结构。
JWS的DAL被调用时，如何参数替换。
```

## @参数在sql节点

@参数在sql节点，其listcache运行时结构如下：

![@参数在sql节点](https://github.com/mojunbin/mojunbin.github.io/blob/master/images/%40%E5%8F%82%E6%95%B0%E5%9C%A8sql%E8%8A%82%E7%82%B9.png?raw=true)

此时sql语句：

```
select * from user u WHERE u.level in( @levels ) ORDER BY level
```

上面例子，即把@levels替换成【1,2 】，最终得到3条记录，最终实际执行SQL如下：

```
select * from user where level in(1,2)
```

@参数在sql节点，JWS框架直接通过String.replaceAll把@levels替换成实际值，无需PreparedStatement替换。

## @参数在param节点

@参数在param节点，其listcache运行时结构如下：

![@参数在param节点](https://github.com/mojunbin/mojunbin.github.io/blob/master/images/%40%E5%8F%82%E6%95%B0%E5%9C%A8sql%E8%8A%82%E7%82%B9.png?raw=true)

此时sql语句：

```
select * from user u WHERE u.level in( ? ) ORDER BY level
```

上面例子，即把@levels替换成【'1,2'】，注意这里的''，最终得到2条记录，最终实际执行SQL如下：

```
select * from user where level in('1,2')
```

@参数在param节点，JWS框架会通过PreparedStatement进行参数替换，会把@levels替换成【'1,2'】。
这时不符合我们期望值，按照上面测试表数据，level=1 or level=2应该会有3条记录。

### 从MSQL SQL角度看问题

```
a	select * from user where level in(1,2)
b	select * from user where level in('1,2')
```

对于【level in (参数1,参数2...)】操作，level是数值类型，如果in里头是字符类型，mysql会尝试把参数1转化成数值，当成in条件的值，至于参数2及参数n，则被丢弃。

因此对于b语句 ，实际等效下面语句:

```
select * from user where level in(1)
```

# 结论

1. JWS的DAL模块中，对@param有2个时机，一是使用String.replace替换，二是交给PreparedStatement替换。其中PreparedStatement参数替换需要考虑数据库类型问题。

2. 整个SQL参数替换流程是先走String.replace，再走PreparedStatement。

3. 对于IN这类操作，建议把@param放在sql节点，一般能够满足期望；如果放在param节点，可以试试【name in ('A','B'),name是字符串】这个操作，估计会跪，这里不细讲了，有空可以自己实践。


**如果还有疑问，代表文章还有优化空间，我们面聊吧。。。。**