只有mac版本，其他版本可以做参考

### 一. 准备条件

1. ssh工具
2. git工具
3. 手

### 二. 配置方法

1. 生成公私钥对

```shell
ssh-keygen -t rsa -C "xxx@xxx.xxx" -f ~/.ssh/id_rsa_gitee
ssh-keygen -t rsa -C "yyy@yyy.yyy" -f ~/.ssh/id_rsa_github
```

> -t 指定加密算法
>
> -C 后面是一个注释信息，并不一定要与Git账号的邮箱或者用户名关联，随便造
>
> -f 密钥生成目录以及文件名，不填写后续命令会让补全



2. 添加私钥到ssh的高速缓存，因为git 应用会默认读取id_rsa,为了让SSH识别新的私钥，需将其添加到SSH agent中

```shell
ssh-add ~/.ssh/id_rsa_gitee
ssh-add ~/.ssh/id_rsa_github
```

> 如果报错 **Could not open a connection to your authentication agent**
>
> 执行如下命令　**ssh-agent bash**

3. 编辑config文件,如果不存在在~/.ssh 目录下新建编辑(没感觉有啥吊用)

```
# gitee
Host gitee.com
    HostName gitee.com
    Port 22
    PreferredAuthentications publickey
    IdentityFile ~/.ssh/id_rsa_gitee
    User shenqiax@vip.qq.com
# github
Host github
    HostName github.com
    Port 443
    IdentityFile ~/.ssh/id_rsa
    PreferredAuthentications publickey
    User shenqiGitHub
    
# 配置文件参数
# Host：对识别的模式，进行配置对应的的主机名和ssh文件
# HostName：登录主机的主机名
# PreferredAuthentications：设置登录方式，publickey公钥，改成password则要输密码
# IdentityFile：私钥全路径名
```

4. 分别把两个公钥的内容复制到github和gitee的ssh设置中
5. 测试

```
  ssh -T git@gitee.com
ssh -T git@github.com
```

### 三. 同一个项目关联两个平台

默认远程项目为origin,查看

```
git remote -v  
origin  https://gitee.com/xxx/xxxxxgit (fetch)        
origin  https://gitee.com/xxx/xxxxx.git (push)
```



建议删除，重命名关联

```
git remote rm origin
git remote add gitee git@gitee.com:xxxx/xxx.git
```





四、You may need to set git config user details for any project.

```
$ cd ~/home_project
$ git config user.name "home_user"
$ git config user.email "your_name@home_email.com" 
```

