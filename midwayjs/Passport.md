
# 权限验证

## CSRF token

1. 安装依赖，security 组件支持 `csrf` 、`xss` 等多种安全策略

```bash 
$ npm i @midwayjs/security --save
```

2. 在 configuration 中引入组件

```TypeScript
// src/configuration.ts
import * as security from '@midwayjs/security';  
@Configuration({  
	imports: [  
		// ...other components  
		security  
	],  
})  
export class MainConfiguration {}
```

3. 配置 security.csrf

```typeScript
// src/config/config.default.ts
import { MidwayConfig } from '@midwayjs/core';

export default {
  // ...其他配置项
  session: {
	// 使 cookie 在关闭窗口时自动失效，实现关闭即退出的效果
	// 但在不关闭窗口的情况，几乎永久有效
    maxAge: 'session', 
  },
  security: {
    csrf: {
      useSession: true, // 使用会话保存 csrfToken
    },
  },
  // ...
}
```

## 本地登陆策略

1. 安装依赖

```bash
$ npm i @midwayjs/jwt passport-jwt --save
```

2. 在 configuration 中引入组件

```TypeScript
// configuration.ts  
import { join } from 'path';  
import * as jwt from '@midwayjs/jwt';  
import { Configuration, ILifeCycle } from '@midwayjs/core';  
import * as passport from '@midwayjs/passport';  
  
@Configuration({  
	imports: [  
		// ...  
		jwt,  
		passport,  
	],  
	importConfigs: [join(__dirname, './config')],  
})  
export class MainConfiguration implements ILifeCycle {}
```

3. 配置 jwt

```typescript
// src/config/config.default.ts
import { MidwayConfig } from '@midwayjs/core';

export default {
  // ...其他配置项
  jwt: {
    secret: 'sUy#zxt!YgSGj59', // 密钥
    expiresIn: '1d',
  },
  // ...
}
```

4. 自定义 jwt 中间件

```typescript
// src/strategy/jwt.strategy.ts
import { CustomStrategy, PassportStrategy } from '@midwayjs/passport';
import { Strategy, ExtractJwt } from 'passport-jwt';
import { Config, Inject } from '@midwayjs/core';

import { UserService } from '../service/user.service';

@CustomStrategy()
export class JwtStrategy extends PassportStrategy(Strategy, 'jwt') {

  @Config('jwt')
  jwtConfig;

  @Inject()
  userService: UserService;

  async validate(payload: any) {
    return payload;
  }

  getStrategyOptions(): any {
    return {
      secretOrKey: this.jwtConfig.secret,
      jwtFromRequest: ExtractJwt.fromAuthHeaderAsBearerToken(),
    };
  }

  serializeUser(user, done) {
    done(null, user?.username);
  }

  // 先执行 deserializeUser， 后执行 serializeUser
  deserializeUser(id, done) {
	// 利用
    this.userService
      .getUser({ username: id })
      .then(user => {
        done(null, user);
      })
      .catch(err => {
        done(err);
      });
  }
}
```