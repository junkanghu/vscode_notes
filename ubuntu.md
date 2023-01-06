[toc]
# Ubuntu Commands

## get home dir
```shell
$HOME # means the dir of home which can be print by echo $HOME
```

## empty trash
```shell
sudo rm -rf ~/.local/share/Trash/*
```

## shell shortcuts
1. ctrl + u/k: 删除光标（cursor）前/后的所有内容
2. ctrl + a/e：将光标移动到行首、尾
3. alt + b/f：光标前/后移一个word
4. ctrl + w：删除光标前一个单词
5. ctrl + l：clear screen

## back to the last dir from current dir
```shell
cd -
```

## watch the history command and re-execute again
```shell
history # watch the history command(each command with a number)
history !n # get the commandkjjj(n is the number of the command)
```

## ```curl cheat.sh/command``` 
will give a brief "cheat sheet" with common examples of how to use a shell command.

## move files
``` shell
mv ./*.jpg ./relight
```

## grep -E = egrep
``` shell
egrep '\.MP4$|\.'
```
used for searching a string in the target string using regular expresion

## |
``` shell
ls | egrep ...
```
take the output of the last command as the input of the next command

## $

1. ```${n:-xxx}```
if $n is empty, take the one follows ":", i.e., -xxx

2. ```${n:-}```
if $n is empty, then the output is empty

## string
1. '' apostrophe

``` shell
kang='kang'
hu='hujunkang$kang'
echo $hu
```
only returns the string content itself: hujunkang$kang

2. "" double quotes
``` shell
kang='123'
hu="hujunkang$kang"
echo
```
returns the quoted one: hujunkang123

## echo 

1. echo auguments
``` shell
hu="hujunkang"
echo $hu # return the value of hu
echo ${hu} # the more safe way compared to the command in line 7
echo ${hu:n} # return the substring starting at index n
echo ${hu:m:n} # return the substring starting at index m and its length is n
```

2. ```echo $# hu```
returns the length of the argument hu


## ll = ls -l
list the information of the file (including permission)

## Modify files with specific extension under current directory
``` shell
find ./ -name "*.JPG" | awk -F "." '{print $2}' | xargs -i -t mv ./{}.JPG  ./{}.jpg
```

## Delete
1. Delete files with specific extension under current directory
``` shell
find . -name "*.ARW" |xargs rm -rfv
find . -name "a*" |xargs rm -rfv
```
2. delete files with specific format
``` shell
ls | egrep "DSC0\d{4}.ARW" | xargs rm
```

## count the files under the directory
``` shell
ls | wc -l
```

## modify file names
``` shell
i=100000001; for f in *.JPG; do mv "$f" ${i#1}.jpg; ((i++)); done
```

## 建立硬、软链接
1. 概念
   1. 软链接，以路径的形式存在，类似于Windows操作系统中的快捷方式。硬链接以文件副本的形式存在，但不占用实际空间。
   2. 软链接可以跨文件系统 ，硬链接不可以。
   3. 软链接可以对一个不存在的文件名进行链接。
   4. 软链接可以对目录进行链接，硬链接不行。
2. 软链接
``` shell
ln -s source target
ln -s log2013.log link2013
```
例如source为/dellnas/dataset/static_recon/Relight/junkangs/head_1118/（这个“/”不能缺，代表将整个head_1118文件夹转移到某个路径下）, target为/dellnas/home/hujunkang/data/（data这个文件夹必须存在，head_1118文件夹将会软链接到data文件夹下，名称为head_1118）。
3. 硬链接
``` shell
ln log2013.log ln2013
```

![soft](./images/soft.png)
![hard](./images/hard.png)

