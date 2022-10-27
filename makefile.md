# Makefile编写笔记
Makefile 文件定义了一系列规则，指明了源文件的编译顺序、依赖关系、是否需要重新编译等。
若要指定特定的makefile文件，可以是使用`-f`或者`--file`参数，如`make -f Make.Linux`

## **一、基本原则**
```
如果这个工程没有编译过，那么我们的所有c文件都要编译并被链接。

如果这个工程的某几个c文件被修改，那么我们只编译被修改的c文件，并链接目标程序。

如果这个工程的头文件被改变了，那么我们需要编译引用了这几个头文件的c文件，并链接目标程序。

```


## **二、makefile编写基本流程**
```
target ... : prerequisites ...
    command
    ...
    ...

#target: 可以是一个object file（目标文件），也可以是一个执行文件，还可以是一个标签（label）
#prerequisites：生成该target所依赖的文件或target，如果这其中有一个以上的文件比target要新的话，就会执行command命令
#command：该target要执行的命令，可以是任意shell脚本
```

**示例**
```cpp
edit : main.o kbd.o command.o display.o \
        insert.o search.o files.o utils.o
    cc -o edit main.o kbd.o command.o display.o \
        insert.o search.o files.o utils.o

main.o : main.c defs.h
    cc -c main.c
kbd.o : kbd.c defs.h command.h
    cc -c kbd.c
command.o : command.c defs.h command.h
    cc -c command.c
display.o : display.c defs.h buffer.h
    cc -c display.c
insert.o : insert.c defs.h buffer.h
    cc -c insert.c
search.o : search.c defs.h buffer.h
    cc -c search.c
files.o : files.c defs.h buffer.h command.h
    cc -c files.c
utils.o : utils.c defs.h
    cc -c utils.c
clean :
    rm edit main.o kbd.o command.o display.o \
        insert.o search.o files.o utils.o
```

### 1、声明变量
可以通过声明变量来简化makefile文件，如下。后续可以使用`$(objects)`来替代

```cpp
objects = main.o kbd.o command.o display.o \
     insert.o search.o files.o utils.o
```
### 2、自动推导
make可以自动推导文件以及文件依赖关系后面的命令。

例如，如果make找到一个 whatever.o ，那么 whatever.c 就会是 whatever.o 的依赖文件。并且 cc -c whatever.

此时，makefile被简化如下。`.PHONY` 表示 `clean` 是个伪目标文件。
```cpp
objects = main.o kbd.o command.o display.o \
    insert.o search.o files.o utils.o

edit : $(objects)
    cc -o edit $(objects)

main.o : defs.h
kbd.o : defs.h command.h
command.o : defs.h command.h
display.o : defs.h buffer.h
insert.o : defs.h buffer.h
search.o : defs.h buffer.h
files.o : defs.h buffer.h command.h
utils.o : defs.h

.PHONY : clean
clean :
    rm edit $(objects)
```
**另一种写法**
```cpp
objects = main.o kbd.o command.o display.o \
    insert.o search.o files.o utils.o

edit : $(objects)
    cc -o edit $(objects)

$(objects) : defs.h
kbd.o command.o files.o : command.h
display.o insert.o search.o files.o : buffer.h

.PHONY : clean
clean :
    rm edit $(objects)
```
### 3、清空目标文件的规则
```cpp
.PHONY : clean
clean :
    -rm edit $(objects)
```

### 4、引用其他makefile
```cpp
bar = e.mk f.mk
include foo.make *.mk $(bar)
# 等价于
include foo.make a.mk b.mk c.mk e.mk f.mk
```
make会在当前目录下寻找这些文件，如果当前目录没有找到，还会去以下目录：
- 当make执行时，指定了-I或者-include-dir参数，则会去该目录寻找
- 如果存在<prefix>/include目录（一般是/usr/local/bin或者/usr/include）存在，也会去这些目录

如果没有找到一些文件，在完成makefile的读取之后，则会报错。可以通过
```cpp
-include <filename>
```
不要报错，继续执行，

### 5、文件搜寻
- 定义特殊变量VPATH
```cpp
VPATH = src:../headers
```
在当前目录搜索之后，再去src目录，再去headers目录

