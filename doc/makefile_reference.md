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



## `makefile`编写方法

这篇文章的目的是为了记录我在学习`makefile`过程中所积累的东西，最终目标是为了给出一个比较专业的`makefile`模板，以便在其他项目中使用。所以计划是从最简单的`makefile`开始，逐步完善，并在这个过程中对一些新的东西进行展开讨论。所以，这节内容开始前，先把上节接触到的新东西补充下。



### 补充材料-`make`选项

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



下面测试下以上参数，首先拷贝`example1`到`example2`。

先看下`make`版本：

```text
$ make -v
GNU Make 4.1
Built for x86_64-pc-linux-gnu
Copyright (C) 1988-2014 Free Software Foundation, Inc.
License GPLv3+: GNU GPL version 3 or later <http://gnu.org/licenses/gpl.html>
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.
```

可以看到，我这里的版本是`4.1`的.下面试下`-b`或`-m`:

```text
$ make -b
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

好像看不出什么区别。下面试试`-B`：

```text
$ make -B
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

可以看到，不管我之前是不是编译过了，`make -B`仍然会重新编译。

下面，退到`example2`的上级目录，试下`-C`:

```text
$ make -C example2
make: Entering directory '/home/duanduanlin/workspace/Pointers_on_C/doc/makefile_example/example2'
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
make: Leaving directory '/home/duanduanlin/workspace/Pointers_on_C/doc/makefile_example/example2'
```

是不是和直接在`example2`目录下执行`make`没什么区别，而且`make`会在执行前后告诉我要进入`XXX`目录以及离开`XXX`目录了。这个一般用在在一个`makefile`中执行另一个`makefile`(后期补充：而且是临时切换`make`的当前目录到指定目录)。

下面看看`--debug[=]`，先试下默认的：

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

这个就比较有意思了，你会看到`make`的执行流程是不是就在调试信息里啊(看到这里，不知道你有没有一种想看到`GNU make`源码的冲动，就是想看看是不是和我想的一样)！为了加深下印象，我再把上一节的实验3，在加上`--debug`:

最终目标不存在：

```text
$ rm main
$ make --debug
...
Reading makefiles...
Updating goal targets....
 File 'main' does not exist.
Must remake target 'main'.
gcc -o main main.o foo.o bar.o
Successfully remade target file 'main'.
```

某个依赖文件不存在：

```text
$ rm foo.o
$ make --debug
...
Reading makefiles...
Updating goal targets....
   File 'foo.o' does not exist.
  Must remake target 'foo.o'.
gcc -c foo.c -o foo.o
  Successfully remade target file 'foo.o'.
 Prerequisite 'foo.o' is newer than target 'main'.
Must remake target 'main'.
gcc -o main main.o foo.o bar.o
Successfully remade target file 'main'.
```

某个依赖需要更新:

```text
$ touch foo.c
$ make --debug
...
Reading makefiles...
Updating goal targets....
   Prerequisite 'foo.c' is newer than target 'foo.o'.
  Must remake target 'foo.o'.
gcc -c foo.c -o foo.o
  Successfully remade target file 'foo.o'.
 Prerequisite 'foo.o' is newer than target 'main'.
Must remake target 'main'.
gcc -o main main.o foo.o bar.o
Successfully remade target file 'main'.
```

下面试试，`--debug=a`,(⊙﹏⊙)结果我就不全列出来了，因为实在是太多了。下面就简单列下一些关键性的信息吧！

```text
Reading makefiles...
Reading makefile 'makefile'...
Updating makefiles....
 Considering target file 'makefile'.
  Looking for a XXX
  ...
 No need to remake target 'makefile'.
```

这是第一块信息，是在导入和读取`makefile`。看信息的话，感觉`make`是把`makefile`也当作了一个目标，然后去查找其上的规则，虽然不知道，它是怎么做的，但从信息上看到了隐式规则和静态规则。

`makefile`读取完成后，会先获取最终目标：

```text
Updating goal targets....
...
Successfully remade target file 'main'.
```

之后,`make`会把最终目标当作执行目标，去查找规则和判断是不是要更新最终目标，直到最终得到最终目标，退出程序。

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
...
Successfully remade target file 'main.o'.
```

这个地方，可以明显看到`make`在生成目标文件时的执行流程，是严格按着依赖链去检查，直到找到叶子节点，并判断是不是有更新，然后执行更新，之后逐级返回,直到生成目标。

之后就是重复上面目标生成的流程了,直到最终目标的所有依赖全部检查完毕：

```text
  Considering target file 'foo.o'.
  ...
  Successfully remade target file 'foo.o'.
  Considering target file 'bar.o'.
  ...
  Successfully remade target file 'bar.o'.
 Finished prerequisites of target file 'main'.
