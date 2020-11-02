---
title: "Makefile"
date: 2020-10-28T13:41:42+08:00
draft: true
tags: ["makefile"]
categories: ["编译原理"]
---

### 1. 概述
makefile定义了一系列的规则来指定、哪些文件需要先编译、哪些文件需要后编译。哪些文件需要重新编译。类似shell命令。带来的好处是自动编译。

### 2. 程序的编译和链接
![](/images/compile_theory/00001.jpeg)
总结：源文件首先会生成中间目标文件、再由中间目标文件生成执行文件。在编译时、编译器只检测程序语法、和函数、变量是否被声明、如果函数未被声明、编译器就会给出一个警告、但可以生成Object File。而在链接程序时。链接器会在所有object file中寻找函数的实现、如果找不到、那就会报链接错误码。
### 3. Makefile介绍
1. make命令执行时、需要一个Makefile文件、以告诉命令是怎么去编译和链接程序
2. Makefile的规则
	```shell
	target...:prerequisites...
	command
	...
	...
	#target:就是目标文件、可以是Object File、也可以是执行文件、还可以是个标签
	#prerequisites: 要生成那个目标文件所需的文件或者是目标
	#command:make需要执行的命令
    ```
	这是一个文件的依赖关系，也就是说target这一个或多个目标文件依赖于prerequisites中的文件、其生成规则定义在command中。说白一点就是prerequisites 中如果有一个以上的文件比target文件要新的话，command 所定义的命令就会被执行。这就是Makefile的规则。也就是 Makefile中核心的内容。 

3. 示例
	```shell
	edit : main.o kbd.o command.o
	cc -o edit main.o kbd.o command.o
	main.o : main.c defs.h
	cc -c main.c
	kbd.o : kbd.c defs.h command.h
	cc -c kbd.c
	command.o : command.c defs.h command.h	
	cc -c command.c
	files.o : files.c defs.h buffer.h command.h
	cc -c files.c
	clean :
	rm edit main.o kbd.o command.o display.o insert.o search.o files.o utils.o 
    ```

4. make是如何工作的
	1. 默认方式下、我们输入make命令、那么
		1. make命令会在当前目录下查找Makefile或者makefile的文件
		2. 如果找到、他会找文件中的第一个目标文件(target)、上面的例子中、他会找到"edit"这个文件、并把这个文件作为最终的目标文件
		3. 如果edit文件不存在、或者edit所依赖的后面的.o文件要比.edit文件要新、那么就会执行后面所定义的命令来生成edit这个文件
		4. 如果edit依赖的.o文件也存在、那么make会在当前文件中找目标为.o文件的依赖性，如果找到则再根据那一个规则生成.o文件。（这有点像一个堆栈的过程） 
		5. 当然，你的C文件和H文件是存在的于是make会生成 .o文件，然后再用.o文件生成make的终极任务，也就是执行文件edit
	2. 可以手动执行 make clean来清除所有目标文件

5. Makefile中使用变量
	1. 定义：obj = main.o kbd.o command.o
	2. 使用：edit: $(obj)

6. 让make自动推导
GNU的make可以自动推导文件以及文件依赖关系后面的命令、于是就没必要去在每一个[.o]文件后都写上类似的命令，因为make会自动识别，并自己推导命令。 只要make看到一个[.o]文件，它就会自动的把[.c]文件加在依赖关系中，如果make找到一个whatever.o，那么whatever.c，就会是whatever.o的依赖文件。并且 cc -c whatever.c 也会被推导出来，于是，我们的 makefile 再也不用写得这么复杂。
	```shell
	objects = main.o kbd.o command.o display.o insert.o search.o files.o utils.o
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
	.PHONY : clean #.PHONY标示、clean是个伪目标文件
	clean :
	rm edit $(objects)   
    ```

7. 另类风格的Makefile：将.h、.c文件收拢起来
	```shell
	objects = main.o kbd.o command.o display.o  insert.o search.o files.o utils.o
	edit : $(objects)
	cc -o edit $(objects)    
	$(objects) : defs.h  
	kbd.o command.o files.o : command.h  
	display.o insert.o search.o files.o : buffer.h    
	.PHONY : clean  
	clean :  
	rm edit $(objects) 
    ```

