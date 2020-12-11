### 快速返回普通模式
| 按键| 用途|
| :-|:-|
|\<Esc>| 退回普通模式|
|<C - [>| 退回普通模式|
|<C - o>| 切换到插入-普通模式|
|<Command - c>|退回到普通模式|

### 技巧8 把撤销但愿切成块
上下左右方向键位 会重置插入模式的撤销块

### 技巧10 用次数做简单的算数运算（普通模式）
<C - a> 加操作 <br>
<C - x> 减操作 <br>
可以添加前缀进行多次

## 插入模式
### 14 返回普通模式
按键操作      用途
<C-o>       切换到插入-普通模式
场景，zz重绘屏幕
### 15 在插入模式下进行粘贴
| 按键| 用途|
| :-|:-|
|yt,| Practical Vim, by Drew Neil <br> Read Drew Neil's,|
|jA| Practical Vim, by Drew Neil <br> Read Drew Neil's,|
|<C - r>0 | 激活面向列块的可视模式|

查询完整的操作符
:h operator

### 技巧16 下进行普通的公式运算（insert mode)
<C - r>=表达式 <br>
注: 大部分第三方编辑器的vim插件不支持这个特性

### 技巧19 用替换模式替换已有文本
用R命令进入替换模式
## 可视模式

### 激活可视模式
| 按键| 用途|
| :-|:-|
|v| 激活面向字符的可视模式|
|V| 激活面向行的可视模式|
|<C - v> | 激活面向列块的可视模式|
|gv|重选上次的高亮选取|

### 切换选区的活动端
```
vbb
o
e
```

### 重复执行面向行的可视命令
```
Vj
>.
```

### 选择标签内内容
```
vit
```

## 命令行模式
### 常见的命令 //todo(使用自定义寄存器编写)
|命令|用途|
|:-|:-|
|:[range] delete [x]                                |删除指定范围内的行[到寄存器x]|
|:[range] yank [x]                                  |复制指定范围的行 |
|:[line] put [x]                                    |在指定行后粘贴寄存器x中的内存|
|:[range] copy {address}                            |把指定范围的内的行拷贝到{address}所指定的行之下|
|:[range] move {address}                            |把指定范围的内的行移动到{address}所指定的行之下|
|:[range] join                                      |连续指定范围内的行|
|:[range] normal {commands}                         |对指定范围内的每一行执行普通命令{commands}|
|:[range] substitute /{pattern}/{string}/[/flags]   |把指定范围内出现{pattern}的地方替换为{string}|
|:[range] global /{pattern}/[cmd]                   |把指定范围内匹配{pattern}的所有行，在其上执行Ex 命令{cmd}|

### 地址规则
数字作为行数地址
符号%表示当前文件中的所有行
```
{start}
{start},{end}
%
```



