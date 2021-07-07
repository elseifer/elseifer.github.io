# Jdk tools


## jar


### 查看jar内容

jar tvf myproject-0.0.1-SNAPSHOT.jar

### 解压jar到指定目录
jar -xvf demo-0.0.1.jar -C demo-0.0.1


## java

每个 Java 程序运行在独立的 JVM 实例上。

>The  java  tool  launches  a  Java application.  It does this by starting a Java runtime environment,
loading a specified class, and invoking that class's main method.  The method must be declared public
and  static  ,  it  must  not return any value, and it must accept a String array as a parameter. The
method declaration must look like the following:
    `public static void main(String args[])`  
By default, the first non-option argument is the name of the class to be invoked.  A  fully-qualified
class  name  should  be  used.  If the -jar option is specified, the first non-option argument is the
name of a JAR archive containing class and resource files for the application, with the startup class
indicated by the Main-Class manifest header.

使用方法：
```
java -jar Test.jar
java -cp com.test.Test Test.jar
```

java -jar 执行过程：

- 启动 JVM，分配内存（Java 内存结构和 GC 知识）
- 找到 MANIFEST.MF 中 Main-Class 指定的入口类，将 Main-class 加载到方法区内，Jvm 从主函数入口启动程序
- 根据引用关系加载类（类加载、类加载器、双亲委托机制)
- 执行程序，在虚拟机栈创建方法栈桢，局部变量等信息

java -cp
 
cp 其实就是 classpath，在 linux 中多个jar包用 `:` 分割，代表了程序运行必须的 jar 包。  
使用 -cp 不用将依赖放到 Test.jar 中。当修改 Test 类后只上传修改后的 Test.jar 到服务器即可，不需要再将所有依赖放到 Test.jar 中再上传一遍，节约了时间。


# jps




