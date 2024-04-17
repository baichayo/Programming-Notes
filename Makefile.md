<h1>Makefile</h1>

首先上个 GCC 编译选项

1. **基本选项**：
    - `-c`：仅编译源文件而不链接，生成目标文件。
    - `-o`：指定生成的可执行文件或目标文件的名称。
    - `-E`：仅执行预处理，生成预处理后的源代码。
    - `-S`：仅执行编译和汇编，生成汇编代码。
2. **优化选项**：
    - `-O0`, `-O1`, `-O2`, `-O3`：设置不同级别的优化级别，`-O0` 关闭优化，`-O1` 到 `-O3` 增加优化程度。
    - `-Os`：优化以减小生成的目标文件的大小。
    - `-ffast-math`：开启快速数学优化，但可能损失一些数学精度。
    - `-march` 和 `-mtune`：指定目标体系结构和目标处理器的优化。
3. **调试选项**：
    - `-g`：生成调试信息，以便在调试时使用。
    - `-ggdb`：生成更多的调试信息，以供 GDB 使用。
    - `-D`：定义预处理宏。
4. **警告选项**：
    - `-Wall`：开启大多数警告。
    - `-Werror`：将警告视为错误。
    - `-Wno-*`：禁用特定的警告。
5. **目标选项**：
    - `-m32` 和 `-m64`：指定生成 32 位或 64 位目标。
    - `-shared`：生成共享库。
    - `-static`：生成静态链接的可执行文件。
6. **其他选项**：
    - `-I`：指定头文件搜索路径。
    - `-L`：指定库文件搜索路径。
    - `-l`：链接特定的库文件。
    - `-D`：定义预处理宏。
    - `-U`：取消定义预处理宏。



### 语法

Makefile 由一组规则组成：

```makefile
targets: prerequisites
	command
	command
	command
```

- 来个简单的示例：

    ```makefile
    hello:
    	echo "Hello, World"
    ```

- 再来个，不过有缺点，如果目标文件已经存在，则不会执行命令。也就是说，在 `blah` 文件已经存在的情况下，即使之后 `blah.c` 发生改变，再执行 `makefile` ，也不会重新编译 `blah.c`

    ```makefile
    blah:
    	cc blah.c -o blah
    ```

- 因此这才是健康的示例，当 目标文件 `blah` 不存在 或 依赖文件更新时 才执行命令

    ```makefile
    blah: blah.c
        cc blah.c -o blah
    ```

- 一些例子：

    - 如果用 `make` ，那么执行默认目标（第一个目标）。首先运行目标 `blah` ，依赖是 `blah.o` ，`blah.o` 有依赖 `blah.c` ，`blah.c` 没有依赖项。因此整个运行顺序是：`blah.c`	`blah.o`	`blah`

        ```makefile
        blah: blah.o
        	cc blah.o -o blah
        
        blah.o: blah.c
        	cc -c blah.c -o blah.o
        
        blah.c:
        	echo "int main() { return 0; }" > blah.c # 重定向
        ```

    - 这个例子也看一看：主要作用在于，由于依赖项 `other_file` 不存在，所以总是会执行目标

        ```makefile
        some_file: other_file
        	echo "This will always run, and runs second"
        	touch some_file
        
        other_file:
        	echo "This will always run, and runs first"
        ```

### Make clean

`make clean` 来执行 `clean` 规则。 `clean` 通常用作删除其他目标

```makefile
some_file: 
	touch some_file

clean:
	rm -f some_file
```

### 变量

变量类型只有字符串。

来点示例

```makefile
files := data1.txt data2.txt

some_file: $(files)
	echo "Look at this variable: " $(files)
	touch some_file

data1.txt:
	touch data1.txt
data2.txt:
	touch data2.txt

clean:
	rm -f data1.txt data2.txt some_file
```

### 多个目标

当规则有多个目标时，将为每个目标运行命令

```makefile
all: f1.o f2.o

f1.o f2.o:
	echo $@
# 相当于:
# f1.o:
#	 echo f1.o
# f2.o:
#	 echo f2.o
```





