# 补充资料-`make`内置目标

`GNU make`中以`.`开通后面跟着大写字母时，一般是有特殊意义的内置参数。而当它是个目标时，那就是内置目标了。下面先看看`make`有那些内置目标。

-   `.PHONY`-伪目标
-   `.SUFFIXES`-后缀
-   `.DEFAULT`-设置默认命令
-   `.PRECIOUS`-保留指定模式的中间文件
-   `.INTERMEDIATE`-标注指定文件为中间文件
-   `.SECONDARY`-标注某文件为次级文件
-   `.SECONDEXPANSION`-辅助扩展
-   `.DELETE_ON_ERROR`-标明指定文件应该在命令出错后删除
-   `.IGNORE`-标明`make`应该忽略指定目标引起的错误，若依赖为空，则忽略所有错误
-   `.LOW_RESOLUTION_TIME`-标明目标是低解析度的时间，也就是是时间对比到秒级
-   `.SLIENT`-标明指定目标的命令不输出信息，如果依赖为空，那么指所有目标
-   `.EXPORT_ALL_VARIABLES`-表示导出所有变量到子进程
-   `.NOTPARALLEL`-表示不允许并行运行命令，即使有添加`-j`选项
-   `.ONESHELL`-处于这个目标下的每行命令使用同一个`shell`
-   `.POSIX`-表示兼容`POSIX`标准

## 写在前面
本文作是我学习`C`语言及其相关工具的使用的系列笔记中，关于`make`工具中一个知识点的学习记录。是和整个系列一起上传到[`github`][pointers_on_c]的,里面有全部源码和笔记。
开始之前呢，先对本文所使用的工程实例模板做下说明,后续测试都是以此为基础。

首先，是目录结构：

```text
./builtin_test
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

.PHONY:cleanall cleanobj

cleanall:cleanobj
	-rm main

cleanobj:
	-rm *.o
```



## `.PHONY`-伪目标

伪目标，简单来说就是告诉`make`这只是个标签，直接执行其命令就好了。这个比较常用，不过我比较好奇的时`make`到底有没有去检查目标和依赖。下面我们设计实验去验证。

首先我们新建一个目录`builtin_test_phony`,然后在里面增加一个`makefile`；内容如下：

```text
$ cat makefile

.PHONY:run

run:temp.file
	@echo target run is come.

temp.file:
	@echo temp.file is not exist,so i will touch it.
    touch temp.file
    
clean:
	-rm temp.file
```

简单讲下，这里做了什么。首先我定义了一个伪目标`run`，且给了一个依赖，`temp.file`其依赖为空，并且会在命令里`touch`一个`temp.file`。当我们首次执行`make`时只需观察是不是会有`temp.file`生成，就可以判断`make`在遇到伪目标时，是会忽略依赖直接执行，还是仍然会检查依赖，不过会忽略检查结果。

我们先看看现象：

```text
$ make
temp.file is not exist,so i will touch it.
touch temp.file
target run is come.

$ make
target run is come.
```

可以看到，其实`make`在遇到伪目标时，还是会正常检查依赖的，只不过会忽略检查结果而已。而且可以看到，第二次执行`make`并没有生成`temp.file`,这说明伪目标并不会继承。



## `SUFFIXES`-后缀

后缀是定义隐式规则的旧方法，目前已基本被模式规则所取代，不过为了兼容旧版`makefile`，`make`仍然保留此功能。后缀有两种形式，单后缀和双后缀。
双后缀规则定义一对后缀：目标文件的后缀和依赖目标的后缀。它匹配所有后缀为目标后缀的文件。对于一个匹配的目标文件，它的依赖文件这样形成：**将匹配的目标文件名中的后缀替换为依赖文件的后缀得到**。如：一个描述目标和依赖后缀的“.o”和“.c”的规则就等价于模式规则“%o : %c”。

首先我们从`builtin_test`拷贝一份到`builtin_test_suffix`，并修改`makefile`如下：

```text
$ cat makefile
main:main.o foo.o bar.o
	gcc -o main main.o foo.o bar.o
.c.o:
	gcc -c $^ -o $@
```

