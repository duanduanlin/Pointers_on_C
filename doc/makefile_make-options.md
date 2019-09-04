# 补充资料-`make`选项

## 写在前面
本文作是我学习`C`语言及其相关工具的使用的系列笔记中，关于`make`工具中一个知识点的学习记录。是和整个系列一起上传到[`github`][pointers_on_c]的,里面有全部源码和笔记。
开始之前呢，先对本文所使用的工程实例模板做下说明,后续所有测试都是以此为基础。

首先，是目录结构：

```text
./example2
	-foo.c
	-bar.c
	-main.c
	-makefile
```


然后简单列下，文件内容(因为内容比较少，就列在一起了)：

```text
$ cat bar.c
/*************************************************************************
        > File Name: doc/example0/bar.c
        > Author:ddl
        > Mail:18899533550@163.com
        > Created Time: Fri Aug 30 00:41:28 2019
 ************************************************************************/

#include<stdio.h>

void bar()
{
    printf("enter bar\r\n");
}

$ cat foo.c
/*************************************************************************
        > File Name: doc/example0/foo.c
        > Author:ddl
        > Mail:18899533550@163.com
        > Created Time: Fri Aug 30 00:39:27 2019
 ************************************************************************/

#include<stdio.h>

void foo()
{
    printf("enter foo\r\n");
}

$ cat main.c
/*************************************************************************
        > File Name: doc/example0/main.c
        > Author:ddl
        > Mail:18899533550@163.com
        > Created Time: Fri Aug 30 00:37:02 2019
 ************************************************************************/

#include<stdio.h>

void main(int argc,char**argv)
{
    printf("enter main\r\n");
    foo();
    bar();

    return 0;
}

$ cat makefile
main:main.o foo.o bar.o
        gcc -o main main.o foo.o bar.o

main.o:main.c
        gcc -c main.c -o main.o

foo.o:foo.c
        gcc -c foo.c -o foo.o

bar.o:bar.c
        gcc -c bar.c -o bar.o
```



## `make`选项概述

还记得，之前指定`make`要查找的文件时，使用的`-f`和`--file=`吗？这是`make`工具的参数，下面看看常用的选项都有哪些：

1.  `-b`或`-m`,忽略其他版本`make`的兼容性。
2.  `-B`或`–-always-make`，重编译。
3.  `-C`或`--directory=`,指定读取`makefile`的目录。
4.  `--debug[=]`或`-d`(`--debug=a`),输出`make`的调试信息，有以下几种级别可选，默认输出最简单信息。
    -   `a`--输出所有调试信息。
    -   `b`--输出简单调试信息。
    -   `v`--输出`b`级之上的信息。
    -   `i`--输出隐式规则(`implicit`)。
    -   `j`--输出执行规则中命令的详细信息。
    -   `m`--输出操作`makefile`时的信息

5.  `-e`或`--environment-overrides`,指明环境变量中的值覆盖`makefile`中的值
6.  `-f`或`--file=`或`--makefile=`,指定需要执行的`makefile`。
7.  `-h`或`--help`,显示帮助信息。
8.  `-i`或`--ignore-errors`,执行时忽略错误。
9.  `-I`或`--include-dir=`,指定包含`makefile`的搜索目录。
10.  `-j`或`--jobs[=]`，指定同时运行命令的数目。默认尽可能多的运行
11.  `-k`或`--keep-going`,出错也不停止运行。
12.  `-l`或`--load-average[=]`或`--max-load[=]`,指定`make`运行的负载。
13.  `-n`或`--just-print`或`--dry-run`或`--recon`,仅输出执行命令序列，但并不执行。

14.  `-o`或`--old-file=`或`--assume-old=`，指定不重新生成的目标。
15.  `-p`或`--print-data-base`，输出`makefile`中的所有数据，包括所有规则和变量。
16.  `-q`或`--question`，仅检查目标是否要更新，如果是0说明要更新，2说明有错误
17.  `-r`或`--no-builtin-variabes`，禁止使用变量上的隐式规则。
18.  `-s`或`--silent`或`--quiet`,在运行命令时不输出命令的输出
19.  `-S`或`--no-keep-going`或`--stop`，取消`-k`选项，一般用在`make`的选项是继承来的，而你又不想要。
20.  `-t`或`--toch`，把目标修改日期变为最新的，也就是阻止生成目标。
21.  `-v`或`--version`，输出`make`版本。
22.  `-w`或`--print-directory`，跟踪`makefile`。
23.  `--no-print-directory`，禁止`-w`选项。
24.  `-W`或`--what-if=`或`--new-file`或`--assume-file=`,假定目标需要更新。

25.  `--warn-undefined-variables`,只要`make`发现未定义变量，那么给出警告。



## `make -v`-查看`make`版本

```text
$ make -v
GNU Make 4.1
Built for x86_64-pc-linux-gnu
Copyright (C) 1988-2014 Free Software Foundation, Inc.
License GPLv3+: GNU GPL version 3 or later <http://gnu.org/licenses/gpl.html>
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.
```

可以看到，我这里的版本是`4.1`的，并且是64位的`linux`机上的。



## `make -B`-重现编译

`make -B`呢是不管之前有没有编译，我都换重新编译，这个比较容易理解，结果就不列出来了，下面我们加上`--debug=v`看下`make`到底做了什么(`--debug`选项后面会讲):

```
$ make --debug=v -B
GNU Make 4.1
Built for x86_64-pc-linux-gnu
Copyright (C) 1988-2014 Free Software Foundation, Inc.
License GPLv3+: GNU GPL version 3 or later <http://gnu.org/licenses/gpl.html>
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.
Reading makefiles...
Reading makefile 'makefile'...
Updating goal targets....
Considering target file 'main'.
  Considering target file 'main.o'.
    Considering target file 'main.c'.
     Finished prerequisites of target file 'main.c'.
    No need to remake target 'main.c'.
   Finished prerequisites of target file 'main.o'.
   Prerequisite 'main.c' is older than target 'main.o'.
  Making 'main.o' due to always-make flag.
  Must remake target 'main.o'.
gcc -c main.c -o main.o
main.c: In function ‘main’:
main.c:13:5: warning: implicit declaration of function ‘foo’; did you mean ‘feof’? [-Wimplicit-function-declaration]
     foo();
     ^~~
     feof
main.c:14:5: warning: implicit declaration of function ‘bar’ [-Wimplicit-function-declaration]
     bar();
     ^~~
main.c:16:12: warning: ‘return’ with a value, in function returning void
     return 0;
            ^
main.c:10:6: note: declared here
 void main(int argc,char**argv)
      ^~~~
  Successfully remade target file 'main.o'.
  Considering target file 'foo.o'.
    Considering target file 'foo.c'.
     Finished prerequisites of target file 'foo.c'.
    No need to remake target 'foo.c'.
   Finished prerequisites of target file 'foo.o'.
   Prerequisite 'foo.c' is older than target 'foo.o'.
  Making 'foo.o' due to always-make flag.
  Must remake target 'foo.o'.
gcc -c foo.c -o foo.o
  Successfully remade target file 'foo.o'.
  Considering target file 'bar.o'.
    Considering target file 'bar.c'.
     Finished prerequisites of target file 'bar.c'.
    No need to remake target 'bar.c'.
   Finished prerequisites of target file 'bar.o'.
   Prerequisite 'bar.c' is older than target 'bar.o'.
  Making 'bar.o' due to always-make flag.
  Must remake target 'bar.o'.
gcc -c bar.c -o bar.o
  Successfully remade target file 'bar.o'.
 Finished prerequisites of target file 'main'.
 Prerequisite 'main.o' is newer than target 'main'.
 Prerequisite 'foo.o' is newer than target 'main'.
 Prerequisite 'bar.o' is newer than target 'main'.
Must remake target 'main'.
gcc -o main main.o foo.o bar.o
Successfully remade target file 'main'.
```