### 自动变量和通配符

- `*` 通配符
  
    `* ` 在文件系统中搜索匹配的文件名。`wildcard`  是一个内置函数，用于获取满足指定模式的文件列表。
    
    ```makefile
    # Print out file information about every .c file
    print: $(wildcard *.c)
    	ls -la  $?
    ```

- `%`  通配符
  
  `%` 通常用于规则的模式匹配。可以直接用。

    ```makefile
    %.o: %.c
    	gcc -c -o $@ $<
    ```


- 自动变量

  1. **$@**：表示规则的目标（Target），即要构建的文件的名称。
  2. **$<**：表示规则中的第一个依赖项（Dependency），即触发构建的源文件的名称。
  3. **$^**：表示规则中的所有依赖项的列表，以空格分隔。
  4. **$?**：表示比目标更新的所有依赖项的列表，以空格分隔。
  5. **$(@D)**：表示规则的目标的目录部分。
  6. **$(@F)**：表示规则的目标的文件名部分。
  7. **$(notdir $@)**：表示规则的目标的文件名部分。
  8. **$(wildcard pattern)**：用于匹配满足指定模式的文件列表，可用于自动变量中。

  ```makefile
  hey: one two
  	# Outputs "hey", since this is the target name
  	echo $@
  
  	# Outputs all prerequisites newer than the target
  	echo $?
  
  	# Outputs all prerequisites
  	echo $^
  
  	touch hey
  
  one:
  	touch one
  
  two:
  	touch two
  
  clean:
  	rm -f hey one two
  ```

  

### 花哨的规则

- 隐含规则

  隐含规则列表：

  - 编译 C 程序：使用以下形式的命令`n.o`自动生成`n.c` `$(CC) -c $(CPPFLAGS) $(CFLAGS) $^ -o $@`
  - 编译 C++ 程序：由以下形式的命令`n.o`自动生成`n.cc` `n.cpp` `$(CXX) -c $(CPPFLAGS) $(CXXFLAGS) $^ -o $@`
  - 链接单个目标文件：通过运行命令`n`自动生成`n.o` `$(CC) $(LDFLAGS) $^ $(LOADLIBES) $(LDLIBS) -o $@`

  隐式规则使用的重要变量是：

  - `CC`：编译C程序的程序；默认`cc`
  - `CXX`：编译C++程序的程序；默认`g++`
  - `CFLAGS`：提供给 C 编译器的额外标志
  - `CXXFLAGS`：提供给 C++ 编译器的额外标志
  - `CPPFLAGS`：给予 C 预处理器的额外标志
  - `LDFLAGS`：当编译器应该调用链接器时提供给编译器的额外标志

  show code：

  ```makefile
  CC = gcc # Flag for implicit rules
  CFLAGS = -g # Flag for implicit rules. Turn on debug info
  
  # Implicit rule #1: blah is built via the C linker implicit rule
  # Implicit rule #2: blah.o is built via the C compilation implicit rule, because blah.c exists
  blah: blah.o
  
  blah.c:
  	echo "int main() { return 0; }" > blah.c
  
  clean:
  	rm -f blah*
  ```

- 静态模式规则

    它长这样：

    ```makefile
    targets...: target-pattern: prereq-patterns ...
       commands
    ```

    比如说你想……（我也不知道这是想干嘛

    ```makefile
    objects = foo.o bar.o all.o
    all: $(objects)
    
    # These files compile via implicit rules
    foo.o: foo.c
    bar.o: bar.c
    all.o: all.c
    
    all.c:
    	echo "int main() { return 0; }" > all.c
    
    %.c:
    	touch $@
    
    clean:
    	rm -f *.c *.o all
    ```

    你就可以这样做：

    ```makefile
    objects = foo.o bar.o all.o
    all: $(objects)
    
    $(objects): %.o: %.c
    
    # 其实可以直接这样写，你可能会有疑问，这不会匹配到全部 .o 文件吗
    # 不会啦，因为这条语句是被动执行的，也就是说需要啥文件就执行啥文件
    # %.o: %.c # 模式规则
    
    all.c:
    	echo "int main() { return 0; }" > all.c
    
    %.c:
    	touch $@
    
    clean:
    	rm -f *.c *.o all
    ```

