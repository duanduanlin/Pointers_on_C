# Makefile 参考

## 写在前面

前段时间因为工作需要，要移植阿里的飞燕平台(`IOT`平台)到我们公司的一款`wifi`模块上。因为这款模块本身是带有小系统的，用的是`RTT`的`rt_thread`系统，而`rt_thread`本身也是有自己的一套基于`scons`的编译规则(类似于`GNU Make`，后面有时间再玩下),所以要考虑把飞燕平台的代码加到`rt_thread`的编译规则中去。当然这个很简单了，照猫画虎加进去就可以了。但是加完后才发现，自己很多东西都是一知半解，明明实例上也是这么用的，而我照搬过来就是不行，然后也找了些文章，学习了下`makefile`以及`menuconfig`的基本规则,才勉强搞定。(^_^|||)感觉自己很水。。。不过想到自己毕竟是业余的，又没什么经验。。。生活还得继续。

刚好最近，终于下定决心要好好补一补基础(实在太烂了)，才有了这一系列文章。好了，别的不多说，一起加油吧！↖(^ω^)↗



## 概述

我们一般所说的`makefile`其实是一个脚本文件，是按照一定的语法书写的，将源文件编译成可执行文件的脚本，它是给像`GNU make`之类的工具使用的。以`GNU make`为例，你在命令行输入`make`后，它会默认在当前文件下搜索名为`makefile`，`Makefile`或`GNUmakefile`的文件，当然你也可以通过设置`-f`或`--file`参数来指定你想要搜索的文件，如 `make -f xxx.xxx`或`make --file xxx.xxx`来搜索一个名叫`xxx.xxx`的文件。`make`工具找到脚本文件之后，会解析它，并按照一定的规则生成可执行文件。所以学习`makefile`，其实就是学习编写`makefile`的语法，以及`make`工具所依赖的规则。

因此，这篇文章也将重心放在，`makefile`的语法，以及其所依赖的规则上。至于编译的过程以及原理，后面有时间会单独写篇文章学习下，这里只是作为补充简单的说下。程序编译首先要把源文件编译成中间文件，这个动作叫编译(`compile`)；之后要把生成的中间文件合成可执行文件，这个动作叫做链接(`link`)。编译时编译器只会检查程序语法是否正确以及函数，变量是否被声明，如果函数未声明，编译器会给出一个警告，但可以正常生成中间文件。也就是说只要语法正确，编译器就可以编译出中间文件。而在链接程序时，链接器是不会管函数所在的源文件的，它只会在所有的中间程序中寻找函数的实现，如果找不到就会报错。



## Makefile介绍

### 为什么要用`makefile`

之前说过，`makefile`只是一个脚本文件，其作用就是告诉`GNU make`之类的工具，你要怎么编译源文件。但是为什么要用`makefile`,我直接写个脚本不就好了吗？比如，你有下面这样一个工程：

```text
-./example0
	-main.c
	-foo.c
	-bar.c
```

这个工程中有三个文件，简单起见，假设每个文件中只有一个函数，而在`main`函数中调用其他两个函数，如下：

```text
---main.c---start
#include<stdio.h>

void main(int argc,char**argv)
{
    foo();
    bar();

    return 0;
}
---main.c---end

---foo.c---start
#include<stdio.h>

void foo()
{
    printf("enter foo\r\n");
}
---foo.c---end

---bar.c---start
#include<stdio.h>

void bar()
{
    printf("enter bar\r\n");
}

---bar.c---end
```

我们先用命令行来编译下：

	gcc -o main main.c bar.c foo.c

执行这条指令，可以看到，虽然我并没有声明`foo()`和`bar()`但编译并没有报错，只是警告我函数未声明。

```text
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
```

执行生成的可执行文件,可以看到结果完全符合预期：

	./main
	enter foo
	enter bar

