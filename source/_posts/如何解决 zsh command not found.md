---
title: 如何解决 zsh command not found
tags:
  - zsh
  - linux
date: 2023-03-15 11:14:28
---

在日常工作中，当我们在终端输入一些命令时，有时会遇到 `zsh: command not found: xxx` 类似的错误。我们可能下意识的通过重新安装命令来解决类似的问题，但是如果重新安装也解决不了呢，那我们就需要从源头上来考虑:是什么样的机制导致命令不能正确被执行?

## 1. PATH 环境变量

当我们把`zsh command not found: xxx`这句话翻译为机器可以识别的语言时应该是这样的:**在操作系统可识别的命名空间里找不到一个名字叫做 `xxx` 的可执行文件**。

这个所谓的命名空间就是 PATH 环境变量，我们依然可以记得在大学时期学习 Java 配置 java_home jdk 时的操作，windows 和 macOS 是同样的，我们需要的就是配置一个可以访问到我们目标执行文件的 PATH 环境变量，通常来说，这里的环境变量是一个地址。

我们可以通过执行下面的命令，查看当前系统的环境变量。

```bash
# exec below command in terminal
echo $PATH
```
输出如下：
> */Users/pengfei.cheng/.bun/bin:/Users/pengfei.cheng/.gvm/pkgsets/system/global/bin:/usr/local/Cellar/go/1.17.2/libexec/bin:/bin:/Users/pengfei.cheng/.gvm/bin:/usr/local/opt/mysql@5.7/bin:/usr/local/Cellar/pyenv-virtualenv/1.1.5/shims:/Users/pengfei.cheng/.pyenv/shims:/Users/pengfei.cheng/.pyenv/bin:/Applications/Visual Studio Code.app/Contents/Resources/app/bin:/Users/pengfei.cheng/.nvm/versions/node/v14.19.0/bin:/Users/pengfei.cheng/.oh-my-zsh/custom/plugins/git-open:/Users/pengfei.cheng/bin:/usr/local/bin:/usr/local/bin:/usr/bin:/bin:/usr/sbin:/sbin:/usr/local/go/bin:/Library/Apple/usr/bin:/usr/local/bin:/Users/pengfei.cheng/go/bin*

我们需要关注的是以下几个路径：

