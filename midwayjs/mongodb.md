
# 使用 MongoDB 
## 数据库

*官方下载*
- 服务器（社区版） [MongoDB Community Server](https://www.mongodb.com/try/download/community)
* 命令行工具 [MongoDB Shell](https://www.mongodb.com/try/download/shell)
* 图形化工具 [MongoDB Compass](https://www.mongodb.com/try/download/compass) 

*官方文档*
* 云端部署工具 [MongoDB Atlas](https://www.mongodb.com/docs/atlas/)
* 数据库手册 [MongoDB Manual](https://www.mongodb.com/docs/manual/)
### 创建数据库（示例）

```bash
## mongosh

# 使用数据库
$ use MLdb
# 如果数据库不存在则创建数据库
$ db

# 创建用户
$ db.createUser({ user: 'MLdb', pwd: '1Qaz@wsx', roles: [{role: 'readWrite', db: 'MLdb'}]}) 

# 创建集合(表)
$ db.createCollection('account')

# 插入账户信息
$ db.account.insert({ username: 'admin', password: 'JAvlGPq9JyTdtvBO6x2llnRI1+gxwIyPqCKAn3THIKk=', email: '123456@xx.com', nickname: 'wanderer' })
```

### 启用登录验证

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
$ mongosh 127.0.0.1:27017/MLdb -u MLdb -p 1Qaz@wsx
```


## 开发
### 安装依赖

```bash
# 进入项目目录下
# ORM: typeorm 
$ npm i @midwayjs/typeorm@3 typeorm mongodb --save

# 注意：mongodb@6.2.0/package.json 中 engines.node 必须 >= 16.20.1
# 使用 nvm 安装 node 升级到指定版本
$ nvm install 16.20.1
$ nvm use 16.20.1
```


### 添加依赖

```TypeScript
// configuration.ts
import { Configuration } from '@midwayjs/core';
import * as orm from '@midwayjs/typeorm';
import { join } from 'path';

@Configuration({
  imports: [
    // ...
    orm                                             // 加载 typeorm 组件
  ],
  importConfigs: [
    join(__dirname, './config')
  ]
})
export class MainConfiguration {

}
```

### 定义实体类

```TypeScript
// src/entity/user.entity.ts
import { Entity, Column, PrimaryColumn } from 'typeorm';

@Entity('account')
export class User {

  @PrimaryColumn()
  username: string;

  @Column()
  password: string;
  
  @Column()
  email: string;

  @Column()
  nickname: string;
}
```

### 添加数据源配置

```TypeScript
// src/config/config.default.ts

export default {
  // ...
  typeorm: {
    dataSource: {
      default: {
        type: 'mongodb',
        host: '127.0.0.1',
        port: 27017,
        database: 'MLdb',
        username: 'MLdb',
        password: '1Qaz@wsx',
        synchronize: false,   // 一般只在初次开发时使用同步创建表功能
        logging: true,
        entities: ['**/entity/*.entity{.ts,.js}'], // 扫描实体文件目录
      }
    }
  },
}
```

### 调用实例

```TypeScript
import { Provide } from '@midwayjs/core';
import { InjectEntityModel } from '@midwayjs/typeorm';
import { Repository } from 'typeorm';
import { User } from '../entity/user.entity';
import { IUserOptions } from '../interface';
  
@Provide()
export class UserService {

  @InjectEntityModel(User)
  userModel: Repository<User>;

  async getUser(options: IUserOptions) {
    const { username, password } = options;

	// 查询用户信息
    return await this.userModel.findOne({
      where: { username, password },
      select: ['username', 'email', 'nickname'],
    });
  }
}
```
