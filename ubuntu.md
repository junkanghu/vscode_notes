[toc]
# Ubuntu Commands

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