- 静态模式规则和过滤器

    给大家表演一下过滤器的用法，比如先有这样的代码

    ```makefile
    obj_files = foo.result bar.o lose.o
    src_files = foo.raw bar.c lose.c
    
    all: $(obj_files)
    # Note: PHONY is important here. Without it, implicit rules will try to build the executable "all", since the prereqs are ".o" files.
    .PHONY: all
    
    $(obj_files): %.o: %.c
    ```

    这样写会给你无情报错“target 'foo.result' doesn't match the target pattern”。那么过滤器来咯

    ```makefile
    obj_files = foo.result bar.o lose.o
    src_files = foo.raw bar.c lose.c
    
    all: $(obj_files)
    # Note: PHONY is important here. Without it, implicit rules will try to build the executable "all", since the prereqs are ".o" files.
    .PHONY: all
    
    # 正确写法在此 只放行 %.o
    $(filter %.o, $(obj_files)): %.o: %.c
    	echo "target: $@ prereq: $<"
    
    $(filter %.result, $(obj_files)): %.result: %.raw
    	echo "target: $@ prereq: $<"
    
    %.c %.raw:
    	touch $@
    
    clean:
    	rm -f $(src_files)
    ```

    其他过滤器有：

    ```makefile
    # $(filter-out pattern, text) 从文本中排除与指定模式匹配的部分
    
    sources := file1.c file2.cpp file3.txt
    # non_cpp_sources 现在包含 file1.c file3.txt
    non_cpp_sources := $(filter-out %.cpp, $(sources))
    
    
    # $(wildcard pattern) 返回与指定模式匹配的文件列表
    
    sources := $(wildcard src/*c) # source_files 包含 src 目录下的所有 .c 文件
    
    
    # $(patsubst pattern, replacement, text)
    # 这个过滤器用于将文本中的模式替换为指定的字符串。
    
    files := file1.c file2.cpp file3.txt
    # obj_files 现在包含 file1.o file2.cpp file3.txt
    obj_files := $(patsubst %.c %.o, $(files))
    
    
    # $(sort list) 对文本中的单词进行排序，并去除重复的单词
    
    names := Alice Bob Carol Bob Dave
    # sorted_names 包含 Alice Bob Carol Dave
    sorted_names := $(sort $(names))
    ```

- 模式规则

    基本语法：（好像在前面已经被用烂了呀）

    ```makefile
    %.target: %.source
        command
    ```

    例子：

    ```makefile
    # 源文件列表
    MD_FILES := file1.md file2.md file3.md
    # 生成目标文件列表（HTML文件）
    HTML_FILES := $(patsubst %.md %.html, $(MD_FILES))
    # 默认目标
    all: $(HTML_FILES)
    
    # 模式规则：将Markdown文件转换为HTML文件
    %.html: %.md
    	pandoc $< -o $@
    
    # 清理生成的文件
    clean:
    	rm -f $(HTML_FILES)
    
    # .PHONY 防止出现同名目录导致出现错误
    # 加上这句话，即使有 all clean 文件，这些目标也会被执行
    .PHONY: all clean
    ```

- 双冒号规则

    允许为同一目标定义多个规则。如果这些是单冒号，则会打印警告，并且只会运行第二组命令。

    ```makefile
    all: blah
    
    blah::
    	echo "hello"
    
    blah::
    	echo "hello again"
    ```

    

### 命令和执行

- 命令回显/静音

    默认情况下，make工具会显示正在执行的命令。不想输出的话就加个 `@`

    ```makefile
    all:
    	@echo "hello, make"
    ```

- 默认 shell

    默认 shell 是 `/bin/sh` ，可以通过变量 `SHELL` 来更改

    ```makefile
    SHELL=/bin/bash
    
    cool:
    	echo "Hello from bash"
    ```

