# C与指针

## 增加编译规则

目录结构如下:

```text
-/.
-src					#source code contents
	-helloworld			#helloworld project
		-helloworld.c	#source file
		-helloworld.mk	#inner layer makefile
	-src.mk				#middle layer makefile,use to manage project compiler
-bin					#xxx.bin executable file
	-helloworld.bin
-build					#build rules
-doc					#document
	-img				#image source
-lib					#general purpose library
-obj					#Compile intermediate files
-makefile				#outer most layer makefile
-README.md
```



## 增加第一个工程 `helloworld`

本来想使用`menuconfi`配合`makefile`进行项目管理，但是在`linux`下怎么使用`menuconfig`还没搞懂。所以先简单通过`include`来选择编译那个工程，只需要修改`/src/src.mk`即可。

```text
include ./src/helloworld/helloworld.mk		#取消屏蔽来选择工程
```



## 写在前面

这个项目主要是用来记录，我在学习`《C与指针》`这本书的过程中，所积累的c代码，以便后面查找使用。整个项目会随着我的学习，不断往前扩展，所以进度基本是跟随着书本的阅读进度，当然后面也可能会有些新的想法，感悟也会加进来，比如一些工具的使用啊，像`menuconfig`管理项目，`makefile`编写之类的。这里简单列下目前能想到的一些要做的事。当然排列是不分先后的，所以以列表的形式给出，后面每做完一项标记下。

---

-   [x] [文档撰写之`Markdown`][Markdown]
-   [ ] [自动化编译之`makefile`][makefile_reference]
-   [ ] 脚步语言之`bash shell`
-   [ ] 项目管理之`menuconfig`
-   [ ] 代码管理之`git`
-   [ ] 章节1-快速上手
-   [ ] 章节2-基本概念
-   [ ] 章节3-数据
-   [ ] 章节4-语句
-   [ ] 章节5-操作符和表达式
-   [ ] 章节6-指针
-   [ ] 章节7-函数
-   [ ] 章节8-数组
-   [ ] 章节9-字符串，字符和字节
-   [ ] 章节10-结构和联合
-   [ ] 章节11-动态内存分配
-   [ ] 章节12=使用结构体和指针
-   [ ] 章节13-高级指针
-   [ ] 章节14-预处理器
-   [ ] 章节15-输入输出函数
-   [ ] 章节16-标准库函数
-   [ ] 章节17-经典抽象数据类型
-   [ ] 章节18-运行时环境

---



## 引用

[markdown]: doc/Markdown_Reference.md "Markdown Reference"

[makefile_reference]: doc/makefile_reference.md "makefile reference"

