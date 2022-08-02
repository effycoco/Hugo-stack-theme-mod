---
title: "Vue Deployment With Github Pages"
description: "将本地vue项目通过github pages部署到网络上的步骤"
date: 2022-08-02T03:13:43+08:00
image:
math:
license:
hidden: false
comments: true
draft: false
tags:
  - vue
---

参考[Youtube 视频](https://www.youtube.com/watch?v=i_XbW-FsLKk&ab_channel=PragmaticReviews)：

1. 新建一个 github 公开仓库。项目将部署到该库的 gh-pages 分支上。

   有两种选择，公开源代码，先将项目源代码推送至 main 分支上，最后该库将有两个分支—— main 存储源代码，gh-pages 存储经`npm run build`生成的浏览器能理解的代码；不公开源代码，建库后无需操作，部署完成后该库只有一个 gh-pages 分支，可另建一个私有库存储源代码。

   若未使用 vue-router，仓库名称将是部署后的网址。若项目使用了 vue-router，部署后的网址只是把本地的`http://localhost:8080/`替换成`https://effycoco.github.io/`，要求在定义 router 时将`/`重定向至某个具体路径，比如`/coaches`，这样多个项目之间地址不会打架。

2. 配置文件。在项目根目录下新建两个文件`vue.config.js` and `deploy.sh`。`vue.config.js`内容如下：

   ```js
   module.exports = {
     publicPath: "/<repo-name>/",
   };
   ```

   `deploy.sh`内容如下：

   ```sh
   #!/usr/bin/env sh

   # 当发生错误时中止脚本
   set -e



   # 构建
   npm run build



   # cd 到构建输出的目录下
   cd dist


   git init

   git add -A

   git commit -m 'deploy'



   # 部署到 https://<USERNAME>.github.io/<REPO>

   git push -f git@github.com:effycoco/find-a-coach.git master:gh-pages

   # master:pages代表将本地master分支push至远程gh-pages分支

   # 返回上一级文件夹
   cd -

   ```

   参考[官方文档](https://cli.vuejs.org/guide/deployment.html#github-pages)

   sh 文件中的内容就是运行 `build` 将手写代码转换成浏览器能理解的代码，转换后的代码放在生成的 `dist `文件夹中，`cd` 到该文件夹，将该文件夹内容上传至远程库的` gh-pages` 分支。

3. 修改 package.json 文件。在 `scripts` 中增加一条`"deploy": "sh deploy.sh"`。
4. 执行文件。打开终端，应在项目根目录下。运行`chmod +x deploy.sh`，增加 deploy.sh 的可执行权限。然后运行`npm run deploy`。
5. 完成。在仓库右侧边栏的 Environment 中可找到部署后的地址。[github 库示例](https://github.com/effycoco/resources-memo-vue)
