# Java 应用程序中的 Java 兼容性问题及解决方案

本文主要内容包括 Java 兼容性的定义与分类、常见兼容性案例、Java 兼容性的规避和解决方法，同时在开源项目基础上定制开发一个用于探测任意三方 jar 和 class 文件 Java 兼容性的检验工具。本文不讨论应用 APIs 变化对上下游应用产生的兼容性，而是关注应用运行在不同版本 JVM 时可能引入的 Java 不兼容性问题。

## 一、什么是 Java 的兼容性

兼容性可以从两个方向上考虑<sup>[4]</sup>：
+ 向后兼容（backward compatibility）：新版本软件可以运行在低版本的环境中，也称为向下兼容（downward compatibility）
+ 向前兼容（forward compatibility）：低版本软件可以运行在高版本的环境中，也称为向上兼容（upward compatibility）

通常，Java 兼容性多指向后兼容，主要包括三种：源码（source）、二进制（binary）和运行时行为（behavioral），同时 Java 提供的 APIs、Tools（如 javac）也涉及兼容性<sup>[5]</sup><sup>[6]</sup><sup>[8]</sup>：

1. 源码兼容性（source compatibility）：源码兼容性考虑的是 Java 源码转为为 .class 文件。
2. 二进制兼容性（binary compatibility）：在 Java 规范中，二进制兼容性被定义保持无链接错误的能力，有时也被称为非源码兼容性。通常，某一版本 JDK 编译出来的 class 文件可以在同版本以及更高版本 JVM 上运行，而不能被低版本 JVM 正确地加载和链接。
3. 行为兼容性（behavioral compatibility）：行为兼容性是指在运行时代码执行表达出的语义应该是相同或者等价的。
4. APIs 兼容性：Java 提供的 API 的所有变化，其中 API 的移除或者签名变化会导致 Java 程序在新版 JDK 中无法编译或运行时错误。 
5. Tools 兼容性：Java 提供的工具，例如编译工具 javac ，这些工具在 Java 发展中参数可能会发生变化。

在 Oracla 官方文档中 Java 提供的 APIs 包含 Java SE APIs 和 JDK APIs 等，Java SE 提供的 APIs 所在模块名以 java 开头，JDK 提供的 APIs 所在的模块名以 jdk 开头<sup>[10]</sup>（Java 9 引入了模块概念，低版本 Java 中可以从包名上区分这些 APIs）。

关于 Java 全部组件可以看 *Description of Java Conceptual Diagram*<sup>[11]</sup>，如下图：

![Java Conceptual Diagram](./images/Java-Conceptual-Diagram.jpg)

### 1.1 应用程序中是如何引入了 Java 兼容性问题

在实际代码开发工作中，我们无法保证程序开发、运行时的 JDK 版本相一致，例如程序开发时使用了高版本 Java API（在低版本中签名有变或不存在）使得编译后的 .class 文件存在二进制兼容性问题，将导致程序在低版本 JVM 中运行时抛出 UnsupportedClassVersionError 等错误。

其次，尽管 JDK 对于 APIs 兼容性策略包括不可破坏二进制兼容性、不可引入源码的不兼容、管理行为兼容性的变化<sup>[9]</sup>，但是低版本中一些废弃或移除的 APIs 会破坏二进制兼容性，因而，如果应用程序中使用了 `@deprecated` 标记的方法，那么应用在后续升级 JDK 时就可能遇到不兼容问题。

