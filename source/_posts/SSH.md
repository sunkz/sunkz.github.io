title: SSH
author: Sunkz
tags:
  - ssh
  - 公钥指纹
  - CA

photos:

- https://s1.ax1x.com/2020/07/08/UZlIHg.png

categories: []
date: 2020-07-08 19:47:00

---
## 密码登录

- 在服务器设置 password

- 服务器收到 ssh 登录请求将自己的公钥发送给用户

  - 发送公钥给用户时, 用户需要先自行验证公钥指纹

  ![UmTBp8.png](https://s1.ax1x.com/2020/07/09/UmTBp8.png)

  ![Um7mjg.png](https://s1.ax1x.com/2020/07/09/Um7mjg.png)

  > 公钥指纹 : 是服务器公钥的一个摘要. 由于 ssh 不存在 CA, 无法保证公钥就是目标服务器的公钥而不是中间人的公钥, 所以需要用户自行拿公钥指纹去目标服务器官网公布的公钥指纹做对比. 
  >
  > 使用公钥指纹作对比而不使用公钥作对比的原因是 : 公钥太长,不方便对比.

  > 第一次登录时需要验证公钥指纹, 输入 yes 之后将保存公钥信息到 ~/.ssh/know_hosts.之后无需验证公钥指纹.

- 用户使用公钥对 password 进行加密, 发送到服务器

  ![UnMw11.png](https://s1.ax1x.com/2020/07/09/UnMw11.png)

  > 使用公钥对 password 加密 : 防止 password 在网络传输中被截获

- 服务器用自己的私钥进行解密

```
整个过程既有对称加密: password.  又有非对称加密 : 使用服务器的 publickey 对 password 加密传输.
使用密码登录缺点 : 每次都需要输入密码, 麻烦, 且存在暴力破解可能
```

## 公钥免密登录

- 用户在本机生成密钥对

  ![UmXMbF.png](https://s1.ax1x.com/2020/07/09/UmXMbF.png)

  ![UmLaz6.png](https://s1.ax1x.com/2020/07/09/UmLaz6.png)

- 将 id_rsa.pub 公钥上传到服务器

  ![Umj1L8.png](https://s1.ax1x.com/2020/07/09/Umj1L8.png)

  > 上传公钥时候需要输入一次 password

- ssh 登录时候, 服务器发送给用户一个随机字符串

- 用户使用私钥对其加密

- 服务器使用用户的公钥对其进行解密

```
公钥登录缺点 : 服务器需要保存用户的公钥. 本质还是对密码登录的简化, 仍存在暴力破解密码的可能.
Git 操作时候 remote repo 配置的就是这种公钥.
```

## 证书登录

- 用户 ssh 登录时, 将自己的证书发给服务器.
- 服务器去 CA 验证用户的证书.
- 服务器将自己的证书发送给用户
- 用户去 CA 验证服务器证书有效性
- 双方建立连接.

```
引入了 CA, 类比 HTTPS 流程.
```