上面就是开启`--debug=v`后在加上`-B`选项后的编译结果了，其实`make`的工作流程全部都在这些信息里面，这里可以不用太纠结这些细节。下面这一句很关键：

```text
Making 'main.o' due to always-make flag.
```

这句话，告诉我们`make`是通过设置标志位来检查是正常编译还是强制重新编译。



## `make -C`-指定`make`执行目录

`-C`后面要跟一个目录名，告诉`make`要执行的路径(相对路径和绝对路径都可以)。先试下绝对路像下面这样：

```text
$ make -C `pwd`
make: Entering directory '/home/duanduanlin/workspace/Pointers_on_C/doc/makefile_example/example2'
gcc -c main.c -o main.o
main.c: In function ‘main’:
main.c:13:5: warning: implicit declaration of function ‘foo’; did you mean ‘feof’? [-Wimplicit-function-declaration]
     foo();
     ^~~
     feof
main.c:14:5: warning: implicit declaration of function ‘bar’ [-Wimplicit-function-declaration]
     bar();
     ^~~
main.c:16:12: warning: ‘return’ with a value, in function returning void
     return 0;
            ^
main.c:10:6: note: declared here
 void main(int argc,char**argv)
      ^~~~
gcc -c foo.c -o foo.o
gcc -c bar.c -o bar.o
gcc -o main main.o foo.o bar.o
make: Leaving directory '/home/duanduanlin/workspace/Pointers_on_C/doc/makefile_example/example2'
```

首先`pwd`会返回当前目录的绝对路径，然后`make -C`会切到该目录下执行。注意这里不像`-f`那样仅仅只是告诉`make`文件在那里，然后读取并执行，`make -C dir`其实是，先执行`cd dir`然后执行`make`。为了展示这个效果，我们先拷贝`example2`到`example2_test_C`,然后稍微修改下目录结构：

```text
./example2
	-foo.c
	-bar.c
	-main.c
	-build
		-makefile
```

因为有时候，我们发现把源码已经编译中间文件甚至是编译规则单独出来，这样工程结构会清晰很多，而且编译管理。其实这里应该在根目录下，也有个`makefile`，然后让他去调用其他`makefile`,不过这里只是测试，就省了。好先看看效果：

```text
$ make -C build
make: Entering directory '/home/duanduanlin/workspace/Pointers_on_C/doc/makefile_example/example2_test_C/build'
make: *** No rule to make target 'main.c', needed by 'main.o'.  Stop.
make: Leaving directory '/home/duanduanlin/workspace/Pointers_on_C/doc/makefile_example/example2_test_C/build'
```

可以看到，这里报错了，因为找不到`main.c`。我们说过`-C`是先切换目录然后执行，当然不是真正切换，只是改变`make`环境变量中的当前目录(`CURDIR`),执行完再改回来。所以上面这个问题，只需简单把源码放在`build`目录下就可以了，当然实际使用时不会这么做，毕竟好不容易分离了代码，现在又搞回去。实际中一般会在`makefile`中定义一个变量，用来指定源码目录。



## `make --debug[=]`-输出不同等级的调试信息

`--debug`选项会根据不同的等级，输出不同的调试信息,可以分为下面几个级别:

-   `a`--输出所有调试信息。
-   `b`--输出简单调试信息。
-   `v`--输出`b`级之上的信息。
-   `i`--输出隐式规则(`implicit`)。
-   `j`--输出执行规则中命令的详细信息。
-   `m`--输出操作`makefile`时的信息

要注意的是`make --debug`默认输出简单信息，以及短式写法`-d`等同于`--debug=a`。

下面先回到我们的`example2`目录下，然后一个一个来看，效果是怎样的。



### `--debug`-默认输出简单信息

```
$ make --debug
GNU Make 4.1
Built for x86_64-pc-linux-gnu
Copyright (C) 1988-2014 Free Software Foundation, Inc.
License GPLv3+: GNU GPL version 3 or later <http://gnu.org/licenses/gpl.html>
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.
Reading makefiles...
Updating goal targets....
 File 'main' does not exist.
   File 'main.o' does not exist.
  Must remake target 'main.o'.
  gcc -c main.c -o main.o
main.c: In function ‘main’:
main.c:12:5: warning: implicit declaration of function ‘foo’; did you mean ‘feof’? [-Wimplicit-function-declaration]
     foo();
     ^~~
     feof
main.c:13:5: warning: implicit declaration of function ‘bar’ [-Wimplicit-function-declaration]
     bar();
     ^~~
main.c:15:12: warning: ‘return’ with a value, in function returning void
     return 0;
            ^
main.c:10:6: note: declared here
 void main(int argc,char**argv)
      ^~~~
  Successfully remade target file 'main.o'.
   File 'foo.o' does not exist.
  Must remake target 'foo.o'.
gcc -c foo.c -o foo.o
  Successfully remade target file 'foo.o'.
   File 'bar.o' does not exist.
  Must remake target 'bar.o'.
  gcc -c bar.c -o bar.o
  Successfully remade target file 'bar.o'.
Must remake target 'main'.
gcc -o main main.o foo.o bar.o
Successfully remade target file 'main'.
```

可以看到，虽然输出的信息只是比不带`--debug`选线多出了一点。但是仔细观察，你会发现其实里面隐藏着`make`工具的执行流程。



### `--debug=a`-输出所有信息

设置调试等级为`all`的话，会把`make`执行过程中所有调试信息全部输出，因为实在是太多了，我这里就不全列了，只是简单列下，又那些比较关键的信息。

```text
Reading makefiles...
Reading makefile 'makefile'...
Updating makefiles....
```

`make`首先会去读取`makefile`,并把`makefile`作为一个目标去尝试解析它，虽然不太清楚里面具体做了些什么，但从调试信息上可以看到，好像在查找一些隐式规则什么的，并最终决定是不是要展开以更新`makefile`。