当然，每次都在命令行上输入`gcc -o main main.c bar.c foo.c`这样一串指令，也太麻烦了吧，不知道那位大神说过，“最美的东西一定是最简单的”。好在，我们可以把它写成脚本,在当前目录下增加`build.sh`,内容如下：

	---build.sh---start
	#!/bin/bash
	gcc -o main main.c bar.c foo.c
	
	---build.sh---end

然后，在命令行中输入`./build.sh`执行脚本，结果如下：

```text
./build.sh
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
```

可以看到结果执行结果和直接在命令行下`gcc`是一样的。但是不知道，你有没有发现一个问题，就是你每次执行一次`build.sh`是不是都会有上面这一堆信息出来，这是编译器给出的调试信息，这说明你每次执行编译脚本，是不是都会编译一次，从调试信息上看的话，至少`main.c`每次都会编译，当然如果你修改下脚本把中间文件也生成出来，你会发现你每编译一次，中间文件都会重新生成。

修改后的`build.sh`:

```text
#!/bin/bash
gcc -c foo.c -o foo.o
gcc -c bar.c -o bar.o
gcc -c main.c -o main.o
gcc -o main main.o bar.o foo.o
```

然后对比下两次编译后,中间文件的时间戳，你会发现确实每次编译，所有文件都会重新编译：

```text
----------------第一次执行----------开始
-rw-rw-rw- 1 ddl ddl  333 Aug 30 00:50 bar.c
-rw-rw-rw- 1 ddl ddl 1536 Aug 30 01:14 bar.o
-rwxrwxrwx 1 ddl ddl  111 Aug 30 01:14 build.sh
-rw-rw-rw- 1 ddl ddl  333 Aug 30 00:40 foo.c
-rw-rw-rw- 1 ddl ddl 1536 Aug 30 01:14 foo.o
-rwxrwxrwx 1 ddl ddl 8416 Aug 30 01:14 main
-rw-rw-rw- 1 ddl ddl  362 Aug 30 00:38 main.c
-rw-rw-rw- 1 ddl ddl 1480 Aug 30 01:14 main.o
----------------第一次执行----------结束

----------------第二执行----------开始
-rw-rw-rw- 1 ddl ddl  333 Aug 30 00:50 bar.c
-rw-rw-rw- 1 ddl ddl 1536 Aug 30 01:16 bar.o
-rwxrwxrwx 1 ddl ddl  111 Aug 30 01:14 build.sh
-rw-rw-rw- 1 ddl ddl  333 Aug 30 00:40 foo.c
-rw-rw-rw- 1 ddl ddl 1536 Aug 30 01:16 foo.o
-rwxrwxrwx 1 ddl ddl 8416 Aug 30 01:16 main
-rw-rw-rw- 1 ddl ddl  362 Aug 30 00:38 main.c
-rw-rw-rw- 1 ddl ddl 1480 Aug 30 01:16 main.o
----------------第二执行----------结束
```

这样是不是很不合理啊！会浪费大量资源和时间，项目小还好，稍微大点的项目有个几千上万源文件，那编译所耗费的时间简直无法忍受(突然想起，以前用过一款模块，那编译速度。。。很感人)。

所以理想情况下应该是，**除了第一次编译，之后每次编译只要编译那些有改动的文件就好了**，而这就是`GNU make`等一系列工具想要实现的核心目标。此外，`makefile`还关系到整个工程的编译规则，因为一个工程中可能有很多源文件，而`makefile`定义了一系列规则来指定，那些文件需要先编译，那些文件需要后编译，那些文件需要重新编译，甚至进行更复杂的操作，比如配合`menuconfig`之类的图形化工具进行功能裁剪啊(这个我也很感兴趣，有时间也要学习学习;-))！而且作为一种脚本语言，`makefile`可以很容易的实现“自动化编译”，一旦写好，只要`make`以下，就会自动编译工程，极大的提高了软件开发效率。



### `makefile`的规则与实现

