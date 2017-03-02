###  Shell echo命令
Shell 的 echo 指令是用于字符串的输出。命令格式：
```
echo string
```
您可以使用echo实现更复杂的输出格式控制。

#### 1.显示普通字符串:
```
echo "It is a test"

[hadoop@test101 ~]$ It is a test
```

这里的双引号完全可以省略，以下命令与上面实例效果一致：
```
echo It is a test
结果是：
It is a test
```
#### 2.显示转义字符
```
echo "\"It is a test\""

结果是:
"It is a test"
```
同样，双引号也可以省略
#### 3.显示变量
read 命令从标准输入中读取一行,并把输入行的每个字段的值指定给 shell 变量 shell脚本内容如下：
```
#!/bin/sh
read name 
echo "$name It is a test"
```
以上代码保存为 test.sh，name 接收标准输入的变量，结果将是:
```
[root@www ~]# sh test.sh
OK                     #标准输入
OK It is a test        #输出
```
#### 4.显示换行
```
echo -e "OK! \n" # -e 开启转义
echo "It it a test"
输出结果：
OK!

It it a test
```
#### 5.显示不换行
```
#!/bin/sh
echo -e "OK! \c" # -e 开启转义 \c 不换行
echo "It is a test"
输出结果：
OK! It is a test
```
#### 6.显示结果定向至文件
```
echo "It is a test" > myfile
```
#### 7.原样输出字符串，不进行转义或取变量(用单引号)
```
echo '$name\"'
输出结果：
$name\"
```
#### 8.显示命令执行结果
```
echo `date`
结果将显示当前日期
Thu Jul 24 10:08:46 CST 2014
```
注意date两边的符号 不是单引号 是键盘Esc下面的反引号。



### Shell printf 命令
 Shell 的另一个输出命令 printf。
printf 命令模仿 C 程序库（library）里的 printf() 程序。
标准所定义，因此使用printf的脚本比使用echo移植性好。
printf 使用引用文本或空格分隔的参数，外面可以在printf中使用格式化字符串，还可以制定字符串的宽度、左右对齐方式等。默认printf不会像 echo 自动添加换行符，我们可以手动添加 \n。

#### printf 命令的语法：

printf  format-string  [arguments...]
参数说明：
- format-string: 为格式控制字符串
- arguments: 为参数列表。
实例如下：
```
$ echo "Hello, Shell"
结果：
Hello, Shell
$ printf "Hello, Shell\n"
结果：
Hello, Shell
$ printf "Hello, Shell"
结果：
Hello,Shell[hadoop@test101 ~]
```
接下来,我来用一个脚本来体现printf的强大功能：
```
#!/bin/bash
# author:菜鸟教程
# url:www.runoob.com
 
printf "%-10s %-8s %-4s\n" 姓名 性别 体重kg  
printf "%-10s %-8s %-4.2f\n" 郭靖 男 66.1234 
printf "%-10s %-8s %-4.2f\n" 杨过 男 48.6543 
printf "%-10s %-8s %-4.2f\n" 郭芙 女 47.9876 
执行脚本，输出结果如下所示：
姓名     性别   体重kg
郭靖     男      66.12
杨过     男      48.65
郭芙     女      47.99
```
**%s %c %d %f都是格式替代符.
%-10s 指一个宽度为10个字符（-表示左对齐，没有则表示右对齐），任何字符都会被显示在10个字符宽的字符内，如果不足则自动以空格填充，超过也会将内容全部显示出来。
%-4.2f 指格式化为小数，其中.2指保留2位小数。**

