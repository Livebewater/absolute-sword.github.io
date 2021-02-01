---
title: learn makefile
date: 2020/2/1/23:30
tags: c/c++
categories: c/c++
---

```bash
g++ -c func1.cpp
g++ -c func2.cpp
g++ -c main.cpp 
g++ -o func main.o fun1.o fun2.o
```

使用g++ -c 进行编译为对象文件，使用g++ -o来进行链接，链接时的文件顺序无要求

```text
g++ -o hello main.cpp function1.cpp function2.cpp
```

也可以一步到位

为了解决：

1. 减少命令数，直接写成一个文件来进行执行
2. 避免每次修改文件之后都需要重新将所有文件进行编译，让make通过时间戳看文件是否改动过，改动过才需要重新编译

另外，在clion中，会将所有文件名为makefile及其变体（任意字母转为大小写）都视为makefile文件，但是在执行make的时候还是只会识别makefile和Makefile文件

 <target>:<dependencies>

```makefile
# the 1-ed
all:
   g++ -o func main.cpp func1.cpp func2.cpp
clean:
   rm -rf *.o func
```

all和clean是一个target，使用make clean和make all就可以激活对应target，分别执行对应的依赖（指令）



```makefile
# the 2-ed
all: func
func: main.o func1.o func2.o
      g++ main.o func1.o func2.o -o func
main.o: main.cpp
      g++ -c main.cpp
func1.o: func1.cpp
      g++ -c func1.cpp
func2.o: func2.cpp
      g++ -c func2.cpp
clean:
      rm -rf *.o func
```

执行make时，等价于make all（等价于执行最上面的一条target），此时all的target依赖于func，但是本地没有func，make继续往下找func，func的target依赖于main.o，本地也没有，就找到main.o的target就是main.cpp，存在本地，继续完成该依赖即g++ -c main.cpp，之后一级一级返回

在第一次执行make时，输出会显示所有文件都被编译了一遍，之后如果修改了某个文件，只会编译有变化的文件，如果没修改执行make，会显示make没有任何改动



```makefile
# the 3-ed

cc = g++
CFLAGS = -c -Wall # -Wall是显示所有的warning
OFLAGS = -Wall
all: func
func: main.o func1.o func2.o
      $(cc) $(OFLAGS) main.o func1.o func2.o -o func
main.o: main.cpp
      $(cc) $(CFLAGS) main.cpp
func1.o: func1.cpp
      $(cc) $(CFLAGS) func1.cpp
func2.o: func2.cpp
      $(cc) $(CFLAGS) func2.cpp
clean:
      rm -rf *.o func
```

添加了一些变量，进行简化





```makefile
# the 4-ed
cc = g++
CFLAGS = -c -Wall # -Wall是显示所有的warning
OFLAGS = -Wall
all: func
func: main.o func1.o func2.o
      $(cc) $(OFLAGS) $^ -o $@
main.o: main.cpp
      $(cc) $(CFLAGS) $<
func1.o: func1.cpp
      $(cc) $(CFLAGS) $<
func2.o: func2.cpp
      $(cc) $(CFLAGS) $<
clean:
      rm -rf *.o func
```



引入\$  ^代表所有的依赖， \$<代表第一个依赖（对于单参数如func1.o的，\$<和\$^都可以）， $@代表target



```makefile
# the 5-ed
cc = g++
CFLAGS = -c -Wall
OFLAGS = -Wall
SOURCE_DIR = .
SOURCE_FILE = $(wildcard $(SOURCE_DIR)/*.cpp) # 用于获取路径下指定模式的文件
OBJS = $(patsubst %.cpp, %.o, $(SOURCE_FILE)) # 用于从一个模式的文件更换为另一个模式文件
all: func
func: main.o func1.o func2.o
      $(cc) $(OFLAGS) $^ -o $@
main.o: main.cpp
      $(cc) $(CFLAGS) $<
func1.o: func1.cpp
      $(cc) $(CFLAGS) $<
func2.o: func2.cpp
      $(cc) $(CFLAGS) $<
clean:
      @rm -rf *.o func 
```



wildcard 用于获取路径下指定模式的文件

patsubst (pattern subsitution)用于从一个模式的文件更换为另一个模式文件

rm处加入@可以避免回显，否则执行clean时也会输出rm命令

可以注意到main.o,func1.o,func2.o基本上都是一样的，只是换了一个名字而已



```makefile
# the 6-ed
cc = g++
CFLAGS = -c -Wall
OFLAGS = -Wall
SOURCE_DIR = .
SOURCE_FILE = $(wildcard $(SOURCE_DIR)/*.cpp)
OBJS = $(patsubst %.cpp, %.o, $(SOURCE_FILE))
all: func
func: $(OBJS)
      $(cc) $(OFLAGS) $^ -o $@
$(OBJS):%.o:$.cpp
      $(cc) $(CFLAGS) $<
clean:
      @rm -rf *.o func
```

Static Pattern Rule

targets: target-pattern: prereq-patterns

即将本来的单个target替换成一组target，将具有类型模式的都归为一组

target-paatern即target文件的模式，prereq-patterns即依赖文件的模式，每个文件都是一一对应的



另外，使用变量的时候，对于单个字母命名的变量不需要使用括号即可引用，但是对于大于1个字母的就都要使用括号

使用

```makefile
objs = xxx\
	yyy\
	zzz\
```

可以定义一个类似数组的结构，之后可以使用OBJS+=ppp.o来添加