其中`.c.o`就是双后缀的写法了，类似于模式规则中的`%.o:%.c`。我们先简单看看`make`后的效果。

```text
$ make
gcc -c main.c -o main.o
gcc -c foo.c -o foo.o
gcc -c bar.c -o bar.o
gcc -o main main.o foo.o bar.o
```

此外要注意的时，后缀规则不能带依赖，否则就变成了一个普通规则。`make`本身有一个后缀列表，我们可以使用没有依赖的`.SUFFIXES`来删除后缀列表，或者给他添加依赖来添加后缀规则到后缀列表。

下面我们在当前文件夹新建`define_double_suffix`目录，并在其中添加`makefile`文件，内容如下：

```test
$ cat makefile

.SUFFIXES:.a .b

target:file a.b b.b c.b
	touch target
.a.b:
	touch $(patsubst %.a,%.b,$^)

file:a.a b.a c.a

%.a:
	touch $@

.PHONY:cleanall cleanb cleana

cleanall:cleanb cleana
	-rm target

cleanb:
	-rm *.b

cleana:
	-rm *.a
```

这里我们定义了一个双后缀规则`.a.b`并把它加如到后缀列表。其中，`.a`是源文件后缀，`.b`是目标文件后缀。然后我们定义了一个目标`target`，依赖于`file`,`a.b`,`b.b`和`c.b`，最后告诉`make`所有`.b`文件均有`.a`文件的文件名生成。`file`是一个没有命令的目标，主要是利用它在检查依赖时生成依赖，也就是我们所需的`.a`文件。

在当前目录下新建然后我们`make`下，可以看到如下效果。

```text
$ make
touch a.a
touch b.a
touch c.a
touch a.b
touch b.b
touch c.b
touch target
```

单后缀没搞懂，等我搞懂了在加上吧！



## `.DEFAULT`-设置默认命令

`.DEFAULT`是给那些没有任何规则的目标设置一个默认的命令。看下面这个例子。

新建`builtin_test_default`,并增加如下``makefile`文件：

```text
$ cat makefile
run:nothing pro1
	@echo target run is coming.

pro1:pro2
	@echo pro1 is coming.

.DEFAULT:
	@echo $@ is not exist
```

考虑这样一种情况,可能有时候我们对于一些依赖的规则还没想好怎么写，或者压根就是忘了写了，而此时我们我们又想正常调试`makefile`的依赖关系，而此时就可以使用`.DEFAULT`定义一个默认的规则。这样一来可以避免`make`出错，二来可以在发现没有规则的依赖时做点什么，比如加个输出。

我们看下上面这个实例的效果：

```text
$ make
nothing is not exist
pro2 is not exist
pro1 is coming.
target run is coming.
```

如果想清除默认规则，使`.DEFAULT`目标的规则为空就可以了。

下面我们来看看一个类似的，“最后手段”。

新建一个`last-resort.mk`,内容如下：

```text
test:hello
	@echo target test is coming.

%::
	@echo $@ is missing.