## 改变权限命令chmod
![rwx](./images/rwx.jpg)
1. 基本格式 
```shell
chmod [-x] 权限命令 filename(dir name)
```
2. -x代表可选参数，一般使用的时候只考虑-R（代表对directory操作），如果不写就是对某个file操作
3. 权限命令
   1. 用数字指定：一共三位数字，每位数字分别代表user、group、others的权限。可以根据二进制得出数字，如777代表对user、group、others都服务rwx权限。
   ![rwx_n](./images/rwx_n.jpg)
   2. 用字母指定:
      1. u,g,o,a：分别代表user、roup、others、all
      2. +-=：分别代表增加某个权限，减去某个权限，设置权限为
      3. 结合起来就是如：ug+r, a+r, ugo-r等
4. example：
``` shell
chmod -R 777 dir1 # 对user、group、others关于某个路径的权限都设置为rwx
chmod ugo+r file1.txt / chmod a+r file1.txt # 设置文件为所有人都可读
chmod ug+w,o-w file1.txt file2.txt # ug加上写的权限，o减去写的权限
chmod -R a+r * # 对当前路径下所有的文件进行递归增加所有人读的权限
chmod a=rwx file # 设置所有人对file有rwx权限
chmod ug=rwx,o=x file # 设置ug权限为rwx，o权限为x

```

## 搜索文本命令grep
一、正则表达式（regular expression）
1. 语法：
   1. 由字符和操作符组成
   2. 操作符：
      note：当以下⼀些字符如’.’、’+’需要以其本来字符形状出现时，需要在其前⾯加⼀个’\’来转义，可以进⼀步认为，所有的字符若要以其本来意思出现时，都需要在其前⾯加⼀个反斜杠。
      1. .：表示任何单个字符
      2. [ ]：字符集，对单个字符给出取值范围。[abc]表示a、b、c中的⼀个字符；[a-z]表示a-z单个字符。
      3. [^]：⾮字符集。[^abc]代表单个字符，其⾮a、b、c。
      4. \*：表示“\*”前⼀个字符的0次或⽆限次扩展。如abc\*代表ab（0次），abc（1次），abcc（2次）等
      5. +：表示“+”前⼀个字符出现1次或⽆限次扩展。
      6. ？：表示“？”前⼀个字符0次或1次扩展
      7. ｜：表示“｜”左右表达式任取其⼀。abc｜def表示abc或def
      8. {m}：表示扩展“{}”前的那个字符m次。ab{2}c代表abbc
      9. {m, n}：表示扩展“{}”前的那个字符m-n次（包含m、n），{:3}代表0次⾄3次。ab{1, 2}c代表abc、abbc
      10. ^：相对于3未出现在“[]”中表示匹配⼀个字符串的开头。^abc代表身处⼀个字符串开头的“abc”字符串
      11. $：匹配字符串结尾。abc$代表身处⼀个字符串结尾的“abc”字符串
      12. ()：分组标记，内部只能使⽤“｜”。（abc）表示“abc”；（abc｜def）表示“abc”或“def”
      13. \d：数字。等价于[0-9]。但是\d{3}代表的不是某⼀个数字重复3次，⽽是代表有连着的3个数字。
      14. \w：单词字符。等价于[A-Za-z0-9_（下划线）]
      note：当上⾯所述的*、+、？没有搭配具体的字符（如abc+）使⽤时，⽽是如[abc]+或[a-z]+或\d{3}这样出现时，代表的是括号中的任意单个字符的组合。
2. example
![RE](./images/RE.png)

二、grep
用于查找指定文件中含有的指定字符串
1. ```grep [option] 文件中要查找的字符串 文件名```
2. -i用于忽略大小写；-v用于显示不包含指定字符串的行；-E同egrep；-n代表显示行号
3. example
``` shell
grep test *file # 查找后缀有file字符串的文件中，包含test字符串的行，并打印该行
grep test file* # 查找前缀有file字符串的文件中，包含test字符串的行，并打印
grep -r update /etc/acpi # 递归查找指定目录下所有文件中包含update字符串的行，并打印所有文件名和行
grep -v test *test* # 查找文件名中包含test字符串，但是文件内容中不含test字符串的行，并打印
grep test file.txt # 查找file.txt中是否含有test字符串，若有则打印
ls | grep txt # 显示当前目录下文件名中含有txt字符串的文件名字并打印
echo "hujunkang" | grep hu # 查看字符串“hujunkang”中是否含有字符串“hu”
```
\*file代表以file字符串作为后缀、file\*代表以file字符串作为后缀、\*file\*代表含有file字符串。若文件名中不含*，即直接```grep test file```，则file必须是一个完整的文件名，其代表在file中查找是否有test字符串，否则会报错。

