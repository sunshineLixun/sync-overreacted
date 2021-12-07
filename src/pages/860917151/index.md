---
title: '记录日常使用 yarn/npm 报错 '
date: '2021-04-19'
spoiler: ''
---

  使用 yarn 安装时，报错node_modules\node sass: Command failed.

解决办法：
```shell
npm install -g mirror-config-china --registry=http://registry.npm.taobao.org
npm install node-sass
yarn install
```
  