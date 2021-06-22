
# Class File

无符号数是一种基本数据类型 u1，u2，u4，u8分 别代表1个字节，2个字节，4个字节，8个字节长度的无符号数（1字节等于8位）。

```
ClassFile {
    u4             magic;
    u2             minor_version;
    u2             major_version;
    u2             constant_pool_count;
    cp_info        constant_pool[constant_pool_count-1];
    u2             access_flags;
    u2             this_class;
    u2             super_class;
    u2             interfaces_count;
    u2             interfaces[interfaces_count];
    u2             fields_count;
    field_info     fields[fields_count];
    u2             methods_count;
    method_info    methods[methods_count];
    u2             attributes_count;
    attribute_info attributes[attributes_count];
}
```

# asm 工具

在asm的源码中看到Opcodes类文件中，有这么一段，定义了java class文件的版本号：
```java
    int V1_1 = 3 << 16 | 45;
    int V1_2 = 0 << 16 | 46;
    int V1_3 = 0 << 16 | 47;
    int V1_4 = 0 << 16 | 48;
    int V1_5 = 0 << 16 | 49;
    int V1_6 = 0 << 16 | 50;
    int V1_7 = 0 << 16 | 51;
```

Java 中的位运算操作符包含：
```
&(and),~(not),^(xor),|(or),>>(右移),>>>(无符号右移),<<(左移)
```

上面代码中主要用到了 **<<左移操作符** 和 **|或** 操作符。左移操作符功能把 << 左边的运算数的各二进位全部左移若干位，由`<<`右边的数指定移动的位数， 高位丢弃低位补0。如 3 << 4，3对应的二进制表达式为：00000011，那么左移4位为：00110000(十进制为48)。而`|`操作符是将两边的数值进行按位或值运算，任何数与0进行或运算的结果为该数本身。

上面的代码中的 V1_2至V1_7，0 << 16 结果为0，与右侧数运算的结果为右侧数，如 V1_2 = 46，V1_3=47 以此类推。在 jdk1.1 中版本号是45.3，其中45是主版本号，3是次版本号，于是 V1_1 先是 3<<16，即十进制的 196608，再与 45 相或后是 196653。

使用 javac -target 1.1 Example.java 编译生成 class 文件，借助 sublime 打开 Example.class 可看到 0003002D，0x0003002d = 3`*`16<sup>4</sup>+2`*`16+13=196635。


# Reference

1.[Java Language and Virtual Machine Specifications](https://docs.oracle.com/javase/specs/index.html)

2.[The Java® Virtual Machine Specification Java SE 9 Edition](https://docs.oracle.com/javase/specs/jvms/se9/jvms9.pdf)

3.[https://www.jianshu.com/p/93318f387d04](https://www.jianshu.com/p/93318f387d04)