**Note：上面的ls、echo配合|的使用可以这样理解。grep命令在搜索某个文件中是否含有某个字符串时，首先将那个文件的内容打印到stdout中，然后在stdout中去搜索，在使用ls或者echo时，内容也被打印到stdout中，因此也相当于在stdout中搜索字符串，因此可以搜索文件名含有指定字符串的文件**


## 命令传参xargs
1. 默认setting
   1. xargs默认与管道搭配使用，即```xxx | xargs ...```，但是也可以接受用户的输入。
   2. xargs默认的命令是echo，即若不指定执行的命令，默认执行echo。
   3. xargs默认一次性接收前置命令的所有内容。
   4. 输入xargs的内容可能包含space和换行，但是在xargs里面都会被转换成space。
2. 读取某个文件，然后以指定格式输出
![xargs](./images/xargs.png)
   1. 当没有-n时，默认一下子读取所有的内容（由于setting的第4点，所以读取进来的内容本来可能含有space和换行，但是最后都被转换为space并输出）。
   2. 当-n指定每次读取多少个以space分隔的内容后，每次只读取几个参数，次与次之间以换行分隔并打印。
   3. setting默认以space或换行符分隔内容，但是可以以-d指定分隔符，使输入的内容以指定的分隔符分为，以space分隔的内容。
3. -I可以将xargs接受的输入传递给后一命令
```shell
ls *.jpg | xargs -n1 -I {} cp {} /data/images 
cat foo.txt | xargs -I file sh -c 'echo file; mkdir file'
# 
```
   1. -I代表将前一个命令输入到xargs中的内容替换到xargs后的命令当中（当xargs后的命令为mkdir这种只需要一个输入的情况时用不到-I(xxx|xargs mkdir即可)；但是cp这种命令后续有两个参数，为了指定传参的位置，必须要用-I来指定传参位置）
   2. {}配合-I使用，将xargs接受的参数传入到后续命令当中
   3. 当xargs后有多个命令同时执行时，需要用某个变量名（上面为“file”）而不是{}。
4. -p 和 -t
-p即为print，代表每次执行命令前打印要执行的命令并询问一下用户是否执行（用户输入yes才会执行）；-t在打印要执行的命令后直接执行，不需要用户输入。
5. 使用 -0配合find命令使用
   1. xargs默认以space作为分隔符，这使得它无法处理一下文件名，因为文件名中可能包含一些空格。一般的文件名都通过find来获得，这时可以配合find一起使用，以专门处理文件名。find命令中的“-print0”可以将获得的文件名以“\0”（即null）分隔，这保证了文件名中的space仍然存在。而在xargs中可以指定以null为分隔符而不是space，以处理这种文件名中有space的情况。
   ![-0]
   ```shell
   find /path -type f -print0 | xargs -0 rm
   ```
6. 

## 搜索进程和杀死进程：pgrep & pkill & kill
1. ```pgrep [option] pname```
   1. ``` shell
      pgrep ssh # 显示以ssh为名称的各个进程的pID
      pgrep -l ssh # 同时显示以ssh为名称的各个进程的pID和进程名称
      pgrep -o ssh # 当匹配多个进程时，显示进程号最小的那个进程
      pgrep -n ssh # 当匹配多个进程时，显示进程好最大的那个进程
      pgrep -f "python main.py" # 找到正在执行某一命令的进程的pid
      ```
