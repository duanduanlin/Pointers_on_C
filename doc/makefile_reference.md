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

简单来说，依赖关系就是说明要要生成目标文件需要那些依赖文件，以及生成规则是怎样的。这样`make`工具在执行的时候，只需判断是不是有依赖文件比目标文件新，如果有的话，执行生成规则。而`makefile`中的依赖关系最终会构成一个依赖树，最终的可执行文件依赖一系列的中间文件，而每个中间文件又依赖于对应的一个或多个源文件。

别的不多说，先写个最简单的实例看看效果，然后我们在逐步完善它。首先我们为之前的`example0`写个简单的`makefile`。

新建目录`example1`,方便起见，这里直接拷贝`example0`,然后稍作修改，如下：

```text
-./example1
	-main.c
	-foo.c
	-bar.c
	-makefile
```

其中，`makefile`内容如下：

```text
main:main.o foo.o bar.o
	gcc -o main main.o foo.o bar.o

main.o:main.c
	gcc -c main.c -o main.o

foo.o:foo.c
	gcc -c foo.c -o foo.o

bar.o:bar.c
	gcc -c bar.c -o bar.o
```

首先我们先`make`下，看看效果,然后在一点一点分析它。

```text
$ make
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

$ ./main
enter foo
enter bar
```

乍看之下，和之前的`example0`的脚本执行的效果差不多嘛！不过是多了一些`gcc`开头的指令而已，但是一旦编译完成，之后在不对源文件做任何修改的情况下，重复执行`make`：

```text
$ make
make: 'main' is up to date.
```

你会发现，不管你怎么执行都好像并没有编译，而是提示你`main`也就是你的可执行程序已经是最新的了。这个好像已经符合我们的目标了，也就是只编译有修改的文件，至于是不是看下时间戳呗！

```text
-------第一次make后，文件信息---------
$ ls -l
total 28
-rw-r--r-- 1 duanduanlin duanduanlin  333 Aug 31 22:31 bar.c
-rw-rw-rw- 1 duanduanlin duanduanlin 1536 Aug 31 23:28 bar.o
-rw-r--r-- 1 duanduanlin duanduanlin  333 Aug 31 22:31 foo.c
-rw-rw-rw- 1 duanduanlin duanduanlin 1536 Aug 31 23:28 foo.o
-rwxrwxrwx 1 duanduanlin duanduanlin 8416 Aug 31 23:28 main
-rw-r--r-- 1 duanduanlin duanduanlin  362 Aug 31 22:31 main.c
-rw-rw-rw- 1 duanduanlin duanduanlin 1480 Aug 31 23:28 main.o
-rw-r--r-- 1 duanduanlin duanduanlin  168 Aug 31 22:31 makefile

---------之后再make时的文件信息-----------
$ ls -l
total 28
-rw-r--r-- 1 duanduanlin duanduanlin  333 Aug 31 22:31 bar.c
-rw-rw-rw- 1 duanduanlin duanduanlin 1536 Aug 31 23:28 bar.o
-rw-r--r-- 1 duanduanlin duanduanlin  333 Aug 31 22:31 foo.c
-rw-rw-rw- 1 duanduanlin duanduanlin 1536 Aug 31 23:28 foo.o
-rwxrwxrwx 1 duanduanlin duanduanlin 8416 Aug 31 23:28 main
-rw-r--r-- 1 duanduanlin duanduanlin  362 Aug 31 22:31 main.c
-rw-rw-rw- 1 duanduanlin duanduanlin 1480 Aug 31 23:28 main.o
-rw-r--r-- 1 duanduanlin duanduanlin  168 Aug 31 22:31 makefile
```

可以看到时间戳，并没有变化，说明目的确实实现了。

下面，我们简单分析下`makefile`。

```text
main:main.o foo.o bar.o
	gcc -o main main.o foo.o bar.o

main.o:main.c
	gcc -c main.c -o main.o

foo.o:foo.c
	gcc -c foo.c -o foo.o

bar.o:bar.c
	gcc -c bar.c -o bar.o
```

如上，可以很清晰的看到，这里总共定义了四个依赖关系，以及对应的生成规则。顺便说下哈！`make`工具会把第一个依赖关系中的第一个目标当作最终目标，也就是这里的可执行程序`main`。根据依赖关系，可以知道，想要生成`main`,需要三个中间文件，`main.o` ,`foo.o`和 `bar.o`(注意这个顺序哦)。而每个`.o`又分别依赖于对应的`.c`文件。所以最终目标的依赖关系树如下：

