---
title: OAuth 2.0
date: 2018-02-25
tags: [node]
categories: node
---

### 一、介绍

  OAuth 2.0是授权相关的标准协议。OAuth 2.0取代了在2006年创建的原始OAuth协议上所做的工作。OAuth 2.0专注于客户端开发人员的简单性，同时为Web应用程序，桌面应用程序，手机和客厅设备提供特定的授权流程.

### 二、核心概念
- OAuth 2.0框架 - RFC 6749
- OAuth 2.0授权类型
  * Authorization Code（ 授权码模式 ）
  * Implicit（ 隐式模式 ）
  * Password（ 密码模式 ）
  * Client Credentials（ 客户端模式 ）
  * Device Code（ 设备码 ）
  * Refresh Token（ 刷新码 ）
- OAuth 2.0 Bearer Tokens（ 无记名令牌 ）

### 三、[协议流程](https://tools.ietf.org/html/rfc6749)

```
+--------+                                    +---------------+
|        |--(A)- Authorization Request ->|   Resource    |
|        |                               |     Owner     |
|        |<-(B)-- Authorization Grant ---|               |
|        |                               +---------------+
|        |
|        |                               +---------------+
|        |--(C)-- Authorization Grant -->| Authorization |
| Client |                               |     Server    |
|        |<-(D)----- Access Token -------|               |
|        |                               +---------------+
|        |
|        |                               +---------------+
|        |--(E)----- Access Token ------>|    Resource   |
|        |                               |     Server    |
|        |<-(F)--- Protected Resource ---|               |
+--------+                               +---------------+
```

流程描述:

- A 客户端请求用户给予授权
- B 用户给予客户端授权
- C 客户端使用B给予授权，申请令牌
- D 认证服务器确认，给予AccessToken
- E 客户端使用令牌，获取资源
- F 服务器确认，返回信息

### 四、服务端实现

#### 4.1、提供一个授权页面（/oauth/authorize）用于登陆授权

页面会重定向到URL

    http://127.0.0.1:5700/auth/****/callback?code=99660996228df4a2947af7c9869b960b8a452e08

![image](/images/oauth/authorize.png)

#### 4.2、根据授权码获取accessToken
根据上一步获取的code，得到accessToken
```
POST /oauth/access_token HTTP/1.1
Host: 127.0.0.1:5700
Content-Type: application/x-www-form-urlencoded
Authorization: Basic d2ViaW1hZ2VyeDpwZjdUYUt4QXlpV0Q3RllN
grant_type=authorization_code&redirect_uri=''&code=99660996228df4a2947af7c9869b960b8a452e08
```

![image](/images/oauth/accessToken.png)
![image](/images/oauth/accessHeader.png)


#### 4.3、根据accessToken获取用户信息
```
GET /api/user HTTP/1.1
Host: localhost:5700
Authorization: Bearer dca31d4ff0d5a8d1a80f9acf149908bcff468b2f
```

![image](/images/oauth/profile.png)

### 五、oauth2-server

授权码模式下需实现方法:
- [ ] generateAccessToken(client, user, scope, [callback]) 可选
- [ ] generateRefreshToken(client, user, scope, [callback]) 可选
- [ ] generateAuthorizationCode(client, user, scope, [callback]) 可选
- [x] getAuthorizationCode(authorizationCode, [callback])
- [x] getClient(clientId, clientSecret, [callback])
- [x] saveToken(token, client, user, [callback])
- [x] saveAuthorizationCode(code, client, user, [callback])
- [x] revokeAuthorizationCode(code, [callback])
- [x] validateScope(user, client, scope, [callback])

#### 5.1、generateAccessToken(client, user, scope, [callback]) 
生成accessToken，默认生成长度为40字节的字符串

#### 5.2、generateRefreshToken(client, user, scope, [callback]) 
生成refreshToken，默认生成长度为40字节的字符串

#### 5.3、generateAuthorizationCode(client, user, scope, [callback]) 
生成authorizationCode，默认生成长度为40字节的字符串

#### 5.4、getAuthorizationCode(authorizationCode, [callback])
获取AuthorizationCode
```js
const getAuthorizationCode = function(authCode, cb) {
  let query = { authorizationCode: authCode };
  authCodeModel.findOne(query)
    .populate('user', '_id name avatar email role scope')
    .populate('client', '_id title client_id user redirectUris grants scope public').lean()
    .exec(function(err, code) {
      let client = null;
      if (err) {
        return cb(err);
      }
      if (!code) {
        return cb('no authCode');
      }
      if (client = code.client) {
        client.id = client.client_id;
        code.client = client;
      }
      cb(null, code);
  });
};
```
#### 5.5、getClient(clientId, clientSecret, [callback])
获取应用
```js
const getClient = function(clientId, clientSecret, cb) {
  let query = { client_id: clientId };
  let projections = '_id title client_id redirectUris grants scope public';
  if (clientSecret) { query.client_secret = clientSecret }
  clientModel.findOne(query, projections).lean().exec(function(err, client{
    if (!client) {
      return cb('client not found');
    }
    client.id = clientId;
    return cb(err, client);
  })
}
```