- `/bin & /sbin`: 系统的一些内置命令集合，所有用户都可以访问
- `/usr/bin`: 如果电脑存在多个账户，适用于所有用户使用的命令集合
- `/usr/local/bin`: 如果电脑存在多个账户,这里存放当前登录用户安装的一些命令集合
- `$CUSTOME_PATH/bin`: 自定义的一些命令路径，如 nvm / python / go 的命令集合
- 更多参考 [https://www.wikiwand.com/en/Filesystem_Hierarchy_Standard](https://www.wikiwand.com/en/Filesystem_Hierarchy_Standard)

所以回过头再看 `zsh command not found` 这个问题，其实就是命令安装的过程中，一些命令独有的 bin 目录并没有被正确的设置到 PATH 环境变量（或者没有被正确识别）。

## 2. 解决 `zsh command not found`

### 2.1 找到命令的安装路径

我们以 `create-react-app` 不能正确访问为例,首先查看当前命令的可执行文件在系统的哪个目录。

```bash
# exec below command in terminal
whereis create-react-app
```
输出如下：
> *create-react-app: /usr/local/bin/create-react-app*

我们可以看到我们的 `create-react-app` 是安装在 `/usr/local/bin` 路径下的。

```bash
# exec below command in terminal
lsa /usr/local/bin | grep create-react-app
```
因为我使用 yarn 全局安装了 create-react-app, 所以输出如下
> *lrwxr-xr-x    1 pengfei.cheng  wheel    83B  8  4  2022 create-react-app -> ../../../Users/pengfei.cheng/.config/yarn/global/node_modules/.bin/create-react-app*

!!! note 正常情况下全局安装的 `create-react-app` 是会在这个目录，如果你的 `creat-reat-app` 命令不能正确执行，`whereis` 可能无法定位到具体的路径，你可以确认`create-react-app`的安装位置后直接设置软链。

### 2.2 方案 1：使用 `ln -s` 配置软链至`/usr/local/bin`

因为一般 bin 目录下的可执行文件都是通过软链的方式绑定的。所以我们可以在目标的 bin 目录下新增一条软链。

因为使用 yarn 安装的 `create-react-app` 在`~/.config/yarn/global/node_modules/`下，所以我们直接配置一个软链至 `/usr/local/bin`。

```bash
ln -s ~/.config/yarn/global/node_modules/.bin/create-react-app /usr/local/bin
```
执行上述命令即可。

!!! note 这种方式软链的配置不太直观，需要你对执行文件的目录有足够的了解。

### 2.3 方案 2：配置 `Alias` 别名

因为使用了 zsh 替代了默认的 bash, 所以我们可以直接修改 `~/.zshrc` 配置文件

```bash
echo 'alias create-react-app="~/.config/yarn/global/node_modules/.bin/create-react-app"' >> ~/.zshrc
```

执行上述命令会在 `~/.zshrc` 中增加一个 Alias 别名，需要手动 `source ~/.zshrc` 使配置生效。

修改完成后可以使用下面的命令查看别名是否配置成功。
```bash
alias | grep creat-react-app
```

!!! note 这种方式需要修改配置文件，但是最简单，比较推荐。

### 2.4 方案 3：直接修改 `$PATH`

```bash
echo 'export PATH="$PATH:$HOME/.config/yarn/global/node_modules/.bin"' >> ~/.zshrc
```
执行上述命令，会将 `creat-react-app` 的安装 bin 目录添加到 PATH 环境变量中。

因为修改了`~/.zshrc` 文件，所以需要手动 `source ~/.zshrc` 使配置生效。

!!! note 这种方式可能会污染 PATH 环境变量，造成非常多的冗余，一些个人化的配置不建议采用这种方式配置。

## 3. ZSH & BASH

当然，除了常见的`zsh command not found`错误，我们也可能遇到一些 `bash command not found` 的错误。

这是因为，假如我们没有使用 zsh ，默认情况下，系统采用的是 bash , bash 的配置文件一般为 `vi ~/.bash_profile`。遇到这类问题，参考 zsh 的修改方式修改即可。

### 3.1 配置文件优先级

假如我们同时使用了 `bash / zsh` ，我们就同时拥有了 `.zshrc` 和 `.bash_profile` 的配置文件, 除此之外还有一些系统默认的配置文件，如 `/etc/profile`, `/etc/bashrc` 。

这些文件的优先级顺序遵循一个原则，个性化的配置会覆盖上一级的配置。
`custom shell (.zshrc) ->  default shell (.bashrc) -> system(/etc/profile)`

### 3.2 切换默认 shell 

因为研发效率问题，不建议使用系统内置的 `bash`, 建议使用 `zsh`, 并设置默认终端为 `zsh`
```
chsh -s /bin/zsh
```


## 4. npm vs yarn 的全局安装路径

### 4.1 查询全局安装的 Node 路径

**使用 whereis**
```bash
whereis node
```
输出如下：
> *node: /Users/pengfei.cheng/.nvm/versions/node/v14.19.0/bin/node /Users/pengfei.cheng/.nvm/versions/node/v14.19.0/share/man/man1/node.1*

**使用 npm config 查看**
```bash
npm config get prefix
```
输出如下：
> */Users/pengfei.cheng/.nvm/versions/node/v14.19.0*

我们也可以使用类似 `npm root -g` 全看全局安装的 node_modules 资源路径，一般会在同级目录的 `lib/` 目录下。

### 4.2 全局包资源目录

使用 npm 安装（未使用 nvm）
```bash
npm root -g

# /usr/local/lib/node_modules
```

使用 npm 安装（使用 nvm）
```bash
npm root -g

# ~/.nvm/versions/node/v14.19.0/lib/node_modules
```

使用 yarn 安装
```bash
yarn global dir

# ~/.config/yarn/global
```

## 5. 参考

- [Filesystem_Hierarchy_Standard](https://www.wikiwand.com/en/Filesystem_Hierarchy_Standard)
- [/etc/profile、/etc/bashrc、~/.bash_profile、~/.bashrc 文件的作用](https://www.cnblogs.com/cwp-bg/p/8257843.html)