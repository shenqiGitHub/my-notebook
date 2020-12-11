

## 所需组件

### **1.ffmpeg安装**

#### 	1.1. YUM方式

##### 	升级系统

```shell
sudo yum install epel-release -y
sudo yum update -y
sudo shutdown -r now
```

##### 	挂载Yum源

​	a.Centos7

```shell
sudo rpm --import http://li.nux.ro/download/nux/RPM-GPG-KEY-nux.ro
sudo rpm -Uvh http://li.nux.ro/download/nux/dextop/el7/x86_64/nux-dextop-release-0-5.el7.nux.noarch.rpm
```

​	b.Centos 6

```shell
sudo rpm --import http://li.nux.ro/download/nux/RPM-GPG-KEY-nux.ro
sudo rpm -Uvh http://li.nux.ro/download/nux/dextop/el6/x86_64/nux-dextop-release-0-2.el6.nux.noarch.rpm
```

##### 	安装ffmpeg和ffmpeg开发包

```shell
sudo yum install ffmpeg ffmpeg-devel -y
```

##### 	测试

```shell
ffmpeg
```



#### **1.2 直接安装源文件方式**

首先在官网http://ffmpeg.org/download.html下载ffmpeg-4.2.1.tar.bz2

放到linux服务器解压

解压完成的ffmpeg-4.2.1目录下安装yasm(NASM汇编器)

```shell
yum install yasm
```









# docker部署

# 启动一个名 nginx81 的 nginx 容器, 添加日志记录启动

```shell
docker run --name nginx81 -d -p 81:80 -v /usr/docker/nginx81/html/:/Users/qishen/Documents/share/nginx/html -d nginx -v /logs:/Users/qishen/Documents/log/nginx -d nginx -v
```



2)拷贝容器内的配置文件到本地，进行个性化配置等操作

```shell
docker cp nginx:/etc/nginx/nginx.conf /Users/qishen/Documents/docker/nginx81/nginx.conf
```



