title: Git
author: Sunkz
tags:

  - git

photos:

- https://cdn.shenlanbao.com/consultants/1666122397_20200102113533.png

categories: []
date: 2018-08-07 10:41:00

---
#### 基础指令

```
git pull origin wxy:skz  将远程wxy分支取回  并于本地skz分支合并
```

```
git rm fileName  从工作去删除  再执行git commit  不会提示   区别于rm fileName
git rm --cached fileName  从缓存区删除
git commit --amend 修改上次提交的message
```

```
git branch --set-upstream-to=origin/remote_branch  local_branch 建立关系
git branch skz v1.0 根据tag建立分支
git branch skz origin/skz  创建分支
git branch -d skz 删除本地skz分支
git checkout -b branch_name commit_id 基于commit创建分支
```

```
git push origin --delete skz 删除远程skz分支
git push origin skz 将本地分支推送到远程库
git push origin skz:master  将本地skz分支推送到远程master
git push -u origin master -f 强制覆盖远程主分支
```

```
git fetch --all  &   git reset --hard origin/master  拉取并覆盖本地
git reset --hard HEAD 
git reset --hard hash 回滚 , 上次修改内容不保留
git reset --soft hash 上次修改内容将会保留
```

```
git rebase -i hash  修改hash之后的几次commit 不包含hash 
```

```
git tag v1.0 给当前分支建立标签
git tag -d 4.5.3 删除本地tag
git push origin v1.0 将标签推送到远程库
git push origin :refs/tags/4.5.0 删除远程tag
```

```
git push —mirror repo 备份repo
```

```
git stash save ' ' 保存
git stash list 
git stash pop stash@{}  出栈
git stash apply stash@{} 查看到不drop
```

```
git remote -v 查看源
git remote remove origin 删掉原来的git源
git remote add origin [.git url] 设置新源地址
```

#### 多个SSH Key

```shell
# .ssh
-rw-r--r-- 1 sunke 197609  226 11月 28 14:56 config
-rw-r--r-- 1 sunke 197609 2578  9月  5 16:15 id_rsa
-rw-r--r-- 1 sunke 197609  554  9月  5 16:15 id_rsa.pub
-rw-r--r-- 1 sunke 197609 1464 12月 30 14:38 known_hosts
-rw-r--r-- 1 sunke 197609 2578 11月 28 14:52 sunkz_rsa
-rw-r--r-- 1 sunke 197609  554 11月 28 14:52 sunkz_rsa.pub
```

```shell
# config
Host code.aliuyn.com
HostName code.aliyun.com
PreferredAuthentications publickey
IdentityFile ~/.ssh/id_rsa

Host sunkz.code.aliuyn.com
HostName code.aliyun.com
PreferredAuthentications publickey
IdentityFile ~/.ssh/sunkz
```