```

最后，再判断是不是要重新生成最终目标:

```text
Must remake target 'main'.
gcc -o main main.o foo.o bar.o
Successfully remade target file 'main'.
```

好了，`--debug=a`基本就这些了，看完后是不是感觉对`make`工具又有了更深的了解呢！反正我看完后有种强烈的冲动想要看看源码去验证是不是和自己想的一样(我也是第一次看哦)。

下面继续，试试`--debug=b`，可以看到这就是`--debug`的功能，就不列出来了。下面看下`--debug=v`:

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

这个应该是比较全面，同时也比较整洁的`make`执行流程了(看到这里，感觉自己对程序的调试信息有了些新的感悟，一个设计良好的调试信息，真的是一种艺术啊)。

继续，下面试试`--debug=i`,(⊙﹏⊙)让我想起了被`--debug=a`支配的恐惧。不过看起来，`make`在检查目标要不要更新前，是先查找并替特殊规则啊！后面讲到这些特殊规则时可以看看，`make`具体都做了什么。

好，继续，`--debug=j`:

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

╰(*°▽°*)╯，好像发现了什么不得了的东西，`make`是开了多线程来执行目标的生成规则的。我记得好像有个`-j`的参数可以指定同时运行多少个规则，后面可以试试。

最后一个`--debug=m`,好像看不出来有什么变化，这样我们引用别的`makefile`试试，先在上级目录从`expamle2`拷贝一份到`example2_test0`,然后在`example2_test0`目录下增加`obj.mk`,内容如下：

```text
main.o:main.c
	gcc -c main.c -o main.o

foo.o:foo.c
	gcc -c foo.c -o foo.o

bar.o:bar.c
	gcc -c bar.c -o bar.o
```

然后修改`makefile`:

```text
main:main.o foo.o bar.o
	gcc -o main main.o foo.o bar.o

include ./obj.mk
```

还是看不出来(╯＾╰),我还就不信了┗|｀O′|┛。`make -d`试下：

```
Reading makefiles...
Reading makefile 'makefile'...
Reading makefile 'obj.mk' (search path) (no ~ expansion)...
Updating makefiles....
```

终于看到了，(^o^)哇(^0^)哈(^○^)哈~~~

但是`--debug=m`现象不是很明显，试试在`makefile`中执行`makefile`会怎样？先修改`makefile`:

```text
$ cat makefile

build:
	$(MAKE) -f build.mk
```

修改，`obj.mk`为`build.mk`并修改其内容：

```text
$ cat build.mk

main:main.o foo.o bar.o
	gcc -o main main.o foo.o bar.o

main.o:main.c
	gcc -c main.c -o main.o

foo.o:foo.c
	gcc -c foo.c -o foo.o

bar.o:bar.c
	gcc -c bar.c -o bar.o
```

再试试，(ˉ▽ˉ；)...还是看不出来，不过我又详细对比了，`--debug=m`和`--debug`,发现区别只是多了一行:`Updating makefiles....`，难道这就是`--debug=m`的效果吗？

好吧！下面，看看`-e`的效果。

新建`example2_test_e`,并增加`test_e.mk`，`test_export.mk`和`makefile`，其中`makefile`内容如下：

```text

param = hello

export param

run:
	@echo "in makefile param = "$(param)
    $(MAKE) -f test_export.mk
    $(MAKE) -f test_e.mk
```

简单讲下这里干了什么哈！主要是为小白准备的，因为好多内容都没讲到。本来是准备把`-e`放到参数传递时讲的，后来想了想补充材料挺多的，所以就想单独抽出来作为一篇文章，考虑到文章的完整性，还是决定放在这里吧！

好吧，言归正传，这里只做了三件事吧！第一行是定义了一个变量叫`param`其值为`hello`;第三行，是导出`param`到环境变量，这样其他的`makefile`可以访问到它(类似于`C`中的全局变量)；第五行定义了一个特殊的依赖关系，也就是目标的依赖为空，而且整个`makefile`只有这一个依赖关系，所以它是个最终目标。通过之前关于`make`执行流程，我们知道`make`执行时，会先检查最终目标存不存在，这里也一样，不过因为其依赖为空，所以有些特殊。我们来看下，如果`run`不存在，会怎样？最终目标不存在，`make`会直接执行生成规则，并在执行前检查依赖是不是要更新，但这里没有依赖，所以直接执行规则。如果`run`存在，那么,`make`会去检查依赖是不是要更新，同样因为没有依赖，所以啥也不干。所以总结起来就是，**目标的依赖为空时，且目标作为`make`的目标，如果此时目标不存在，那么必然要执行其生成规则，反之，必然不会执行规则**。后面第六行，可以不用管他的细节，只要知道它会输出一段文字和`param`的值到控制台。

那么这里就很清晰了，当你在命令行下输入`make`时，`make`也做了三件时，第一声明一个变量并导出到环境变量，之后执行`run`的生成规则，也就是输出一段文字，然后执行`test_e.mk`和`test_export.mk`。

然后我们在看看，`test_e.mk`里有些什么：

```text

param = world

test:
	@echo "in test.mk param = "$(param)
```

`test_e.mk`主要做了两件事，第一声明一个变量，也叫`param`,不过其值为`world`。然后简单输出一段包含`param`值的文字。

然后是`test_export.mk`：

```text
test_export:
	@echo "param in test_export.mk = "&(param)