Oracle 提供了 Java 兼容性指导手册，如 *[Compatibility Guide for JDK 8](https://www.oracle.com/java/technologies/javase/8-compatibility-guide.html)*，我们可以通过这份文档了解 Java 版本之间的兼容性变化。

*关于代码开发、编译、运行时涉及 JDK 版本的相关内容参考 [Class 文件格式版本](./ClassFile-Version.md)。*

## 二、Java 兼容性影响应用的若干案例 

下面创建一个示例工程 [compatibility-demo](https://github.com/elseifer/compatibility-demo.git) 演示 Java 兼容性，本文内涉及的开发环境均为 JDK 8 和 Maven 3.6.3： 
![Maven 和 JDK 环境](./images/env.jpg)

### 2.1 源码兼容性





### 2.2 二进制兼容性


### ConcurrentHashMap.keySet

在 JDK8 中 keySet 方法的返回类型为 KeySetView，而在低版本 JDK 中为 Set。



### 2.3 运行时行为兼容性




## 三、如何规避代码中引入 Java 兼容性问题

从上面的案例知道，设置 `target` 选项并不能保证代码可以正确地在某一版本的 JRE 上运行，一些较晚出现的 APIs 会在代码运行时产生连接错误，为了避免这个问题，我们可以配置 Java 编译器的引导类路径以匹配目标 JRE 或者使用 Animal Sniffer Maven Plugin 插件。同样的，设置 `source` 选项也不能保证代码可以在某一版本的 JDK 上编译通过，为了解决这个问题，我们需要使用与启动 Maven 不同的特定 JDK 版本编译代码<sup>[1]</sup>。

继续以 compatibility-demo<sup>[7]</sup> 为例，如何规避引入 Java 兼容性问题。

### 3.1 Animal Sniffer Maven Plugin

The Animal Sniffer Plugin 可以用于构建 APIs 签名以及通过对照 APIs 签名对 class 进行检查，其中使用到的最重要的 APIs 签名就是各版本的 Java APIs 的签名。

#### 3.1.1 如何配置

为了对照 APIs 签名检查我们的应用，必须在 `pom.xml` 中配置需要参考的签名<sup>[2]</sup>，例如：
配置 java1.6 的签名作为参考，在 mvn 编译代码后手动执行 `animal-sniffer:check` 可以进行签名检查，

```
<plugin>
    <groupId>org.codehaus.mojo</groupId>
    <artifactId>animal-sniffer-maven-plugin</artifactId>
    <version>1.16</version>
    <!-- 配置 jdk1.6 的签名 -->
    <configuration>
        <signature>
            <groupId>org.codehaus.mojo.signature</groupId>
            <artifactId>java16</artifactId>
            <version>1.0</version>
        </signature>
    </configuration>
</plugin>
```

或者参考一下方式把检查工作加入到 mvn 构建过程中：

```
<plugin>
    <groupId>org.codehaus.mojo</groupId>
    <artifactId>animal-sniffer-maven-plugin</artifactId>
    <version>1.16</version>
    <!-- jdk1.6 的签名 -->
    <configuration>
        <signature>
            <groupId>org.codehaus.mojo.signature</groupId>
            <artifactId>java16</artifactId>
            <version>1.0</version>
        </signature>
    </configuration>

    <!-- compile 阶段自动执行 mvn animal-sniffer:check -->
    <executions>
        <execution>
            <id>animal-sniffer-check</id>
            <phase>compile</phase>
            <goals>
                <goal>check</goal>
            </goals>
        </execution>
    </executions>
</plugin>
```

#### 3.1.2 使用效果


### 3.2 Compiling Sources Using A Different JDK

对于 Maven Compiler Plugin 3.6.0 版本并且 Maven 3.3.1+ 版本<sup>[3]</sup>

#### 3.2.1 如何配置


#### 3.2.2 使用效果


### 3.3 小结

通过上述方式基本可以的避免引入 Java 兼容性问题，但我们日常的研发环境中代码编译过程大部分都在统一的构建平台进行，一些外部环境是 maven 插件不能控制的，构建平台升级 JDK 也可能把 Java 兼容性问题引入到应用中。时刻保持代码编译、打包和运行时的 JDK 版本一致是解决应用引入 Java 兼容性问题的最好方式<sup>[8]</sup>。

## 四、如何探测应用引入的 jar 潜在 Java 兼容问题呢

不论是设置编译时 JDK 还是使用 Sniffer 插件，都默认了一个前提，即我们拥有源代码并可以修改代码的构建配置与流程，但如果我们在应用中更新或者引入一个三方依赖（其他组织发布的 jar），最经典的案例莫过于，基于 Spring 开发的应用避免不了升级 Spring 版本或者相关依赖，我们有必要清楚这些 jar 是否潜在 Java 兼容性问题。

### 4.1 改造 animal-sniffer

通过阅读 animal-sniffer-maven-plugin 的源码，我发现它是基于 ASM 对字节码进行检查，加之有过 ASM 开发工具的经验，于是我设想把参考的 APIs 签名默认限制为 Java APIs 签名，调整检测目标为任意三方 jar 和class 目录来对它进行改造。在[我的 github](https://github.com/elseifer) 上可以获取 [animal-sniffer-jar-with-dependencies](https://github.com/elseifer/animal-sniffer/blob/enhance-signature-checker/animal-sniffer/src/main/java/org/codehaus/mojo/animal_sniffer/enhanced/EnhancedSigChecker.java) 的源代码，作为 animal-sniffer 变种，它完整包含所有依赖、可脱离 Maven 环境独立运行，可检查第三方 jar 而无需源代码，同时修复了一些 NPE 问题。

继续以 compatibility-demo 为例，从 pom.xml 将 Animal Sniffer Maven Plugin 和 Maven Compiler Plugin 中移除，参考 [ConcurrentHashMap#keySet 的二进制不兼容案例](#ConcurrentHashMap.keySet)。 执行 `mvn clean package -Dmaven.test.skip` 编译

```
java -jar animal-sniffer-jar-with-dependencies.jar /Users/qingqin/git/compatibility-demo/target/classes -v 6
```

示例运行结果：</br>
![jar checker](./images/jar-checker.jpg)

### 4.2 实践

`-i` 用于忽略一些包路径，一般可以把自身以及无需关注的依赖包忽略来减少干扰；
`-v` 用于设置期望的 Java 版本，并使用该版本的 Java APIs 签名作为参考；

通过对 

animal-sniffer-jar-with-dependencies 可以检查第三方依赖的二进制不兼容，但不适用于源码兼容、运行时行为兼容的检测。

## 五、总结

本文讨论了应用中的 Java 兼容性，涉及 Java 兼容性分类和案例、并对应给出解决方案，同时实验性质的开发 [animal-sniffer-jar-with-dependencies](https://github.com/elseifer/animal-sniffer/tree/enhance-signature-checker) 工具，用于检查 jar 中存在的二进制不兼容。

1. 如果应用发布的 jar 会被其他应用在低版本 JVM 上运行，需要测试与研发环境的 JDK 和最低运行 JVM 版本相一致，或者参考本文第三部分的内容，保证代码构建无兼容性问题；
2. 如果应用引入其他外部依赖，在无法确定该 jar 是否可以在应用基线的 JVM 版本上运行时，则可以参考 `animal-sniffer-jar-with-dependencies` 工具对 jar 进行兼容性探测；

Java 兼容性是宽泛的话题，不可能依赖单一手段解决或者规避，更需要研发规范、评审机制和测试手段协同。

# Reference

1.[Setting the -source and -target of the Java Compiler](http://maven.apache.org/plugins/maven-compiler-plugin/examples/set-compiler-source-and-target.html)

2.[animal-sniffer-maven-plugin](http://www.mojohaus.org/animal-sniffer/animal-sniffer-maven-plugin/usage.html)

3.[Compiling Sources Using A Different JDK](http://maven.apache.org/plugins/maven-compiler-plugin/examples/compile-using-different-jdk.html)

4.[BINARY COMPATIBILITY FOR LIBRARY AUTHORS](https://docs.scala-lang.org/overviews/core/binary-compatibility-for-library-authors.html)

5.[Compatibility Guide for JDK 8](https://www.oracle.com/java/technologies/javase/8-compatibility-guide.html)

6.[Kinds of Compatibility](https://wiki.openjdk.java.net/display/csr/Kinds+of+Compatibility)

7.[Java 兼容性的演示代码 compatibility-demo](https://github.com/elseifer/compatibility-demo.git)

8.[Java compatibility, Apache Maven and Jenkins Best Practices](https://support.cloudbees.com/hc/en-us/articles/360020783632-Java-compatibility-Apache-Maven-and-Jenkins-Best-Practices?mobile_site=true)

9.[Compatibility & Specification Review](https://wiki.openjdk.java.net/display/csr)

10.[Java® Platform, Standard Edition & Java Development Kit Version 9 API Specification](https://docs.oracle.com/javase/9/docs/api/overview-summary.html)

11.[Java Platform Standard Edition 8 Documentation](https://docs.oracle.com/javase/8/docs/)