2. kill命令（会向指定id的进程发送信号，如终止、杀死等信号）
   1. ``` shell
      kill -l # 列出所有可以发送的信号
      kill -l KILL # 显示KILL信号对应的简化的数值（KILL=9）
      kill -KILL 12345 = kill -9 12345 # 杀死指定pid的进程
      kill -9 %n # 杀死当前shell中id为n的进程
      ```
3. pkill命令（可以以进程名称来处理整个进程组）
   1. ``` shell
      pkill -o ssh # 杀死以ssh为名称的进程组中进程id最小的进程
      pkill -n ssh # 杀死以ssh为名称的进程组中进程id最大的进程
      pkill -9 ssh # 杀死以ssh为名称的整个进程组
      pkill -u mark # 杀死制定user（mark）的所有进程
      pkill -u mark，hank # 杀死多个user的所有进程
      pkill -9 -u mark ssh # 强制杀死user（mark）的以ssh为名称的整个进程组
      pkill -9 -f "ping 8.8.8.8" # 强制杀死明确带有auguments的命令产生的进程（shell执行ping 8.8.8.8会产生一个进程去处理，我们要杀死这个进程）
      ```
4. 加```-KILL或-9```与不加的区别：不加会正常中断进程，然后一步步将其kill（较为安全）。加了之后直接不讲道理杀死，没有正常中断的过程。一般先用不加```-KILL```，如果无效就使用它，可以理解为不加```-KILL```就是正常关闭软件，加了相当于在任务管理器界面强制终止进程。

## 进程的后台运行、suspend、continue等
1. ```jobs [args]```
   1. 作用：显示在当前shell下产生的进程（running、suspend or terminating）。
   2. 直接调用```jobs```只显示进程在当前shell中的序号和进程运行的命令，但是不显示进程的pid。使用```jobs -l```可以显示pid。
2. ```disown [agrs] [pid | %{当前shell中的进程序号}]``` 
   1. args中-a代表移除当前shell中的所有作业，-r代表移除运行中的作业。使用-a或者-r后不需要指定后面的pid或者在当前shell中的进程序号**注意：移除只是代表不被当前shell看到，即使用*jobs*命令无法看到移除的进程，但是这些进程还在后台运行。**
   2. ```shell
      jobs -l # 查看当前shell中的进程及其id和pid
      disown %n # 使用通过jobs查看的id移除进程
      disown pid # 使用通过jobs查看的pid移除进程
      ```
3. ```nohup command [&]```：
   1. 作用：nohup命令用于运行程序或者命令，并忽略所有中断信号SIGHUP。SIGHUP是当前shell关闭时发送到进程的信号。
   2. ```shell
      nohup ./clash -d . & # &用于将命令放到后台运行，不在shell显示
      nohup /root/runoob.sh > runoob.log 2>&1 & # nohup命令执行后shell不会有输出，如果想把输出存下来，可以使用当前命令将所有的输出或者错误都存放到runoob.log（可以自己指定）中
      ```
   3. 在2中加&可以把进程放到后台运行，但是由于nohup命令不会在关闭shell后终止进程的运行，因此即使不加&而进程仍在运行时关闭shell，进程仍可以在后台找到。
   4. 使用nohup后shell的本来的输出会放到当前目录下的*nohup.out*文件中。
   5. 与disown的关联和区别：在disown中，通过disown移除进程后，进程也会在后台运行，但是无法在当前shell中通过jobs命令查看；而nohup虽然能够保持进程在后台的持续运行，但是仍能够在当前shell中通过jobs命令查看。因此为了保证进程在后台的永久运行，要么直接使用disown，要么使用nohup后使用disown清理在此shell中可查看到的进程，要么使用nohup后直接关闭shell。
4. 进程的挂起和终止：
   1. 前台挂起和终止进程：*ctrl+z*是suspend进程，*ctrl+c*是终止进程。进程的挂起只在当前shell有效，只要关闭当前shell，进程就会消失，不存在与后台之中。**前台进程的终止用ctrl+c，后台进程的终止用kill；前台进程的挂起用ctrl+z，后台进程的挂起用kill -stop [pid | %{当前shell进程id}]**
