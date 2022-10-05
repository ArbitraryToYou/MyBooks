g++ main.cpp -o out.exe **-g**	// -g表示编译时产生必要的调试信息

-O0 ~ -O4 表示优化等级，0表示不优化
对GDB更友好的编译选项是-Og，其优化介于O0~O1之间

常用gdb调试指令：
1.b xxx // 在某一行设置断点break
	break location	// location可以是linenum，filename:linenum, +offset, -offset, function, filename:offset
	break ...if cond // ...可以是上述打断点的位置，每次都执行cond进行判断，如果true则中断
2.tbreak // 断点一次有效
3.rbreak // 只对c,c++函数有效，会在指定函数开头打断点
	rbreak regex // 满足正则表达式的函数名会中断，和break一样
4.r  	// 运行/重新开始run，run p1 p2 ... 可指定输入参数
5.start // 运行到main()函数位置停止，start p1 p2 ...
6.c	// 继续执行continue
7.n	// 执行一行代码next
8.p xxx	// 打印指定变量的值print
9.l	// 显示源程序代码内容list，默认显示10行代码，Enter键继续往后看（也可显示是否找到目标程序文件）
10.q	// 终止调试quit
11.help
12.file xxx	// 指定调试文件
13.attach PID, -p PID, filename PID, 调试正在运行的程序用PID，pidof filename 可查看pid号
14.detach	// 调试器与程序分离
15.分析调试core文件
	ulimit -c 查看是否开启core dump功能
	ulimit -c unlimited	 不限制core文件大小
	gdb xxx.exe core	// 可用where, print, bt查看详细信息
16.s file	// 仅从file加载符号表
17.cd dir	// 以dir作为启动gdb调试的工作目录
18.--args p1 p2 ...// 向可执行程序传入参数，如gdb --args main.exe **10 a.txt**
19.set args p1 p2 ... // 调试器启动后，可以这样传递参数
20.path xxx	// 临时修改环境变量
21.gdb -q	// 进入时不显示多余消息
22.watch cond	// 监控变量值的变化，观察断点，只要变量值变化就会中断
	实现原理：**硬件观察点**，**软件观察点**
	软件观察点：程序会以单步执行，每次执行完毕会检查变量的值是否变化，一定程度影响执行效率；
	set can-use-hw-watchpoints 0 // 只建立软件观察点
	硬件观察点：不影响程序执行效率，系统会为其提供少量寄存器，数量有限，观察点太多时有些会失效；
23.rwatch	// 只要程序中出现读取目标变量值的操作就中断，只能硬件观察点
24.awatch	// 只要程序中出现读取目标变量值或改变值的操作就中断，只能硬件观察点
25.info watchpoints // 查看当前观察点的数量
26.catch event	// 建立捕捉断点，监控程序中某一个事件的发生，如异常，动态库加载，gcc>4.8才可
	常用的event事件：
	throw [exception] // 程序中抛出指定异常时，中断
	catch [exception] // 捕获到异常时，中断
	load [dllname]
	unload [dllname]
	catch捕获的异常往往位于系统库，up可以回到用户的代码
27.tcatch event	// 只有一次有效
28.condition // 对条件断点的表达式进行修改，也可为普通断点、观察断点和捕捉断点加上条件
	condition bnum expr // bnum是目标断点的编号
	condition bnum // 置空，使其变成无条件的普通断点
29.info break	// 查看断点信息
30.up // ?
31.ignore bnum count	// 断点功能失效指令，失效次数过了就正常中断
32.finish // 结束当前函数的执行，跳出函数后暂停程序的执行。缩写fi
33.return // 结束当前函数的调用，并返回指定值，再上一层函数调用处暂停执行
	可以指定函数的返回值，剩下未执行的代码就忽略了
34.jump location // 跳转执行
35.list // 显示全部源码
36.help // 显示全部命令的分类
	help xxx // 显示更详细的命令
37.<TAB>键自动补全命令，输入命令之后双击<TAB>可以显示所有参数
----------------------------单步调试--------------------------------
1.next	count // 将函数视为一行，默认为1行，缩写n
2.step	count // 会进入到函数内部，在函数第一行代码处停止执行，缩写s
3.until // 单步调试指令，缩写u
	until	// 是程序快速运行完当前的循环体，并在循环体外停止（只有执行到循环体尾部才会发挥此作用，其余同next）
	until location // loc为行号，执行到指定位置停止