- 使用vpath关键字
```cpp
# 为符合pattern的文件指定搜索目录dir
vpath <pattern> <dir>
vpath %.h ../headers

# 清除符合pattern的文件的搜索目录
vpath <pattern>

# 清除所有设置好的文件搜索目录
vpath

```


### 6、伪目标
避免出现与文件重名的情况。如下程序可以生成多个可执行文件，而不需要编写多个makefile
```cpp
.PHONY : clean
clean :
    rm *.o temp
```
定义了伪目标，就不会生成该目标
```cpp
all : prog1 prog2 prog3
.PHONY : all

prog1 : prog1.o utils.o
    cc -o prog1 prog1.o utils.o

prog2 : prog2.o
    cc -o prog2 prog2.o

prog3 : prog3.o sort.o utils.o
    cc -o prog3 prog3.o sort.o utils.o
```

```cpp
.PHONY : cleanall cleanobj cleandiff

cleanall : cleanobj cleandiff
    rm program

cleanobj :
    rm *.o

cleandiff :
    rm *.diff
```
### 7、多目标
使用自动化变量 `$@`，其表示目标集。`-$(subst output,,$@)` 中的 `$` 表示执行一个Makefile的函数，函数名为subst，后面的为参数。
```cpp
bigoutput littleoutput : text.g
    generate text.g -$(subst output,,$@) > $@


bigoutput : text.g
    generate text.g -big > bigoutput
littleoutput : text.g
    generate text.g -little > littleoutput
```

### 8、静态模式
```cpp
# target-pattern 目标集模式
# prereq-patterns 目标的依赖模式
<targets ...> : <target-pattern> : <prereq-patterns ...>
    <commands>
    ...
```

如下，$< 表示第一个依赖文件， $@ 表示目标集
```cpp
objects = foo.o bar.o

all: $(objects)

$(objects): %.o: %.c
    $(CC) -c $(CFLAGS) $< -o $@
```
可展开为
```cpp
foo.o : foo.c
    $(CC) -c $(CFLAGS) foo.c -o foo.o
bar.o : bar.c
    $(CC) -c $(CFLAGS) bar.c -o bar.o
```
<font color=red>也可使用变量值替换</font>
```cpp
foo := a.o b.o c.o
bar := $(foo:.o=.c)
```

再举例, `$(filter %.o,$(files))`表示调用Makefile的filter函数，过滤“$files”集，只要其中模式为“%.o”的内容
```cpp
files = foo.elc bar.o lose.o

$(filter %.o,$(files)): %.o: %.c
    $(CC) -c $(CFLAGS) $< -o $@
$(filter %.elc,$(files)): %.elc: %.el
    emacs -f batch-byte-compile $<
```

```cpp
a_objects := a.o b.o c.o
1_objects := 1.o 2.o 3.o

sources := $($(a1)_objects:.o=.c)

# 结果
$(a1) 的值是“a”的话，那么， $(sources) 的值就是“a.c b.c c.c”；如果 $(a1) 的值是“1”，那么 $(sources) 的值是“1.c 2.c 3.c”
```

### 9、自动生成依赖性
编译器支持-M选项，自动寻找源文件中包含的头文件，生成依赖关系
```cpp
cc -M main.c
```
输出
```cpp
main.o: main.c defs.h
```
<font color=red>注：GNU的C/C++编译器，如gcc，使用 -MM 参数，否则 -M 参数会把一些标准库的头文件也包含进来  </font>

为每个源文件自动生成的依赖关系存放在一个文件中，即为name.c生成name.d，.d中存放对应.c文件的依赖关系

```cpp
# to be understand
%.d: %.c
    @set -e; rm -f $@; \
    $(CC) -M $(CPPFLAGS) $< > $@.$$$$; \
    sed 's,\($*\)\.o[ :]*,\1.o $@ : ,g' < $@.$$$$ > $@; \
    rm -f $@.$$$$
```


## 三、命令
### 1、显示命令

- `@`不显示命令
```cpp
@echo 正在编译。。。

# 输出：
正在编译。。。

echo 正在编译。。。

#输出：
echo 正在编译。。。
正在编译。。。
```
make的参数
- `-n`或者`--just-print`只显示，不执行命令
- `-s`或者`--silent`或者`--quiet` 全面禁止命令的显示

