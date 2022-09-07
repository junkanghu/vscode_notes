# Ubuntu Commands

## common
### ```curl cheat.sh/command``` 
will give a brief "cheat sheet" with common examples of how to use a shell command.


## grep -E = egrep
``` shell
egrep '\.MP4$|\.'
```
used for searching a string in the target string using regular expression

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