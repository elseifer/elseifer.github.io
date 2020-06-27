# Java 应用的兼容性问题

## 什么是兼容性问题

兼容性可以从两个方向上考虑<sup>[4]</sup>：
1. 向后兼容 backward compatibility：新版本软件可以运行在低版本的环境中，也称为向下兼容 downward compatibility；
2. 向前兼容 forward compatibility：低版本软件可以运行在高版本的环境中，也称为向上兼容 upward compatibility。

通常，Java 应用兼容性指的是向后兼容，分类主要有三种：源码（source）、二进制（binary）和运行时行为（behavioral）<sup>[5]</sup><sup>[6]</sup>，同时 Java 平台提供的APIs、Tools（如 javac）的变更也涉及兼容性<sup>[6]</sup><sup>[8]</sup>。

1. 源码兼容性（source compatibility）：源码兼容性考虑的是 Java 源码转为为 class 文件。
2. 二进制兼容性（binary compatibility）：在 Java 规范中，二进制兼容性被定义保持无链接错误的能力，有时也被称为非源码兼容性。通常，某一版本 JDK 编译出来的 class 文件可以被该版本以及更高版本 JDK 正确地加载和链接，而未必能被低版本 JDK 正确地加载和链接。
3. 行为兼容性（behavioral compatibility）：行为兼容性是指在运行时代码执行表达出的语义应该是相同或者等价的。
4. APIs 兼容性：Java 平台中提供的 API 的所有变化，其中 API 的移除或者签名变化会导致 Java 程序在新版 JDK 中无法编译或运行时错误。 
5. Tools 兼容性：Java 平台提供的工具，例如编译工具 javac ，这些工具在 Java 发展中参数可能会发生变化。

这里我们把 APIs、Tools 兼容性约定为 JDK 兼容性（或者 Java 兼容性），JDK 的不兼容对 Java 应用的源码、二进制和运行时的兼容产生重要影响。

## 程序中是如何引入了 JDK 兼容性问题呢

在实际代码开发工作中，我们无法保证程序开发时使用的 JDK 版本和程序运行时的 JDK 版本相一致，例如程序内使用的某一 Java API 在不同版本 JDK 中有签名变化则会导致程序运行时抛出 UnsupportedClassVersionError 等错误，这为应用引入 JDK 兼容性问题提供了现实基础。

JDK 有一套通用的针对外部 APIs 的兼容性策略：不可破坏二进制兼容性、不可引入源码的不兼容、管理行为兼容性的变化<sup>[9]</sup>，我们可以通过 oracle 官方文档获取兼容性变化的内容，如 *Compatibility Guide for JDK 8*<sup>[5]</sup>。

*关于代码开发、编译、运行时的 JDK 版本相关部分参考 [Class 文件格式版本](./ClassFile-Version.md)。*

## JDK 兼容性在 Java 应用中的若干案例 

### 源码兼容性





### 二进制兼容性


### ConcurrentHashMap#keySet

在 jdk8 中 keySet 方法的返回类型为 KeySetView，而在 jdk6、7 等低版本中为 Set 



### 运行时行为兼容性







## 如何规避引入 JDK 兼容性问题

Merely setting the `target` option does not guarantee that your code actually runs on a JRE with the specified version. The pitfall is unintended usage of APIs that only exist in later JREs which would make your code fail at runtime with a linkage error. To avoid this issue, you can either configure the compiler's boot classpath to match the target JRE or use the Animal Sniffer Maven Plugin to verify your code doesn't use unintended APIs. In the same way, setting the `source` option does not guarantee that your code actually compiles on a JDK with the specified version. To compile your code with a specific JDK version, different than the one used to launch Maven, refer to the Compile Using A Different JDK example.<sup>[1]</sup>

我们可以通过设置 maven 编译时 JDK、使用 Animal Sniffer 插件等方式来一定程度上规避 JDK 兼容性问题（源码和二进制兼容性）。

### Animal Sniffer Maven Plugin

The Animal Sniffer Plugin is used to build signatures of APIs and to check your classes against previously generated signatures. This plugin is called animal sniffer because the principal signatures that are used are those of the Java Runtime, and since Sun traditionally names the different versions of its Java Runtimes after different animals, the plugin that detects what Java Runtime your code requires was called "Animal Sniffer"<sup>[2]</sup>.


#### 


### Compiling Sources Using A Different JDK

适用于 maven-compiler-plugin 3.6.0 版本并且 Maven 3.3.1+ 版本<sup>[3]</sup>

#### 


### 真的避免应用引入 JDK 兼容性问题了么

答案是并不完全解决，我们通过上述方式虽然极大程度地避免引入 JDK 兼容性问题，但是只要有高版本 JDK 将 .java 文件编译成低版本格式的 .class 文件，或者高版本格式的 .class 文件运行在低版本 JDK 中的场景，那么 JDK 兼容性问题（主要是影响 Java 应用的二进制和行为兼容性，源码兼容性很容易在编译期内解决）就可能被引入。也因此，保持代码编译、打包和运行时的 JDK 版本一致才是最好的解决方式<sup>[8]</sup>。


## 如何探测应用引入的 jar 潜在 JDK 兼容问题呢


不论是通过设置编译时 JDK 还是使用 maven 插件，都默认了一个前提，即我们拥有源代码并可以修改代码的构建流程，如果我们的应用在一次变更中引入了新的三方依赖（其他组织发布的 jar），如何知道它们在运行时的兼容性呢？下面我们就开始讨论下检测任意三方 jar 的 JDK 兼容性。




# Reference

1.[Setting the -source and -target of the Java Compiler](http://maven.apache.org/plugins/maven-compiler-plugin/examples/set-compiler-source-and-target.html)

2.[animal-sniffer-maven-plugin](http://www.mojohaus.org/animal-sniffer/animal-sniffer-maven-plugin/)

3.[Compiling Sources Using A Different JDK](http://maven.apache.org/plugins/maven-compiler-plugin/examples/compile-using-different-jdk.html)

4.[BINARY COMPATIBILITY FOR LIBRARY AUTHORS](https://docs.scala-lang.org/overviews/core/binary-compatibility-for-library-authors.html)

5.[Compatibility Guide for JDK 8](https://www.oracle.com/java/technologies/javase/8-compatibility-guide.html)

6.[Kinds of Compatibility](https://wiki.openjdk.java.net/display/csr/Kinds+of+Compatibility)

7.[Determining the Java version used to compile a class](https://fabianlee.org/2018/01/19/java-determining-the-java-version-used-to-compile-a-class-class-file-has-the-wrong-version/)

8.[Java compatibility, Apache Maven and Jenkins Best Practices](https://support.cloudbees.com/hc/en-us/articles/360020783632-Java-compatibility-Apache-Maven-and-Jenkins-Best-Practices?mobile_site=true)

9.[Compatibility & Specification Review](https://wiki.openjdk.java.net/display/csr)

10.[Java API Compliance Checker](https://lvc.github.io/japi-compliance-checker/)

11.[tool to compare compatibility of Java library API](https://launchpad.net/ubuntu/trusty/+package/japi-compliance-checker)

12.[SignatureTests](http://wiki.apidesign.org/wiki/SignatureTests)