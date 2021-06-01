### 常规的启动命令

1. 基本含义

- dev/null 表示空设备文件
- 0 表示stdin标准输入
- 1 表示stdout标准输出
- 2 表示stderr标准错误
- \> file 表示将标准输出输出到file中，也就相当于 1 > file
- 2> error 表示将错误输出到error文件中
- 2>&1 也就表示将错误重定向到标准输出上，注意这里的 >& 是一体的，放在>后面的&，表示重定向的目标不是一个文件，而是一个文件描述符
- 2>&1 >file ：错误输出到终端，标准输出重定向到文件file，等于 > file 2>&1(标准输出重定向到文件，错误重定向到标准输出)。
- & 放在命令到结尾，表示后台运行，对SIGINT（sign interrupt 关联ctrl+c, 结束前台进程)信号免疫。防止终端一直被某个进程占用，这样终端可以执行别到任务，配合 >file 2>&1可以将log保存到某个文件中，但如果终端关闭，则进程也停止运行。如 command > file.log 2>&1 & 。
- nohup放在命令的开头，表示不挂起（no hang up，或者说是对SIGHUP信号免疫），也即关闭终端或者退出某个账号，进程也继续保持运行状态，一般配合&符号一起使用。如nohup command &。

```
 nohup command(java -jar 包路径包名) > 输出路径 2>&1 &
```

Ctrl+C

终止并退出前台命令的执行，回到SHELL

Ctrl+Z

暂停前台命令的执行，将该进程放入后台，回到SHELL

jobs

查看当前在后台执行的命令，可查看命令进程号码

```
$ ./test10.sh > testout
^Z
[1]+  Stopped                 ./test10.sh > testout

# 查看当前作业
$ jobs
[1]+  Stopped                 ./test10.sh > testout

# -l，列出进程的PID和作业号
$ jobs -l
[1]+ 96267 Suspended: 18           ./test10.sh > testout

# -p，只列出作业的PID
$ jobs -p
96267

# -s，只列出停止的作业
$ jobs -s
[1]+  Stopped                 ./test10.sh > testout

# -r，只列出运行的作业
$ jobs -r

$ jobs -l
[1]+ 96267 Suspended: 18           ./test10.sh > testout
[3]- 96292 Done                    ./test10.sh > testb
```

&

运行命令时，在命令末尾加上&可让命令在后台执行

fg N

将命令进程号码为N的命令进程放到前台执行，同%N

bg N

将命令进程号码为N的命令进程放到后台执行