![chain][]

结合上图，我们先看看在你在命令行输入`make`时，`make`都干了什么。`make`工具首先会从当前文件下查找`makefile`或`Makefile`,如果找到了，就会载入此`makefile`,然后获取文件中的第一个依赖关系的第一个目标，并把它作为最终目标；然后会检查最终可执行`main`是否存在，如果不存在，那么直接执行生成规则，但如果此时`main`所依赖的文件也不存在，那么就要先生成它，而如果此时依赖文件存在，就要先检查依赖文件是不是比源文件要旧，如果是，先更新依赖文件；如果最终目标存在，那么接着依次检查`main`所依赖的文件(这里是`main.o` ,`foo.o`和 `bar.o`)是否存在，同样如果不存在，那么先执行`.o`文件的生成规则生成所需要的依赖文件；如果`.o`文件也存在的话，就会去检查`.o`文件的依赖关系，如果`.o`文件比对应的源文件旧，那么就去执行`.o`文件的生成规则去重新生成`.o`文件；当所有`main`所依赖的文件都检查完后，`make`再去判断是否需要执行`main`的生成规则去更新最终目标文件。

以上就是`make`程序的大致执行流程了，下面我们针对上面的流程做些实验。



#### 实验1-查找`makefile`

我们在`example1`的基础上,建立`example1_test1`,并修改`makefile`文件名为`Makefile`,然后`make`下：

```text
$ make
gcc -c main.c -o main.o
...中间信息省略...
gcc -c foo.c -o foo.o
gcc -c bar.c -o bar.o
gcc -o main main.o foo.o bar.o
```

可以看到，是可以做正常编译的。下面在改成`GNUmakefile`试试，可以发现一样可以。然后在改成`my_makefile`试试：

```text
$ make
make: *** No targets specified and no makefile found.  Stop.
```

可以看到，`make`出错了，而且提示我们没找到`makefile`。下面试试给`make`带个参数，指定要查找的文件,先试下-f：

```text
$ make -f my_makefile
gcc -c main.c -o main.o
...中间信息省略...
gcc -c foo.c -o foo.o
gcc -c bar.c -o bar.o
gcc -o main main.o foo.o bar.o
```

没毛病，此外输入`make --file=my_makefile`可以实现一样的效果。

结论：

1.  执行`make`命令时，`make`工具会默认查找当前目录下的`makefile`或`Makefile`或`GNUmakefile`文件。
2.  可以通过设置,-f或--file=来指定要查找的`makefile`文件。



#### 实验2-`make`的最终目标

同样拷贝`example1`到`example1_test2`,并修改`makefile`文件，如下：

```text
foo.o:foo.c
	gcc -c foo.c -o foo.o

main:main.o foo.o bar.o
	gcc -o main main.o foo.o bar.o

main.o:main.c
	gcc -c main.c -o main.o

bar.o:bar.c
	gcc -c bar.c -o bar.o
```

然后`make`下:

```text
$ make
gcc -c foo.c -o foo.o
```

可以看到，`make`仅仅只编译生成`foo.o`结束了,再试试首个依赖有多个目标的会怎样？修改`makefile`如下：

```text
first foo.o:foo.c
	gcc -c foo.c -o $@

main:main.o foo.o bar.o
	gcc -o main main.o foo.o bar.o

main.o:main.c
	gcc -c main.c -o main.o

bar.o:bar.c
	gcc -c bar.c -o bar.o
```

为了更好的演示多目标的效果，上面第二行，我使用了自动变量`$@`,自动变量会在后面讲，这里可以简单的把第一个依赖关系用下面的关系替换：

```text
first:foo.c
	gcc -c foo.c -o first
	
foo.o:foo.c
	gcc -c foo.c -o foo.o
```

然后执行`make`,记得先把之前生成的`.o`文件删掉：

```test
$ make
gcc -c foo.c -o first

$ ls
bar.c  first  foo.c  main.c  makefile
```

你会发现，`make`在生成`first`就结束了。这是因为`make`把`first`当作最终目标，而一旦最终目标生成，`make`也就结束了。

结论：

1.  `make`会把`makefile`文件中的第一个依赖关系的第一个目标作为最终目标。
2.  一旦最终目标生成，`make`就结束了。



#### 实验3-最终目标的生成过程