5. ```fg bg```
   1. ```shell
      ./clash -d . # 运行命令然后使用ctrl+z将其suspend
      jobs -l # 查看当前shell中的进程信息
      fg %n # 恢复被suspend的进程到前台运行 | 将后台运行的某个进程换到前台运行。若想将前台运行的进程换到后台运行，则先用ctrl+z将其suspend，然后使用bg将其放在后台运行。
      bg %n # 恢复被suspend的进程到后台运行
      ```
   2. 注意：没有disown和nohup命令，这些进程都只在当前shell中运行，关闭当前shell后，shell会向进程发送Sighup信号，kill这些进程。也即，bg和fg只能操纵当前shell中的进程而不能操纵后台进程。
6. ```&```：将&加到command的最后可以使command在后台运行，若有些command搭配&运行后仍占用前台，即使用ctrl+c终止进程，仍可以在jobs或者后台看到这个进程。
```shell
./clash -d . &
ctrl + c # 执行完ctrl+c后仍可以看到进程在运行
```

## 复制、粘贴文本中的内容到clipboard（用于复制记录的git token或者公钥）
1. mac
```shell
pbcopy < ~/.ssh/id_rsa.pub
echo `pbpaste` # 若要在shell中打印内容，可以用pbpaste
```
2. windows
```shell
clip < ~/.ssh/id_rsa.pub
```
3. linux
```shell
sudo apt-get install xclip
xclip -sel clip < ~/.ssh/id_rsa.pub
# Copies the contents of the id_rsa.pub file to your clipboard
```
4. 将linux中的xclip命令换为pbcopy
在~/.zshrc中加入以下内容：
```shell
alias pbcopy='xclip -selection clipboard'
alias pbpaste='xclip -selection clipboard -o'
```
5. 解决pbcopy命令会在paste时多出一个换行符的几种方法
``` shell
printf $(pwd) | pbcopy # 1
echo -n $(pwd) | pbcopy # 2
echo -n "$(pwd)" | pbcopy # 3（如果目录名称中包含空格字符用3，否则用2）
echo "abcd\c" | pbcopy # 如果直接在shell中手动输入要copy的内容，可以在内容后面加上\c
```

## 输入输出重定向
一、基础概念：
1. stdin、stdout、stderr是三个文件，分别代表标准输入、标准输出、标准错误输出，它们的文件描述符分别为0、1、2。
2. unix程序默认从stdin读取数据，默认从stdout输出数据，默认从stderr写入错误信息。

二、用法
1. 输出重定向（>、>>）：
``` shell
command > file # 泛化用法，代表把command的标准输出重定向到file中
command >> file # 泛化用法。>会把file中原有的内容全部清除之后再写入，但是>>会在原来的文件内容末尾继续写下去
command 2>file # 将stderr重定向到file
command 2>>file # 追加输入
who > list.txt
who >> list.txt
```
2. 输入重定向
``` shell
command < file # 泛化用法，command获取file中的内容作为其输入
wc -l < list.txt
```
3. 特殊例子
``` shell
command > file 2>&1 # >后面默认跟file，但是这里跟的是文件描述符1，因此要用&1，代表重定向到标准输出中，如果不加&代表重定向到当前路径下文件名为1的file中；这个命令要从左到右理解，代表输出先重定向到file中，此时在这条命令中已经默认file为stdout而不是/dev/stdout，因此标准输出会写入file，而后面的2>&1，代表将标准错误输出重定向到stdout中，因为此时stdout已经为file，因此错误信息也会被写入file。
command >> file 2>&1 # 追加输入
command > file 2>&1  =  command &> file  =  command >& file # 在csh中无法使用上面的命令，只能使用这里的简化方式
command > /dev/null # /dev/null是一个特殊的文件，写入其中的内容都会被丢弃，读取其内容也不会有结果。这条命令可以起到禁止输出的作用。
command < file1 > file2 # 先将stdin重定向到file1，然后将stdout重定向到file2。代表将file1的内容输入命令，然后将命令的输出写入到file2。
```

