---
title: 部署 Hexo 
date: 2020-06-04 16:00:02
categories:
- 前端
tags:
- hexo
---
使用 hexo 搭建个人博客，并且可以根据喜好切换主题。

## hexo 基础操作

### 新建仓库
创建 \<user-name\>.github.io 仓库，其中 \<user-name\> 只能是 github 昵称，我们需要以此免费获取博客域名。 
比如我的博客：sugelalin.github.io

### 全局安装hexo

```bash
npm install -g hexo
```

### 初始化项目

```bash
hexo init
```

// 本地运行
```bash
hexo s
```

### 部署到github
根目录下修改 _congif.yml 的 deploy 配置

```bash
deploy:
  type: git
  repo: https://github.com/sugelalin/sugelalin.github.io.git
  branch: master
```

安装部署工具：
```bash
npm install hexo-deployer-git --save
```

清理缓存文件
```bash
hexo clean
```

项目部署
```bash
hexo deploy
```

部署成功后可通过项目名称： https://sugelalin.github.io 访问

## hexo 高阶操作

### 项目布局

[layout] 为布局，可选项为 `post`、`page`、`draft`，这将决定文章所在文件路径
\<title\> 为文章标题 

格式：
```bash
hexo new [layout] \<title\>
```

示例：
新建文章
```bash
hexo new post article-name
```

### 添加标签
生成分类文件 source/tags/index.md
```bash
hexo new page tags
```

添加type:
```markdown
- - -
title: 文章标签
date: 2020-06-04 16:00:00
type: "tags"
- - -

```

### 添加分类
生成分类文件 source/categories/index.md
```bash
hexo new page categories
```

添加type:
```markdown
- - -
title: 文章分类
date: 2020-06-04 16:00:00
type: "categories"
- - -

```

修改文章生成的模板文件
修改 scaffolds/post.md ， 添加 tags 和 categories
```markdown
- - -
title: {{ title }}
date: {{ date }}
tags: 
categories: 
- - -
```


在文章中使用标签和分类
```markdown
- - -
title: 文章标题
date: 2020-06-04 16:00:00
categories: 
- 前端
tags: 
- web前端
- hexo
- - -
```

### 添加归档
生成分类文件 source/archives/index.md
```bash
hexo new page archives
```

添加type:
```markdown
- - -
title: 文章分类
date: 2020-06-04 16:00:00
type: "archives"
- - -

```

修改主题 _config.yml 配置
```markdown
menu:
  主页: /
  归档: /archives/
```

### 修改主题
下载主题到themes文件夹下
```bash
git clone https://github.com/litten/hexo-theme-yilia.git themes/yilia
```

修改项目 _config.yml 配置
```bash
theme: yilia
```

### 部署优化
每次都要执行 hexo clean 和 hexo deploy，合并为新脚本

在 package.json 的script 中添加如下：
```bash
    "dev": "hexo s",
    "build": "hexo clean & hexo deploy"
```

部署命令
```bash
npm run build
```

### 接入评论系统 -valine
[valine地址](https://valine.js.org/quickstart.html)


### 其他
a. 解决主题BUG：yilia主题分类跳转多了个 / 
去掉 themes/yilia/layout/_partial/post/category.ejs 文件中，tag.path 前面的 /

b. hexo 排序是依据文章的 date 降序排列；若date 值相同，就按文章文件的创建时间升序排列


本文参考：
https://www.jianshu.com/p/390f202c5b0e
https://www.jianshu.com/p/e17711e44e00