```

其中`%::`可以匹配任何目标，任何找不到规则的目标最后都会找到这里。效果如下：

```text
$ make -f last-resort.mk
hello is missing.
target test is coming.
```



## `.INTERMEDIATE`-标注指定文件为中间文件

通过把某文件指定为中间文件，则`make`每次执行完成都会删除该文件。我们先来看看什么是中间文件，简单以编译`C`代码为例，我们一般情况下直接在命令行下输入`gcc -c main.c main`就可以直接生成可执行文件了。但我们也知道这个过程细分的话，是包括预处理，编译，汇编，以及链接等过程。这里我们以编译链接为例，首先`.c`文件会被编译成`.o`文件，之后链接器把所有`.o`文件链接成可执行程序。这里`.o`文件就是中间文件。

而对于中间文件，也即是为了得到目标而临时引入的文件，`make`会和其他文件一样检查依赖并执行规则生成，但有以下两点区别：第一点，当某个目标依赖于中间文件，而中间文件又不存在且中间文件的依赖也不比目标新，那`make`就不会去生成中间文件，而是直接认为目标不需要更新。第二点，如果中间文件确实需要生成，那么`make`会先生成中间文件，然后在不需要的时候删除它。

默认情况下，`make`不会把目标和依赖当作中间文件，这时你可以手动指定某文件为中间文件。

下面我们看个实例，首先我们拷贝`builtin_test`到`builtin_test_intermediate`,然后在`makefile`中添加如下内容：

```text
.INTERMEDIATE:main.o foo.o bar.o
```

然后，`make`下看下效果：

```text
$ make
gcc -c main.c -o main.o
gcc -c foo.c -o foo.o
gcc -c bar.c -o bar.o
gcc -o main main.o foo.o bar.o
rm foo.o bar.o main.o
```

你会看到，除了在最后`make`会删掉所有指定的`.o`文件外，和不加`.INTERMEDIATE`并没有什么区别。

然后我们再`make`下：

```text
$ make
make: 'main' is up to date.
```

你会看到，虽然我们最终目标的依赖全都不存在，但`make`也没有更新依赖，因为他此时是比较中间文件的依赖是不是比最终目标新。

下面我们修改源文件时间戳试试：

```text
$ touch foo.c

$ make
gcc -c main.c -o main.o
gcc -c foo.c -o foo.o
gcc -c bar.c -o bar.o
gcc -o main main.o foo.o bar.o
rm foo.o bar.o main.o
```

你会发现，我任意改变一个中间文件所依赖的文件的时间戳，都会导致`make`重新编译所有中间文件。



## `.PRECIOUS`-设置目标是精贵的

现在我们知道，对于中间文件，`make`会在不需要它们的时候删除。但如果我们不想删除，则可以使用`.PRECIOUS`。

方便起见，我们直接拷贝`builtin_test_intermediate`到`builtin_test_precious`,然后在`makefile`中增加下面内容：

```text
.PRECIOUS:main.o
```

然后我们`make`下，看下效果：

```text
$ make
gcc -c main.c -o main.o
gcc -c foo.c -o foo.o
gcc -c bar.c -o bar.o
gcc -o main main.o foo.o bar.o
rm foo.o bar.o
```

你会看到，现在`make`不会再删除`main.o`了。



## `.SECONDARY`-标注某文件为次级文件

次级文件是一种特殊的中间文件，它不会自动删除。也就是说，它只有中间文件的特点一。

同样还是直接从`builtin_test_intermediate`拷贝到`builtin_test_secondary`,并修改如下内容：

```text
把这一行
.INTERMEDIATE:main.o foo.o bar.o
改成
.SECONDARY:main.o foo.o bar.o
```

然后编译下，你会看到和正常编译没有任何区别。下面手动删除`.o`文件后再试试。

```text
$ rm *.o
$ make
make: 'main' is up to date.
```

你会看到这一点是不是和中间文件一样，也就是当中间文件不存在时，`make`直接检查中间文件的依赖是不是有更新，从而决定是不是要重新生成中间文件。

同样，下面我们试试手动更新源码时间戳。你会看到和中间文件一样，改动任意一个文件，都会导致`make`重现编译所有中间文件。

下面看看这样会怎样：

```text
$ rm foo.o

$ touch foo.c

$ make
gcc -c foo.c -o foo.o
gcc -o main main.o foo.o bar.o
```

你会看到，对于次级文件，如果`make`检查到源码有更新，它只会重新生成需要更新的中间文件(不存在的和依赖有更新的)，并链接程序。



## `.SECONDEXPANSION`-辅助扩展



## `.DELETE_ON_ERROR`-命令出错后删除



## `.IGNORE`-忽略错误



## `.LOW_RESOLUTION_TIME`-低解析度时间



## `.SLIENT`-不输出命令信息



## `.EXPORT_ALL_VARIABLES`-导出所有变量



## `.NOTPARALLEL`-不允许并行运行命令



## `.ONESHELL`-使用同一个`shell`



## `.POSIX`-兼容`POSIX`标准

