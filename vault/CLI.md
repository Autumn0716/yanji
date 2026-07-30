---
title: CLI
description: 1. 查看网络端口使用ports lsof -iTCP -sTCP:LISTEN -P -n -i表示internet 筛选网络连接相关的文件 -s表示state 按照连接状态过滤,比如筛选LISTEN状态的连接 -P表示只看端口数字,不显示服务名
tags:
- imported-from-obsidian
---

1. 查看网络端口使用*ports*
   `lsof -iTCP -sTCP:LISTEN -P -n`
   `-i`表示internet 筛选网络连接相关的文件
   `-s`表示state 按照连接状态过滤,比如筛选LISTEN状态的连接
   `-P`表示只看端口数字,不显示服务名
   `-n`表示不做DNS反查
2. 目前打算长期使用[otty](./otty.md),可以使用`cmd+j`来快速打开一个命令面板,可以自如切换recipe
   tips: cmd可以使用`place of interest` ⌘J这样子类似还有 ⌥🌐
   这个叫做`字符检视器`
3. ⇧+↔️可以用==键盘==选中文字(不必鼠标)
3. SSH密钥配置

```shell
#1.生成一对ssh密钥(如有可跳过)
ssh-keygen -t ed25519 -f ~/.ssh/id_ed25519

#2.输出pub
cat ~/.ssh/id_ed25519.pub

#3.公钥移植到目标服务器
mkdir  ~/.ssh
vim ~/.ssh/authorized_keys
echo "YOUR_PUB_KEYS" >> ~/.ssh/authorized_keys
```

## 迁移备注

以下原 wiki 链接在库中无对应文件，已改为纯文本：
- `DNS`
- `recipe`
- `端口`