8. 清空目标文件的规则
	```shell
	clean:  
	rm edit $(objects)
	#更稳妥的办法
	.PHONY:clean
	clean :  
	-rm edit $(objects)  #前面-号作用是：也许某些文件出问题、不用管、继续做后面的事、一般clean放最后
    ```

### 4.Makefile总述 
1. Makefile 里有什么
	1. 显式规则：显式规则说明了，如何生成一个或多的的目标文件。这是由Makefile的书写者明显指出，要生成的文件，文件的依赖文件，生成的命令。 
	2. 隐晦规则：由于我们的 make 有自动推导的功能，所以隐晦的规则可以让我们比较粗糙地简略地书写 Makefile，这是由 make 所支持的。 
	3. 变量的定义：在Makefile中我们要定义一系列的变量，变量一般都是字符串，这个有点你C语言中的宏，当Makefile被执行时，其中的变量都会被扩展到相应的引用位置上
	4. 文件指示：其包括了三个部分，一个是在一个Makefile中引用另一个Makefile，就像C语言中的include一样；另一个是指根据某些情况指定Makefile中的有效部分，就像 C 语言中的预编译#if一样；还有就是定义一个多行的命令
	5. 注释:Makefile 中只有行注释，和 UNIX 的 Shell 脚本一样，其注释是用“#”字符，这个就 像 C/C++中的“//”一样。如果你要在你的 Makefile 中使用“#”字符，可以用反斜框进行 转义，如：“\#”
	最后，还值得一提的是，在 Makefile 中的命令，必须要以[Tab]键开始
2. 引用其它的Makefile:
	1. 使用include把别的Makefile包含进来
	2. 语法：include <filename> 、注意、绝对不能是[tab]键开始、include与<filename>之间可以有多个空格
	3. 举例：有a.mk,b.mk,c.mk 还有个文件叫foo.make、一个变量$(bar)、其中包含了e.mk,f.mk那么写成
	```shell
		include foo.make *.mk $(bar)
		#等价于
		include foo.make a.mk b.mk c.mk e.mk f.mk
    ```
3. include命令查找顺序
make命令开始时，会把找寻include 所指出的其它 Makefile，并把其内容安置在当前的位。就好像 C/C++的#include指令一样。如果文件都没有指定绝对路径或是相对路径的话， make 会在当前目录下首先寻找如果当前目录下没有找到，那么make还会在下面的几个目录下找：
	1. 如果 make 执行时，有“-I”或“--include-dir”参数，那么make就会在这个参数 所指定的目录下去寻找。  
	2. 如果目录<prefix>/include（一般是：/usr/local/bin 或/usr/include存在的话，make也会去找。如果有文件没有找到的话，make 会生成一条警告信息，但不会马上出现致 命错误。它会继续载入其它的文件，一旦完成 makefile 的读取，make会再重试这些没有找到或是不能读取的文件，如果还是不行，make 才会出现一条致命信息。如果你想让 make 不理那些无法读取的文件，而继续执行，你可以在 include 前加一个减号“-”。 
	如： -include <filename>  其表示，无论 include 过程中出现什么错误，都不要报错继续执行。和其它版本 make 兼 容的相关命令是 sinclude，其作用和这一个是一样的。
4. make 的工作方式
	1. 读入所有的 Makefile。  
	2. 读入被 include 的其它 Makefile。  
	3. 初始化文件中的变量。  
	4. 推导隐晦规则，并分析所有规则。  
	5. 为所有的目标文件创建依赖关系链。  
	6. 根据依赖关系，决定哪些目标要重新生成。 
	7. 执行生成命令

### 5.书写规则
1. 规则包含两个部分
	1. 依赖关系、
	2. 生成目标的方法。
2. 在 Makefile中，规则的顺序是很重要的，因为，Makefile中只应该有一个终目标，其它的目标都是被这个目标所连带出来的，所以一定要让 make 知道你的终目标是什么。 一般来说，定义在Makefile中的目标可能会有很多，但是第一条规则中的目标将被确立为终的目标。如果第一条规则中的目标有很多个，那么，第一个目标会成为终的目标。make 所完成的也就是这个目标。 
3. 例子一:
	```shell
	foo.o:foo.c defs.h #foo 模块 ----依赖关系
	cc -c -g foo.c     #         ----生成目标的方法
    ```
    说明两件事：
    	1. 文件的依赖关系，foo.o依赖于foo.c和defs.h的文件，如果foo.c和defs.h的文件日期要比 foo.o文件日期要新，或是 foo.o不存在，那么依赖关系发生
    	2. 如果生成（或更新）foo.o 文件。也就是那个cc命令，其说明了，如何生成foo.o这个文件。（当然 foo.c 文件 include 了 defs.h 文件）