```text
Updating goal targets....
```

`makefile`读取完成后，`make`会去查找最终目标，也就是`makefile`文件的第一个依赖关系的第一个目标，然后会去检查最终目标是不是要更新，并决定是不是要更新最终目标。

```text
Considering target file 'main'.
 File 'main' does not exist.
  Considering target file 'main.o'.
   File 'main.o' does not exist.
    Considering target file 'main.c'.
    Looking for XXX
    ...
    No need to remake target 'main.c'.
   Finished prerequisites of target file 'main.o'.
  Must remake target 'main.o'.
gcc -c main.c -o main.o
```

之后，如果`make`觉得某个目标需要更新的话，`make`会直接执行依赖下的命令，然后这时候会看到类似下面这样的信息：

```text
Reaping winning child 0x7fffea1aded0 PID 12295
Removing child 0x7fffea1aded0 PID 12295 from chain.
gcc -c foo.c -o foo.o
```

可以看到，对于命令的执行，`make`会给每个命令单独开个线程去执行，默认情况下是一次只执行一个指令，当然也可以通过`-j`选项选择同时执行的命令数(当然前提是你系统支持多线程)。

之后就是不断重复，检查目标，更新目标这样的过程。但如果你仔细观察的话，你会发现`make`是严格按照目标依赖关系的顺序来检查的，直到检查完所有依赖并决定是否更新最终目标。就像下面这样：

```text
Considering target file 'main.o'.
...
  Successfully remade target file 'main.o'.
...
Considering target file 'foo.o'.
...
  Successfully remade target file 'foo.o'.
Considering target file 'bar.o'.
 ...
  Successfully remade target file 'bar.o'.
 Finished prerequisites of target file 'main'.
```



### `--debug=b`-输出简单信息

这个效果和`--debug`一样，就不说了。



### `--debug=v`-输出`b`之上的信息

这个是我比较喜欢用的，信息既不会太多的让人眼花缭乱，也不会太少的缺少细节。

```text
$ make --debug=v
...
Reading makefiles...
Reading makefile 'makefile'...
Updating goal targets....
Considering target file 'main'.
 File 'main' does not exist.
  Considering target file 'main.o'.
   File 'main.o' does not exist.
    Considering target file 'main.c'.
     Finished prerequisites of target file 'main.c'.
    No need to remake target 'main.c'.
   Finished prerequisites of target file 'main.o'.
   Must remake target 'main.o'.
gcc -c main.c -o main.o
...
  Successfully remade target file 'main.o'.
  Considering target file 'foo.o'.
   File 'foo.o' does not exist.
    Considering target file 'foo.c'.
     Finished prerequisites of target file 'foo.c'.
    No need to remake target 'foo.c'.
   Finished prerequisites of target file 'foo.o'.
  Must remake target 'foo.o'.
gcc -c foo.c -o foo.o
  Successfully remade target file 'foo.o'.
  Considering target file 'bar.o'.
   File 'bar.o' does not exist.
    Considering target file 'bar.c'.
     Finished prerequisites of target file 'bar.c'.
    No need to remake target 'bar.c'.
   Finished prerequisites of target file 'bar.o'.
  Must remake target 'bar.o'.
gcc -c bar.c -o bar.o
  Successfully remade target file 'bar.o'.
 Finished prerequisites of target file 'main'.
Must remake target 'main'.
gcc -o main main.o foo.o bar.o
Successfully remade target file 'main'.
```

这应该也是研究`make`执行流程最好的范例了吧，里面很详细的追踪了`make`从读取`makefile`开始直到生成最终目标的过程，而且条理清晰，层次感很强。每次看都会对作者的水平叹为观止，简直是艺术。



### `--debug=i`-输出所有隐式规则

这个信息也比较多，就不列出来了，有兴趣的可以自己试试。



### `--debug=j`-输出命令执行时的信息

因为`make`每次执行命令都会单独开一个线程，而`j`等级的话会显示此时线程的相关信息，像`PID`和返回码之类的。

```text
$ make --debug=j
gcc -c main.c -o main.o
Putting child 0x7fffcc14bed0 (main.o) PID 774 on the chain.
Live child 0x7fffcc14bed0 (main.o) PID 774
...
Reaping winning child 0x7fffcc14bed0 PID 774
Removing child 0x7fffcc14bed0 PID 774 from chain.
gcc -c foo.c -o foo.o
Putting child 0x7fffcc14a9b0 (foo.o) PID 777 on the chain.
Live child 0x7fffcc14a9b0 (foo.o) PID 777
Reaping winning child 0x7fffcc14a9b0 PID 777
Removing child 0x7fffcc14a9b0 PID 777 from chain.
gcc -c bar.c -o bar.o
Putting child 0x7fffcc14b1f0 (bar.o) PID 780 on the chain.
Live child 0x7fffcc14b1f0 (bar.o) PID 780
Reaping winning child 0x7fffcc14b1f0 PID 780
Removing child 0x7fffcc14b1f0 PID 780 from chain.
gcc -o main main.o foo.o bar.o
Putting child 0x7fffcc14db20 (main) PID 783 on the chain.
Live child 0x7fffcc14db20 (main) PID 783
Reaping winning child 0x7fffcc14db20 PID 783
Removing child 0x7fffcc14db20 PID 783 from chain.
```



### `--debug=m`-显示执行`makefile`前后的信息

这个主要是显示对`makefile`操作是的信息，不管效果不是很明显，可以看下：

```text
Reading makefiles...
Reading makefile 'makefile'...
Updating makefiles....
```



## `make -e`-用环境变量覆盖本地变量

这个主要是用在不同`makefile`之前传递参数，`make`在参数传递时一般使用`export`导出参数，到环境变量，这样你后面执行别的`makefile`时，就可以从环境变量中读取了。但是如果你`makefile`中本身是有一个同名变量时，`make`默认是会忽略环境变量的值，而是以你当前变量为准。但有时如果你需要环境变量中的值的话，你可以在执行`makefile`时加上`-e`，这样会用你环境变量中俄值覆盖你本地的变量。不过需要注意的是这样会覆盖所有重名变量，如果你不想它覆盖的话，可以在变量声明前`unexport`下。

好的，为了测试这个效果，我们新建`example2_test_e`,然后增加`example2_test_e/test_e.mk`和`example2_test_e/test_export.mk`,并修改`example2_test_e/makefile`。各文件内容如下：

```text
$cat makefile
param = hello

export param

run:
	@echo "in makefile param = "$(param)
    $(MAKE) -f test_export.mk
    $(MAKE) -f test_e.mk
    
$ cat test_e.mk
param = world

test:
	@echo "in test.mk param = "$(param)
	
$ cat test_export.mk

test_export:
	@echo "param in test_export.mk = "$(param)
```

