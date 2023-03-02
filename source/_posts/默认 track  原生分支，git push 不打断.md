---
title: git 默认 track 原生分支，git push 不打断
tags:
  - git
date: 2023-01-09 15:41:54
---

每次 checkout 一个新分支时，git push 总会提示类似下面的提示，略微难受

![image](./images/git-track-demo.png)

在大部分情况下，我们的本地分支和远程分支名应该是一致的（这也方便你管理需求），因此可以使用下面的命令设置默认 track

```sh
git config --global --replace-all push.default current
git config --global --replace-all push.autoSetupRemote true
```

更多配置看这里↓：

[https://git-scm.com/docs/git-config](https://git-scm.com/docs/git-config)

![image](/images/git-push-doc.png)


某些情况下，你需要修改你的需求分支名，在修改分支名称后，你先修改了本地分支与远程分支的映射，但 `git push` 后发现还是会映射到旧分支名。这时可以尝试直接修改本地的 cache 。

```
vi ~/.git/config
```

![image](./images/git-track-config.png)