4. 规则的语法
	```shell
	targets : prerequisites
		command  ...  
	#或是这样：
	targets : prerequisites ; command
	command  ... 
    ```
    targets是文件名，以空格分开，可以使用通配符。一般来说，我们的目标基本上是一 个文件，但也有可能是多个文件。  command 是命令行，如果其不与“target:prerequisites”在一行，那么，必须以[Tab 键]开头，如果和 prerequisites 在一行，那么可以用分号做为分隔。（见上） prerequisites 也就是目标所依赖的文件（或依赖目标）。如果其中的某个文件要比目标文件要新，那么， 目标就被认为是“过时的”，被认为是需要重生成的。这个在前面已经讲过了。    如果命令太长，你可以使用反斜框（‘\’）作为换行符。make 对一行上有多少个字符 没有限制。规则告诉 make 两件事，文件的依赖关系和如何成成目标文件。    一般来说，make 会以 UNIX 的标准 Shell，也就是/bin/sh 来执行命令。
5. 在规则中使用通配符
	1. make支持三个通配符、\"*\" 、\"?\"、\"[...]\"
	2. 波浪号"~"字符在文件名中也有特殊用途、如果是"~/test"、这就表示当前用户$HOME目录下的test目录。如果"~test/test"则表示用户test的宿主目录下的test目录
	3. 例子
	```shell
	clean:
		rm -rf *.o #表示删除所有.o结尾的文件
	例子：
		print:*.c #print 依赖于所有的.c文件
		lpr -p $? #是一个自动化变量
		touch print
	例子：
		objects = *.o #这里注意objects的值就是*.o
	例子：
		objects = $(wildcard *.o)
    ```
6. 文件搜索
	1. 方法一、VPATH:如果没指定这个变量、make只会在当前目录下寻找依赖文件、如果指定了、就在指定目录下去寻找`VPATH = src:../headers`指定两个目录、src 和"../headers"、make会按照这个顺序去寻找
	2. 方法二、使用make的"vpath"关键字、这里不是变量、是一个关键字、
		1. vpath <pattern> <directories> 为符合模式<pattern>的文件指定搜索目录<directories>
		2. vpath <pattern> 清除符合模式<pattern>的文件的搜索目录
		3. vpath 清除所有已被设置好的文件搜索目录
		说明：vpath使用方法中的<pattern>需要包含"%"字符。"%"的意思是匹配零或若干字符、例如：%.h表示所有以".h"结尾的文件。<pattern>指定了要搜索的文件集。，而 <directories>则指定了<pattern>的文件集的搜索的目录
		4. 例子：`vpath %.h ../headers`
7. 伪目标
		```shell
		clean:
			rm *.o temp
        ```
    1. 我们生成了许多文件编译文件、我们也应该提供一个清除他们的“目标”以备完整地重编译而用。因为我们并不生成clean这个文件。“伪目标”并不是一个文件、只是一个标签、不能和文件重名
    2. .PYONY 来显示的指明一个目标是“伪目标”、向make说明、不管是否有个这个文件、这个目标就是“伪目标”`.PYONY:clean`
	    ```shell
	    .PYONY clean
	    clean:
	    	rm *.o temp
        ```
    3. 伪目标一般没有依赖的文件。但是我们也可以为伪目标指定所依赖的文件。伪目标同样可以作为“默认目标”，只要将其放在第一个。一个示例就是、如果的Makefile需要一口气生成若干个可执行文件、但你只想简单的敲一个命令完事、并且所有的目标文件都写在一个Makefile中、那么可以使用“伪目标”这个特性
	    ```shell
	    all : prog1 prog2 prog3
		.PHONY : all
		prog1 : prog1.o utils.o
		cc -o prog1 prog1.o utils.o
		prog2 : prog2.o
		cc -o prog2 prog2.o
		prog3 : prog3.o sort.o utils.o
		cc -o prog3 prog3.o sort.o utils.o
        ```