更多实例：
```
#!/bin/bash
# author:菜鸟教程
# url:www.runoob.com
 
# format-string为双引号
printf "%d %s\n" 1 "abc"

# 单引号与双引号效果一样 
printf '%d %s\n' 1 "abc" 

# 没有引号也可以输出
printf %s abcdef

# 格式只指定了一个参数，但多出的参数仍然会按照该格式输出，format-string 被重用
printf %s abc def

printf "%s\n" abc def

printf "%s %s %s\n" a b c d e f g h i j

# 如果没有 arguments，那么 %s 用NULL代替，%d 用 0 代替
printf "%s and %d \n" 
执行脚本，输出结果如下所示：
1 abc
1 abc
abcdefabcdefabc
def
a b c
d e f
g h i
j  
 and 0
```
printf的转义序列
```
序列	说明
\a	警告字符，通常为ASCII的BEL字符
\b	后退
\c	抑制（不显示）输出结果中任何结尾的换行字符（只在%b格式指示符控制下的参数字符串中有效），而且，任何留在参数里的字符、任何接下来的参数以及任何留在格式字符串中的字符，都被忽略
\f	换页（formfeed）
\n	换行
\r	回车（Carriage return）
\t	水平制表符
\v	垂直制表符
\\	一个字面上的反斜杠字符
\ddd	表示1到3位数八进制值的字符。仅在格式字符串中有效
\0ddd	表示1到3位的八进制值字符
```
实例
```
$ printf "a string, no processing:<%s>\n" "A\nB"
a string, no processing:<A\nB>

$ printf "a string, no processing:<%b>\n" "A\nB"
a string, no processing:<A
B>

$ printf "www.runoob.com \a"
www.runoob.com $                  #不换行
```




### Shell test 命令
Shell中的 test 命令用于检查某个条件是否成立，它可以进行数值、字符和文件三个方面的测试。
1. 数值测试
```
参数	说明
-eq	等于则为真
-ne	不等于则为真
-gt	大于则为真
-ge	大于等于则为真
-lt	小于则为真
-le	小于等于则为真
```
实例演示：
```
num1=100
num2=100
if test $[num1] -eq $[num2]
then
    echo '两个数相等！'
else
    echo '两个数不相等！'
fi
输出结果：
两个数相等！
```
2. 字符串测试
```
参数	说明
=	等于则为真
!=	不相等则为真
-z 字符串	字符串的长度为零则为真
-n 字符串	字符串的长度不为零则为真
```
实例演示：
```
num1="ru1noob"
num2="runoob"
if test $num1 = $num2
then
    echo '两个字符串相等!'
else
    echo '两个字符串不相等!'
fi
输出结果：
两个字符串不相等!
```
3. 文件测试
```
参数	说明
-e 文件名	如果文件存在则为真
-r 文件名	如果文件存在且可读则为真
-w 文件名	如果文件存在且可写则为真
-x 文件名	如果文件存在且可执行则为真
-s 文件名	如果文件存在且至少有一个字符则为真
-d 文件名	如果文件存在且为目录则为真
-f 文件名	如果文件存在且为普通文件则为真
-c 文件名	如果文件存在且为字符型特殊文件则为真
-b 文件名	如果文件存在且为块特殊文件则为真
```
实例演示：
```
cd /bin
if test -e ./bash
then
    echo '文件已存在!'
else
    echo '文件不存在!'
fi
输出结果：
文件已存在!
```
另外，Shell还提供了与( -a )、或( -o )、非( ! )三个逻辑操作符用于将测试条件连接起来，其优先级为："!"最高，"-a"次之，"-o"最低。例如：
```
cd /bin
if test -e ./notFile -o -e ./bash
then
    echo '有一个文件存在!'
else
    echo '两个文件都不存在'
fi
输出结果：
有一个文件存在!
```



### Shell 流程控制
和Java、PHP等语言不一样，**sh的流程控制不可为空**，如(以下为PHP流程控制写法)：
<?php
```
if (isset($_GET["q"])) {
    search(q);
}
else {
    // 不做任何事情
}
```

在sh/bash里可不能这么写，如果else分支没有语句执行，就不要写这个else。

#### if 语句语法格式：
```
if condition
then
    command1 
    command2
    ...
    commandN 
fi
```
写成一行（适用于终端命令提示符）：
```
if [ $(ps -ef | grep -c "ssh") -gt 1 ]; then echo "true"; fi
```
末尾的fi就是if倒过来拼写，后面还会遇到类似的。

if else 语法格式：
```
if condition
then
    command1 
    command2
    ...
    commandN
else
    command
fi
```

