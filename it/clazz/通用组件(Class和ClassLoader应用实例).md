# 通用组件(Class和ClassLoader应用实例)

需求：根据包名，获取包下所有Class对象。

思路：
1. 找到包对应的类目录(如 code.entity ===> 项目路径/target/classes/code/entity)。
2. 获取类目录所有class的全限定类名(如 code.entity.User)。
3. 依次加载所有类，得到Class实例。

## 获取类加载器

要获取类目录，一般需要先获得ClassLoader实例。
根据Java类加载器结构，可以自底向上获取ClassLoader。

```java
/**
 * 参见org.springframework.util.ClassUtils
 * 自底向上获取类加载器
 * <p/>
 * Return the default ClassLoader to use: typically the thread context
 * ClassLoader, if available; the ClassLoader that loaded the ClassUtils
 * class will be used as fallback.
 * <p/>
 * Call this method if you intend to use the thread context ClassLoader
 * in a scenario where you clearly prefer a non-null ClassLoader reference:
 * for example, for class path resource loading (but not necessarily for
 * {@code Class.forName}, which accepts a {@code null} ClassLoader
 * reference as well).
 *
 * @return the default ClassLoader (only {@code null} if even the system
 * ClassLoader isn't accessible)
 * @see Thread#getContextClassLoader()
 * @see ClassLoader#getSystemClassLoader()
 */
public static ClassLoader getDefaultClassLoader() {

    ClassLoader cl = null;
    try {
        cl = Thread.currentThread().getContextClassLoader();
    } catch (Throwable ex) {
        // Cannot access thread context ClassLoader - falling back...
    }
    if (cl == null) {
        // No thread context class loader -> use class loader of this class.
        cl = ClassUtils.class.getClassLoader();
        if (cl == null) {
            // getClassLoader() returning null indicates the bootstrap ClassLoader
            try {
                cl = ClassLoader.getSystemClassLoader();
            } catch (Throwable ex) {
                // Cannot access system ClassLoader - oh well, maybe the caller can live with null...
            }
        }
    }
    return cl;
}

```

## 获取包对应的类目录


拿到ClassLoader实例后，我们需要获得包对应的类目录：如 code.entity ===> 项目路径/target/classes/code/entity。

ClassLoader#getResource方法拿到URL实例，再通过URL#getPath()可以得到包对应的类目录。
**注意**：这种方式获得的路径是平台无关的，可兼容linux、windows。


```java
/**
 * 获取包对应的类目录
 * 如 code.entity ===> 项目路径/target/classes/code/entity
 *
 * @param packagePath 包名
 * @return 类目录
 */
public static String getClassPath(String packagePath) {

    if (packagePath == null) {
        throw new NullPointerException();
    }

    URL url = getDefaultClassLoader().getResource(packagePath.replace('.', '/'));
    if (url == null) {
        throw new NullPointerException();
    }

    return url.getPath();
}

```

## 获取类目录所有Class的全限定类名

拿到包对应的类目录后，可遍历目录下所有扩展名为**.class**的文件，进而获得每个类的全限定类名。

```java

private static final String CLASS_SUFFIX = "class";
private static final String CLASS_SEPARATOR = ".";
private static final FileFilter fileFilter = new FileFilter() {
    @Override
    public boolean accept(File pathname) {

        String name = pathname.getName();
        int extIndex = name.lastIndexOf(CLASS_SEPARATOR);
        String ext = name.substring(extIndex + 1, name.length());
        return CLASS_SUFFIX.equals(ext) || pathname.isDirectory();
    }
};

/**
 * 获取目录下所有File
 *
 * @param path
 * @return
 */
public static Collection<File> listFiles(String path) {

    File directory = new File(path);
    Collection<File> list = new LinkedList<File>();
    innerListFiles(list, directory);

    return list;
}

/**
 * 递归获取目录下所有File
 *
 * @param list      存放File的集合
 * @param directory 目录
 */
private static void innerListFiles(Collection<File> list, File directory) {

    File[] files = directory.listFiles(fileFilter);

    if (files == null) { // 空目录
        return;
    }

    for (File file : files) {
        if (file.isDirectory()) {
            innerListFiles(list, file);
        } else {
            list.add(file);
        }
    }
}

```


## 获取Class实例

获得一个类的Class对象，下面四种方式：
1. 类.class
2. 类实例.getClass()
3. ClassLoader.loadClass(String name)
4. Class.forName(String className)

能够**动态**获取Class实例，只有3、4两种方式，一般优先使用方式3。

ClassLoader.loadClass(String name)和Class.forName(String className)的区别，简单说**默认**Class.forName会初始化类，而ClassLoader.loadClass不会初始化类。

```java

    /**
     * 根据类名和类加载器获取Class实例
     *
     * @param className   全限定类名
     * @param classLoader 类加载器
     * @return Class实例
     */
    public static Class<?> resolveClassName(String className, ClassLoader classLoader) {

        try {
            return classLoader != null ? classLoader.loadClass(className) : Class.forName(className);
        } catch (ClassNotFoundException e) {
            throw new IllegalArgumentException(e);
        }
    }

    /**
     * 根据包名，获取包下所有Class对象
     *
     * @param packagePath
     * @return
     */
    public static Set<Class<?>> getClasses(String packagePath) {

        if (packagePath == null) {
            throw new NullPointerException();
        }

        Set<Class<?>> classes = new LinkedHashSet<Class<?>>();

        String packageDirectory = getClassPath(packagePath);
//        System.out.println("packageDirectory = " + packageDirectory);

        //org.apache.commons.io.FileUtils
        Collection<File> files = listFiles(packageDirectory);

        for (File file : files) {
            String className = packagePath + CLASS_SEPARATOR + file.getName().substring(0, file.getName().indexOf(CLASS_SEPARATOR + CLASS_SUFFIX));
//            System.out.println("className = " + className);
            classes.add(resolveClassName(className, getDefaultClassLoader()));
        }


        return classes;
    }

```

所有代码封装在ClassUtils中，嫌麻烦的同学可以联系。


针对Class的开源库有许多，有兴趣可以参见：
- org.apache.commons.lang3.ClassUtils
- org.springframework.util.ClassUtils
- com.google.common.reflect.ClassPath
