# make 命令

在默认的方式下，在当前目录提示符下输入 make 命令，将自动完成以下操作：

- 1.依次读取变量 MAKEFILES 定义的 makefile 文件列表
- 2.读取工作目录下的 makefile文件，根据命名的查找顺序 GNUmakefile、makefile、Makefile，读取首先找到的文件
- 3.依次读取工作目录 makefile 文件中使用指示符 include 包含的文件
- 4.查找重建所有已读取的 makefile 文件的规则（如果存在一个目标是当前读取的某一个makefile 文件，则执行此规则重建此 makefile 文件，完成以后从第一步开始重新执行）
- 5.初始化变量值并展开那些需要立即展开的变量和函数并根据预设条件确定执行分支
- 6.根据 终极目标 以及其他目标的依赖关系建立依赖关系链表
- 7.执行除 终极目标 以外的所有的目标的规则（规则中如果依赖文件中任一个 文件的时间戳比目标文件新，则使用规则所定义的命令重建目标文件）
- 8.执行 终极目标 所在的规则

## 制作一个Makefile




