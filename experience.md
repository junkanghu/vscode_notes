[TOC]
# Experience for Solving Problems

## about codes
1. 在安装oh-my-zsh时，先安装oh-my-zsh，再安装conda，否则conda的环境变量会找不到。
2. 

## about system
1. 文件名不要带空格：首先，带空格无法在shell中表示；其次，带空格无法在代码中引用；最后，带空格的文件若是参与pc之间的共享，会造成传输识别错误或极度延迟。
2. 在shell中打开名字含有空格的file时，需要以"\ "代替那个空格

example：
``` shell
open /Applications/ReadPaper\ (Beta).app
```
3. 