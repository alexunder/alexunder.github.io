---
layout: post
title: MIT Missing Course---Bash Scripting Note
category: Linux
---

# Introduction

Bash脚本编程在平时的工作中可以起到解放生产力的作用。因为通过bash脚本可以将繁琐，可重复的工作进行自动化运行。然而有时用脚本解决一些问题以后，长时间不用，就忘了很多细节。正好missing课程总结了一些常见的规则，语法，以及秘籍，我在这里罗列一些。

# 变量定义

```bash
foo=bar
```

变量定义不能有空格，否则bash会当作调用命令来对待。

# 字符串

字符串可以用单引号或者双引号来包围住。但是他们俩还是有区别的。单引号不能引用字符串变量而双引号可以。

```bash
echo "$foo"
# prints bar
echo '$foo'
# prints $foo
```

第一个echo命令就可以把刚才定义的字符串变量打印出来，而第二个只能打印个寂寞。

# 输入参数

bash脚本跟一般的编程语言一样对if, for, while都有支持。同样地，bash脚本同样也支持类似函数调用的机制，还可以传入参数。下面是一个实现了创建了一个目录并且进入目标目录的函数：

```bash
mcd () {
    mkdir -p "$1"
    cd "$1"
}
```

在这里，``$1`` 是表示函数或者整个脚本的第一个参数。Bash有很多类似的输入变量，以及错误编码。下面是一些常用的变量：

- ``$0`` 表示脚本的名字。
- ``$1`` 到 $9 表示脚本的从左到右的输入参数。
- ``$@`` 表示所有的参数
- ``$#`` 表示输入参数的个数
- ``$?`` 表示上一个命令的返回码
- ``$$`` 表示当前脚本的PID
- ``!!`` 表示整个上一条命令，包括参数。有时候如果执行失败了，前面加上sudo就可以。

# 脚本的返回值或者运行结果

一般情况下，很多命令将一般信息打印到标准输出(STDOUT)，将错误打印到标准错误输出(STDERR)。通常脚本会有返回码，这样就比较方便的控制和监控脚本运行情况。返回0代表运行成功，其他表示运行有错误，所以上节里的常用变量``$?``就表示上个命令返回的返回码。当然也可以直接在调用命令的地方，将脚本的返回值做一些二进制的运算，例子如下；

```bash
false || echo "Oops, fail"
# Oops, fail

true || echo "Will not be printed"
#

true && echo "Things went well"
# Things went well

false && echo "Will not be printed"
```

另外还有一种情况是将命令得到的结果作为变量。把调用命令放到这个里面，``$( cmd )``，即cmd的位置，等脚本运行的时候，命令运行的结果就会被替换到这个位置。举个例子，比如这样一句脚本：

```bash
for file in $(ls)
```

这条脚本会先运行ls命令，然后再遍历所有通过ls找到的所有文件，再进行相关操作。

还有一个类似的秘术，即 ``< (cmd)`` ，他的做法是，先运行命令，然后将运行结果放到临时文件，然后在刚才cmd的位置上，将临时文件的整个路径将cmd替换掉。比如命令：``diff <(ls foo) <(ls bar)`` 将会显示在foo，bar两个命令下面文件的不同情况。

# 条件比较语句

下面有个例子，将上面说的几个点综合了一下：

```bash
#!/bin/bash

echo "Starting program at $(date)" # Date will be substituted
echo "Running program $0 with $# arguments with pid $$"

for file in "$@"; do
    grep foobar "$file" > /dev/null 2> /dev/null
    # When pattern is not found, grep has exit status 1
    # We redirect STDOUT and STDERR to a null register since we do not care about them
    if [[ $? -ne 0 ]]; then
        echo "File $file does not have any foobar, adding one"
        echo "# foobar" >> "$file"
    fi
done
```

首先 ``$(date)`` 打印出时间，然后遍历脚本的输入参数，其实应该是各个文件，每一次循环都将相关文件全目录赋给``$file``变量，然后用grep命令在当前文件里搜索关键字``foobar``。后面这段命令：``> /dev/null 2> /dev/null``的意思是将grep命令的标准输入，和标准错误输出都重新定位到/dev/null设备上，其实就是舍弃，不要的意思。``/dev/null``是一种特殊的linux虚拟设备，专门用来倾倒不需要的垃圾数据。再后面的命令就是如果grep没有找到相关关键字，就将这个关键字添加到当前的文件里。

说了这么多，才要说到正题：条件判断语句，其实很简单。上面的例子中，两个中括号里面就是条件判断。其实单个中括号也可以作为判断语句的容器，但是可能会和其他有冲突，所以为了减少错误，用双中括号更能降低出错的机会。

# 用通配符和大括号实现的shell globbing便利功能

当调用脚本的时候，你可能会输入一些类似的，同质的选项。正好Bash有一些技巧可以扩展文件的后缀名,叫做shell globbing，中文我也暂时不知道怎么翻译。

- 通配符(Wildcards ). 做通配符匹配的时候，我们可以使用``?``或者``*``。问号``?``只匹配一个字符，星号``*``可以匹配任意个字符。比如有这样几个文件：foo, foo1, foo2, foo10 以及 bar，命令``rm foo?``将会删除foo1和foo2，而``rm foo*``将会删除除了bar以外所有的文件。
- 大括号(``{}``). 当一些类似的命令里里有一些字符串，他们虽然不一样，但是比较类似，就可以用大括号来做一些处理。

看看下面的例子：

```bash
convert image.{png,jpg}
# Will expand to
convert image.png image.jpg

cp /path/to/project/{foo,bar,baz}.sh /newpath
# Will expand to
cp /path/to/project/foo.sh /path/to/project/bar.sh /path/to/project/baz.sh /newpath

# Globbing techniques can also be combined
mv *{.py,.sh} folder
# Will move all *.py and *.sh files


mkdir foo bar
# This creates files foo/a, foo/b, ... foo/h, bar/a, bar/b, ... bar/h
touch {foo,bar}/{a..h}
touch foo/x bar/y
# Show differences between files in foo and bar
diff <(ls foo) <(ls bar)
# Outputs
# < x
# ---
# > y
```

编写Bash脚本是一件非常反直觉的事情，因此有个工具[shell check](https://github.com/koalaman/shellcheck)可以像编译器那样检查你的脚本写的是否有问题。

# References

https://missing.csail.mit.edu/2020/shell-tools/