----------------------------打印变量--------------------------------
1.print variable	// 打印指定变量的值，或者修改变量或表达式的值
	print高级用法：print [options --] [/fmt] expr
		-address on|off // 查看某一指针变量的值时，同时打印其指向的内存地址，默认on，等同于set print address on
		-array on|off // 是否以便于阅读的方式输出数组
		-array-indexes on|off // 是否同时显示其下标
		-pretty on|off // 以便于阅读的形式打印结构体
	还支持@和::运算符：
		@用于输出数组中指定区域的元素，print arr[x]@len，x是数组查看区域内某个元素的值，len是长度
		::用于区分作用域
	
2.display/fmt expr	// 每当程序暂停执行时，都会打印变量的值，fmt是指定输出变量的格式
	/fmt的值：
	/x 十六进制
	/d 有符号十进制
	/u 无符号十进制
	/o 八进制
	/t 二进制
	/f 浮点数
	/c 字符形式
3.info display // 可打印display的变量信息
4.undisplay num... // 删除display中的一个或多个变量
5.delete display num... // 删除变量
6.disable display num... // 禁用display中的变量
7.enable display num... // 激活display变量
----------------------------删除断点--------------------------------
1.clear location // 行号
2.delete num // 断点编号，默认删除所有
3.disable num... // 禁用断点，默认禁用所有
4.enable num... // 激活断点
	enable once num... // 激活一次
	enable count num... // 激活count次
	enable delete num... // 中断后删除该断点
----------------------------调试多线程程序--------------------------------
1.info threads
2.thread id // 将线程号为id的线程设置为当前线程
3.thread apply id... command // 将command命令作用于指定编号的线程,当对某个线程进行单步执行时，其他线程也会执行，可能不止一行代码
4.break location thread id // 在location建立普通断点，并且该断点作用于编号为id的线程
5.set scheduler-locking on|off|step // 默认情况当一个线程暂停，所有线程暂停，当运行continue时，所有线程运行，设置这个可使单个线程运行
	off // 不锁定线程，任何线程都可以随时执行
	on // 锁定线程，只有当前线程和指定线程可以执行
	step // 单步执行某一线程时，其他线程不会执行，除非是continue, until, finish会使其他执行，然后线程会切换到其他线程中断的位置
6.show scheduler-locking // 查看线程锁定的状态
7.non-stop模式(只有gdb 7.0以上版本可用)
	此模式下，调试单个线程不会影响其他线程的执行，即当某一线程暂停，其他线程可以继续执行。
	此模式下，continue, next, step都是作用在当前线程，**continue -a**可使其作用于所有线程
	如何从all-stop模式转换到non-stop模式，在未执行程序前执行**set non-stop on/off**
	查看：show non-stop
----------------------------后台异步执行调试命令--------------------------------
同步执行：执行下一个命令必须等待前一个命令执行完毕
异步执行：可以并行执行命令，语法**command&**，如continue&, 多用于non-stop模式中。
支持的后台命令：
	run, attach(调试运行的程序), step, stepi, next, nexti, continue, finish, until, 