```

和之前`test_e.mk`类似，不过这里没有定义变量。

好，我们先`make`下，看看效果怎样：

```text
$ make
in makefile param = hello
make -f test_export.mk
make[1]: Entering directory '/home/duanduanlin/workspace/Pointers_on_C/doc/makefile_example/example2_test_e'
param in test_export.mk = hello
make[1]: Leaving directory '/home/duanduanlin/workspace/Pointers_on_C/doc/makefile_example/example2_test_e'
make -f test_e.mk
make[1]: Entering directory '/home/duanduanlin/workspace/Pointers_on_C/doc/makefile_example/example2_test_e'
in test.mk param = world
make[1]: Leaving directory '/home/duanduanlin/workspace/Pointers_on_C/doc/makefile_example/example2_test_e'
```

可以看到，首先，我们在`makefile`中定义了一个变量`param`值为`hello`,并把它导出，然后我们在`test_export.mk`中直接引用`param`可以看到，其值还是`hello`。最后，我们在`test_e.mk`也定义了一个`param`，其值为`world`。尽管此时全局的值为`hello`，但默认下`make`并不会用环境变量的值覆盖`makefile`中的变量。如果想要使用环境变量覆盖`makefile`中的变量的话，可以使用`-e`,如下：

```text
$ make -e
in makefile param = hello
make -f test_export.mk
make[1]: Entering directory '/home/duanduanlin/workspace/Pointers_on_C/doc/makefile_example/example2_test_e'
param in test_export.mk = hello
make[1]: Leaving directory '/home/duanduanlin/workspace/Pointers_on_C/doc/makefile_example/example2_test_e'
make -f test_e.mk
make[1]: Entering directory '/home/duanduanlin/workspace/Pointers_on_C/doc/makefile_example/example2_test_e'
in test.mk param = hello
make[1]: Leaving directory '/home/duanduanlin/workspace/Pointers_on_C/doc/makefile_example/example2_test_e'
```



因为前面`-f`讲过好多次了，这里只是试试参数的传递：

```text
$ make -f=build.mk
make: =build.mk: No such file or directory
make: *** No rule to make target '=build.mk'.  Stop.

$ make -f= build.mk
make: =: No such file or directory
make: *** No rule to make target '='.  Stop.

$ make -fbuild.mk
make: 'main' is up to date.

$ make -f   build.mk
make: 'main' is up to date.

$ make -f   build.mk test.mk
make: Nothing to be done for 'test.mk'.
```

可以看到传参时，短式写法`-f`是把字符`f`后的字符去掉前置空格，当作参数，也试了传递多个参数，不过没有成功，应该是只能传入一个`makefile`。

```text
$ make --file=build.mk
make: 'main' is up to date.

$ make --file build.mk
make: 'main' is up to date.

$ make --filebuild.mk
make: unrecognized option '--filebuild.mk'
```

而长式写法`--file`则需要一个分隔符，可以是`=`也可以是空格。

为了统一起见，避免混乱，不管长式还是短式写法，一律用空格分隔参数。

下面试试`-h`,可以看到比较全的选项信息:

```text
$ make -h
Usage: make [options] [target] ...
Options:
  -b, -m                      Ignored for compatibility.
  -B, --always-make           Unconditionally make all targets.
  -C DIRECTORY, --directory=DIRECTORY
                              Change to DIRECTORY before doing anything.
  -d                          Print lots of debugging information.
  --debug[=FLAGS]             Print various types of debugging information.
  -e, --environment-overrides
                              Environment variables override makefiles.
  --eval=STRING               Evaluate STRING as a makefile statement.
  -f FILE, --file=FILE, --makefile=FILE
                              Read FILE as a makefile.
  -h, --help                  Print this message and exit.
  -i, --ignore-errors         Ignore errors from recipes.
  -I DIRECTORY, --include-dir=DIRECTORY
                              Search DIRECTORY for included makefiles.
  -j [N], --jobs[=N]          Allow N jobs at once; infinite jobs with no arg.
  -k, --keep-going            Keep going when some targets can't be made.
  -l [N], --load-average[=N], --max-load[=N]
                              Don't start multiple jobs unless load is below N.
  -L, --check-symlink-times   Use the latest mtime between symlinks and target.
  -n, --just-print, --dry-run, --recon
                              Don't actually run any recipe; just print them.
  -o FILE, --old-file=FILE, --assume-old=FILE
                              Consider FILE to be very old and don't remake it.
  -O[TYPE], --output-sync[=TYPE]
                              Synchronize output of parallel jobs by TYPE.
  -p, --print-data-base       Print make's internal database.
  -q, --question              Run no recipe; exit status says if up to date.
  -r, --no-builtin-rules      Disable the built-in implicit rules.
  -R, --no-builtin-variables  Disable the built-in variable settings.
  -s, --silent, --quiet       Don't echo recipes.
  -S, --no-keep-going, --stop
                              Turns off -k.
  -t, --touch                 Touch targets instead of remaking them.
  --trace                     Print tracing information.
  -v, --version               Print the version number of make and exit.
  -w, --print-directory       Print the current directory.
  --no-print-directory        Turn off -w, even if it was turned on implicitly.
  -W FILE, --what-if=FILE, --new-file=FILE, --assume-new=FILE
                              Consider FILE to be infinitely new.
  --warn-undefined-variables  Warn when an undefined variable is referenced.

This program built for x86_64-pc-linux-gnu
Report bugs to <bug-make@gnu.org>
```

下面试下`-i`选项(方便起见，回到`example2`目录下)：

```text
$ make -i
gcc -c main.c -o main.o
...
gcc -c foo.c -o foo.o
gcc -c bar.c -o bar.o
gcc -o main main.o foo.o bar.o
```

好像看不到什么区别，我们先改下`makefile`,让`make`出错试下。

把下面这两行：

```text
main:main.o foo.o bar.o
	gcc -o main main.o foo.o bar.o
