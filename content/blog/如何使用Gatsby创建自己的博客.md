---
title: "如何使用Gatsby创建自己的博客"
date: 2022-09-28T11:16:00+0800
categories: javascript
---

首先使用npm安装gatsby，使用gatsby –version命令可以查看是否安装

```shell
npm install -g gatsby-cli
```

使用new命令后面跟着目录名，即可创建一个gatsby项目

```shell
gatsby new website
```

使用develop参数即可启用gatsby服务
```shell
gatsby develop
```

进入到8000端口后即可访问默认的页面，不过本文章是创建个人博客

![](/assets/Snipaste_2022-09-28_09-38-35.png)

在创建项目时后面跟着一个github地址，这个是gatsby官方的一个博客模板

```shell
gatsby new blog  https://github.com/gatsbyjs/gatsby-starter-blog   
```

![](/assets/Snipaste_2022-09-28_09-46-26.png)

这个是项目的目录结构

```
C:.
├─content
│  └─blog
│      ├─hello-world
│      ├─my-second-post
│      └─new-beginnings
├─src
│  ├─components
│  ├─images
│  ├─pages
│  └─templates
└─static
```

content下主要是markdown文件，但如果要被识别为blog还需要在blog里创建markdown文件

主要看blog里都有哪些文件，每个文件夹下都有一个index.md文件，这里会根据文件夹的名称来决定地址名是什么

```
C:.
├─hello-world
│      index.md
│      salty_egg.jpg
│
├─my-second-post
│      index.md
│
└─new-beginnings
        index.md
```

这里可以直接在blog中创建markdown文件，地址名就是markdown文件名

每个markdown文件的前几行都会有用3个横杠抱起来的内容，这个就是Front-matter格式，博客的信息都会从这里获取，title是博客的标题，date是发布时间，description是描述，其中时间必须遵守正确的格式

```
---
title: "如何使用Gatsby创建自己的博客"
date: 2022-09-28T11:16:00+0800
categories: javascript
---
```

现在来介绍如何使用GitHub的pages服务，首先需要在项目下安装gh-pages

```shell
npm install gh-pages --save-dev
```

在项目的gatsby的config文件中添加pathPrefix，这里选择没前缀

```javascript
module.exports = {
  pathPrefix: "/"
}
```

在scripts中添加deploy，意思就是让gatsby生成页面，随后用gh-pages将public的目录中所有文件推送到项目的gh-pages分支中，使用npm run deploy即可部署到GitHub中

```json
"scripts": {
    "deploy": "gatsby build --prefix-paths && gh-pages -d public"
}
```

进入到项目的setting页面，选择左侧的pages，将分支选为gh-pages这个分支

如果需要没有自己的域名就把仓库设置为这个格式，注意的是GitHub的用户名而不是昵称，如果有自己域名那就让域名解析CNAME到这个地址中，并且在项目的static目录下创建CNAME文件，填入你的域名
