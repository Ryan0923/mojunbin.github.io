spring源码分析
====

bean初始化
----


#1

    java
public class Main {
public static void main(String[] args)throws Exception{
Context ctx = new InitialContext();
Context context = (Context) ctx.lookup("java:comp/env");
context.lookup("props");

}
    
