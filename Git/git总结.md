### git密钥

Q:新建的仓库使用git clone ，但是却无法拉下来。

A:这是因为需要预先指定密钥。添加密钥的方法为

```
ssh-agent bash
ssh-add ~/.ssh/id_rsa
```

用此方法，指定 .ssh/id_rsa 为默认的git 密钥

### git回退

* \\1. git回退到上一个版本

```
git reset --hard HEAD^
```

\\2. 回退到前3次提交之前，以此类推，回退到n次提交之前

```
git reset --hard HEAD~3
```

\\3. 查看commit的sha码

```
git log` `git show dde8c25694f34acf8971f0782b1a676f39bf0a46
```

\\4. 退到/进到 指定commit的sha码

```
git reset --hard dde8c25694f34acf8971f0782b1a676f39bf0a46
```

\\5. 强推到远程

```
git push origin HEAD --force
```