拷贝`example1`到`example1_test3`。`makefile`如下：

```text
main:main.o foo.o bar.o
	gcc -o main main.o foo.o bar.o

main.o:main.c
	gcc -c main.c -o main.o

foo.o:foo.c
	gcc -c foo.c -o foo.o

bar.o:bar.c
	gcc -c bar.c -o bar.o
```

先`make`下：

```text
$ make
gcc -c main.c -o main.o
...中间信息省略...
gcc -c foo.c -o foo.o
gcc -c bar.c -o bar.o
gcc -o main main.o foo.o bar.o
```

删除最终目标`main`，然后再`make`试试：

	$ make
	gcc -o main main.o foo.o bar.o
可以看到，`make`直接链接生成目标文件了。这是符合预期的，因为最终目标不存在，且中间目标不但存在而且不用更新。

然后删除`foo.o`,再`make`试试,此时预计会先生成`foo.o`,然后链接程序:

```text
$ make
gcc -c foo.c -o foo.o
gcc -o main main.o foo.o bar.o
```

可以看到，结果完全符合预期。这时，最终目标存在，而且仅有一个中间目标需要重新生成(因为不存在)，所以只需先生成缺少的中间目标，之后因为有一个中间目标比最终目标新，导致最终目标也会重新生成。

下面修改`foo.c`内容如下：

```text
#include<stdio.h>

void foo()
{
    printf("enter foo\r\n");
    printf("leave foo\r\n");
}
```

然后编译并执行：

```text
$ make
gcc -c foo.c -o foo.o
gcc -o main main.o foo.o bar.o

$ ./main
enter foo
leave foo
enter bar
```

此时，最终文件存在和中间文件都存在，且只有一个中间文件比对应源文件旧，所以只需更新此中间文件，然后链接程序即可。

还记得，首次编译时，中间文件的生成顺序吗？是不是和目标文件的依赖顺序很像。下面修改下依赖顺序试试：

```text
main:foo.o bar.o main.o
	gcc -o main main.o foo.o bar.o

main.o:main.c
	gcc -c main.c -o main.o

foo.o:foo.c
	gcc -c foo.c -o foo.o

bar.o:bar.c
	gcc -c bar.c -o bar.o
```

编译结果如下：

```text
$ make
gcc -c foo.c -o foo.o
gcc -c bar.c -o bar.o
gcc -c main.c -o main.o
...中间信息省略...
gcc -o main main.o foo.o bar.o
```

如上，反复实验后可以看到，中间程序的生成顺序是和最终文件的依赖顺序是一致的。这说明，**`makefile`的执行顺序是基于依赖链顺序的**。这个很重要，因为这样的话，就可以通过修改依赖链顺序来决定源文件编译的顺序了。

总结：

1.  `make`在生成目标文件时，会首先检查最终文件是否存在
2.  如果最终目标不存在，那么就去检查依赖是不是要更新(依赖文件不存在，或者依赖文件比较旧)，然后决定是直接使用现有依赖文件生成最终文件，还是更新依赖文件后在生成最终文件。
3.  如果目标文件存在，一样要去检查依赖文件是不是要更新，之后再判断是不是要重新生成目标文件。
4.  `makefile`的执行顺序是基于依赖链顺序的。



目前为止，我们已经对``makefile`的依赖关系有了个初步的了解，而这正是`makefile`的核心，其他的规则和语法都不过是对此规则的扩充，目的只是为了让`make`工具更好用而已。而且值得注意的是`make`工具并不会管你的生成规则是怎样的，它只会关注目标文件和依赖之间的新旧关系，并决定是不是要执行规则，所以`make`不仅仅只是`C`的编译工具，而是更像一种代码编译通用的方法，可以很容易的套用到其他语言上。所以，`make`工具研究一个也就够了，一通百通嘛！

与此同时，我们对`make`工具的执行流程也有了个大致的了解，这是`make`工具的主线。要知道不管`makefile`写的有多复杂，主线是基本不变的。

后面，我会从最基本的`makefile`开始，一点一点的完善它的功能，在此过程中会穿插`makefile`的常用规则和语法，并最终给出一个比较专业的`makefile`实例。



#### 补充材料

因为我也是新手，很多东西都不懂，所以在学习过程中遇到些新的概念什么的，如果内容比较多的话，我会专门抽出时间单独为其写篇文章。这是第一篇，[补充材料之`make`选项][make-options]。



## `makefile`编写方法

下面我们按着计划，一步一步的完善我们的`makefile`。在开始之前呢！我们再看下关于依赖的定义：

```text
target...:prerequisites...
	command
	...
	...