```

改成：

```text
main:main.o fo.o bar.o
	gcc -o main main.o fo.o bar.o
```

我们把`main`所依赖的`foo.o`改成`fo.o`，这样就会找不到依赖而出错。先`make --debug=v`看下，这样对`make`的执行流程有什么影响：

```text
Considering target file 'fo.o'.
   File 'fo.o' does not exist.
   Finished prerequisites of target file 'fo.o'.
  Must remake target 'fo.o'.
make: *** No rule to make target 'fo.o', needed by 'main'.  Stop.
```

可以看到，`make`在检查`fo.o`的时候，发现`fo.o`并不存在，所以尝试生成它，但又没有找到`fo.o`的生成方法，所以出错了。下面我们再试试从别处拷贝一个`fo.o`试试：

```text
 Considering target file 'fo.o'.
   Finished prerequisites of target file 'fo.o'.
  No need to remake target 'fo.o'.
```

可以看到，已经可以正常编译了。而且从调试信息上，可以看到，`make`在找到`fo.o`之后就不在往下找了，因为`fo.o`并没有依赖与谁。从这里可以看到，`make`真的不会管目标是什么，目标是怎么得到的，它只会关注，能不能找到目标，有没有依赖(也就是没找到怎么办？)，如果有依赖的话，要不要更新。而这里，之前是没找到目标`fo.o`,而且也没给出目标依赖谁，所以`make`就不知道该怎么办了，现在是找到目标了，而且目标没有依赖，那就可以结束了啊。

好，下面，我们继续测试`-i`,给`fo.o`改个名字吧，先别删，以防后面会用，就叫`fo.obj`吧！然后`make -i`试下：

```text
$ make -i
gcc -c main.c -o main.o
main.c: In function ‘main’:
...
make: *** No rule to make target 'fo.o', needed by 'main'.  Stop.
```

可以看到，并不是我们想看到的结果，`make`并没有继续，而是里面停止了。这说明`-i`不是忽略`makefile`本身的错误。

下面看看是不是执行指令的错误，修改`makefile`(记得先复原下`makefile`)。

把下面这行

```text
	gcc -c main.c -0 main.o
```

改成:

```text
	gcc -c main.c -0 main.o
```

这样，`gcc`执行时，会有参数错误。

还是先`--debug=v`下：

```text
gcc -c main.c -0 main.o
gcc: error: main.o: No such file or directory
gcc: error: unrecognized command line option ‘-0’
makefile:5: recipe for target 'main.o' failed
make: *** [main.o] Error 1
```

可以看到，`make`在执行生成目标文件时出错了。此时`make`停止运行，并告诉你我是在`[main.o]`这个目标里出错的。

下面试试，`make -i`会怎样：

```text
$ make -i
gcc -c main.c -0 main.o
gcc: error: main.o: No such file or directory
gcc: error: unrecognized command line option ‘-0’
makefile:5: recipe for target 'main.o' failed
make: [main.o] Error 1 (ignored)
gcc -c foo.c -o foo.o
gcc -c bar.c -o bar.o
gcc -o main main.o foo.o bar.o
gcc: error: main.o: No such file or directory
makefile:2: recipe for target 'main' failed
make: [main] Error 1 (ignored)
```

(●ˇ∀ˇ●)可以看到，`make`发现了两处错误，第一处是在执行`main.o`生成规则时出错，第二处是在最终目标生成时出错。此外`make`是正常执行完了，而且可以看下当前目录下的文件：

```text
$ ls
bar.c  bar.o  fo.obj  foo.c  foo.o  main.c  makefile
```

发现除了，`main.o`和`main`其他中间文件是不是都正常生成了。这说明，当使用`-i`选项时，`make`会忽略的是执行生成`xxx`的指令时的错误，继续执行，但依赖于`XXX`的目标并不会被更新。

下面看看，`-I`,方便起见，还是先从`example2`拷贝到`example2_test_I`。

然后改下`example2_test_I`的目录结构，如下：

```text
./example2_test_I
	-src
		-main.c
		-foo.c
		-bar.c
	makefile
```

老规矩先`make --debug=v`下看看,是在哪里出错的：

```text
Considering target file 'main'.
 File 'main' does not exist.
  Considering target file 'main.o'.
   File 'main.o' does not exist.
    Considering target file 'main.c'.
     File 'main.c' does not exist.
     Finished prerequisites of target file 'main.c'.
    Must remake target 'main.c'.
make: *** No rule to make target 'main.c', needed by 'main.o'.  Stop.
```

可以看到，`make`在查找`main.c`的时候，没有找到，之后又找了下`main.c`的依赖也没找到，就出错了。现在是不是感觉`make`的查找规则更清晰了呢？总结下哈！

1.  `make`在检查目标时，总是先看看目标存不存在，如果不存在的话，就去看看有没有依赖可以生成目标，如何目标不存在有没有依赖，那对不起，我不玩了。如果目标不催在但有办法可以生成目标，那么直接生成目标。
2.  如果目标存在，那就去看看目标有没有依赖，如果有，那就检查下要不要更新依赖，如果没有依赖，那更好，直接返回。

好，言归正传，看看增加`-I`时，发生了什么：

```text
$ make -I src
make: *** No rule to make target 'main.c', needed by 'main.o'.  Stop.
```

(⊙﹏⊙)好像和我想得不一样。。。

`-I`是指定`makefile`的目录，好那就再增加一个`src/src.mk`，内容如下：

```text
main:main.o foo.o bar.o
	gcc -o main main.o foo.o bar.o

