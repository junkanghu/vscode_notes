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

## 建立软链接
``` shell
ln -s source target
```
例如source为/dellnas/dataset/static_recon/Relight/junkangs/head_1118/（这个“/”不能缺，代表将整个head_1118文件夹转移到某个路径下）, target为/dellnas/home/hujunkang/data/（data这个文件夹必须存在，head_1118文件夹将会软链接到data文件夹下，名称为head_1118）。

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
      1. u,g,o,a：分别代表user、group、others、all
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

二、grep语法
1. asdf 


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