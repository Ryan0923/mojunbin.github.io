# spring自定义标签

可以划分二种：

- 标准标签添加非标准属性

```xml
    <bean id="bean" name="bean" class="Class" orther="xxx" />
```

- 自定义标签


```xml
    <myns:dateformat id="dateFormat"
                     pattern="yyyy-MM-dd HH:mm"
                     lenient="true"/>
```