先看下效果：

```text
$ make
in makefile param = hello
make -f test_export.mk
make[1]: Entering directory '/home/duanduanlin/workspace/Pointers_on_C/doc/makefile_example/example2_test_e'
param in test_export.mk = hello
make[1]: Leaving directory '/home/duanduanlin/workspace/Pointers_on_C/doc/makefile_example/example2_test_e'
make -f test_e.mk
make[1]: Entering directory 
'/home/duanduanlin/workspace/Pointers_on_C/doc/makefile_example/example2_test_e'
in test.mk param = world
make[1]: Leaving directory '/home/duanduanlin/workspace/Pointers_on_C/doc/makefile_example/example2_test_e'
```

可以看到，首先，我们在`makefile`中定义了一个变量`param`值为`hello`,并把它导出，然后我们在`test_export.mk`中直接引用`param`可以看到，其值还是`hello`。最后，我们在`test_e.mk`也定义了一个`param`，其值为`world`。尽管此时全局的值为`hello`，但默认下`make`并不会用环境变量的值覆盖`makefile`中的变量。

如果想要使用环境变量覆盖`makefile`中的变量的话，可以使用`-e`,如下：

```text
$ make -e
in makefile param = hello
make -f test_export.mk
make[1]: Entering directory '/home/duanduanlin/workspace/Pointers_on_C/doc/makefile_example/example2_test_e'
param in test_export.mk = hello
make[1]: Leaving directory '/home/duanduanlin/workspace/Pointers_on_C/doc/makefile_example/example2_test_e'
make -f test_e.mk
make[1]: Entering directory 
'/home/duanduanlin/workspace/Pointers_on_C/doc/makefile_example/example2_test_e'
in test.mk param = hello
make[1]: Leaving directory '/home/duanduanlin/workspace/Pointers_on_C/doc/makefile_example/example2_test_e'
```



## `make -f`-指定`makefile`文件路径

我们知道`make`执行时，默认是搜索当前目录下的`Makefile`或`makefile`以及`GNUmakefile`等。如果你想给你的`makefile`取个别的名字，或者你要执行的`makefile`并不在当前目录下的话，你可以使用`-f dir/file`或`--file=dir/file`来指定要执行的`makefile`文件。当然就像之前的`-C`一样，路径既可以是绝对路径也可以是相对路径。

比如，假设你当前处于`example2`目录下，你可以这样：

```text
$ make -f `pwd`/makefile
gcc -c main.c -o main.o
main.c: In function ‘main’:
main.c:13:5: warning: implicit declaration of function ‘foo’; did you mean ‘feof’? [-Wimplicit-function-declaration]
     foo();
     ^~~
     feof
main.c:14:5: warning: implicit declaration of function ‘bar’ [-Wimplicit-function-declaration]
     bar();
     ^~~
main.c:16:12: warning: ‘return’ with a value, in function returning void
     return 0;
            ^
main.c:10:6: note: declared here
 void main(int argc,char**argv)
      ^~~~
gcc -c foo.c -o foo.o
gcc -c bar.c -o bar.o
gcc -o main main.o foo.o bar.o
```

或者：

```text
 make --file=`pwd`/makefile
```

也可以实现同样的效果。

也可能你的`makefile`的名称是`build.mk`等非标准名称，且在你当前目录下，你可以像这样：

```text
make -f build.mk
```

执行`make`指令。



## `make -i`-忽略指令执行错误

`-i`选项会忽略指令执行时的错误，并继续执行，且如果出错的话就会生成目标文件。这个一般用在调试的时候。比如你的关注点是想看看`makefile`的执行流程对不对，而对些局部指令错误没那么关心，这时就可以加上`-i`选项。要注意`-i`选项是忽略执行指令时的错误继续执行，如果出错的是`makefile`本身的规则，`make`仍会直接停止运行。试试下面这样。

先拷贝`example2`到`example2_test_i`,并像下面这样修改最终目标的依赖：

```text
main:main.o fo.o bar.o
	gcc -o main main.o foo.o bar.o
```

其中目标的依赖`foo.o`改成`fo.o`,来模拟你在书写时，不小心写错依赖了(这是完全有可能的)。

然后，`make`下：

```text
$ make
gcc -c main.c -o main.o
main.c: In function ‘main’:
main.c:13:5: warning: implicit declaration of function ‘foo’; did you mean ‘feof’? [-Wimplicit-function-declaration]
     foo();
     ^~~
     feof
main.c:14:5: warning: implicit declaration of function ‘bar’ [-Wimplicit-function-declaration]
     bar();
     ^~~
main.c:16:12: warning: ‘return’ with a value, in function returning void
     return 0;
            ^
main.c:10:6: note: declared here
 void main(int argc,char**argv)
      ^~~~
make: *** No rule to make target 'fo.o', needed by 'main'.  Stop.
```

你会看到`make`报了个错，告诉你没找到`fo.o`的规则。

下面试试`make -i`怎么样？

```text
$ make -i
gcc -c main.c -o main.o
main.c: In function ‘main’:
main.c:13:5: warning: implicit declaration of function ‘foo’; did you mean ‘feof’? [-Wimplicit-function-declaration]
     foo();
     ^~~
     feof
main.c:14:5: warning: implicit declaration of function ‘bar’ [-Wimplicit-function-declaration]
     bar();
     ^~~
main.c:16:12: warning: ‘return’ with a value, in function returning void
     return 0;
            ^
main.c:10:6: note: declared here
 void main(int argc,char**argv)
      ^~~~
make: *** No rule to make target 'fo.o', needed by 'main'.  Stop.
```

你会看到，是不是和之前没有任何区别。

下面我们再改下，比如这次你不小心把`gcc`编译选项的`-o`写成了`-0`。当然方便起见我们在当前目录下拷贝一份`makefile`到`test_i.mk`，内容修改如下：

```text
$ cat test_i.mk
main:main.o foo.o bar.o
	gcc -o main main.o foo.o bar.o

main.o:main.c
	gcc -c main.c -0 main.o

foo.o:foo.c
	gcc -c foo.c -o foo.o

bar.o:bar.c
	gcc -c bar.c -o bar.o
```

然后再`make -i -f test_i.mk`:

```text
$ make -i -f test_i.mk
gcc -c main.c -0 main.o
gcc: error: main.o: No such file or directory
gcc: error: unrecognized command line option ‘-0’
test_i.mk:5: recipe for target 'main.o' failed
make: [main.o] Error 1 (ignored)
gcc -c foo.c -o foo.o
gcc -c bar.c -o bar.o
gcc -o main main.o foo.o bar.o
gcc: error: main.o: No such file or directory
test_i.mk:2: recipe for target 'main' failed
make: [main] Error 1 (ignored)
```

你会看到，虽然`gcc`报错了，但`make`仍然继续往下运行了。然后在最后告诉你，出现了一个错误，所以`main`的生成被忽略了。

