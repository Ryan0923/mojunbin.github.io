spring源码分析
====

bean初始化
----


#格式化java代码

```java
public class Main {
    public static void main(String[] args)throws Exception{
        Context ctx = new InitialContext();
        Context context = (Context) ctx.lookup("java:comp/env");
    }
```