main.o:main.c
	gcc -c main.c -0 main.o

foo.o:foo.c
	gcc -c foo.c -o foo.o

bar.o:bar.c
	gcc -c bar.c -o bar.o
```

然后修改下`./makefile`:

```text

include src.mk
```

然后先`make --debug=v`试下：

```text
Reading makefiles...
Reading makefile 'makefile'...
Reading makefile 'src.mk' (search path) (no ~ expansion)...
makefile:2: src.mk: No such file or directory
make: *** No rule to make target 'src.mk'.  Stop.
```

可以看到，出错了。另外需要注意的是，`make`是在尝试生成`src.mk`的时候，发现没有方法可用才出错的。这说明什么呢？这说明`make`是支持自动生成`.mk`的(是不是很激动呢？对于我这种懒癌晚期的人来说，如果能自动生成，那真是太棒了！)。这个会在后面自动生成依赖时讲(没错，你没有看错，依赖是可以自动生成的)。

好的，我们回来，(ಥ _ ಥ)终于回来了。

```text
Reading makefiles...
Reading makefile 'makefile'...
Reading makefile 'src.mk' (search path) (no ~ expansion)...
Updating goal targets....
Considering target file 'main'.
 File 'main' does not exist.
  Considering target file 'main.o'.
   File 'main.o' does not exist.
    Considering target file 'main.c'.
     File 'main.c' does not exist.
     Finished prerequisites of target file 'main.c'.
    Must remake target 'main.c'.
make: *** No rule to make target 'main.c', needed by 'main.o'.  Stop.
```

可以看到，虽然`src.mk`可以正常读取了，但还是不对/(ㄒoㄒ)/~~

看下问什么哈！`make`告诉我们，找不到`main.c`的生成规则，这说明什么呢？再看一这个实例的目录结构。

```text
./example2_test_I
	-src
		-main.c
		-foo.c
		-bar.c
		-src.mk
	makefile
```

可以看到`main.c`是在`src`目录下且和`src.mk`是在同级目录。而`makefile`是在`main.c`的上级目录。我们在`makefile`中导入了`src.mk`,并使用`-I`选项告诉`make`如果在当前目录下，找不到`src.mk`，就去`src`目录试试。这样是可以找到`src.mk`了，也成功导入了，但执行时，`make`却找不到`main.c`。这说明，`include`指令只是简单把目标文件的内容展开到当前位置中(想一想`C`中的`include`)。但`make`再检查目标时，还是会在当前目录下检查。所以，只需改下目录结构就可以了。

新的目录结构如下：

```text
./example2_test_I
	-src
		-src.mk
	-main.c
	-foo.c
	-bar.c
	makefile
```

不容易啊！终于好了。这个`-I`感觉可以用在`makefile`的嵌套执行上，只要在`makefile`执行`makefile`时指定目录，要执行的`makefile`中如果需要其他`makefile`，就不用写路径了，会方便些，特别时路径可能会变，比如自动生成的`makefile`。当然这时，也可以用变量保存路径，但总没有什么都不管直接`include`要好，而且这样的话还有个好处，就是便于移植。这也是一种抽象吧！按我的理解，抽象就是**只做该你做的事，其他的，谁让你做的找谁**。

`(～ o ～)~zZ`今天先这样吧！

好的，下面接着昨天的继续，看看`-j`,还记得之前`--debug=j`吗？通过观察调试信息，我们发现，`make`在每次执行规则时，都会开启一个线程，那这样的话，是不是可以同时处理多条规则呢？其实是可以的，`-j`参数就是干这个的(当然前提是你系统得支持，必然`MS-DOS`就不支持)。

`-j`后面啥也不带，也即是尽可能多的运行命令：

```text
$ make --debug=j -j
gcc -c main.c -o main.o
Putting child 0x7fffcd0e8ed0 (main.o) PID 6161 on the chain.
Live child 0x7fffcd0e8ed0 (main.o) PID 6161
Reaping winning child 0x7fffcd0e8ed0 PID 6161
Removing child 0x7fffcd0e8ed0 PID 6161 from chain.
gcc -o main main.o foo.o bar.o
Putting child 0x7fffcd0eb560 (main) PID 6164 on the chain.
Live child 0x7fffcd0eb560 (main) PID 6164
Reaping winning child 0x7fffcd0eb560 PID 6164
Removing child 0x7fffcd0eb560 PID 6164 from chain.
```

速度上，可能文件比较少，感觉不是太明显，但是执行流程上就比较明显了。之前不加`-j`是一个线程执行完才会执行下一个会有明显的卡顿，但加了`-j`之后，好像线程间完全没有顺序了。

下面我们试下`-j1`,感觉和默认差不多。好了，别的就不多试了，条件允许的话，尽量编译时都加上`-j`选项吧！效率要高很多。

下面看看`-k`，`--keep-going`,还记得之前的`-i`吗？`-i`会忽略执行规则时的错误，而且如果此错误导致生成目标失败，那么依赖于此目标的目标并不会更新。但`-i`并无法忽略检查依赖时遇到的错误。同样的，我们也从这两个方面探讨下，`-k`的作用。

首先，我们先看看依赖错误，方便起见还是拷贝`example2`到`example2_test_k`,然后修改`makefile`如下。

将下面一行：

```text
main:main.o foo.o bar.o
```
修改为：
```text
main:main.o fo.o bar.o
```

然后，`make`下：

```text
$ make
gcc -c main.c -o main.o
make: *** No rule to make target 'fo.o', needed by 'main'.  Stop.
```

显然，`make`找不到`fo.o`,停止运行。下面试试加上`-k`:

```text
$ make -k
gcc -c main.c -o main.o
...
make: *** No rule to make target 'fo.o', needed by 'main'.
gcc -c bar.c -o bar.o
make: Target 'main' not remade because of errors.
```

加了`-k`后，可以看到，虽然`make`检查`fo.o`时遇到了错误，但还是把剩下的依赖检查完了，只是没有更新最终目标而已。

下面再试试，`-k`在遇到指令执行错误会怎样？

方便起见，拷贝`makefile`到`test_k.mk`,并修改。

将下面这两行：

```text
main:main.o fo.o bar.o
...
	gcc -c main.c -o main.o