与`-i`类似的是`-k`,但`-k`不仅可以忽略指令错误，而且还能忽略`makefile`规则错误(语法错误除外)。



## `make -k`-出错也不停止运行

`-k`可以说是`-i`的高级版，不仅可以忽略指令错误，而且还能忽略`makefile`规则错误。

我们直接从`example2_test_i`上拷贝一份到`example2_test_k`，并把`test_i.mk`改成`test_k.mk`，然后先直接`make -k`:

```text
$ make -k
gcc -c main.c -o main.o
main.c: In function ‘main’:
main.c:12:5: warning: implicit declaration of function ‘foo’; did you mean ‘feof’? [-Wimplicit-function-declaration]
     foo();
     ^~~
     feof
main.c:13:5: warning: implicit declaration of function ‘bar’ [-Wimplicit-function-declaration]
     bar();
     ^~~
main.c:15:12: warning: ‘return’ with a value, in function returning void
     return 0;
            ^
main.c:10:6: note: declared here
 void main(int argc,char**argv)
      ^~~~
make: *** No rule to make target 'fo.o', needed by 'main'.
gcc -c bar.c -o bar.o
make: Target 'main' not remade because of errors.
```

你会看到，之前`-i`束手无策的依赖错误，在`-k`这里再也不是问题，`make`仍然运行完了，只是在最后告诉你，最终目标因为有错误，我就不生成了。

然后试下`make -k -f test_k.mk`:

```text
$ make -k -f test_k.mk
gcc -0 main.c -o main.o
gcc: error: unrecognized command line option ‘-0’
test_k.mk:5: recipe for target 'main.o' failed
make: *** [main.o] Error 1
gcc -c foo.c -o foo.o
gcc -c bar.c -o bar.o
make: Target 'main' not remade because of errors.
```

同样，可以看到对于指令错误`-k`也可以忽略。

下面来看看，对于语法错误`-k`能忽略吗？比如不小心`:`写成了`：`。

先拷贝`test_k.mk`到`test_alg.mk`，然后修改如下：

```text
$ make -k -f test_alg.mk
test_alg.mk:4: *** missing separator.  Stop.
```

可以看到，`make`直接停止运行了。你也可以`--debug=v`看下，发生了什么：

```text
$ make --debug=v -k -f test_alg.mk
GNU Make 4.1
Built for x86_64-pc-linux-gnu
Copyright (C) 1988-2014 Free Software Foundation, Inc.
License GPLv3+: GNU GPL version 3 or later <http://gnu.org/licenses/gpl.html>
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.
Reading makefiles...
Reading makefile 'test_alg.mk'...
test_alg.mk:4: *** missing separator.  Stop.
```

显然，是读取`makefile`文件的过程中发现了错误。

所以，`-k`不仅可以忽略指令执行的错误，而且可以忽略``makefile`规则错误，但是不能忽略`makefile`的语法错误。



## `make -S`-取消`-k`的效果

有时候，你可能并不想让`make`忽略执行错误继续执行，与之相反，你可能需要`make`在遇到错误时立即停止。这个主要是用在嵌套调用的时候，这时下层`makefile`的执行会从上层继承一些选项，比如`-k`。而如果你不想忽略错误的话，就可以使用`-S`来取消`-k`的效果。

还是先拷贝`example2`到`example2_test_S`,然后增加一个`build.mk`,内容如下：

```text
main:main.o fo.o bar.o
	gcc -o main main.o foo.o bar.o

main.o:main.c
	gcc -c main.c -o main.o

foo.o:foo.c
	gcc -c foo.c -o foo.o

bar.o:bar.c
	gcc -c bar.c -o bar.o
```

可以看到，默认情况下`make`在检查`main`的依赖时会因找不到`fo.o`而停止。同时，修改`makefile`,在里面调用`build.mk`。

```text
$ cat makefile
build:
	$(MAKE) -f build.mk
```

先试下效果：

```text
$ make
make -f build.mk
make[1]: Entering directory '/home/duanduanlin/workspace/Pointers_on_C/doc/makefile_example/example2_test_S'
gcc -c main.c -o main.o
main.c: In function ‘main’:
main.c:12:5: warning: implicit declaration of function ‘foo’; did you mean ‘feof’? [-Wimplicit-function-declaration]
     foo();
     ^~~
     feof
main.c:13:5: warning: implicit declaration of function ‘bar’ [-Wimplicit-function-declaration]
     bar();
     ^~~
main.c:15:12: warning: ‘return’ with a value, in function returning void
     return 0;
            ^
main.c:10:6: note: declared here
 void main(int argc,char**argv)
      ^~~~
make[1]: *** No rule to make target 'fo.o', needed by 'main'.  Stop.
make[1]: Leaving directory '/home/duanduanlin/workspace/Pointers_on_C/doc/makefile_example/example2_test_S'
makefile:2: recipe for target 'build' failed
make: *** [build] Error 2
```

可以看到效果完全符合预期，然后，我们在最外层调用`make`时传入`-k`选项：

```text
$ make -k
make -f build.mk
make[1]: Entering directory '/home/duanduanlin/workspace/Pointers_on_C/doc/makefile_example/example2_test_S'
gcc -c main.c -o main.o
main.c: In function ‘main’:
main.c:12:5: warning: implicit declaration of function ‘foo’; did you mean ‘feof’? [-Wimplicit-function-declaration]
     foo();
     ^~~
     feof
main.c:13:5: warning: implicit declaration of function ‘bar’ [-Wimplicit-function-declaration]
     bar();
     ^~~
main.c:15:12: warning: ‘return’ with a value, in function returning void
     return 0;
            ^
main.c:10:6: note: declared here
 void main(int argc,char**argv)
      ^~~~
make[1]: *** No rule to make target 'fo.o', needed by 'main'.
gcc -c bar.c -o bar.o
make[1]: Target 'main' not remade because of errors.
make[1]: Leaving directory '/home/duanduanlin/workspace/Pointers_on_C/doc/makefile_example/example2_test_S'
makefile:2: recipe for target 'build' failed
make: *** [build] Error 2
```

我们可以看到，虽然`make`出错了，但是仍然继续执行了，这说明，我们在命令行调用`makefile`时传入的选项，被我们在`makefile`中调用的`build.mk`时继承了(关于参数继承，我会在后面递归调用时讲)。

下面，修改`makefile`,增加`-S`选项，如下：

```text
build:
	$(MAKE) -S -f build.mk
```

再试试：

```text
$ make -k
make -S -f build.mk
make[1]: Entering directory '/home/duanduanlin/workspace/Pointers_on_C/doc/makefile_example/example2_test_S'
gcc -c main.c -o main.o
main.c: In function ‘main’:
main.c:12:5: warning: implicit declaration of function ‘foo’; did you mean ‘feof’? [-Wimplicit-function-declaration]
     foo();
     ^~~
     feof
