# CMakeLists编写文档
## 一、构造流程
### 1、基础结构

```cpp
在需要进行编译的文件夹内编写CMakeLists.txt，即含有.cpp/.c/.cc的文件夹内
project(xxx)                                          #必须

add_subdirectory(子文件夹名称)                         #父目录必须，子目录不必

add_library(库文件名称 STATIC 文件)                    #通常子目录(二选一)
add_executable(可执行文件名称 文件)                     #通常父目录(二选一)
dangqiuan 
include_directories(路径)                              #必须
link_directories(路径)                                 #必须

target_link_libraries(库文件名称/可执行文件名称 链接的库文件名称)       #必须
此外，就是set变量语句，if判断语句，以及一些编译选项
```

至少要包含以下内容：
```cpp
cmake_minimum_required (VERSION 3.5)
project (Demo)
add_executable(Demo demo.cxx)
```
### 2、添加版本号 & 配置头文件
```cpp
cmake_minimum_required (VERSION 3.5)
project (Demo)
add_executable(Demo demo.cxx)

set (Tutorial_VERSION_MAJOR 1)
set (Tutorial_VERSION_MINOR 2)
set (Tutorial_VERSION_PATCH 3)

message(STATUS ${PROJECT_NAME})

# 配置一个头文件将一些设置传入到源代码中
configure_file (
  "${PROJECT_SOURCE_DIR}/TutorialConfig.h.in"
  "${PROJECT_BINARY_DIR}/TutorialConfig.h"
  )

# 将构建目录添加到 include 的搜索路径中以便找到
# TutorialConfig.h 文件
include_directories("${PROJECT_BINARY_DIR}")

# 添加可执行文件
add_executable(Tutorial tutorial.cxx)

```
注：二者区别

