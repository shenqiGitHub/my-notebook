登陆mysql客户端

```shell
mysql -h主机名或者IP地址 -u用户名 -p密码
```

查看mysql是否是自动提交

```sql
show variables like 'autocommit'
```

关闭自动提交

```sql
set autocommit=off; 
```

Todo