---
title: SSHLink
date: 2025-10-24 15:49:27
tags: Tips
---

# SSH Link
在用vscode连接远端服务器，账号之前删除过，现在重现创建后连不上了。很可能是以前连的本地密钥没删，重装后连不上了。

当你用 VS Code (或其他任何 SSH 客户端) 连接服务器时，主要涉及两个地方的"信任"：

- 服务器信任你（你的电脑）：通过authorized_keys文件。

- 你（你的电脑）信任服务器：通过known_hosts文件。

情况一：一般报错Permission denied (publickey)，需要在服务器上重新添加自己的公钥

情况二：一般报错REMOTE HOST IDENTIFICATION HAS CHANGED 或 OFFENDING_KEY_IN_KNOWN_HOSTS，需要修改自己本地的~/.ssh/known_hosts 文件