if else-if else 语法格式：
```
if condition1
then
    command1
elif condition2 
then 
    command2
else
    commandN
fi
```
以下实例判断两个变量是否相等：
```
a=10
b=20
if [ $a == $b ]
then
   echo "a 等于 b"
elif [ $a -gt $b ]
then
   echo "a 大于 b"
elif [ $a -lt $b ]
then
   echo "a 小于 b"
else
   echo "没有符合的条件"
fi
输出结果：
a 小于 b
```
if else语句经常与test命令结合使用，如下所示：
```
num1=$[2*3]
num2=$[1+5]
if test $[num1] -eq $[num2]
then
    echo '两个数字相等!'
else
    echo '两个数字不相等!'
fi
输出结果：
两个数字相等!
```
#### for 循环
与其他编程语言类似，Shell支持for循环。
for循环一般格式为：
```
for var in item1 item2 ... itemN
do
    command1
    command2
    ...
    commandN
done
```
写成一行：
```
for var in item1 item2 ... itemN; do command1; command2… done;
```
当变量值在列表里，for循环即执行一次所有命令，使用变量名获取列表中的当前取值。命令可为任何有效的shell命令和语句。in列表可以包含替换、字符串和文件名。
in列表是可选的，如果不用它，for循环使用命令行的位置参数。
例如，顺序输出当前列表中的数字：
```
for loop in 1 2 3 4 5
do
    echo "The value is: $loop"
done
输出结果：
The value is: 1
The value is: 2
The value is: 3
The value is: 4
The value is: 5
```
顺序输出字符串中的字符：
```
for str in 'This is a string'
do
    echo $str
done
输出结果：
This is a string
```
#### while 语句
while循环用于不断执行一系列命令，也用于从输入文件中读取数据；命令通常为测试条件。其格式为：
```
while condition
do
    command
done
```
以下是一个基本的while循环，测试条件是：如果int小于等于5，那么条件返回真。int从0开始，每次循环处理时，int加1。运行上述脚本，返回数字1到5，然后终止。
```
#!/bin/sh
int=1
while(( $int<=5 ))
do
        echo $int
        let "int++"
done
运行脚本，输出：
1
2
3
4
5
```
使用中使用了 Bash let 命令，它用于执行一个或多个表达式，变量计算中不需要加上 $ 来表示变量，具体可查阅：Bash let 命令
。
while循环可用于读取键盘信息。下面的例子中，输入信息被设置为变量FILM，按<Ctrl-D>结束循环。
```
echo '按下 <CTRL-D> 退出'
echo -n '输入你最喜欢的电影名: '
while read FILM
do
    echo "是的！$FILM 是一部好电影"
done
```
运行脚本，输出类似下面：
```
按下 <CTRL-D> 退出
输入你最喜欢的电影名: w3cschool菜鸟教程
是的！w3cschool菜鸟教程 是一部好电影
```
无限循环
无限循环语法格式：
```
while :
do
    command
done
或者
while true
do
    command
done
或者
for (( ; ; ))
```
#### until 循环
**until循环执行一系列命令直至条件为真时停止。
until循环与while循环在处理方式上刚好相反**。