```

修改为：

```text
main:main.o foo.o bar.o
...
	gcc -0 main.c -o main.o
```

然后，`make -f test_k.mk -k`试下：

```text
$ make -f test_k.mk -k
gcc -0 main.c -o main.o
gcc: error: unrecognized command line option ‘-0’
test_k.mk:5: recipe for target 'main.o' failed
make: *** [main.o] Error 1
gcc -c foo.c -o foo.o
gcc -c bar.c -o bar.o
make: Target 'main' not remade because of errors.
```

可以看到，效果和`-i`一样，`make`会忽略，执行错误，继续执行，并且同样不会更新依赖目标。

所以，`-k`可以看作为高级版的`-i`，它即会忽略执行错误，又会忽略`makefile`本身错误，而且同样不会根据错误的依赖生成目标。

下面看看`-n`,它会仅输出执行过程，但并不会执行。

进入`example2`目录，并执行`make -n`:

```text
$ make -n
gcc -c main.c -o main.o
gcc -c foo.c -o foo.o
gcc -c bar.c -o bar.o
gcc -o main main.o foo.o bar.o
```

可以看到，调试信息上，好像和正常执行没什么区别，只是少了些警告。但正因为它少了些警告，就可以判定，它确实没执行。我们也可以，`ls`下，看下当前目录下的文件变了没：

```text
$ ls
bar.c  fo.obj  foo.c  main.c  makefile
```

可以看到，并没有生成任何文件。而这就是`-n`的效果，它只会输出执行命令，但并不会执行。这个一般常用于调试`makefile`，看看其执行流程是不是符合预期。

`-o`选项。我们先试试在一个未编译过的目录下直接`make -o`会怎样：

```text
$ make -o
make: option requires an argument -- 'o'
```

嗯，他提示我需要个参数。那就给他个参数呗！我们把最终目标作为参数传给他：

```text
$ make -o main
make: 'main' is up to date.

$ ls
bar.c  fo.obj  foo.c  main.c  makefile
```

这个比较有意思，明明我目录下都不存在`main`，`make`却提示我，`main`已经是最新的了。这说明，`-o file` 就是告诉`make`关于`file`你就不用检查了，他已经是新的了。

下面看看`-p`:

(⊙﹏⊙)，信息太多了，不列了，这个选项主要是，把执行当前`makefile`时，环境中的所有变量和规则全部输出。



下面看看一个比较有意思的`-q`，先看下说明：不运行命令也不输出，仅仅检查指定目标是不是要更新。

```text
$ make -q
```

好像啥也看不到，因为它只会返回一个状态，0表示要更新，2表示错误。我不知道其他控制台是怎样的，不过我的会在我执行后，在命令提示符后多了一个下面这样的标志。

```text
C:1
```

这应该表示，不需要更新吧！不过又有点奇怪，我的目录是一个没编译过的，它却告诉我不需要更新。然后发现它可以带参数，加了参数试试，还是一样。有点搞不懂。

算了，下面试试`-s`,先看下效果吧：

```text
$ make -s
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

可以看到`-s`屏蔽命令执行的输出。如果你觉得命令信息太多了，不便于调试的话，可以`-s`下，这样就可以专注于代码调试了(是这样吗？我编的使用场景哦)。

还有，用于取消`-k`效果的`-S`。因为`make`嵌套执行时会默认从环境变量中继承些选项和参数，如果你不想要的话，你可以在`make`时明确取消它。

试下吧！还是先拷贝`example2_test`到`example2_test_S`,然后增加一个`build.mk`,内容如下：

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

`-t`,这个和`-o`有点像。都是阻止目标更新，我们看看两者有什么区别，先试试干净的目录：

```text
$ make -t main
touch main.o
touch foo.o
touch bar.o
touch main
```