- `$$` 是用来转义字符 `$`。额，还涉及 shell 变量。不太懂

    ```makefile
    make_var = I am a make variable
    all:
    	# Same as running "sh_var='I am a shell variable'; echo $sh_var" in the shell
    	sh_var='I am a shell variable'; echo $$sh_var
    
    	# Same as running "echo I am a make variable" in the shell
    	echo $(make_var)
    ```

- 递归使用make

    要递归调用 makefile，使用$(MAKE)`。（看不懂哇看不懂

    ```makefile
    new_contents = "hello:\n\ttouch inside_file"
    all:
    	mkdir -p subdir
    	printf $(new_contents) | sed -e 's/^ //' > subdir/makefile
    	cd subdir && $(MAKE)
    
    clean:
    	rm -rf subdir
    ```



### 条件

- if/else

    ```makefile
    foo = ok
    
    all:
    ifeq ($(foo), ok)
    	echo "foo equals ok"
    else
    	echo "nope"
    endif
    ```

- 检查变量是否为空

    ```makefile
    nullstring =
    foo = $(nullstring) # end of line; there is a space here
    
    # nullstring 是 空字符串，foo 包含一个空格（空格个屁，一样是空，我被骗了）
    # strip 删除前后空格
    
    all:
    ifeq ($(strip $(foo)),)
    	echo "foo is empty after being stripped"
    endif
    ifeq ($(nullstring),)
    	echo "nullstring doesn't even have spaces"
    endif
    ```

- 检查变量是否定义

    ```makefile
    bar = 
    foo = $(bar)
    
    all:
    ifdef foo # 都不用写成 $(foo)
    	echo "foo is defined"
    endif
    ifndef bar
    	echo "bar is undefined"
    endif
    ```

- $(MAKEFLAGS)

    ```makefile
    all:
    # Search for the "-i" flag. MAKEFLAGS is just a list of single characters, one per flag. So look for "i" in this case.
    ifneq (,$(findstring i, $(MAKEFLAGS)))
    	echo $(findstring i, $(MAKEFLAGS))
    	echo "i was passed to MAKEFLAGS"
    endif
    ```



### 函数

函数主要用于文本处理。`$(fn, arguments)` 或 `${fn, arguments}`

- `$(subst find,replace,text)` ：在字符串中执行替换操作

    ```makefile
    comma := ,
    empty:=
    space := $(empty) $(empty)
    foo := a b c
    bar := $(subst $(space),$(comma),$(foo)) # 将 空格 替换为 ，
    
    all: 
    	@echo $(bar)
    ifeq ($(strip $(space)))
    ```

    注意函数的 arguments 之间不要加空格，这会被视为字符串的一部分

    ```makefile
    comma := ,
    empty:=
    space := $(empty) $(empty)
    foo := a b c
    bar := $(subst $(space), $(comma) , $(foo))
    
    all: 
    	# Output is ", a , b , c". Notice the spaces introduced
    	@echo $(bar)
    ```

- `$(patsubst pattern,replacement,text)` ：用于在字符串中进行**模式**替换

    ```makefile
    sources := file1.c file2.cpp file3.c
    objects := $(patsubst %.c,%.o,$(sources))
    
    all:
    	@echo $(objects) # output: file1.o file2.cpp file3.o
    ```

- `$(foreach var, list, text)` ：

    - `var`：循环中的临时变量名称，用于代表列表中的每个元素。
    - `list`：要迭代的列表，可以包含多个元素，用空格或逗号分隔。
    - `text`：对每个元素执行的操作，通常包含使用 `$(var)` 来引用当前元素的值

    ```makefile
    # 定义一个列表
    fruits := apple banana cherry
    
    # 使用foreach迭代处理列表中的每个水果
    processed_fruits := $(foreach fruit,$(fruits),Eat $(fruit);)
    
    all:
        @echo $(processed_fruits)
        # output: Eat apple; Eat banana; Eat cherry;
    ```

- `$(if condition, then-part, else-part)` ：如果条件为真（非空字符串），则执行 `then-part`；否则执行 `else-part`

    ```makefile
    file := myfile.txt
    exists := $(wildcard $(file))
    result := $(if $(exists), File exists, File does not exist)
    ```

    