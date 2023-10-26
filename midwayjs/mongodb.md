
#创建用户

```bash
## mongosh

# 使用数据库
> use MLdb
# 如果数据库不存在则创建数据库
> db

# 创建用户
> db.createUser({ user: 'MLdb', pwd: '1Qaz@wsx', roles: [{role: 'readWrite', db: 'MLdb'}]}) 

# 创建集合(表)
> db.createCollection('account')

# 插入账户信息
> db.account.insert({ username: 'admin', password: 'JAvlGPq9JyTdtvBO6x2llnRI1+gxwIyPqCKAn3THIKk=', email: '123456@xx.com', nickname: 'wanderer' })
```

#启用登录验证

```config
## Linux: /etc/mongod.conf
## Windows: <install directory>\bin\mongod.cfg

# Where and how to store data.
storage:
  dbPath: ..\MongoDB\Server\7.0\data

# where to write logging data.
systemLog:
  destination: file
  logAppend: true
  path:  ..\MongoDB\Server\7.0\log\mongod.log

# network interfaces
net:
  port: 27017
  bindIp: 127.0.0.1

# 启用登录验证 
security:
  authorization: enabled
```

```bash
# 登录本地数据库
> mongosh 127.0.0.1:27017/MLdb -u MLdb -p 1Qaz@wsx
```

