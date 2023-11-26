
# 数据库维护

## 修复 WSL1 Ubuntu MySQL8.0 启动服务出错

```bash
# 重新创建一个已丢失的 mysql 保存运行时临时文件与进程信息的目录
$ mkdir -p /var/run/mysqld
# 赋予权限
$ chown mysql:mysql /var/run/mysqld
```