### 2、执行命令
两条命令之间需要用 ; 隔开

### 3、命令出错时
- 命令行前加`-`
```cpp
clean:
    -rm -f *.o
```
- make添加`-i`或者`--ignore-errors`参数，忽略所有命令错误
- `.IGNORE`为目标的规则，忽略该规则中的所有命令错误
- `-k`参数或者`--keep-going`，如果某规则中的命令出错，终止该规则执行，继续执行其他规

### 4、传递变量

总控Makefile可以传递到下级Makefile中，但是不会覆盖下层的Makefile中所定义的变量，除非指定了`-e`参数
```cpp
# 传递所有变量
export 

# 传递一个变量
export variable = value
export variable := value

variable = value
export variable

# 不传递
unexport variable
```
有两个参数：`SHELL`和`MAKEFLAGS `，不管是否export，总是会传递到下层

如果不想往下层传递参数，可以
```cpp
subsystem:
    cd subdir && $(MAKE) MAKEFLAGS=
```


### 5、定义命令包
```cpp
define run-yacc
yacc $(firstword $^)
mv y.tab.c $@
endef

foo.c : foo.y
    $(run-yacc)
```
### 6、 定义多行变量
```cpp
define two-lines
echo foo
echo $(bar)
endef

```

### 7、override指示符

如果有变量是通常make的命令行参数设置的，那么Makefile中对这个变量的赋值会被忽略。但是可以通过override覆盖
```cpp
override <variable>; = <value>;

override <variable>; := <value>;
```






## 四、 变量
- 大小写敏感
- 变量名可以包含字符、数字，下划线（可以是数字开头），但不应该含有 : 、 # 、 = 或是空字符（空格、回车等）
- 声明时赋予初始值，使用时$()或者${}
- 变量赋值：=，可以使用在任何一处定义的变量赋值； :=，只能使用已定义的变量
- 可以用`#`结束变量的定义
- `?=` 若先前未定义，则赋值。否则不作用
- `+=`追加变量值

```cpp
objects = main.o foo.o bar.o utils.o
objects += another.o
```
- 定义多行变量



## 五、条件判断

<font color=red>make是在读取Makefile时就计算条件表达式的值，并根据条件表达式的值来选择语句，不要把自动化变量（如 $@ 等）放入条件表达式中，因为自动化变量是在运行时才有的。</font>
- **ifeq**
```cpp
ifeq (<arg1>, <arg2>)
ifeq "<arg1>" '<arg2>' # 单双引号随意

# 参数中可以使用make的函数
ifeq ($(strip $(foo)),)
<text-if-empty>
endif
```

- **ifneq**
用法同`ifeq`

- **ifdef**
若已经定义过，则表达式为真，否则为假
```cpp
bar =
foo = $(bar)
ifdef foo
    frobozz = yes
else
    frobozz = no
endif
```
- **ifndef**
用法同`ifdef`



## 六、函数
1、字符串处理函数
```cpp
# subst：将text中的from替换为to
$(subst <from>,<to>,<text>)

# patsubst：将text中符合pattern的以 <replacement> 替换
$(patsubst <pattern>,<replacement>,<text>)

# strip：去首尾空格函数
$(strip <string>)

# findstring：查找字符串函数。如果在字符串<in>中找到<find>，则返回<find>,否则返回空字符串
$(findstring <find>,<in>)

# filter：以 <pattern> 模式过滤 <text> 字符串中的单词，保留符合模式 <pattern> 的单词
$(filter <pattern...>,<text>)

# filter-out：去除<text>字符串中符合模式 <pattern> 的单词
$(filter-out <pattern...>,<text>)

# sort：按字典升序排序，会去掉重复值
$(sort <list>)

# word：取字符串 <text> 中第 <n> 个单词
$(word 2, foo bar baz) # 返回bar

# wordlist：返回从i到j的单词串
$(wordlist <i>,<j>,<text>)$(wordlist 2, 3, foo bar baz) # 返回bar baz


# words：统计单词个数
$(words, foo bar baz) # 返回3
```

