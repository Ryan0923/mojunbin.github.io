# 通用组件(Class和ClassLoader应用实例)

需求：根据包名，获取包下所有Class对象。

思路：
1. 找到包对应的类目录(如 code.entity ===> 项目路径/target/classes/code/entity)。
2. 获取类目录所有class的全限定类名(如 code.entity.User)。
3. 依次加载所有类，得到Class实例。

## 获取包对应的类目录


## 获取类目录所有Class的全限定类名


## 获取Class实例