main.c:13:5: warning: implicit declaration of function ‘bar’ [-Wimplicit-function-declaration]
     bar();
     ^~~
main.c:15:12: warning: ‘return’ with a value, in function returning void
     return 0;
            ^
main.c:10:6: note: declared here
 void main(int argc,char**argv)
      ^~~~
make[1]: *** No rule to make target 'fo.o', needed by 'main'.  Stop.
make[1]: Leaving directory '/home/duanduanlin/workspace/Pointers_on_C/doc/makefile_example/example2_test_S'
makefile:2: recipe for target 'build' failed
make: *** [build] Error 2
```

可以看到，结果和不加`-k`没什么区别。



## `make -I`-指定包含`makefile`的搜索目录

这个主要用在`include`其他`makefile`时，如果你没有指定要搜索的目录时，`make`会默认在当前目录和系统环境变量中搜索。

比如，你想把编译规则单独放在一个文件夹中，然后再需要的地方`include`一下，以实现编译规则和你的源码的分离。这时候，如果你的目录嵌套比较深的话，直接输入编译规则的路径就比较麻烦，而且这样的话可移植性也很差，因为你一旦修改目录，所有`include`处都要改。因此一般的做法是`include`时直接导入你想要的文件，然后在调用处指定搜索路径，当然你也可以多套一层`makefile`，从而利用递归调用来帮你指定搜索路径。这样你就可以直接在最外层`make`下，就能实现你的逻辑，从而简化调用。

同样还是拷贝`example2`到`example2_test_I`,增加`build/build.mk`,并修改`makefile`如下：

```text
$ cat makefile

include build.mk

$ cat build/build.mk

main:main.o foo.o bar.o
        gcc -o main main.o foo.o bar.o

main.o:main.c
        gcc -c main.c -o main.o

foo.o:foo.c
        gcc -c foo.c -o foo.o

bar.o:bar.c
        gcc -c bar.c -o bar.o
```

当然简单起见，这里还是命令行下传参，执行`make -I build`：

```text
$ make -I build
gcc -c main.c -o main.o
main.c: In function ‘main’:
main.c:12:5: warning: implicit declaration of function ‘foo’; did you mean ‘feof’? [-Wimplicit-function-declaration]
     foo();
     ^~~
     feof
main.c:13:5: warning: implicit declaration of function ‘bar’ [-Wimplicit-function-declaration]
     bar();
     ^~~
main.c:15:12: warning: ‘return’ with a value, in function returning void
     return 0;
            ^
main.c:10:6: note: declared here
 void main(int argc,char**argv)
      ^~~~
gcc -c foo.c -o foo.o
gcc -c bar.c -o bar.o
gcc -o main main.o foo.o bar.o
```



## `make -j[n]`-并行处理指令

之前有说到`make`在执行指令时，会单独开个线程去执行，并且默认是每次执行一个。那么使用`-j`就可以指定同时可以处理多少条指令，默认是尽可能多的执行。

首先，我们进入`example2`目录下，然后执行`make -j`。

是不是感觉编译要快了很多呢？不过因为工程比较小，可能感觉不到，不过没关系，我们可以添加`--debug=j`选项，这样会明显些：

```text
$ make --debug=j -j
gcc -c main.c -o main.o
Putting child 0x7fffb898aed0 (main.o) PID 13868 on the chain.
Live child 0x7fffb898aed0 (main.o) PID 13868
gcc -c foo.c -o foo.o
Putting child 0x7fffb898cb10 (foo.o) PID 13869 on the chain.
Live child 0x7fffb898cb10 (foo.o) PID 13869
Live child 0x7fffb898aed0 (main.o) PID 13868
gcc -c bar.c -o bar.o
Putting child 0x7fffb898e640 (bar.o) PID 13870 on the chain.
Live child 0x7fffb898e640 (bar.o) PID 13870
Live child 0x7fffb898cb10 (foo.o) PID 13869
Live child 0x7fffb898aed0 (main.o) PID 13868
main.c: In function ‘main’:
main.c:13:5: warning: implicit declaration of function ‘foo’; did you mean ‘feof’? [-Wimplicit-function-declaration]
     foo();
     ^~~
     feof
main.c:14:5: warning: implicit declaration of function ‘bar’ [-Wimplicit-function-declaration]
     bar();
     ^~~
main.c:16:12: warning: ‘return’ with a value, in function returning void
     return 0;
            ^
main.c:10:6: note: declared here
 void main(int argc,char**argv)
      ^~~~
Reaping winning child 0x7fffb898cb10 PID 13869
Removing child 0x7fffb898cb10 PID 13869 from chain.
Live child 0x7fffb898e640 (bar.o) PID 13870
Live child 0x7fffb898aed0 (main.o) PID 13868
Reaping winning child 0x7fffb898aed0 PID 13868
Removing child 0x7fffb898aed0 PID 13868 from chain.
Live child 0x7fffb898e640 (bar.o) PID 13870
Reaping winning child 0x7fffb898e640 PID 13870
Removing child 0x7fffb898e640 PID 13870 from chain.
gcc -o main main.o foo.o bar.o
Putting child 0x7fffb8991e90 (main) PID 13877 on the chain.
Live child 0x7fffb8991e90 (main) PID 13877
Reaping winning child 0x7fffb8991e90 PID 13877
Removing child 0x7fffb8991e90 PID 13877 from chain.

# duanduanlin @ ddl-pc in ~/workspace/Pointers_on_C/doc/makefile_example/example2 on git:master x [2:01:25]
$ rm *.o main

# duanduanlin @ ddl-pc in ~/workspace/Pointers_on_C/doc/makefile_example/example2 on git:master x [2:01:44]
$ make -j
gcc -c main.c -o main.o
gcc -c foo.c -o foo.o
gcc -c bar.c -o bar.o
main.c: In function ‘main’:
main.c:13:5: warning: implicit declaration of function ‘foo’; did you mean ‘feof’? [-Wimplicit-function-declaration]
     foo();
     ^~~
     feof
main.c:14:5: warning: implicit declaration of function ‘bar’ [-Wimplicit-function-declaration]
     bar();
     ^~~
main.c:16:12: warning: ‘return’ with a value, in function returning void
     return 0;
            ^
main.c:10:6: note: declared here
 void main(int argc,char**argv)
      ^~~~
gcc -o main main.o foo.o bar.o

# duanduanlin @ ddl-pc in ~/workspace/Pointers_on_C/doc/makefile_example/example2 on git:master x [2:01:48]
$ rm *.o main