2、文件名操作函数
```cpp
# dir：取目录
$(dir src/foo.c hacks) # 返回值是 src/ ./ 

# notdir: 取文件
$(notdir src/foo.c hacks) # 返回值是 foo.c hacks 。

# suffix: 取后缀
$(suffix src/foo.c src-1.0/bar.c hacks) # 返回值是 .c .c

# basename: 取前缀
$(basename src/foo.c src-1.0/bar.c hacks) # 返回值是 src/foo src-1.0/bar hacks

# addsuffix：加后缀
$(addsuffix .c,foo bar) # 返回值是 foo.c bar.c 。

# addperfix: 加前缀
$(addprefix src/,foo bar) # 返回值是 src/foo src/bar 。

# join: 连接函数
$(join aaa bbb , 111 222 333) # 返回值是 aaa111 bbb222 333
```

3、循环函数
```cpp
# 把参数 <list> 中的单词逐一取出放到参数 <var> 所指定的变量中，然后再执行 <text> 所包含的表达式
$(foreach <var>,<list>,<text>)

names := a b c d
files := $(foreach n,$(names),$(n).o) # $(files) 的值是 a.o b.o c.o d.o
```

4、判断函数
```cpp
$(if <condition>,<then-part>,<else-part>)
```

5、call函数
```cpp
reverse =  $(2) $(1)
foo = $(call reverse,a,b) # foo 的值是 b a
```

6、origin函数
```cpp
$(origin <variable>)
# 返回值有如下：
undefined
如果 <variable> 从来没有定义过，origin函数返回这个值 undefined

default
如果 <variable> 是一个默认的定义，比如“CC”这个变量，这种变量我们将在后面讲述。

environment
如果 <variable> 是一个环境变量，并且当Makefile被执行时， -e 参数没有被打开。

file
如果 <variable> 这个变量被定义在Makefile中。

command line
如果 <variable> 这个变量是被命令行定义的。

override
如果 <variable> 是被override指示符重新定义的。

automatic
如果 <variable> 是一个命令运行中的自动化变量。关于自动化变量将在后面讲述。
```

7、shell函数

8、控制make的函数
```cpp
# 会停止make的运行
$(error <text ...>)

# 输出警告，make继续执行
$(warning <text ...>)
```





## **七、make运行**
### 1、工作流程
```cpp
(1)读入所有的Makefile。
(2)读入被include的其它Makefile。
(3)初始化文件中的变量。
(4)推导隐晦规则，并分析所有规则。
(5)为所有的目标文件创建依赖关系链。
(6)根据依赖关系，决定哪些目标要重新生成。
(7)执行生成命令。
```
### 2、make的退出码
```cpp
 0：代表成功执行
 1：运行时出现任何错误，返回1
 2：使用了make的“-q”选项，并且make使得一些目标不需要更新，那么返回2
```
### 3、指定Makefile
```cpp
make –f hchen.mk
```

### 4、make中常用目标的书写规则
```cpp
all:这个伪目标是所有目标的目标，其功能一般是编译所有的目标。

clean:这个伪目标功能是删除所有被make创建的文件。

install:这个伪目标功能是安装已编译好的程序，其实就是把目标执行文件拷贝到指定的目标中去。

print:这个伪目标的功能是例出改变过的源文件。

tar:这个伪目标功能是把源程序打包备份。也就是一个tar文件。

dist:这个伪目标功能是创建一个压缩文件，一般是把tar文件压成Z文件。或是gz文件。

TAGS:这个伪目标功能是更新所有的目标，以备完整地重编译使用。

check和test:这两个伪目标一般用来测试makefile的流程。
```

