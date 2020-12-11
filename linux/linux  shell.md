# shell常见命令入门

## seq命令
序列化缩写，主要是用来输出序列化的东西
指定分隔符  横着输出
```
qishen ~ % seq -s '#' 5
1#2#3#4#5 
```
以空格作为分格，且输出单数
```
qishen ~ % seq -s ' ' 5
1 2 3 4 5
```
```shell
qishen ~ % seq -w 1 10
01
02
03
04
05
06
07
08
09
10
```

```shell
# error
$ echo 'test' | echo
# valid
$ echo 'test' | xargs echo
```

==继续==

http://www.ruanyifeng.com/blog/2019/08/xargs-tutorial.html



==前台运行==， ==后台运行==

## 