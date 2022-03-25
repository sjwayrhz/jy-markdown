# Github Container Registry

[TOC]

## Token

账户为sjwayrhz，token如下

```
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
```

权限为

```
repo	write:packages	read:packages		delete:packages 
```

## 登陆方法

```bash
$ export CR_PAT=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
$ echo $CR_PAT | docker login ghcr.io -u sjwayrhz --password-stdin
```