### 5、make的参数
```cpp
-b, -m
这两个参数的作用是忽略和其它版本make的兼容性。

-B, --always-make
认为所有的目标都需要更新（重编译）。

-C <dir>, --directory=<dir>
指定读取makefile的目录。如果有多个“-C”参数，make的解释是后面的路径以前面的作为相对路径，并以最后的目录作为被指定目录。如：“make -C ~hchen/test -C prog”等价于“make -C ~hchen/test/prog”。

-debug[=<options>]
输出make的调试信息。它有几种不同的级别可供选择，如果没有参数，那就是输出最简单的调试信息。下面是<options>的取值：

a: 也就是all，输出所有的调试信息。（会非常的多）

b: 也就是basic，只输出简单的调试信息。即输出不需要重编译的目标。

v: 也就是verbose，在b选项的级别之上。输出的信息包括哪个makefile被解析，不需要被重编译的依赖文件（或是依赖目标）等。

i: 也就是implicit，输出所有的隐含规则。

j: 也就是jobs，输出执行规则中命令的详细信息，如命令的PID、返回码等。

m: 也就是makefile，输出make读取makefile，更新makefile，执行makefile的信息。

-d
相当于“–debug=a”。

-e, --environment-overrides
指明环境变量的值覆盖makefile中定义的变量的值。

-f=<file>, --file=<file>, --makefile=<file>
指定需要执行的makefile。

-h, --help
显示帮助信息。

-i , --ignore-errors
在执行时忽略所有的错误。

-I <dir>, --include-dir=<dir>
指定一个被包含makefile的搜索目标。可以使用多个“-I”参数来指定多个目录。

-j [<jobsnum>], --jobs[=<jobsnum>]
指同时运行命令的个数。如果没有这个参数，make运行命令时能运行多少就运行多少。如果有一个以上的“-j”参数，那么仅最后一个“-j”才是有效的。（注意这个参数在MS-DOS中是无用的）

-k, --keep-going
出错也不停止运行。如果生成一个目标失败了，那么依赖于其上的目标就不会被执行了。

-l <load>, --load-average[=<load>], -max-load[=<load>]
指定make运行命令的负载。

-n, --just-print, --dry-run, --recon
仅输出执行过程中的命令序列，但并不执行。

-o <file>, --old-file=<file>, --assume-old=<file>
不重新生成的指定的<file>，即使这个目标的依赖文件新于它。

-p, --print-data-base
输出makefile中的所有数据，包括所有的规则和变量。这个参数会让一个简单的makefile都会输出一堆信息。如果你只是想输出信息而不想执行makefile，你可以使用“make -qp”命令。如果你想查看执行makefile前的预设变量和规则，你可以使用 “make –p –f /dev/null”。这个参数输出的信息会包含着你的makefile文件的文件名和行号，所以，用这个参数来调试你的 makefile会是很有用的，特别是当你的环境变量很复杂的时候。

-q, --question
不运行命令，也不输出。仅仅是检查所指定的目标是否需要更新。如果是0则说明要更新，如果是2则说明有错误发生。

-r, --no-builtin-rules
禁止make使用任何隐含规则。

-R, --no-builtin-variabes
禁止make使用任何作用于变量上的隐含规则。

-s, --silent, --quiet
在命令运行时不输出命令的输出。

-S, --no-keep-going, --stop
取消“-k”选项的作用。因为有些时候，make的选项是从环境变量“MAKEFLAGS”中继承下来的。所以你可以在命令行中使用这个参数来让环境变量中的“-k”选项失效。

-t, --touch
相当于UNIX的touch命令，只是把目标的修改日期变成最新的，也就是阻止生成目标的命令运行。

-v, --version
输出make程序的版本、版权等关于make的信息。

-w, --print-directory
输出运行makefile之前和之后的信息。这个参数对于跟踪嵌套式调用make时很有用。

--no-print-directory
禁止“-w”选项。

-W <file>, --what-if=<file>, --new-file=<file>, --assume-file=<file>
假定目标<file>;需要更新，如果和“-n”选项使用，那么这个参数会输出该目标更新时的运行动作。如果没有“-n”那么就像运行UNIX的“touch”命令一样，使得<file>;的修改时间为当前时间。

--warn-undefined-variables
只要make发现有未定义的变量，那么就输出警告信息。
```

### 6、make的隐含规则
（1）在make的“隐含规则库”中，每一条隐含规则都在库中有其顺序，越靠前的则是越被经常使用的
```cpp
# 如以下命令：
foo.o : foo.p

# 在隐含规则中，Pascal的规则出现在C的规则之后
# 如果目录下存在了 foo.c 文件，隐含规则一样会生效，会通过 foo.c 调用C的编译器生成 foo.o 文件。而不会使用foo.p
```
（2）隐含规则一览
- 编译C程序隐含规则
  
  `<n>.o` 的目标的依赖目标会自动推导为 <n>.c ，并且其生成命令是 $(CC) –c $(CPPFLAGS) $(CFLAGS)