# duanduanlin @ ddl-pc in ~/workspace/Pointers_on_C/doc/makefile_example/example2 on git:master x [2:03:30]
$ make --debug=j -j
gcc -c main.c -o main.o
Putting child 0x7fffd137eed0 (main.o) PID 13959 on the chain.
Live child 0x7fffd137eed0 (main.o) PID 13959
gcc -c foo.c -o foo.o
Putting child 0x7fffd1380b10 (foo.o) PID 13960 on the chain.
Live child 0x7fffd1380b10 (foo.o) PID 13960
Live child 0x7fffd137eed0 (main.o) PID 13959
gcc -c bar.c -o bar.o
Putting child 0x7fffd1382640 (bar.o) PID 13961 on the chain.
Live child 0x7fffd1382640 (bar.o) PID 13961
Live child 0x7fffd1380b10 (foo.o) PID 13960
Live child 0x7fffd137eed0 (main.o) PID 13959
main.c: In function ‘main’:
main.c:13:5: warning: implicit declaration of function ‘foo’; did you mean ‘feof’? [-Wimplicit-function-declaration]
     foo();
     ^~~
     feof
main.c:14:5: warning: implicit declaration of function ‘bar’ [-Wimplicit-function-declaration]
     bar();
     ^~~
main.c:16:12: warning: ‘return’ with a value, in function returning void
     return 0;
            ^
main.c:10:6: note: declared here
 void main(int argc,char**argv)
      ^~~~
Reaping winning child 0x7fffd1382640 PID 13961
Removing child 0x7fffd1382640 PID 13961 from chain.
Live child 0x7fffd1380b10 (foo.o) PID 13960
Live child 0x7fffd137eed0 (main.o) PID 13959
Reaping winning child 0x7fffd1380b10 PID 13960
Removing child 0x7fffd1380b10 PID 13960 from chain.
Live child 0x7fffd137eed0 (main.o) PID 13959
Live child 0x7fffd137eed0 (main.o) PID 13959
Reaping winning child 0x7fffd137eed0 PID 13959
Removing child 0x7fffd137eed0 PID 13959 from chain.
gcc -o main main.o foo.o bar.o
Putting child 0x7fffd1380d60 (main) PID 13968 on the chain.
Live child 0x7fffd1380d60 (main) PID 13968
Reaping winning child 0x7fffd1380d60 PID 13968
Removing child 0x7fffd1380d60 PID 13968 from chain.
```

加了`--debug=j`的话，会显示线程调用信息。通过对比不加`-j`和加了的，你会发现不加`-j`的其线程调用是一个一个的，而加了的话顺序就没那么明显了。



## `make -n`-仅输出指令调用，但不执行

`-n`一般用在调试`makefile`编译流程上，加上它的的，`make`在执行命令时，仅会输出执行过程，并不会真的执行。

还是首先进入`example2`目录，然后`make -n`试下：

```text
$ make -n
gcc -c main.c -o main.o
gcc -c foo.c -o foo.o
gcc -c bar.c -o bar.o
gcc -o main main.o foo.o bar.o
```

你可以看到，虽然输出结果和之前没太大区别(仅仅只是少了下编译时的信息，这也说明它没编译)。但如果你看下当前目录下的文件的话，你会发现没有任何变化。

下面我们在加上`--debug=v`来看看`make`干了什么：

```text
$ make --debug=v -n
GNU Make 4.1
Built for x86_64-pc-linux-gnu
Copyright (C) 1988-2014 Free Software Foundation, Inc.
License GPLv3+: GNU GPL version 3 or later <http://gnu.org/licenses/gpl.html>
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.
Reading makefiles...
Reading makefile 'makefile'...
Updating goal targets....
Considering target file 'main'.
 File 'main' does not exist.
  Considering target file 'main.o'.
   File 'main.o' does not exist.
    Considering target file 'main.c'.
     Finished prerequisites of target file 'main.c'.
    No need to remake target 'main.c'.
   Finished prerequisites of target file 'main.o'.
  Must remake target 'main.o'.
gcc -c main.c -o main.o
  Successfully remade target file 'main.o'.
  Considering target file 'foo.o'.
   File 'foo.o' does not exist.
    Considering target file 'foo.c'.
     Finished prerequisites of target file 'foo.c'.
    No need to remake target 'foo.c'.
   Finished prerequisites of target file 'foo.o'.
  Must remake target 'foo.o'.
gcc -c foo.c -o foo.o
  Successfully remade target file 'foo.o'.
  Considering target file 'bar.o'.
   File 'bar.o' does not exist.
    Considering target file 'bar.c'.
     Finished prerequisites of target file 'bar.c'.
    No need to remake target 'bar.c'.
   Finished prerequisites of target file 'bar.o'.
  Must remake target 'bar.o'.
gcc -c bar.c -o bar.o
  Successfully remade target file 'bar.o'.
 Finished prerequisites of target file 'main'.
Must remake target 'main'.
gcc -o main main.o foo.o bar.o
Successfully remade target file 'main'.
```

是不是看起来和正常编译一样。



## `make -o`-不重新生成某目标

`-o xxx`作用是告诉`make`关于`xxx`你就不用检查了，它是最新的。

下面，我们直接在一个干净的目录下`make -o xxx`试试:

```text
$ make -o main
make: 'main' is up to date.
```

可以看到，这里我是把最终目标作为`-o`选项的参数，可以看到`make`直接就提示我们,`main`已经是新的了，即使当前目录下`main`并不存在。

同样我们`--debug=v`看看发生了什么:

```text
$ make --debug=v -o main
GNU Make 4.1
Built for x86_64-pc-linux-gnu
Copyright (C) 1988-2014 Free Software Foundation, Inc.
License GPLv3+: GNU GPL version 3 or later <http://gnu.org/licenses/gpl.html>
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.
Reading makefiles...
Reading makefile 'makefile'...
Updating goal targets....
Considering target file 'main'.
File 'main' was considered already.
make: 'main' is up to date.
```

可以看到，`make`压根就没检查`main`的依赖，甚至连目标是不是存在都没检查。



## `make -t`-用`touch`替换要执行的命令

和`-o`效果类似，`-t`也会阻止目标的生生成。不过和`-o`不同的是，`-t`是在目标需要生成时，`touch`一个目标。

下面还是在`example2`下直接`make -t`,并加上`--debug=v`，来看看发生了什么:

```text
$ make --debug=v -t
GNU Make 4.1
Built for x86_64-pc-linux-gnu
Copyright (C) 1988-2014 Free Software Foundation, Inc.
License GPLv3+: GNU GPL version 3 or later <http://gnu.org/licenses/gpl.html>
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.
Reading makefiles...
Reading makefile 'makefile'...
Updating goal targets....
Considering target file 'main'.
 File 'main' does not exist.
  Considering target file 'main.o'.
   File 'main.o' does not exist.
    Considering target file 'main.c'.
     Finished prerequisites of target file 'main.c'.
    No need to remake target 'main.c'.
   Finished prerequisites of target file 'main.o'.
  Must remake target 'main.o'.