```

我们知道`make`执行时都是要检查目标依赖，然后决定是不是要执行指令。那么如果目标压根就没有依赖呢？我们先看个实例，然后对其执行过程进行分析。

我们新建一个目录`example3`，然后在里面增加一个`makefile`，内容如下:

```text
$ cat makefile

run:
	@echo runing
```

可以看到，这里我们有一个目标`run`，其依赖为空，而且`run`是我们的最终目标。

我们先`make`看看：

```text
$ make
runing
```

可以看到，我们每次`make`都会打印`runing`，也就是`make`总是执行目标`run`的指令。

然后我们加上`--debug=v`看下发生了什么：

```text
Reading makefiles...
Reading makefile 'makefile'...
Updating goal targets....
Considering target file 'run'.
 File 'run' does not exist.
 Finished prerequisites of target file 'run'.
Must remake target 'run'.
runing
Successfully remade target file 'run'.
```

通过输出信息，我们可以看到，`make`是先检查最终目标是不是存在，然后发现目标不存在，这个时候就已经可以判定命令是要执行了。之后继续检查所需依赖要不要更新，然后发现压根就没有依赖，所以`make`直接就执行命令了。

然后，我们在目录下新建`run`，然后再试试：

```text
$ make --debug=v
Reading makefiles...
Reading makefile 'makefile'...
Updating goal targets....
Considering target file 'run'.
 Finished prerequisites of target file 'run'.
No need to remake target 'run'.
make: 'run' is up to date.
```

和前面那个类似，这里也是先检查`run`是不是存在，不过和之前不一样的是，这里发现我最终目标已经存在了，然后`make`就要去检查其依赖是不是要更新，然后发现没有依赖，所以就不用更新了。

通过前面这两个例子，我们可以看到，对于依赖为空的目标来说，如果目标本身不存在，那么`make`在检查目标时一定会执行其指令，而如果目标存在，则一定不会执行指令。利用这个再加上`make`本身支持的指定目标，我们可以让`make`去执行一些特定功能，就像脚本一样。



### 增加清理功能

下面我们为`makefile`增加清除功能。来删除中间文件，为了演示这个，我们从`example2`拷贝一份到`example3_test_clean`,并修改`makefile`，在末尾添加如下内容:

```text

clean:
	rm *.o main
```

可以看到，这里我们添加了一个依赖为空的目标`clean`，而如果`clean`不存在的情况下，每次`make`去检查`clean`总是会执行其指令。即删除当前目录下`main`和所有`.o`文件。

下面我们试试效果如何,首先先编译下。

```text
$ ls
bar.c  bar.o  foo.c  foo.o  main  main.c  main.o  makefile
```

你会看到目录下多了一些文件，然后`make clean`。

```text
$ make clean
rm *.o main

$ ls
bar.c  foo.c  main.c  makefile
```

你会看到，当你指定`make`的目标为`clean`时，就可以执行`clean`下的指令了，而且当前目录下的相关文件也确实删掉了。

但是这样的话，有一个前提条件是`makefile`的同级目录下，不能有和`clean`重名的文件(文件夹也不行，因为`make`并不会管你目标是什么)。

好在，针对这种情况`make`工具专门提供了规则来规避目标检查，那就是伪目标，规则如下：

```
.PHONY:target
```

可以看到伪目标其实就是一个依赖关系，其目标是`make`内置目标`.PHONY`，依赖就是你想要设置的目标。这样`make`在执行时就会知道这个目标是伪目标从而忽略对其的检查，直接执行其指令。

我们先把`clean`变成伪目标。

```text
.PHONY:clean
clean:
	rm *.o main
```

然后加上`--debug=v`，看看`make`执行时干了什么：

```text
$ make --debug=v clean
Reading makefiles...
Reading makefile 'makefile'...
Updating goal targets....
Considering target file 'clean'.
 File 'clean' does not exist.
 Finished prerequisites of target file 'clean'.
Must remake target 'clean'.
rm *.o main
Successfully remade target file 'clean'.
```

好像看不出来，我们在当前目录下新建个`clean`文件试试。

```text
$ make --debug=v clean
Reading makefiles...
Reading makefile 'makefile'...
Updating goal targets....
Considering target file 'clean'.
 File 'clean' does not exist.
 Finished prerequisites of target file 'clean'.
