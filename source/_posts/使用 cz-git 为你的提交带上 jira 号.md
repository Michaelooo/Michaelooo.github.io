---
title: 使用 cz-git 为你的提交带上 jira 号
date: 2022-07-21 15:58:53
tags:
  - git
---

## 问题

1. 在开发规范里其实是有要求带上 jira 号的，但是目前大部分 commit 都没有带，因为输入起来比较麻烦
2. 如果可以在 commit 信息中带上一些业务模块的信息，也比较方便排查问题

## cz-git 介绍

一句话，可以帮助你规范 commit 提交。

详细介绍可以参考 [cz-git 介绍](https://cz-git.qbb.sh/zh/guide/introduction.html#%E4%BB%8B%E7%BB%8D)

## 如何使用

###  全局安装（推荐）

推荐全局安装此命令，好处是：

* 不侵入业务仓库，业务仓库更干净
* 不会影响其他人的提交，如果后续作为强制 commit 规范，可在工程内部安装

#### step 1: 下载全局依赖

```sh
npm install -g cz-git commitizen
```

#### step 2: 全局配置适配器类型

```
echo '{ "path": "cz-git", "useEmoji": true }' > ~/.czrc
```

#### step 3: 自定义配置，支持 jira 号
step 3: 添加自定义配置(可选，使用默认配置)

```js
// .commitlintrc.js
/** @type {import('cz-git').UserConfig} */
const { execSync } = require('child_process');

// @tip: git branch name = feature/SPXFM-12345   =>    auto get feature = #SPXFM-12345
const branch = execSync('git rev-parse --abbrev-ref HEAD').toString().trim();
let feature = '';
if (branch.startsWith('feature/SPXFM')) {
  const content = branch.split('/')[1];
  const splitArr = content.split('-');
  feature = ` ${splitArr[0]}-${splitArr[1]} `
}

module.exports = {
  rules: {
    // @see: https://commitlint.js.org/#/reference-rules
  },
  prompt: {
    messages: {
      type: "Select the type of change that you're committing:",
      scope: "Denote the SCOPE of this change (optional):",
      customScope: "Denote the SCOPE of this change:",
      subject: "Write a SHORT, IMPERATIVE tense description of the change:\n",
      body: 'Provide a LONGER description of the change (optional). Use "|" to break new line:\n',
      breaking: 'List any BREAKING CHANGES (optional). Use "|" to break new line:\n',
      footerPrefixsSelect: "Select the ISSUES type of changeList by this change (optional):",
      customFooterPrefixs: "Input ISSUES Prefix:",
      footer: "List any ISSUES by this change. E.g.: #31, #34, #I972S:\n",
      confirmCommit: "Are you sure you want to proceed with the commit above ?"
    },
    types: [
      { value: "feat", name: "feat:     ✨  A new feature", emoji: ":sparkles:" },
      { value: "fix", name: "fix:      🐛  A bug fix", emoji: ":bug:" },
      { value: "docs", name: "docs:     📝  Documentation only changes", emoji: ":memo:" },
      { value: "style", name: "style:    💄  Changes that do not affect the meaning of the code", emoji: ":lipstick:" },
      { value: "refactor", name: "refactor: ♻️   A code change that neither fixes a bug nor adds a feature", emoji: ":recycle:" },
      { value: "perf", name: "perf:     ⚡️  A code change that improves performance", emoji: ":zap:" },
      { value: "test", name: "test:     ✅  Adding missing tests or correcting existing tests", emoji: ":white_check_mark:" },
      { value: "build", name: "build:    🏗️   Changes that affect the build system or external dependencies", emoji: ":building_construction:" },
      { value: "ci", name: "ci:       💚  Changes to our CI configuration files and scripts", emoji: ":green_heart:" },
      { value: "chore", name: "chore:    🔨  Other changes that don't modify src or test files", emoji: ":hammer:" },
      { value: "revert", name: "revert:   ⏪️  Reverts a previous commit", emoji: ":rewind:" }
    ],
    useEmoji: true,
    scopes: [],
    allowCustomScopes: true,
    allowEmptyScopes: true,
    customScopesAlign: "bottom",
    customScopesAlias: "custom",
    emptyScopesAlias: "empty",
    upperCaseSubject: false,
    allowBreakingChanges: ['feat', 'fix'],
    breaklineNumber: 100,
    breaklineChar: "|",
    skipQuestions: ['scope', 'body', 'breaking', 'footer', 'footerPrefix'],
    issuePrefixs: [{ value: "closed", name: "closed:   ISSUES has been processed" }],
    customIssuePrefixsAlign: "top",
    emptyIssuePrefixsAlias: "skip",
    customIssuePrefixsAlias: "custom",
    confirmColorize: true,
    maxHeaderLength: Infinity,
    maxSubjectLength: Infinity,
    minSubjectLength: 0,
    scopeOverrides: undefined,
    defaultBody: "",
    defaultIssues: "",
    defaultScope: "",
    defaultSubject: feature
  }
};
```

#### step4 : 使用 cz 替代 git commit

只需要将过去的 `git commit -m 'aaaa'` 替换为 `cz`即可， 其他人使用正常的 `git commit` 也不会受影响。


### 项目内安装

如果需要在某个项目内安装，只需要把  `.commitlintrc.js` 复制到执行项目根目录即可。 详细的步骤可以参考 [快速开始](https://cz-git.qbb.sh/zh/guide/#%E5%85%A8%E5%B1%80%E4%BD%BF%E7%94%A8) ，这里不再单独介绍。

## 已知问题

一个已知的问题：提交历史不能被 `zsh-autosuggestions` 工具检测到，所以存在下面的问题：
在 commit 时遇到 eslint error, 无法提示到上一次的 commit 信息。