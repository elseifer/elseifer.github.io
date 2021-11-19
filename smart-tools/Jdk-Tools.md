# Jdk tools
以 JDK 8 为例，以下所有的引用都来自文献 1、2

## jar

1.查看jar内容

jar tvf myproject-0.0.1-SNAPSHOT.jar

2.解压jar到指定目录

jar -xvf demo-0.0.1.jar -C demo-0.0.1

## java

>The  java  tool  launches  a  Java application.  It does this by starting a Java runtime environment,
loading a specified class, and invoking that class's main method.  The method must be declared public
and  static  ,  it  must  not return any value, and it must accept a String array as a parameter. The
method declaration must look like the following:
    `public static void main(String args[])`  
By default, the first non-option argument is the name of the class to be invoked.  A  fully-qualified
class  name  should  be  used.  If the -jar option is specified, the first non-option argument is the
name of a JAR archive containing class and resource files for the application, with the startup class
indicated by the Main-Class manifest header.

java 命令用于发布 Java 应用，它会启动一个 JVM 实例进程，Java 应用在 JVM 中以主线程的形式运行，主线程结束则 JVM 进程终止，而对于 javaa gent 代理，它和宿主应用在同一个 JVM 实例上。

使用方法：
```
java -jar Test.jar
java -cp com.test.Test Test.jar
```

`java -jar` 执行过程：

- 启动 JVM 实例，分配内存（内存结构、垃圾回收）
- 找到 MANIFEST.MF 中 Main-Class 指定的入口类，将 Main-class 加载到方法区内，Jvm 从主函数入口启动程序
- 根据引用关系加载类（类加载、类加载器、双亲委托机制)
- 执行程序，在虚拟机栈创建方法栈桢，局部变量等信息

`java -cp` 
 
cp 其实就是 classpath，在 linux 中多个jar包用 `:` 分割，代表了程序运行必须的 jar 包。

使用 -cp 不用将依赖放到 Test.jar 中。当修改 Test 类后只上传修改后的 Test.jar 到服务器即可，不需要再将所有依赖放到 Test.jar 中再上传一遍，节约了时间。


## jps



## -verbose:class


# jmap
dump 指定 pid 的 java 进程，存放在 heap.bin 文件
`jmap -dump:live,format=b,file=heap.bin [pid]`

分析 heap.bin 文件，启动一个 web server，访问 http://ip:7000 即可查阅
`jhat -J-mx800m heap.bin`

# REF

1.[https://docs.oracle.com/javase/8/docs/technotes/tools/windows/java.html](https://docs.oracle.com/javase/8/docs/technotes/tools/windows/java.html)

2.[https://docs.oracle.com/en/java/javase/16/docs/specs/man/index.html](https://docs.oracle.com/en/java/javase/16/docs/specs/man/index.html)