Must remake target 'clean'.
rm *.o main
Successfully remade target file 'clean'.
```

可以看到，虽然目录下已经有了`clean`，但`make`还是提示说`clean`不存在。这就是伪目标的作用了，就是告诉`make`这不是个真正的目标，你不用检查它，直接执行指令就好了。

到此我们为`makefile`增加了`clean`功能，但是考虑一下，其实有时候我们并不希望删除我们的可执行程序，而是像保留它，仅仅把比较杂乱的中间文件删掉。

```text
.PHONY:cleanall cleanobj

cleanall:
	rm *.o main

cleanobj:
	rm *.o
```

要实现有选择的删除某些文件，你可以像上面这样，定义两个伪目标，分别执行你要删除的操作。但是你会发现，这两个命令之间其实是有重合的。所以我们可以利用依赖检查可能会更新依赖，把重合的部分抽出来作为一个伪目标的依赖。这样每当伪目标执行时，都会去检查依赖，如果此时依赖也是伪目标的话，那不就可以执行重合的功能了吗，然后因为目标本身也是伪目标，所以同时也会执行目标本身的指令。就像下面这样。

```text
.PHONY:cleanall cleanobj

cleanall:cleanobj
	rm main

cleanobj:
	rm *.o
```

现在，我们的清除功能是比较完善了。但是考虑这样一种情况，如果我在执行`cleanall`之前，先执行了`cleanobj`会怎样？

```text
$ make cleanall
rm *.o
rm: cannot remove '*.o': No such file or directory
makefile:19: recipe for target 'cleanobj' failed
make: *** [cleanobj] Error 1
```

你会发现，因为`.o`文件已经不存在了，这是你去执行`cleanall`它会因找不到`.o`而报错。而此时`rm main`是没有执行的。也就是我们想删除的东西其实是没删完的。

还记得`make`选项里的`-k`选项吗？它会告诉`make`忽略错误继续执行，我们试试这里可不可以。

```text
$ make -k cleanall
rm *.o
rm: cannot remove '*.o': No such file or directory
makefile:19: recipe for target 'cleanobj' failed
make: *** [cleanobj] Error 1
make: Target 'cleanall' not remade because of errors.
```

我们发现并不行，这是因为如果中间出错，最终目标是不会更新的，这里也就意味着`cleanall`的命令并不会执行。其实这里我们可以直接在`rm`前加个`-`表示忽略命令执行错误。

```text
.PHONY:cleanall cleanobj

cleanall:cleanobj
	-rm main

cleanobj:
	-rm *.o
```

效果如下：

```text
$ make cleanall
rm *.o
rm: cannot remove '*.o': No such file or directory
makefile:19: recipe for target 'cleanobj' failed
make: [cleanobj] Error 1 (ignored)
rm main
```

嗯，现在我们的删除功能就比较完善了。



#### 补充资料

嗯又到了该充电的时候，上面有学习到伪目标(`.PHONY`),其实它是`make`的内置目标。下面我们就看看`make`有那些内置目标，以及都有什么用吧！这是第二篇，[补充资料-`make`内置目标][builtin-targets]。



### 分离源码和中间文件

现在考虑这个问题，就是如果我文件比较多的话，最好把生成的中间文件单独放起来，不然混在一起的话会非常混乱。下面看看该怎么做。

首先还是先从`example3_test_clean`拷贝一份到`example3_test_detach`，然后我们新建一个`obj`目录来存放中间文件。下面考虑该怎么把所有`*.o`都放到`obj`目录中去，很容易想到的办法就是使用`.o`时连带着路径呗！就像下面这样：

```test
$ cat makefile
main:obj/main.o obj/foo.o obj/bar.o
	gcc -o main obj/main.o obj/foo.o obj/bar.o

obj/main.o:main.c
	gcc -c main.c -o obj/main.o

obj/foo.o:foo.c
	gcc -c foo.c -o obj/foo.o

obj/bar.o:bar.c
	gcc -c bar.c -o obj/bar.o

.PHONY:cleanall cleanobj

cleanall:cleanobj
	-rm main

cleanobj:
	-rm obj/*.o
```

然后编译下，发现确实可以。但是这也太麻烦了。

## 参考

[chain]: img/dependent_chain.png

[make-options]: makefile_make-options.md

[builtin-targets]: makefile_builtin-targets

