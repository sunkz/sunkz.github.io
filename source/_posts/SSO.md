title: SSO
author: Sunkz
tags:

  - OAuth
  - LDAP

photos:

- https://s1.ax1x.com/2020/07/31/alB3wQ.png

categories: []
date: 2020-07-31 17:20:00

---

### OAuth 2.0

- 核心目的

  ```
  使用 临时令牌 代替 用户名密码, 保证用户数据的安全性.
  ```

- 为什么 APP 都用授权码模式

  > The authorization code flow offers a few benefits over the other grant types. When the user authorizes the application, they are redirected back to the application with a temporary code in the URL. The application exchanges that code for the access token. When the application makes the request for the access token, that request is authenticated with the client secret, which reduces the risk of an attacker intercepting the authorization code and using it themselves. This also means the access <b>token is never visible to the user</b>, so it is the most secure way to pass the token back to the application, reducing the risk of the token leaking to someone else.
  >
  > https://www.oauth.com/oauth2-servers/server-side-apps/authorization-code/
  > 

### LDAP



