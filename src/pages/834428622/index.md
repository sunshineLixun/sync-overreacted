---
title: 'npm link的使用'
date: '2021-03-18'
spoiler: ''
---

  最近在做一个小程序自动化打包的事情。了解了下npm发包的过程，其实很简单。

1. 首先要注册npm账户。

2. 终端执行：
`npm login` 输入账户名、密码、邮箱就行。
这里要切换到npm自己源：`npm config set registry https://registry.npmjs.org/ `。
我们大多数都在用淘宝的镜像：`npm config set registry https://registry.npm.taobao.org/`

3. 然后cd 到将要发布到npm上的项目路径下。执行`npm publish`。

在调试/开发[taro-deploy-email](https://github.com/sunshineLixun/taro-deploy)过程中，一开始方法确实太笨了。每次改动都要去发布到npm中，然后版本号version一直在累加。整个过程也很傻瓜式。后来了解到了npm link才大悟。


首先进入到我们的npm module中，也就是要发布的npm工程下，执行`npm link`。

```shell
cd npm-link-module
npm link
```

然后会在全局路径中链接上我们的开发的npm库，可以这么理解吧：相当于一个本地全局安装的库。
Windows平台下的路径如下：

![1616047104(1)](https://user-images.githubusercontent.com/15700015/111580130-0cb5ab80-87f2-11eb-9bcb-5f8ed2d7853d.png)


然后在要接入的项目路径下执行

```shell
npm link 我们的npm包名称
比如： npm link taro-deploy-email
```

在接入项目的node_modules中会存在一个临时的npm包。 _ydk-taro-deploy是根据公司业务修改的小程序打包npm库。这里做一个例子_

![1616047312(1)](https://user-images.githubusercontent.com/15700015/111580528-a1200e00-87f2-11eb-8fc4-49f66bcefcda.png)

可以看到右边有一个箭头，表示临时安装的npm包。

自此，本地项目与本地的npm库相链接，开发调试再也不用经过发布到npm新版本来解决了。


如何断开链接呢？
那就在要接入的项目路径下执行`npm unlink 你的npm包名称` 就行。自动会与之前创建的本地npm库断开链接。








  