感觉是根据依赖链，把所有需要更新的文件全部`touch`一遍。这个和`-o`有很大的不同，`-o`比较文雅，是直接告诉`make`和谁谁谁相关的文件你都不用检查了，而`-t`是十分粗暴的，把所有需要更新的全部手动`touch`一遍。

很好奇，加个调试看看，`-t`到底干了什么：

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

果然简单粗暴，`-t`只是直接用`touch`替换了生成指令而已，相当于更新文件的时间戳。可以在看看之后，正常编译会怎样：

```text
$ make
make: 'main' is up to date.
```

它会提示你文件已更新，当然了手动更新的。所以，`-t`只是利用`make`通过检查时间戳来判断是不是要更新文件，即通过手动更新来阻止规则生成。

下面看看`-o`:

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

同样在看下正常编译：

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
```

可以看到，虽然`-o`直接从源头阻止了检查，但只是零时阻止一次，下次如果不加`-o`的话，还是会检查。

所以`-t`的优点就很明显了。因为有时候，对于一些文件，你并不想马上更新它，而且又不想每次都用`-o`去忽略他，这样就可以`-t`一下，强行更新时间戳，一劳永逸。

下一个，`-w`,这个感觉和`--debug=m`很像啊！不过之前测试时发现`--debug=m`效果并不明显，下面看看`-w`怎么样：

方便起见，我们直接拷贝`example2_test_S`到`example2_test_w`,并修复其中的错误，而且为了效果更好，我们把`build.mk`修改为`build/makefile`，同时对应修改`makefile`。

```text
$ make
make: 'build' is up to date.
```

(⊙﹏⊙)出现了一个非常奇怪的事情，我们来分析下为什么哈！

首先列下我的当前目录结构：

```text
./
	-build
		-makefile
	-bar.c
	-main.c
	-foo.c
	-makefile
```

然后我们看看`./makefile`里面做了什么：

```text
$ cat makefile
build:
	$(MAKE) -C  build
```

可以看到，我们定义了一个依赖为空的目标，并把它作为最终目标。回忆一下，之前说过的依赖为空的目标的原则，如果目标不存在，那么必然执行命令，如果目标存在，那么必然不执行。再看看我们的目录结构，刚好有个目录叫`build`,所以对`make`来说，它发现目标存在后，就直接退出了，它并不会管你目标是什么，或者是怎样得到的。对于这种问题，有两种办法，第一改个文件名或改个目标名。不过作为专业人士(自封的，不接受反驳)，我不建议这么做，因为`make`本身支持一些内置规则，用来应对这种情况。也就是伪目标，所谓伪目标就是告诉`make`,这个目标是个假的，你就不用费劲去检查依赖了，直接执行指令吧！语法如下：

```text
.PHONY: targets
targets : prerequisites
	command
```

可以看到，伪目标其实是把你的目标作为依赖传给了`.PHONY`(这是一个内置参数，我会在[补充资料-内置参数][material_builtin-para]里讲解)，这样`make`就知道那些目标是伪目标了。

好了，这里我们把`build`目标设置为伪目标：

```text
.PHONY: build
build:
	$(MAKE) -C build
```

然后再`make`下：

```text
$ make
make -C build
make[1]: Entering directory '/home/duanduanlin/workspace/Pointers_on_C/doc/makefile_example/example2_test_w/build'
make[1]: *** No rule to make target 'main.c', needed by 'main.o'.  Stop.
make[1]: Leaving directory '/home/duanduanlin/workspace/Pointers_on_C/doc/makefile_example/example2_test_w/build'
makefile:4: recipe for target 'build' failed
make: *** [build] Error 2
```

(o_O???)什么情况，再分析下哈(好像又发现了什么有趣的东西，好激动啊！)！

`make`提示我，在执行`./build/makefile`时，找不到`main.c`。这说明什么？这说明不同于`make -f`指定要搜索的`makefile`,`make -C`是直接把当前目录切换过去了！！！下面修改目录结构如下：

```text
./
	-build
		-makefile
		-bar.c
		-main.c
		-foo.c
	-makefile
```

然后再试下：

```text
$ make
make -C build
make[1]: Entering directory '/home/duanduanlin/workspace/Pointers_on_C/doc/makefile_example/example2_test_w/build'
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
make[1]: Leaving directory '/home/duanduanlin/workspace/Pointers_on_C/doc/makefile_example/example2_test_w/build'
```

可以看到现在可以正常编译了，而且`ls`下，可以看到，生成的中间文件和可执行文件也全在`build`下，这说明`make`在执行`make -C build`时相当于是先执行`cd build`,然后在`make`。当然之后也会切回来。

哦呼！！！好吧！继续，这里也可以加上调试，看下`make`对伪目标做了什么：

```text
Reading makefiles...
Reading makefile 'makefile'...
Updating goal targets....
Considering target file 'build'.
 File 'build' does not exist.
 Finished prerequisites of target file 'build'.