#### 5.6、saveToken(token, client, user, [callback])
生成 accessToken和refreshToken
```js
const saveToken = function(token, client, user, cb) {
  let fns = [
    accessTokenModel.create({
      accessToken: token.accessToken,
      accessTokenExpiresAt: token.accessTokenExpiresAt,
      client: client._id,
      user: user._id,
      scope: token.scope
    }), token.refreshToken ? refreshTokenModel.create({
      refreshToken: token.refreshToken,
      refreshTokenExpiresAt: token.refreshTokenExpiresAt,
      client: client._id,
      user: user._id,
      scope: token.scope
    }) : Promise.resolve()
  ];
  return Promise.all(fns)
  .then(function(result) {
    let ret = _.assign({
      client: client,
      user: user
    }, token);
    return cb(null, ret);
  }).catch(function(err) {
    return cb(err);
  });
};
```

#### 5.7、saveAuthorizationCode(code, client, user, [callback])
生成 authorizationCode
```js
const saveAuthorizationCode = function(code, client, user, cb) {
  let doc = {
    expiresAt: code.expiresAt,
    client: client._id,
    authorizationCode: code.authorizationCode,
    user: user._id,
    scope: code.scope
  };
  return authCodeModel.create(doc, function(err, code) {
    if (err) {
      return cb(err);
    }
    code = _.pick(code, ['authorizationCode', 'expiresAt']);
    return cb(null, code);
  });
};
```

#### 5.8、revokeAuthorizationCode(code, [callback])
将authorizationCode置为无效
```js
const revokeToken = function(token, cb) {
  let query = {
    refreshToken: token.refreshToken
  };
  let update = {
    expiresAt: Date.now()
  };
  return refreshTokenModel.findOneAndUpdate(query, update, cb);
};
```

#### 4.9、validateScope(user, client, scope, [callback])
验证scope的有效性
```js
const validateScope = function(user, client, scope) {
  return true;
};
```

### 六、表模型
#### 6.1、accessToken
```js
const mongoose = require('mongoose');

let accessTokenSchema = new mongoose.Schema({
  user: {
    type: mongoose.Schema.Types.ObjectId,
    ref: 'user',
    required: true
  },
  client: {
    type: mongoose.Schema.Types.ObjectId,
    ref: 'client',
    required: true
  },
  accessToken: {
    type: String,
    required: true,
    unique: true
  },
  scope: String,
  accessTokenExpiresAt: Date
});

accessTokenSchema.index({
  userId: 1
});

accessTokenSchema.index({
  clientId: 1
});

const accessTokenModel = mongoose.model('oauthAccessToken', accessTokenSchema);
```

#### 6.2、authCode
```js 
const mongoose = require('mongoose');

let authCodeSchema = new mongoose.Schema({
  authorizationCode: {
    type: String,
    required: true
  },
  user: {
    type: mongoose.Schema.Types.ObjectId,
    ref: 'user',
    required: true
  },
  client: {
    type: mongoose.Schema.Types.ObjectId,
    ref: 'client',
    required: true
  },
  redirectUri: String,
  scope: String,
  expiresAt: Date
});

authCodeSchema.index({
  userId: 1
});

authCodeSchema.index({
  clientId: 1
});

const authCodeModel = mongoose.model('oauthCode', authCodeSchema);
```

#### 6.3、refreshToken
```js
const  mongoose = require('mongoose');

let refreshTokenSchema = new mongoose.Schema({
  user: {
    type: mongoose.Schema.Types.ObjectId,
    ref: 'user',
    required: true
  },
  client: {
    type: mongoose.Schema.Types.ObjectId,
    ref: 'client',
    required: true
  },
  refreshToken: {
    type: String,
    required: true,
    unique: true
  },
  scope: String,
  refreshTokenExpiresAt: Date
});

refreshTokenSchema.index({
  userId: 1
});

refreshTokenSchema.index({
  clientId: 1
});

const refreshTokenModel = mongoose.model('oauthRefreshToken', refreshTokenSchema);
```
#### 6.4、scope
```js

const mongoose = require('mongoose');

let scopeSchema = new mongoose.Schema({
  scope: String,
  is_default: Boolean
});

scopeSchema.index({
  scope: 1
});

scopeSchema.index({
  is_default: 1
});

const scopeModel = mongoose.model('oauthScope', scopeSchema);
```

#### 6.5、client
```js
const mongoose = require('mongoose');

let clientSchema = new mongoose.Schema({
  title: String,
  client_name: {
    type: String,
    required: true
  },
  client_id: {
    type: String,
    required: true,
    unique: true
  },
  client_secret: {
    type: String,
    required: true
  },
  url: String,
  user: {
    type: mongoose.Schema.Types.ObjectId,
    ref: 'user'
  },
  owners: [
    {
      type: mongoose.Schema.Types.ObjectId,
      ref: 'user'
    }
  ],
  "public": {
    type: Boolean,
    "default": false
  },
  dongle: {
    type: Boolean,
    "default": false
  },
  redirectUris: {
    type: [String]
  },
  grants: {
    type: [String],
    "default": ['authorization_code', 'password', 'refresh_token', 'client_credentials']
  },
  scope: {
    type: String,
    "default": 'profile'
  },
  createAt: {
    type: Date,
    "default": Date.now
  },
  updateAt: {
    type: Date,
    "default": Date.now
  },
  icon: String,
  summary: String
});

clientSchema.index({
  client_id: 1,
  title: 1,
  createAt: 1
});

const clientModel = mongoose.model('client', clientSchema);
```
### 参考文件
- [oauth](https://oauth.net/2/)
- [理解OAuth 2.0](http://www.ruanyifeng.com/blog/2014/05/oauth_2_0.html)
- [node-oauth2-server-implementation](https://github.com/manjeshpv/node-oauth2-server-implementation)