touch main.o
  Successfully remade target file 'main.o'.
  Considering target file 'foo.o'.
   File 'foo.o' does not exist.
    Considering target file 'foo.c'.
     Finished prerequisites of target file 'foo.c'.
    No need to remake target 'foo.c'.
   Finished prerequisites of target file 'foo.o'.
  Must remake target 'foo.o'.
touch foo.o
  Successfully remade target file 'foo.o'.
  Considering target file 'bar.o'.
   File 'bar.o' does not exist.
    Considering target file 'bar.c'.
     Finished prerequisites of target file 'bar.c'.
    No need to remake target 'bar.c'.
   Finished prerequisites of target file 'bar.o'.
  Must remake target 'bar.o'.
touch bar.o
  Successfully remade target file 'bar.o'.
 Finished prerequisites of target file 'main'.
Must remake target 'main'.
touch main
Successfully remade target file 'main'.
```

可以看到，所有需要重新生成的目标，`make`在执行命令时全部用`touch`替换了一遍。

下面简单对比下`-o`和`-t`,虽然都是阻止目标更新，但其区别还是蛮大的。`-o`是直接阻止`make`去检查目标，所以目标其实仍处于未更新状态，但`-t`确实把目标以及目标需要更新的依赖全部`touch`了一遍，其结果是虽然目标可能内容上没有任何变化，但是其时间戳是最新的。也就是说，对于`make`来说，这个目标就是最新的。

所以`-t`的优点就很明显了。因为有时候，对于一些文件，你并不想马上更新它，而且又不想每次都用`-o`去忽略他，这样就可以`-t`一下，强行更新时间戳，一劳永逸。



## `make -p`-输出当前环境中所有变量和规则

这个选项会把执行`makefile`时，环境中的所有变量和规则全部输出。



## `make -q`-检查某目标是不是要更新

`-p`选择既不会运行任何命令，也不会输出任何信息，它只是检查目标是不是需要更新。同时返回一个状态，其中，0表示要更新，2表示错误。



`make -s`-禁止输出命令

加上`-s`，可以让`make`静默的执行命令。下面看看效果：

```text
$ make -s
main.c: In function ‘main’:
main.c:13:5: warning: implicit declaration of function ‘foo’; did you mean ‘feof’? [-Wimplicit-function-declaration]
     foo();
     ^~~
     feof
main.c:14:5: warning: implicit declaration of function ‘bar’ [-Wimplicit-function-declaration]
     bar();
     ^~~
main.c:16:12: warning: ‘return’ with a value, in function returning void
     return 0;
            ^
main.c:10:6: note: declared here
 void main(int argc,char**argv)
```

可以看到，是不是所有的命令执行的输出全没了，这就是`-s`选项的效果。



## `make -w`-跟踪嵌套调用

这个会在你调用`makefile`前后显示相关信息。可以简单看下效果。

首先，方便起见，我们直接拷贝`example2_test_C`到`example2_test_w`，先直接`make`下:

```text
$ make
make -C build
make[1]: Entering directory '/home/duanduanlin/workspace/Pointers_on_C/doc/makefile_example/example2_test_w/build'
gcc -c main.c -o main.o
gcc -c foo.c -o foo.o
gcc -c bar.c -o bar.o
gcc -o main main.o foo.o bar.o
make[1]: Leaving directory '/home/duanduanlin/workspace/Pointers_on_C/doc/makefile_example/example2_test_w/build'
```

你会看到在`make -c build`，进入`build`目录执行`build.mk`时，打印了一些目录信息，这个就是`-w`的效果了，可能你会奇怪，我这里没加`-w`啊！为什么会有`-w`的效果呢？其实`make -C`时会默认带上`-w`选项。

如果你在当前目录下执行`make`也加上`-w`选项，看看会发生什么：

```text
$ make -w
make: Entering directory '/home/duanduanlin/workspace/Pointers_on_C/doc/makefile_example/example2_test_w'
make -C build
make[1]: Entering directory '/home/duanduanlin/workspace/Pointers_on_C/doc/makefile_example/example2_test_w/build'
make[1]: 'main' is up to date.
make[1]: Leaving directory '/home/duanduanlin/workspace/Pointers_on_C/doc/makefile_example/example2_test_w/build'
make: Leaving directory '/home/duanduanlin/workspace/Pointers_on_C/doc/makefile_example/example2_test_w'
```

你会看到，是不是多了个当前目录下的信息啊！其实这是`make`把当前目录下的`makefile`的追踪信息也输出了。

和之前`-k`一样，如果你不想追踪`makefile`，你可以使用`--no-print-directory`取消它。



## `make -W`-假定目标需要更新

`-W`感觉像是`-o`和`-t`的结合，首先`-W`并不会去生成目标，这个和`-o`比较像。其次`-W`会去检查目标依赖，并更新依赖，这个和`-t`比较像，不过不同的是`-t`只是更新时间戳，而`-W`是真的有更新目标。也就是最终，`-W`会实现这样的效果，也就是它会更新目标的依赖，但唯独不更新目标。

为了`-W`,我们拷贝`example2`到`example2_test_W`:

首先，我们`make -W mian`：

```text
$ make -W main
gcc -c main.c -o main.o
main.c: In function ‘main’:
main.c:13:5: warning: implicit declaration of function ‘foo’; did you mean ‘feof’? [-Wimplicit-function-declaration]
     foo();
     ^~~
     feof
main.c:14:5: warning: implicit declaration of function ‘bar’ [-Wimplicit-function-declaration]
     bar();
     ^~~
main.c:16:12: warning: ‘return’ with a value, in function returning void
     return 0;
            ^
main.c:10:6: note: declared here
 void main(int argc,char**argv)
      ^~~~
gcc -c foo.c -o foo.o
gcc -c bar.c -o bar.o
```

你会看到，`make`把所有要更新的依赖全都更新了，唯独`main`没有更新。

下面加上`--debug=v`，看看发生了什么：

```text
...
  Successfully remade target file 'bar.o'.
 Finished prerequisites of target file 'main'.
 Prerequisite 'main.o' is newer than target 'main'.
 Prerequisite 'foo.o' is newer than target 'main'.
 Prerequisite 'bar.o' is newer than target 'main'.
No need to remake target 'main'.
```

可以看到，前面和正常编译没有任何区别，只是在最后即将生成目标时，取消生成。这个`-o`就完全相反了，`-o`是在一开始就拒绝检查，而`-W`是在最后拒绝生成。



## 参考

[跟我一起写 Makefile](https://www.cnblogs.com/BigBang/articles/403511.html)

[`GNU make`](http://www.gnu.org/software/make/manual/make.html)

[GNU make参数详解](http://www.ha97.com/961.html)

[Makefile概念入门](https://zhuanlan.zhihu.com/p/29910215)

[pointers_on_c]:https://github.com/duanduanlin/Pointers_on_C "pointers_on_c"