[PROJECT_SOURCE_DIR与PROJECT_BINARY_DIR区别](https://juejin.cn/post/6844903999448055815)
```cpp
# 假设工程目录如下：

project
|   CMakeLists.txt
|   source1.cxx

# 如果在CMakeLists.txt所在的目录运行cmake . 的话，可以看到两个变量的值分别为：

project
|   CMakeLists.txt
|   source1.cxx

# 实际上一般成熟的工程不会把source文件与cmake生成的文件放在一起，而是会放到不同的目录中，如下：

project
|____sources
|   |   CMakeLists.txt
|   |   source1.cxx
____build

#在build目录中运行cmake ../sources/，可以看到两个变量的值分别为：

-- PROJECT_BINARY_DIR=C:/Codebase/Learnings/camkelearn3/build
-- PROJECT_SOURCE_DIR=C:/Codebase/Learnings/camkelearn3/sources
```


### 3、添加一个库
```cpp
假设现在文件目录如下
.
├── CMakeLists.txt
├── MathFunctions
│   ├── CMakeLists.txt
│   ├── MathFunctions.h
│   └── mysqrt.cxx
├── TutorialConfig.h.in
└── tutorial.cxx

# 子目录下CmakeLists.txt
add_library(MathFunctions mysqrt.cxx)

# 根目录下CMakeLists.txt
include_directories ("${PROJECT_SOURCE_DIR}/MathFunctions")
add_subdirectory (MathFunctions) 
 
# add the executable
add_executable (Tutorial tutorial.cxx)
target_link_libraries (Tutorial MathFunctions)
```

### 4、构建可选选项
```cpp
# 选择是否使用math函数
option (USE_MYMATH "Use tutorial provided math implementation" ON)

if (USE_MYMATH)
  include_directories ("${PROJECT_SOURCE_DIR}/MathFunctions")
  add_subdirectory (MathFunctions)
  set (EXTRA_LIBS ${EXTRA_LIBS} MathFunctions)
endif (USE_MYMATH)
 
# add the executable
add_executable (Tutorial tutorial.cxx)
target_link_libraries (Tutorial  ${EXTRA_LIBS})

# 同时可以在配置文件TutorialConfig.h.in中定义选项
# cmakedefine USE_MYMATH

# 然后就可在源文件c/cpp中使用
#ifdef USE_MYMATH

```

### 5、安装和测试
（1）安装
```cpp

install (TARGETS Tutorial DESTINATION bin)
install (FILES "${PROJECT_BINARY_DIR}/TutorialConfig.h"        
         DESTINATION include)

INSTALL(TARGETS myrun mylib mystaticlib
       RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
       LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
       ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
)

```

（2）测试

```cpp

include(CTest)

#define a macro to simplify adding tests, then use it
macro (do_test arg result)
  add_test (TutorialComp${arg} Tutorial ${arg})
  set_tests_properties (TutorialComp${arg}
    PROPERTIES PASS_REGULAR_EXPRESSION ${result})
endmacro (do_test)
 
# do a bunch of result based tests
do_test (25 "25 is 5")
do_test (-25 "-25 is 0")

```

### 6、流程和循环

（1）在if条件表达式中，是不必用${var}来取变量的值的，可以直接写var
```cpp

if(expression)
  # then section.
  COMMAND1(ARGS ...)
  COMMAND2(ARGS ...)
  #...
elseif(expression2)
  # elseif section.
  COMMAND1(ARGS ...)
  COMMAND2(ARGS ...)
  #...
else(expression)
  # else section.
  COMMAND1(ARGS ...)
  COMMAND2(ARGS ...)
  #...
endif(expression)

```
（2）foreach循环，需要用${var}取变量值，可以使用break和continue
```cpp

foreach(loop_var arg1 arg2 ...)  /  foreach(loop_var RANGE total)  /  foreach(loop_var RANGE start stop [step])
  COMMAND1(ARGS ...)
  COMMAND2(ARGS ...)
  ...
endforeach(loop_var)

```
（3）while循环，需要用${var}取变量值，可以使用break和continue
```cpp

while(condition)
  COMMAND1(ARGS ...)
  COMMAND2(ARGS ...)
  ...
endwhile(condition)

```

（4）option与if结合
```cpp

include(${CMAKE_ROOT}/Modules/CMakeDependentOption.cmake)
option(USE_CURL "use libcurl" ON)
option(USE_MATH "use libm" ON)
cmake_dependent_option(DEPENT_USE_CURL "this is dependent on USE_CURL" ON "USE_CURL;NOT USE_MATH" OFF)
if(DEPENT_USE_CURL)
    message(STATUS "using lib curl")
else()
    message(STATUS "not using lib curl")
endif()

# 输出
➜  StepTest git:(master) ✗ cmake -P optionc.cmake
-- not using lib curl

```


### 7、macro宏定义 和 function函数

```cpp

macro(<name> [arg1 [arg2 [arg3 ...]]])
  COMMAND1(ARGS ...)
  COMMAND2(ARGS ...)
  ...
endmacro(<name>)

function(<name> [arg1 [arg2 [arg3 ...]]])
  COMMAND1(ARGS ...)
  COMMAND2(ARGS ...)
  ...
function(<name>)


ARGV# 是一个下标，0指向第一个参数，累加

ARGV 所有的定义时要求传入的参数

ARGN 定义时要求传入的参数以外的参数，比如定义宏（函数）时，要求输入1个，书记输入了3个，则剩下的两个会以数组形式存储在ARGN中

ARGC 传入的实际参数的个数，也就是调用函数是传入的参数个数

```

### 8、交叉编译
编译命令：cmake .. -DCMAKE_TOOLCHAIN_FILE=/path/to/toochain.cmake src_path
```cpp
# 必须存在，执行完该指令之后，CMAKE_CROSSCOMPILING 会自动被设置为TRUE
set(CMAKE_SYSTEM_NAME Linux)
​
# 可选，目标系统的处理器名，arm或者X86 
set(CMAKE_SYSTEM_PROCESSOR arm)

# 需要显式指明编译工具
set(TOOL_CHAIN_DIR /home/wh/work/cross_compile/gcc-linaro-7.3.1-2018.05-x86_64_arm-linux-gnueabihf)
set(CMAKE_C_COMPILER ${TOOL_CHAIN_DIR}/bin/arm-linux-gnueabihf-gcc)
set(CMAKE_CXX_COMPILER ${TOOL_CHAIN_DIR}/bin/arm-linux-gnueabihf-g++)

# 设置Cmake查找主路径
set(CMAKE_FIND_ROOT_PATH ${TOOL_CHAIN_DIR}/arm-none-linux-gnueabi)

set(CMAKE_FIND_ROOT_PATH_MODE_PROGRAM NEVER)
# 只在指定目录下查找库文件
set(CMAKE_FIND_ROOT_PATH_MODE_LIBRARY ONLY)
# 只在指定目录下查找头文件
set(CMAKE_FIND_ROOT_PATH_MODE_INCLUDE ONLY)
# 只在指定目录下查找依赖包
set(CMAKE_FIND_ROOT_PATH_MODE_PACKAGE BOTH)


# 其他设置
add_definitions(-D__ARM_NEON)
add_definitions(-DLINUX)
```



## 二、常用命令
```cpp
# 设置cmake的最低版本
cmake_minimum_required(VERSION 3.5)

# 本CMakeLists.txt的project名称
# 会自动创建两个变量，PROJECT_SOURCE_DIR和PROJECT_NAME
# ${PROJECT_SOURCE_DIR}：本CMakeLists.txt所在的文件夹路径
# ${PROJECT_NAME}：本CMakeLists.txt的project名称
project(xxx)

# 查找包并从外部项目加载设置
find_package(catkin REQUIRED)

# 设置指定的C++编译器版本是必须的，如果不设置，或者为OFF，则指定版本不可用时，会使用上一版本。
set(CMAKE_CXX_STANDARD_REQUIRED ON)

# 指定为C++11 版本
set(CMAKE_CXX_STANDARD 11)

# 获取路径下所有的.cpp/.c/.cc文件，并赋值给变量中
# 只会搜集当前设置语言的文件如.c/.cpp
aux_source_directory(dir VAR) 

# 配置一个头文件来传递一些CMake设置到源代码
configure_file(
    "${PROJECT_SOURCE_DIR}/TutorialConfig.h.in"
    "${PROJECT_BINARY_DIR}/TutorialConfig.h"
)

# 给文件名/路径名或其他字符串起别名，用${变量}获取变量内容
set(变量 文件名/路径/...)

# 添加编译选项
add_definitions(-Wall -g)

# 打印消息
message(消息)

# 编译子文件夹
# 在add_subdirector之前set的各个变量，在子文件夹中是可以调用的
add_subdirectory(子文件夹名称 输出目录)

# 在当前目录下生成`libsubdir.a`
add_subdireftory(subdir)

# 在output目录下生成`libsubdir.a`
add_subdireftory(subdir output)

# 设置目标属性
set_target_properties(hello PROPERTIES CLEAN_DIRECT_OUTPUT 1)
set_target_properties(test PROPERTIES LINKER_LANGUAGE CXX) // 指定C++

#自定义搜索规则
file(GLOB SRC_LIST "*.cpp")
file(GLOB_RECURSE SRC_PROTOCOL "*.cpp") #递归搜索
FILE(GLOB SRC_PROTOCOL_LIST RELATIVE "protocol" "*.cpp") # 相对protocol目录下搜索
add_library(demo ${SRC_LIST} ${SRC_PROTOCOL} ${SRC_PROTOCOL_LIST})

# 将.cpp/.c/.cc文件生成.a静态库，默认生成是静态库
# 注意，库文件名称通常为libxxx.so，在这里只要写xxx即可
add_library(common STATIC util.cpp)
add_library(common SHARED util.cpp)

# 将.cpp/.c/.cc文件生成可执行文件
add_executable(可执行文件名称 文件)

# 规定.h头文件路径
include_directories(路径)

# 规定.so/.a库文件路径
link_directories(路径)

# 对add_library或add_executable生成的文件进行链接操作
# 注意，库文件名称通常为libxxx.so，在这里只要写xxx即可
target_link_libraries(库文件名称/可执行文件名称 链接的库文件名称)
```

## 三、一些命令的详解
### 1、`add_definitions` & `add_compile_options` & `target_compile_definitions` & `CMAKE_CXX_FLAGS`

```cpp

#为当前以下层路径的所有源文件和target增加编译定义，编译选项是对所有编译器都生效
add_definitions(-DABC)

add_definitions(ABC)

# 为指定target增加编译定义
target_compile_definitions(target PUBLIC ABC)

# 单独设置C++或C的编译选项，编译选项放在“”内，同时要将“${CMAKE_C_FLAGS}字段保留
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Werror") 
set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS}-Werror ")    

```

### 2、find_package()
#### （1）cmake官方预定义了许多寻找依赖包的Module,每个以Find.cmake命名的文件都可以帮我们找到一个包。

[Find Modules](https://cmake.org/cmake/help/latest/manual/cmake-modules.7.html)

```cpp
# 对于官方定义好的包，查找如下
find_package(CURL)
add_executable(curltest curltest.cc)
if(CURL_FOUND)
    target_include_directories(clib PRIVATE ${CURL_INCLUDE_DIR})
    target_link_libraries(curltest ${CURL_LIBRARY})
else(CURL_FOUND)
    message(FATAL_ERROR ”CURL library not found”)
endif(CURL_FOUND)

# 对于未定义的包，可以先自己编译安装，然后再按照上述模式查找
```

#### （2）find_package的两种模式：Module模式与Config模式
```cpp

Module模式
cmake需要找到一个叫做Find<LibraryName>.cmake的文件。这个文件负责找到库所在的路径，为我们的项目引入头文件路径和库文件路径。cmake搜索这个文件的路径有两个，一个是上文提到的cmake安装目录下的share/cmake-<version>/Modules目录，另一个使我们指定的CMAKE_MODULE_PATH的所在目录

Config模式
如果Module模式搜索失败，没有找到对应的Find<LibraryName>.cmake文件，则转入Config模式进行搜索。它主要通过<LibraryName>Config.cmake or <lower-case-package-name>-config.cmake这两个文件来引入我们需要的库。以我们刚刚安装的glog库为例，在我们安装之后，它在/usr/local/lib/cmake/glog/目录下生成了glog-config.cmake文件，而/usr/local/lib/cmake/<LibraryName>/正是find_package函数的搜索路径之一。
```

### 3、file命令
（1）文件写入与追加
```cpp
# 将content写入或者追加到filename中
file(WRITE <filename> <content>...)
file(APPEND <filename> <content>...)

```

（2）文件读取
```cpp
# OFFSET读取的位置， LIMIT读取的长度， HEX是否以16进制方式读取
file(READ <filename> <variable>
     [OFFSET <offset>] [LIMIT <max-in>] [HEX])

# 将filename中的字符串读取到variable中
# variable是一个list，每个元素保存每行的内容
# 不读取二进制文件，换行符会被忽略
file(STRINGS <filename> <variable> [<options>...])
```

（3）提取文件的hash码
```cpp
# MD5,SHA1,SHA224,SHA256,SHA384,SHA512,SHA3_224,SHA3_256,SHA3_384,SHA3_512
file(<HASH> <filename> <variable>)

file(SHA256 test.txt hash)
message(STATUS ${hash})

```

（4）收集文件
```cpp
# 匹配文件，以字典顺序排序
set(CUR ${CMAKE_CURRENT_SOURCE_DIR})
file(GLOB files  LIST_DIRECTORIES false RELATIVE ${CUR}/.. *)
foreach(file IN LISTS files)
    message(STATUS ${file})
endforeach(file)
# 输出
-- Stepfile/filelist.cmake
-- Stepfile/hash.cmake
-- Stepfile/string.cmake
-- Stepfile/test.txt
-- Stepfile/write.cmake

#列出当前所有文件、子文件夹中的文件以及链接中的文件
set(CUR ${CMAKE_CURRENT_SOURCE_DIR})
file(GLOB_RECURSE files FOLLOW_SYMLINKS LIST_DIRECTORIES true RELATIVE ${CUR}/.. *)
foreach(file IN LISTS files)
    message(STATUS ${file})
endforeach(file)

```

（5）对文件的操作
```cpp
# 重命名文件或者文件夹
file(RENAME <oldname> <newname>)

# 删除指定的文件
file(REMOVE [<files>...])
# 删除文件和文件夹
file(REMOVE_RECURSE [<files>...])

# 递归创建文件，包括路径中的文件夹
file(MAKE_DIRECTORY [<directories>...])

```

### 4、install命令
install用于指定在安装时运行的规则。它可以用来安装很多内容，可以包括目标二进制、动态库、静态库以及文件、目录、脚本等

install的默认目录为DESTINATION为/usr/local（ Windows：c:/Program Files/${PROJECT_NAME}）
若要改变可以有以下方法：
- set(CMAKE_INSTALL_PREFIX "${CMAKE_BINARY_DIR}/install" CACHE STRING "The path to use for make install" FORCE)
- cmake -DCMAKE_INSTALL_PREFIX=./install ..
- make DESTDIR=./install

```cpp

install(TARGETS <target>... [...])
install({FILES | PROGRAMS} <file>... [...])
install(DIRECTORY <dir>... [...])
install(SCRIPT <file> [...])
install(CODE <code> [...])
install(EXPORT <export-name> [...])

```
（1）目标文件的安装
目标文件有以下几种：
| 目标文件 | 内容 | 安装目录变量 | 默认安装文件夹 | 
| :-----| ----: | :----: | :----: |
| ARCHIVE | 静态库 | ${CMAKE_INSTALL_LIBDIR} | lib |
| LIBRARY | 动态库 | ${CMAKE_INSTALL_LIBDIR} | lib |
| RUNTIME | 可执行二进制文件 | ${CMAKE_INSTALL_BINDIR} | bin |
| PUBLIC_HEADER | 与库关联的PUBLIC头文件 | ${CMAKE_INSTALL_INCLUDEDIR}	 | include |
| PRIVATE_HEADER | 与库关联的PRIVATE头文件 | ${CMAKE_INSTALL_INCLUDEDIR}	 | include |

```cpp

INSTALL(TARGETS myrun mylib mystaticlib
       RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
       LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
       ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
)

# configurations 参数：指定安装规则适用的构建配置列表(DEBUG或RELEASE等)
install(TARGETS target
        CONFIGURATIONS Debug
        RUNTIME DESTINATION Debug/bin)
install(TARGETS target
        CONFIGURATIONS Release
        RUNTIME DESTINATION Release/bin)

```

（2）普通文件的安装

```cpp

# FILES为普通的文本文件，PROGRAMS指的是非目标文件的可执行程序(如脚本文件)
# PERMISSIONS参数：指定文件的权限
# 默认情况下，普通的文本文件将具有OWNER_WRITE，OWNER_READ，GROUP_READ和WORLD_READ权限，即644权限
# 而非目标文件的可执行程序将具有OWNER_EXECUTE, GROUP_EXECUTE,和WORLD_EXECUTE，即755权限。
install (FILES|PROGRAMS "${PROJECT_BINARY_DIR}/TutorialConfig.h"        
         DESTINATION include)
```

（3）目录的安装

```cpp
# 将icons 目录安装到 <prefix>/share/myproj
# 将 scripts/中的内容安装到<prefix>/share/myproj
# PATTERN用于使用正则表达式进行过滤，表示不包含目录名为CVS的目录
# 对于scripts'/*'指定权限
INSTALL(DIRECTORY icons scripts/ DESTINATION share/myproj
PATTERN "CVS" EXCLUDE
PATTERN "scripts/*"
PERMISSIONS OWNER_EXECUTE OWNER_WRITE OWNER_READ
GROUP_EXECUTE GROUP_READ)

```

（4）安装时脚本的运行

```cpp
# SCRIPT参数将在安装过程中调用给定的CMake脚本文件(即.cmake脚本文件)，
# CODE参数将在安装过程中调用给定的CMake代码
install([[SCRIPT <file>] [CODE <code>]]
        [COMPONENT <component>] [EXCLUDE_FROM_ALL] [...])

install(CODE "MESSAGE(\"Sample install message.\")")
```

5、list命令

list命令即对列表的一系列操作

Cmake中的列表变量是用分号;分隔的一组字符串，创建列表可以使用set命令，例如：set (var a b c d)创建了一个列表 "a;b;c;d"，而set (var "a b c d")则是只创建了一个变量"a c c d"。

（1）获取长度
```cpp
# 获取列表长度
set (list_test a b c d) 
list (LENGTH list_test length)
message (">>> LENGTH: ${length}")

# 输出
>>> LENGTH: 4
```

（2）读取列表中指定索引的的元素
```cpp

set (list_test a b c d) # 创建列表变量"a;b;c;d"
list (GET list_test 0 1 -1 -2 list_new)
message (">>> GET: ${list_new}")

# 输出
>>> GET: a;b;d;c

```

（3）连接字符
```cpp

set (list_test a b c d) # 创建列表变量"a;b;c;d"
list (JOIN list_test -G- list_new)
message (">>> JOIN: ${list_new}")

# 输出
>>> JOIN: a-G-b-G-c-G-d
```

（4）移除元素
```cpp

set (list_test a a b c c d) # 创建列表变量"a;a;b;c;c;d"
list (REMOVE_ITEM list_test a)
message (">>> REMOVE_ITEM: ${list_test}")

# 输出
>>> REMOVE_ITEM: b;c;c;d

```
（5）还有修改，插入，查找等命令，在这篇[博客](https://www.jianshu.com/p/89fb01752d6f)

## 四、其他语法内容
### 1、Cmake文件格式
cmake能识别CMakeLists.txt和*.cmake格式的文件。cmake能够以三种方式 来组织文件：
```cpp
Dierctor（文件夹） CMakeLists.txt 

Script（脚本） <script>.cmake 

cmake -P <script>.cmake
 
Module（模块） <module>.cmake
```

### 2、编译选项汇总
[gcc警告选项汇总](https://blog.csdn.net/qq_17308321/article/details/79979514)

- -rdynamic编译选项通知链接器将所有符号添加到动态符号表中。（目的是能够通过使用 dlopen 来实现向后跟踪）

- -O3是一个优化选项，告诉编译器优化我们的代码。

- -fPIC 作用于编译阶段，告诉编译器产生与位置无关代码，即，产生的代码中，没有绝对地址，全部使用相对地址，故而代码可以被加载器加载到内存的任意位置，都可以正确的执行。这正是共享库所要求的，共享库被加载时，在内存的位置不是固定的。

- -ggdb选项使编译器生成gdb专用的更为丰富的调试信息。

        -g
        该选项可以利用操作系统的“原生格式（native format）”生成调试
        信息。GDB 可以直接利用这个信息，其它调试器也可以使用这个调试信息

        -ggdb
        使 GCC 为 GDB 生成专用的更为丰富的调试信息，但是，此时就不能用其
        他的调试器来进行调试了 (如 ddx)

- -std=c++11设置为使用C++11标准

- -Wall选项告诉编译器 编译后显示所有警告。

- -Wno-deprecated不要警告使用已弃用的功能

- -Werror告诉编译器视所有警告为错误，出现任何警告立即放弃编译

- -Wno-unused-function关闭函数被定义了但没有被使用 而产生的警告，即，不使用的函数不警告。

- -Wno-builtin-macro-redefined
如果某些内置宏被重新定义，请不要警告。这抑制了警告的重新定义__TIMESTAMP__，TIME，DATE，FILE，和__BASE_FILE__。
- -Wno-deprecated-declarations关闭使用废弃API的警告

### 3、预定义变量

- PROJECT_SOURCE_DIR 工程的根目录
- PROJECT_BINARY_DIR 运行cmake命令的目录,通常是${PROJECT_SOURCE_DIR}/build
- CMAKE_INCLUDE_PATH 环境变量,非cmake变量
- CMAKE_LIBRARY_PATH 环境变量
- CMAKE_CURRENT_SOURCE_DIR 当前处理的CMakeLists.txt所在的路径
- CMAKE_CURRENT_BINARY_DIR target编译目录
- 使用ADD_SURDIRECTORY(src bin)可以更改此变量的值
- SET(EXECUTABLE_OUTPUT_PATH <新路径>)并不会对此变量有影响,只是改变了最终目标文件的存储路径
- CMAKE_CURRENT_LIST_FILE 输出调用这个变量的CMakeLists.txt的完整路径
- CMAKE_CURRENT_LIST_LINE 输出这个变量所在的行
- CMAKE_MODULE_PATH 定义自己的cmake模块所在的路径
- SET(CMAKE_MODULE_PATH ${PROJECT_SOURCE_DIR}/cmake),然后可以用INCLUDE命令来调用自己的模块
- EXECUTABLE_OUTPUT_PATH 重新定义目标二进制可执行文件的存放位置
- LIBRARY_OUTPUT_PATH 重新定义目标链接库文件的存放位置
- PROJECT_NAME 返回通过PROJECT指令定义的项目名称
- CMAKE_ALLOW_LOOSE_LOOP_CONSTRUCTS 用来控制IF ELSE语句的书写方式


## 五、一些教程
[CMake官方教程翻译](https://juejin.cn/post/6844903557183832078)

[CMake中文教程](https://github.com/BrightXiaoHan/CMakeTutorial)

[Cmake编写CmakeList.txt 语法备忘](https://blog.csdn.net/hw140701/article/details/90203141#1_CMake_1)
