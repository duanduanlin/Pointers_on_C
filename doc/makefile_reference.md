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

`makefile`要解决的核心问题是，每次只编译有修改的文件。而要实现这一点最简单的方法是对比编译所生成的中间文件与对应源文件的时间戳，如果发现源文件比较新，那么就要重新编译源文件。

据此，`makefile`定义了一套称为"依赖关系"的规则，如下：

```text
target...:prerequisites...
	command
	...
	...
```

其中`target`是目标文件，可以是中间文件，也可以是执行文件，还可以是一个标签。`prerequisites`是依赖文件，也就是要生成`target`所需要的文件。`command`就是生成规则了(shell 命令)。

简单来说，依赖关系就是说明要要生成目标文件需要那些依赖文件，以及生成规则是怎样的。这样`make`工具在执行的时候，只需判断是不是有依赖文件比目标文件新，如果有的话，执行生成规则。而`makefile`中的依赖关系最终会构成一个依赖树，最终的可执行文件依赖一系列的中间文件，而每个中间文件又依赖于对应的一个或多个源文件。还是以上面那个工程为例，其依赖树如下：

![dependent chain][]

如上图所示，`make`工具在检查可执行文件`main`的依赖时，会找到`foo.o`,`bar.o`和`main.o`三个中间文件，而这三个中间文件又分别有自己的依赖，所以最终会找到依赖树的叶子节点，然后进行时间比较，需要的话执行对应规则更新上一节点，以此类推并最终导致根节点更新。

当然上面这个实例比较简单，只有源文件，而实际工程中会有很多头文件，所以实际的`makefile`规则要复杂些，如下：

1.  如果这个工程没有编译过，那么我们的所有`C`文件都要编译并链接。
2.  如果这个工程的某几个`C`文件被修改，那么我们只编译被修改的`C`文件，并链接目标程序。
3.  如果这个工程的头文件被改变了，那么我们需要编译引用了这几个头文件的`C`文件，并链接程序。

以上就是`makefile`的核心规则了，掌握这些，书写一个能用的`makefile`也就够了，至于其他的像自动推导，伪目标，变量定义等功能都是为了让`make`工具更好用而已。当然了，如果你想更高效的写出稳健的`makefile`，仅仅掌握核心规则是远远不够的。

正所谓万丈高楼拔地起，在正式开始学习之前呢，我们先写个最简单的`makefile`看看效果。在此之前呢，再简单理一下`make`的工作流程。

1.  首先`make`会去查找`makefile`文件，默认是当前目录下的“Makefile”或“makefile”。
2.  如果找到，它会找到文件中的第一个目标文件，并把这个文件作为最终的目标文件。
3.  如果目标文件不存在或者它所依赖的文件比目标新，那么执行对应的规则生成目标。
4.  如果目标文件的依赖文件存在，那么`make`会去寻找依赖文件所依赖的文件，然后进行时间检查，并决定是否执行对应规则更新依赖文件。

下面，我们为之前的`example0`写个简单的`makefile`。



## 参考

[dependent chain]: img/dependent_chain.png