----------------------------暂停后台线程执行--------------------------------
1.interrupt
----------------------------调试多进程程序--------------------------------
GDB默认只调试父进程，可以使用attach child_pid 调试子进程
1.对于使用fork(), vfork()构建的多进程程序，可以使用**set follow-fork-mode parent/child** 来设置默认调试父进程还是子进程
2.可以通过show follow-fork-mode 来查看当前默认调试的进程，**此命令一经选择，调试过程无法改变**。
3.set detach-on-fork on/off // on=只调试一个进程，可以是子进程或父进程；off=可以是所有进程
4.show detach-on-fork
5.info inferiors // 查看当前环境中有多少个进程
6.inferiors id // 切换到指定进程id号的进程进行调试
7.detach inferior id // 切断与指定进程间的联系，是该进程可以独立运行
8.kill inferior id // 切断联系，中断该进程的执行
9.remove-inferior id // 需先执行detach inferior id 或 kill inferior id，将其彻底删除，不存在info表中
----------------------------反向调试--------------------------------
要求：gdb7.0以上
临时改变程序执行的方向，消除已经执行的指定行数代码的影响，还原到这些代码未执行前的状态
1.record // 启动程序之后，反向调试之前，需要执行这个，记录相关信息，否则无法反向调试
2.record btrace // 同上
3.reverse-continue/(rc) // 反向运行，直到遇到断点或者开始record的地方
4.reverse-step // 反向指向一行代码，遇到函数时，会暂停到函数最后一行代码开头处
5.reverse-next
6.reverse-finish
7.set exec-direction forward/reverse // forward表示正常执行所有命令，reverse表示反向执行所有命令，命令和正常时一样
----------------------------handle命令：信号处理--------------------------------
信号，常常作为进程间通信的一种重要手段
1.info handles // 用于常看gdb可以处理的信号种类
2.handle signal mode // signal表示信号全名或简称，mode有如下值：
	nostop // 信号发生时，程序不会暂停，但会打印信息
	stop // 信号发生时，暂停程序执行
	print // 信号发生时打印必要的提示信息
	noprint
	pass	// 在捕获信号的同时，允许程序自动处理该信号
	nopass
	当GDB捕获信号，并暂停程序的那一刻，程序是捕获不到信号的，只有当程序继续执行之后
3.info signals name // 查看相关信号的配置信息
----------------------------frame和backtrace: 查看栈信息--------------------------------
对于c, c++程序而言，每个调用的函数在执行时，都会生成必要的信息，包括：
	①函数调用发生在程序中的具体位置；
	②函数调用时的参数；
	③函数体内各变量的值等；
这些信息会集中存储在一块叫做“栈帧”的内存空间中，程序执行调用多少个函数就产生多少个栈帧，在函数调用时产生，函数结束时释放，它们集中位于栈区。【程序的运行会占用一整块内存区域，包括：栈区，堆区，常量区，全局数据区等。】
1.frame spec // 根据栈帧编号或栈帧地址，选定要查看的栈帧，spec常用的指定方法有三种：
	①栈帧的编号，0为当前调用的函数，1为调用它的函数，则main是最大的；
	②借助栈帧的地址指定，可由info frame获取
	③函数名，如果是递归的话，则指定为编号最小的那个
2.up num // 在当前栈帧cur的基础上，选定cur+num为编号的栈帧作为当前帧，默认值为1
3.down num // 相反，选定cur-num
4.info frame // 打印当前栈帧信息
5.info args // 查看当前函数各个参数值
6.info locals // 查看当前函数各个局部变量的值
7.backtrace [-full] [n] // 当n>0,表示打印最里层的n个栈帧的信息，当n<0,表示打印最外层的n个栈帧的信息；-full打印栈帧的同时打印局部变量的值，**只用于打印当前线程中的所有栈帧**。若要打印所有线程中的栈帧信息，应用**thread apply all backtrace**
----------------------------编辑和搜索源码--------------------------------
1.edit // 编辑文件
	edit location	// 激活文件中指定位置然后进行编辑，如edit 16激活第16行，edit func激活func函数开头位置，edit main.c:16等
	edit filename:location	// edit命令默认使用的是ex编辑器，可能找不到
	遇到找不到ex编辑器，可以在调试前指定编辑器：export EDITOR=/usr/bin/vim，只是临时生效，修改配置文件可以永久生效，修改当前home目录下的：~/.bashrc文件，在文件末尾添加上述命令即可！
2.search // 搜索文件
	search <regexp>	// regexp表示正则表达式
	reverse-search <regexp>	// 反向查找

断点调试：
通过在程序适当位置打断点，观察某些变量的值，近而缩小导致程序异常或bug的搜索范围，并最终找到的过程。

运行gdb run/start之前：
1.指定调试的目标程序
2.有些需要传入参数
3.可能需要临时设置环境变量
4.可能需要指定工作目录，默认当前目录
5.允许对程序的输入输出做重定向

常见错误：
1.段错误=访问权限冲突，访问了不可访问的地址