Must remake target 'build'.
make -C build
```

好像和`-o`的效果类似，直接告诉`make`忽略检查`build`，不过这里还是重新生成了，试下吧，先把伪目标删掉，然后`make -o build`:

```text
Considering target file 'build'.
File 'build' was considered already.
make: 'build' is up to date.
```

不对，(⊙﹏⊙)，`-o`的话根本就不会检查。而对于依赖为空的目标，要执行命令的前提是目标不存在。也就是要先检查，发现目标不存在，然后尝试生成目标，因为依赖不存在所以省掉依赖检查，从而导致直接执行指令的效果。而`-o`压根就不会检查目标是不是存在。

然后我们加上`-w`试下：

```text
$ make -w
make: Entering directory '/home/duanduanlin/workspace/Pointers_on_C/doc/makefile_example/example2_test_w'
make -C  build
make[1]: Entering directory '/home/duanduanlin/workspace/Pointers_on_C/doc/makefile_example/example2_test_w/build'
make[1]: *** No rule to make target 'main.c', needed by 'main.o'.  Stop.
make[1]: Leaving directory '/home/duanduanlin/workspace/Pointers_on_C/doc/makefile_example/example2_test_w/build'
makefile:4: recipe for target 'build' failed
make: *** [build] Error 2
make: Leaving directory '/home/duanduanlin/workspace/Pointers_on_C/doc/makefile_example/example2_test_w'
```

对比不加`-w`的效果，可以看到，区别只是，加了`-w`的话，每次运行`makefile`都会显示前后信息。还有，不加`-w`好像也显示了部分信息，那是因为`-C`选项会默认带上`-w`。同样这个选项也是会继承的，如果想要取消的话，可以加上`--no-print-directory`。

哦呼！最后一个`-W`，和`-t`类似，也是假定目标要更新，不过如果配合`-n`使用，就会仅显示运行的指令，但不做任何修改。

同样，我们直接在`example2`目录下执行`make -W`:

```text
$ make -W
make: option requires an argument -- 'W'
Usage: make [options] [target] ...
```

发现，它报错了，提示我需要个参数，拿给它个参数。

```text
$ make -W main
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
```

好像，和`-t`不太一样，最起码对于一个干净的目录来说是这样的。`-t`是直接`touch`一个文件就好了，这里好像是真的全部生成了文件。这里说明下啊，`touch`时，如果文件不存在则新建文件，如果文件存在则更新时间戳。

进一步加上调试看下，`-W`到底做了什么？

```text
$ make --debug=v -W main
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
   File 'main.o' does not exist.
    Considering target file 'main.c'.
     Finished prerequisites of target file 'main.c'.
    No need to remake target 'main.c'.
   Finished prerequisites of target file 'main.o'.
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
 Prerequisite 'main.o' is newer than target 'main'.
 Prerequisite 'foo.o' is newer than target 'main'.
 Prerequisite 'bar.o' is newer than target 'main'.
No need to remake target 'main'.
```

通过调试信息可以看到，`-W`是把所有需要更新的依赖全部更新了，但唯独没有更新指定文件`main`。

为了确定下，我们在执行一次看看：

```text
$ make --debug=v -W main
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
  No need to remake target 'main.o'.
  Considering target file 'foo.o'.
    Considering target file 'foo.c'.
     Finished prerequisites of target file 'foo.c'.
    No need to remake target 'foo.c'.
   Finished prerequisites of target file 'foo.o'.
   Prerequisite 'foo.c' is older than target 'foo.o'.
  No need to remake target 'foo.o'.
  Considering target file 'bar.o'.
    Considering target file 'bar.c'.
     Finished prerequisites of target file 'bar.c'.
    No need to remake target 'bar.c'.
   Finished prerequisites of target file 'bar.o'.
   Prerequisite 'bar.c' is older than target 'bar.o'.
  No need to remake target 'bar.o'.
 Finished prerequisites of target file 'main'.
 Prerequisite 'main.o' is older than target 'main'.
 Prerequisite 'foo.o' is older than target 'main'.
 Prerequisite 'bar.o' is older than target 'main'.
No need to remake target 'main'.
make: 'main' is up to date.
```

可以看到，因为我们之前已经编译过了，所以此次编译并没有文件需要更新，而且`main`被我们指定了不用更新，所以这次啥也没干。

我们再试试，修改`main.c`加个打印：

```text
#include<stdio.h>

void main(int argc,char**argv)
{
    printf("enter main\r\n");
    foo();
    bar();

    return 0;
}
```

然后，再试下，预计是只更新`main.o`:

```text
   Considering target file 'main.c'.
     Finished prerequisites of target file 'main.c'.
    No need to remake target 'main.c'.
   Finished prerequisites of target file 'main.o'.
   Prerequisite 'main.c' is newer than target 'main.o'.
  Must remake target 'main.o'.
  ...
 Prerequisite 'main.o' is newer than target 'main'.
 Prerequisite 'foo.o' is older than target 'main'.
 Prerequisite 'bar.o' is older than target 'main'.
No need to remake target 'main'.
```

可以看到，确实如此，而且比较有意思的是，最后几行打印告诉我们，依赖`main.o`要比`main`新，但是仍然不用更新`main`,即使`main`并不存在。

所以可以看到`-W`的作用，只是在最后要生成某个目标是忽略生成指令，而其并不会影响依赖的更新。

[补充资料-`make`选项][material_make-options]到这里结束了，明天会整理成单独的一篇文章，因为实在是太长了/(ㄒoㄒ)/~~。

## 参考

[chain]: img/dependent_chain.png