## ssh使用
1. 如何配置ssh：ssh的配置文件在*/etc/ssh/sshd_config*中，其中可以看到端口号、空闲超时时间等配置项。
2. 构建本地ssh密钥
``` shell
ssh-keygen -t rsa
```
3. 将本地公钥放到远程服务器以免去输入密码
``` shell
ssh-keygen -t rsa # 如果已经生成过密钥就不需要再次生成
export USER_AT_HOST="your-user-name-on-host@hostname"
export PUBKEYPATH="$HOME/.ssh/id_ed25519.pub"
ssh-copy-id -i "$PUBKEYPATH" "$USER_AT_HOST"
```

## 改变文件所属者（user、group）
查看文件所属者和所属组可用```ll(ls -l)```
1. chown
   1. 作用：主要用来改变文件的拥有者，root权限才能改。
   2. ``` shell
      chown root /home/hujunkang/a.log # 改变文件的所有者为root
      chown root:group file.txt # 改变文件的所有者为root，改变其所属组为group
      chown -R root:group * # 改变当前目录下所有文件的所有者为root，所属组为group
      ```
2. chgrp
   1. 作用：用于非root用户改变文件所属组（要求用户为组内成员）
   2. ``` shell
      chgrp -v bin log2012.log # 将文件的所属组改为bin。-v代表显示指令执行过程
      chgrp -R -v bin * = chgrp -Rv bin * # 将当前目录下所有文件的所属组改为bin
      chgrp --reference=log2012.log log2013.log # 将log2013.log的所属组改为跟log2012.log一样
      ```

## 显示目录或文件的大小：du
1. -s代表显示总计大小
2. -h代表以KB、MB等为单位，不加-h时默认以Byte为单位
``` shell
du -s # 显示当前目录的总大小
du -s * # 显示当前目录下每个目录或文件的总大小
du -sh # 显示当前目录总大小（以KB、MB等为单位）
du -sh file.txt # 显示file.txt文件的大小
```

## 用域名查询ip地址：dig
``` shell
dig www.baidu.com # 查询百度的ip地址
```
![dig](./images/dig.png)
dig 命令默认的输出信息比较丰富，大概可以分为 5 个部分。
第一部分显示 dig 命令的版本和输入的参数。
第二部分显示服务返回的一些技术详情，比较重要的是 status。如果 status 的值为 NOERROR 则说明本次查询成功结束。
第三部分中的 "QUESTION SECTION" 显示我们要查询的域名。
第四部分的 "ANSWER SECTION" 是查询到的结果。
第五部分则是本次查询的一些统计信息，比如用了多长时间，查询了哪个 DNS 服务器，在什么时间进行的查询等等。

## 查询包到达某个目的地经过的路由：traceroute
``` shell
traceroute www.google.com # 查询本机到google经过的路由
```

## 从网上下载的software是.deb文件，如何安装软件
``` shell
sudo apt-get -f -y ./xxx.deb
```

## 查看系统文件：lsof
1. 文件类型：普通文件、目录、网络文件系统的文件、字符或设备文件、共享库、管道、符号链接、网络文件（socket等）等。
2. args
   1. -a：不同选项的结果相与（默认不同选项的结果相或，即都显示出来）
   2. -c<进程名>：列出指定进程所打开的文件
   3. -t：只返回进程id
   4. -p：查看指定pid打开的内容
3. example
``` shell
lsof -u hujunkang # 列出指定用户打开了哪些文件
lsof -u ^hujunkang # 列出除了指定用户外所有其他用户所做的事情
lsof -t -u hujunkang # 只列出指定用户所有进程的进程id
kill -9 `lsof -t -u hujunkang` # 杀死指定用户所有的进程
lsof -p pid # 显示指定进程已打开的内容
```

## 查看系统上各个用户运行了多长时间
``` shell
uptime # 显示系统运行了多长时间，有几个用户
w # 显示每个用户登入系统多长时间
```
