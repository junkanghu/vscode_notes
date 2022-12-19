# Ubuntu Commands

## common
### ```curl cheat.sh/command``` 
will give a brief "cheat sheet" with common examples of how to use a shell command.

### delete files with specific format
``` shell
ls | egrep "DSC0\d{4}.ARW" | xargs rm
```

### move files
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

### ${n:-xxx}
if $n is empty, take the one follows ":", i.e., -xxx

### ${n:-}
if $n is empty, then the output is empty

## string
### '' apostrophe

``` shell
kang='kang'
hu='hujunkang$kang'
echo $hu
```
only returns the string content itself: hujunkang$kang

### "" double quotes
``` shell
kang='123'
hu="hujunkang$kang"
echo
```
returns the quoted one: hujunkang123

## echo 

### echo auguments
``` shell
hu="hujunkang"
echo $hu # return the value of hu
echo ${hu} # the more safe way compared to the command in line 7
echo ${hu:n} # return the substring starting at index n
echo ${hu:m:n} # return the substring starting at index m and its length is n
```

### echo $# hu
returns the length of the argument hu


## ll = ls -l
list the information of the file (including permission)

## Modify files with specific extension under current directory
``` shell
find ./ -name "*.JPG" | awk -F "." '{print $2}' | xargs -i -t mv ./{}.JPG  ./{}.jpg
```

## Delete files with specific extension under current directory
``` shell
find . -name "*.ARW" |xargs rm -rfv
find . -name "a*" |xargs rm -rfv
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

## find命令
格式：```find pathname -options [-print -exec -ok]```
1. -options:
   1. -name：按照文件名查找。
   2. -mtime -n/+n：按照文件更改时间查找。-n代表n天之内，+n代表n天之外。
   3. -type：查找某一类文件。d代表目录，f代表普通文件。
2. -print：将匹配的文件输出到标准输出
   1. -print0:可以将文件名以‘\0’隔开
3. -exec：对匹配的文件执行给出的shell命令。
4. -ok：与-exec作用相同，但是在执行命令前会请求用户确认。

```shell
find ~ -name "*.txt" -print # 在用户目录中查找txt文件并显示
find . -name "[A-Z]*" -print # 当前目录查找以大写字母开头的文件
find / -name "host*" -print # 整个系统查找以host开头的文件
find / -mtime -1 -type f -exec ls -l {} \; # 查询当天修改过的文件（后面的\;不能少，也是命令一部分）
find . -name 'kk' -exec mv {} jk \; # 将名为kk的file或dir重新命令为jk
```