一般while循环优于until循环，但在某些时候—也只是极少数情况下，until循环更加有用。
until 语法格式:
```
until condition
do
    command
done
```
条件可为任意测试条件，测试发生在循环末尾，因此循环至少执行一次—请注意这一点。
#### case
Shell case语句为多选择语句。可以用case语句匹配一个值与一个模式，如果匹配成功，执行相匹配的命令。case语句格式如下：
```
case 值 in
模式1)
    command1
    command2
    ...
    commandN
    ;;
模式2）
    command1
    command2
    ...
    commandN
    ;;
esac
```
case工作方式如上所示。取值后面必须为单词in，每一模式必须以右括号结束。取值可以为变量或常数。匹配发现取值符合某一模式后，其间所有命令开始执行直至 ;;。
取值将检测匹配的每一个模式。一旦模式匹配，则执行完匹配模式相应命令后不再继续其他模式。如果无一匹配模式，使用星号 * 捕获该值，再执行后面的命令。
下面的脚本提示输入1到4，与每一种模式进行匹配：
```
echo '输入 1 到 4 之间的数字:'
echo '你输入的数字为:'
read aNum
case $aNum in
    1)  echo '你选择了 1'
    ;;
    2)  echo '你选择了 2'
    ;;
    3)  echo '你选择了 3'
    ;;
    4)  echo '你选择了 4'
    ;;
    *)  echo '你没有输入 1 到 4 之间的数字'
    ;;
esac
```
输入不同的内容，会有不同的结果，例如：
```
输入 1 到 4 之间的数字:
你输入的数字为:
3
你选择了 3
```
#### 跳出循环
在循环过程中，有时候需要在未达到循环结束条件时强制跳出循环，Shell使用两个命令来实现该功能：break和continue。
**break命令**
break命令允许跳出所有循环（终止执行后面的所有循环）。
下面的例子中，脚本进入死循环直至用户输入数字大于5。要跳出这个循环，返回到shell提示符下，需要使用break命令。
```
#!/bin/bash
while :
do
    echo -n "输入 1 到 5 之间的数字:"
    read aNum
    case $aNum in
        1|2|3|4|5) echo "你输入的数字为 $aNum!"
        ;;
        *) echo "你输入的数字不是 1 到 5 之间的! 游戏结束"
            break
        ;;
    esac
done
```
执行以上代码，输出结果为：
```
输入 1 到 5 之间的数字:3
你输入的数字为 3!
输入 1 到 5 之间的数字:7
你输入的数字不是 1 到 5 之间的! 游戏结束
```
**continue**
continue命令与break命令类似，只有一点差别，它不会跳出所有循环，仅仅跳出当前循环。
对上面的例子进行修改：
```
#!/bin/bash
while :
do
    echo -n "输入 1 到 5 之间的数字: "
    read aNum
    case $aNum in
        1|2|3|4|5) echo "你输入的数字为 $aNum!"
        ;;
        *) echo "你输入的数字不是 1 到 5 之间的!"
            continue
            echo "游戏结束"
        ;;
    esac
done
```
运行代码发现，当输入大于5的数字时，该例中的循环不会结束，语句 echo "Game is over!" 永远不会被执行。
#### esac
case的语法和C family语言差别很大，它需要一个esac（就是case反过来）作为结束标记，每个case分支用右圆括号，用两个分号表示break。




### Shell 函数
linux shell 可以用户定义函数，然后在shell脚本中可以随便调用。

