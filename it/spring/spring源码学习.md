spring源码分析
====

bean初始化
----




```java
public interface InputStreamSource {

	InputStream getInputStream() throws IOException;

}
```
InputStreamSource封装任何能够返回**InputStream**的类。

org.springframework.util.ClassUtils#getDefaultClassLoader可以获取默认类加载器，可以
用来实现从类中读取文件。


```java
	/**
	 * Create a new AbstractAutowireCapableBeanFactory.
	 */
	public AbstractAutowireCapableBeanFactory() {
		super();
		ignoreDependencyInterface(BeanNameAware.class);
		ignoreDependencyInterface(BeanFactoryAware.class);
		ignoreDependencyInterface(BeanClassLoaderAware.class);
	}
```
ignoreDependencyInterface忽略给定接口的自动装配功能。


##spring-LoadBeanDefinitions 函数执行时序图


InputSource来自JDK，而不是spring

##默认标签的解析