- 编译C++隐含规则
  
  `<n>.o` 的目标的依赖目标会自动推导为 <n>.cc 或是 <n>.C ，并且其生成命令是 $(CXX) –c $(CPPFLAGS) $(CXXFLAGS) 。（建议使用 .cc 作为C++源文件的后缀，而不是 .C ）

- 编译Pascal程序的隐含规则
  
  `<n>.o`的目标的依赖目标会自动推导为 <n>.p ，并且其生成命令是 $(PC) –c  $(PFLAGS)

- 编译Fortran/Ratfor程序的隐含规则
  
  `<n>.o` 的目标的依赖目标会自动推导为 <n>.r 或 <n>.F 或 <n>.f ，并且其生成命令是:

  .f $(FC) –c  $(FFLAGS)

  .F $(FC) –c  $(FFLAGS) $(CPPFLAGS)

  .f $(FC) –c  $(FFLAGS) $(RFLAGS)

- 预处理Fortran/Ratfor程序的隐含规则
  
  `<n>.f` 的目标的依赖目标会自动推导为 `<n>.r` 或 `<n>.F` 。这个规则只是转换Ratfor 或有预处理的Fortran程序到一个标准的Fortran程序。其使用的命令是：
  .F $(FC) –F $(CPPFLAGS) $(FFLAGS)

  .r $(FC) –F $(FFLAGS) $(RFLAGS)

- 编译Modula-2程序的隐含规则

  `<n>.sym` 的目标的依赖目标会自动推导为 `<n>.def` ，并且其生成命令是： $(M2C) $(M2FLAGS) $(DEFFLAGS) 。 
  
  `<n>.o` 的目标的依赖目标会自动推导为 `<n>.mod` ，并且其生成命令是： $(M2C) $(M2FLAGS) $(MODFLAGS) 

- 汇编和汇编预处理的隐含规则。

  `<n>.o` 的目标的依赖目标会自动推导为 `<n>.s` ，默认使用编译器 as ，并且其生成命令是： $ (AS) $(ASFLAGS) 。
  
  ` <n>.s` 的目标的依赖目标会自动推导为 `<n>.S` ，默认使用C预编译器 cpp ，并且其生成命令是： $(AS) $(ASFLAGS)

- 链接Object文件的隐含规则。

    `<n>` 目标依赖于 `<n>.o` ，通过运行C的编译器来运行链接程序生成（一般是 ld ），其生成命令是： $(CC) $(LDFLAGS) `<n>.o` $(LOADLIBES) $(LDLIBS) 。这个规则对于只有一个源文件的工程有效，同时也对多个Object文件（由不同的源文件生成）的也有效。例如如下规则
    ```cpp
    x : y.o z.o

    # 当 x.c 、 y.c 和 z.c 都存在时，隐含规则将执行如下命令:

    cc -c x.c -o x.o
    cc -c y.c -o y.o
    cc -c z.c -o z.o
    cc x.o y.o z.o -o x
    rm -f x.o
    rm -f y.o
    rm -f z.o
    
    ```

（3）隐含规则使用的变量

make的 -R 或 --no–builtin-variables 参数来取消你所定义的变量对隐含规则的作用

- **命令相关变量**
```cpp
AR : 函数库打包程序。默认命令是 ar

AS : 汇编语言编译程序。默认命令是 as

CC : C语言编译程序。默认命令是 cc

CXX : C++语言编译程序。默认命令是 g++

CO : 从 RCS文件中扩展文件程序。默认命令是 co

CPP : C程序的预处理器（输出是标准输出设备）。默认命令是 $(CC) –E

FC : Fortran 和 Ratfor 的编译器和预处理程序。默认命令是 f77

GET : 从SCCS文件中扩展文件的程序。默认命令是 get

LEX : Lex方法分析器程序（针对于C或Ratfor）。默认命令是 lex

PC : Pascal语言编译程序。默认命令是 pc

YACC : Yacc文法分析器（针对于C程序）。默认命令是 yacc

YACCR : Yacc文法分析器（针对于Ratfor程序）。默认命令是 yacc –r

MAKEINFO : 转换Texinfo源文件（.texi）到Info文件程序。默认命令是 makeinfo

TEX : 从TeX源文件创建TeX DVI文件的程序。默认命令是 tex

TEXI2DVI : 从Texinfo源文件创建军TeX DVI 文件的程序。默认命令是 texi2dvi

WEAVE : 转换Web到TeX的程序。默认命令是 weave

CWEAVE : 转换C Web 到 TeX的程序。默认命令是 cweave

TANGLE : 转换Web到Pascal语言的程序。默认命令是 tangle

CTANGLE : 转换C Web 到 C。默认命令是 ctangle

RM : 删除文件命令。默认命令是 rm –f
```