shell中函数的定义格式如下：
```
[ function ] funname [()]

{

    action;

    [return int;]

}
```
说明：
1. 可以带function fun() 定义，也可以直接fun() 定义,不带任何参数。
2. 参数返回，可以显示加：return 返回，如果不加，将以最后一条命令运行结果，作为返回值。 return后跟数值n(0-255
下面的例子定义了一个函数并进行调用：
```
#!/bin/bash
# author:菜鸟教程
# url:www.runoob.com

demoFun(){
    echo "这是我的第一个 shell 函数!"
}
echo "-----函数开始执行-----"
demoFun
echo "-----函数执行完毕-----"
输出结果：
-----函数开始执行-----
这是我的第一个 shell 函数!
-----函数执行完毕-----
```
下面定义一个带有return语句的函数：
```
#!/bin/bash
# author:菜鸟教程
# url:www.runoob.com

funWithReturn(){
    echo "这个函数会对输入的两个数字进行相加运算..."
    echo "输入第一个数字: "
    read aNum
    echo "输入第二个数字: "
    read anotherNum
    echo "两个数字分别为 $aNum 和 $anotherNum !"
    return $(($aNum+$anotherNum))
}
funWithReturn
echo "输入的两个数字之和为 $? !"
```
输出类似下面：
```
这个函数会对输入的两个数字进行相加运算...
输入第一个数字: 
1
输入第二个数字: 
2
两个数字分别为 1 和 2 !
输入的两个数字之和为 3 !
```
- 函数返回值在调用该函数后通过 $? 来获得。
- 注意：所有函数在使用前必须定义。这意味着必须将函数放在脚本开始部分，直至shell解释器首次发现它时，才可以使用。调用函数仅使用其函数名即可。
#### 函数参数
在Shell中，调用函数时可以向其传递参数。在函数体内部，通过 $n 的形式来获取参数的值，例如，$1表示第一个参数，$2表示第二个参数...
带参数的函数示例：
```
#!/bin/bash
# author:菜鸟教程
# url:www.runoob.com

funWithParam(){
    echo "第一个参数为 $1 !"
    echo "第二个参数为 $2 !"
    echo "第十个参数为 $10 !"
    echo "第十个参数为 ${10} !"
    echo "第十一个参数为 ${11} !"
    echo "参数总数有 $# 个!"
    echo "作为一个字符串输出所有参数 $* !"
}
funWithParam 1 2 3 4 5 6 7 8 9 34 73
输出结果：
第一个参数为 1 !
第二个参数为 2 !
第十个参数为 10 !
第十个参数为 34 !
第十一个参数为 73 !
参数总数有 11 个!
作为一个字符串输出所有参数 1 2 3 4 5 6 7 8 9 34 73 !
```
- 注意，**$10 不能获取第十个参数，获取第十个参数需要${10}。当n>=10时，需要使用${n}来获取参数。**
另外，还有几个特殊字符用来处理参数：
```
参数处理	说明
$#	传递到脚本的参数个数
$*	以一个单字符串显示所有向脚本传递的参数
$$	脚本运行的当前进程ID号
$!	后台运行的最后一个进程的ID号
$@	与$*相同，但是使用时加引号，并在引号中返回每个参数。
$-	显示Shell使用的当前选项，与set命令功能相同。
$?	显示最后命令的退出状态。0表示没有错误，其他任何值表明有错误。
```






### Shell 输入/输出重定向
大多数 UNIX 系统命令从你的终端接受输入并将所产生的输出发送回​​到您的终端。一个命令通常从一个叫标准输入的地方读取输入，默认情况下，这恰好是你的终端。同样，一个命令通常将其输出写入到标准输出，默认情况下，这也是你的终端。
重定向命令列表如下：
命令	说明
```
command > file	将输出重定向到 file。
command < file	将输入重定向到 file。
command >> file	将输出以追加的方式重定向到 file。
n > file	将文件描述符为 n 的文件重定向到 file。
n >> file	将文件描述符为 n 的文件以追加的方式重定向到 file。
n >& m	将输出文件 m 和 n 合并。
n <& m	将输入文件 m 和 n 合并。
<< tag	将开始标记 tag 和结束标记 tag 之间的内容作为输入。
需要注意的是文件描述符
0 通常是标准输入（STDIN），
1 是标准输出（STDOUT），
2 是标准错误输出（STDERR）。
```
#### 输出重定向
重定向一般通过在命令间插入特定的符号来实现。特别的，这些符号的语法如下所示:
```
command1 > file1
```
上面这个命令执行command1然后将输出的内容存入file1。

**注意任何file1内的已经存在的内容将被新内容替代。如果要将新内容添加在文件末尾，请使用>>操作符。**
实例
执行下面的 who 命令，它将命令的完整的输出重定向在用户文件中(users):
```
$ who > users
```
执行后，并没有在终端输出信息，这是因为输出已被从默认的标准输出设备（终端）重定向到指定的文件。
你可以使用 cat 命令查看文件内容：
```
$ cat users
_mbsetupuser console  Oct 31 17:35 
tianqixin    console  Oct 31 17:35 
tianqixin    ttys000  Dec  1 11:33 
```
输出重定向会覆盖文件内容，请看下面的例子：
```
$ echo "菜鸟教程：www.runoob.com" > users
$ cat users
菜鸟教程：www.runoob.com

```
如果不希望文件内容被覆盖，可以使用 >> 追加到文件末尾，例如：
```
$ echo "菜鸟教程：www.runoob.com" >> users
$ cat users
菜鸟教程：www.runoob.com
菜鸟教程：www.runoob.com
$
```
#### 输入重定向
和输出重定向一样，Unix 命令也可以从文件获取输入，语法为：
command1 < file1
这样，本来需要从键盘获取输入的命令会转移到文件读取内容。
注意：输出重定向是大于号(>)，输入重定向是小于号(<)。
实例
接着以上实例，我们需要统计 users 文件的行数,执行以下命令：
```
$ wc -l users
       2 users
```
也可以将输入重定向到 users 文件：
```
$  wc -l < users
       2 
```
注意：上面两个例子的结果不同：第一个例子，会输出文件名；第二个不会，因为它仅仅知道从标准输入读取内容。
```
command1 < infile > outfile
```
同时替换输入和输出，执行command1，从文件infile读取内容，然后将输出写入到outfile中。
重定向深入讲解
一般情况下，每个 Unix/Linux 命令运行时都会打开三个文件：
标准输入文件(stdin)：stdin的文件描述符为0，Unix程序默认从stdin读取数据。
标准输出文件(stdout)：stdout 的文件描述符为1，Unix程序默认向stdout输出数据。
标准错误文件(stderr)：stderr的文件描述符为2，Unix程序会向stderr流中写入错误信息。
默认情况下，command > file 将 stdout 重定向到 file，command < file 将stdin 重定向到 file。
如果希望 stderr 重定向到 file，可以这样写：
```
$ command 2 > file
```
如果希望 stderr 追加到 file 文件末尾，可以这样写：
```
$ command 2 >> file
```
2 表示标准错误文件(stderr)。
如果希望将 stdout 和 stderr 合并后重定向到 file，可以这样写：
```
$ command > file 2>&1

或者

$ command >> file 2>&1
```
如果希望对 stdin 和 stdout 都重定向，可以这样写：
```
$ command < file1 >file2
```
command 命令将 stdin 重定向到 file1，将 stdout 重定向到 file2。
#### Here Document
Here Document 是 Shell 中的一种特殊的重定向方式，用来将输入重定向到一个交互式 Shell 脚本或程序。
它的基本的形式如下：
command << delimiter
    document
delimiter
它的作用是将两个 delimiter 之间的内容(document) 作为输入传递给 command。
注意：
结尾的delimiter 一定要顶格写，前面不能有任何字符，后面也不能有任何字符，包括空格和 tab 缩进。
开始的delimiter前后的空格会被忽略掉。
实例
在命令行中通过 wc -l 命令计算 Here Document 的行数：
```
$ wc -l << EOF
    欢迎来到
    菜鸟教程
    www.runoob.com
EOF
```
3          # 输出结果为 3 行
$
我们也可以将 Here Document 用在脚本中，例如：
```
#!/bin/bash
# author:菜鸟教程
# url:www.runoob.com

cat << EOF
欢迎来到
菜鸟教程
www.runoob.com
EOF
```
执行以上脚本，输出结果：
欢迎来到
菜鸟教程
www.runoob.com


#### /dev/null 文件
如果希望执行某个命令，但又不希望在屏幕上显示输出结果，那么可以将输出重定向到 /dev/null：
$ command > /dev/null
/dev/null 是一个特殊的文件，写入到它的内容都会被丢弃；如果尝试从该文件读取内容，那么什么也读不到。但是 /dev/null 文件非常有用，将命令的输出重定向到它，会起到"禁止输出"的效果。
如果希望屏蔽 stdout 和 stderr，可以这样写：
$ command > /dev/null 2>&1
注意：0 是标准输入（STDIN），1 是标准输出（STDOUT），2 是标准错误输出（STDERR）。







### Shell 文件包含
和其他语言一样，Shell 也可以包含外部脚本。这样可以很方便的封装一些公用的代码作为一个独立的文件。
Shell 文件包含的语法格式如下：
. filename   # 注意点号(.)和文件名中间有一空格

或

source filename
实例
创建两个 shell 脚本文件。
test1.s
h 代码如下：
#!/bin/bash
# author:菜鸟教程
# url:www.runoob.com

url="http://www.runoob.com"
test2.sh 代码如下：
#!/bin/bash
# author:菜鸟教程
# url:www.runoob.com

#使用 . 号来引用test1.sh 文件
. ./test1.sh

# 或者使用以下包含文件代码
# source ./test1.sh

echo "菜鸟教程官网地址：$url"
接下来，我们为 test2.sh 添加可执行权限并执行：
$ chmod +x test2.sh 
$ ./test2.sh 
菜鸟教程官网地址：http://www.runoob.com
注：被包含的文件 test1.sh 不需要可执行权限。