- **命令参数相关的变量**
```cpp
ARFLAGS : 函数库打包程序AR命令的参数。默认值是 rv

ASFLAGS : 汇编语言编译器参数。（当明显地调用 .s 或 .S 文件时）

CFLAGS : C语言编译器参数。

CXXFLAGS : C++语言编译器参数。

COFLAGS : RCS命令参数。

CPPFLAGS : C预处理器参数。（ C 和 Fortran 编译器也会用到）。

FFLAGS : Fortran语言编译器参数。

GFLAGS : SCCS “get”程序参数。

LDFLAGS : 链接器参数。（如： ld ）

LFLAGS : Lex文法分析器参数。

PFLAGS : Pascal语言编译器参数。

RFLAGS : Ratfor 程序的Fortran 编译器参数。

YFLAGS : Yacc文法分析器参数。
```

（4）隐含规则链

一个 .o 的文件生成，可能会是先被 Yacc的[.y]文件先成 .c ，然后再被C的编译器生成
.c被称为中间目标，其最终会被rm -f 删除。

可以使用`.INTERMEDIATE`强制声明中间目标。如：.INTERMEDIATE : mid

可以使用伪目标 `.SECONDARY` 来强制声明阻止自动删除中间目标（如： .SECONDARY : sec ）

也可以以模式的方式来指定（如： %.o ）成伪目标 .PRECIOUS 的依赖目标，阻止删除

（5）定义模式规则
- 模式规则中的自动化变量
  ```cpp
  $@ : 表示规则中的目标文件集。在模式规则中，如果有多个目标，$@ 就是匹配于目标中模式定义的集合。

  $% : 仅当目标是函数库文件中，表示规则中的目标成员名。如一个目标是 foo.a(bar.o) , $% 就是 bar.o ， $@ 就是 foo.a 。如果目标不是函数库文件（Unix是 .a ，Windows是 .lib ），其值为空。

  $< : 依赖目标中的第一个目标名字。如果依赖目标是以模式（即 % ）定义的，那么 $< 将是符合模式的一系列的文件集。其是一个一个取出来的。

  $? : 所有比目标新的依赖目标的集合。以空格分隔。

  $^ : 所有的依赖目标的集合。以空格分隔。这个变量会去除重复的依赖目标，只保留一份。

  $+ : 所有依赖目标的集合。但是不去除重复的依赖目标。

  $* : 表示目标模式中 % 及其之前的部分。如目标是 dir/a.foo.b ，并且目标的模式是 a.%.b ，那么， $* 的值就是 dir/foo 。
     这个变量对于构造有关联的文件名是比较有效。如果目标中没有模式的定义，那么 $* 也就不能被推导出，但如果目标文件的后缀是make所识别的，那么 $* 就是除了后缀的那一部分。
     如目标是 foo.c ，因为 .c 是make所能识别的后缀名，所以， $* 的值就是 foo 。、
     这个特性是GNU make的，很有可能不兼容于其它版本的make，应该尽量避免使用 $* ，除非是在隐含规则或是静态模式中。如果目标中的后缀是make所不能识别的，那么 $* 就是空值。
  
  ```


## 八、参考教程

[跟我一起写Makefile](https://seisman.github.io/how-to-write-makefile/introduction.html)

[makefile文件的编写](https://www.kancloud.cn/dlover/note/1644663)

[从零开始快速编写Makefile](https://zhuanlan.zhihu.com/p/430029724)

