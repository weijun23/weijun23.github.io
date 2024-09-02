---
title: 使用Jekyll + GitHub Pages搭建个人博客
date: 2023-09-03 10:56:51 +0800
categories: [Blogging]
tags: [jekyll, github pages]
---
本文将介绍如何使用Jekyll搭建个人博客，并部署在GitHub Pages上。

## 1.简介
Jekyll是一个强大的静态网站生成器，可以将Markdown、HTML、Liquid模板等文件转换为静态网站。Jekyll支持模板引擎、主题、插件、集成GitHub Pages等特性，可以帮助用户快速搭建和发布静态网站。

官方网站：<https://jekyllrb.com/>

## 2.安装
Jekyll支持大多数操作系统，依赖如下：
* [Ruby](https://www.ruby-lang.org/en/downloads/) 2.5.0+
* [RubyGems](https://rubygems.org/pages/download)
* [GCC](https://gcc.gnu.org/install/)和[Make](https://www.gnu.org/software/make/)

下面介绍在Windows系统上的安装步骤，其他操作系统见官方文档安装指引：<https://jekyllrb.com/docs/installation/>。

### 2.1 安装Ruby
在Windows上安装Ruby最简单的方式是RubyInstaller。

下载地址：<https://rubyinstaller.org/downloads/>，选择 **Ruby+Devkit** 版本，使用默认选项安装即可。

在安装向导最后一步勾选“运行ridk install”：

![ridk install](/assets/images/creating-personal-blog-site/ridk-install.png)

在弹出的命令行窗口中选择 "MSYS2 and MINGW development tool chain"：

![MSYS2 installation choice](/assets/images/creating-personal-blog-site/msys2-installation-choice.png)

检查是否安装成功：

```shell
ruby -v
gem -v
```

### 2.2 安装Jekyll
打开一个新的CMD窗口，执行以下命令安装Jekyll和Bundler：

```shell
gem install jekyll bundler
```

检查是否安装成功：

```shell
jekyll -v
```

## 3.Jekyll基础
### 3.1 Jekyll教程
在正式开始搭建个人博客之前，建议先按照Jekyll官方教程[Step by Step Tutorial](https://jekyllrb.com/docs/step-by-step/01-setup/)搭建一个Demo网站，了解**Liquid模板**（变量、标签、过滤器）、**前页**(front matter)、**布局**(layout)等基本概念，并学会使用`jekyll`命令。

Demo网站：<https://zzy979.github.io/jekyll-tutorial/>

![Demo site](/assets/images/creating-personal-blog-site/demo-site.png){: .shadow }

### 3.2 Jekyll目录结构
Jekyll项目的典型目录结构如下：

```
mysite/
  _data/
    foo.yml
  _includes/
    foo.html
  _layouts/
    default.html
    post.html
  _posts/
    2018-08-20-bananas.md
    2018-08-21-apples.md
  _sass/
    main.scss
  _site/
  assets/
    css/
    images/
    js/
  _config.yml
  index.html
  Gemfile
```

作为博客作者，主要关注以下文件/目录即可（其他目录用于存放主题样式文件）：

| 文件/目录 | 描述 |
| --- | --- |
| _config.yml | 配置文件 |
| _posts | 文章内容，文件命名格式为`YYYY-MM-DD-TITLE.EXTENSION` |
| _site | Jekyll生成的网站文件 |
| assets/images | 文章中的图片文件 |
| index.html | 网站主页 |

详见官方文档[Directory Structure](https://jekyllrb.com/docs/structure/)。

### 3.3 主题
**主题**(theme)提供了网站页面的布局和样式，详见官方文档[Themes](https://jekyllrb.com/docs/themes/)。

可以在 <https://jekyllthemes.org/> 选择自己喜欢的主题并在线预览。我选择的主题是[Chirpy](https://jekyllthemes.org/themes/jekyll-theme-chirpy/)，该主题提供了**分类**(category)、**标签**(tag)、目录、语法高亮、数学公式、搜索文章、评论系统等特性，能够满足博客网站的绝大部分需求。
* 在线Demo & 使用教程：<https://chirpy.cotes.page/>
* 项目主页：<https://github.com/cotes2020/jekyll-theme-chirpy/>

## 4.搭建个人博客
下面正式开始搭建个人博客网站。参考：[Chirpy - Getting Started](https://chirpy.cotes.page/posts/getting-started/)。

### 4.1 创建网站
打开[chirpy-starter](https://github.com/cotes2020/chirpy-starter)仓库，点击按钮 "Use this template" → "Create a new repository"。

![create-repository-step1](/assets/images/creating-personal-blog-site/create-repository-step1.png)

将新仓库命名为`<username>.github.io`，其中`<username>`是你的GitHub用户名，如果包含大写字母需要转换为小写。

![create-repository-step2](/assets/images/creating-personal-blog-site/create-repository-step2.png)

注：如果不需要自定义主题样式，则推荐使用这种方式，因为容易升级，并且能隔离无关文件，使你能够专注于文章内容写作。

### 4.2 安装依赖
使用`git clone`将新创建的仓库克隆到本地，并在项目根目录下执行

```shell
bundle
```

### 4.3 配置
根据需要更新_config.yml中的变量，例如`url`、`avatar`、`timezone`、`lang`等。

### 4.4 启动本地服务器
要在本地预览网站内容，执行

```shell
bundle exec jekyll serve
```

在浏览器访问 <http://127.0.0.1:4000/>。

### 4.5 部署
[GitHub Pages](https://pages.github.com/)是一个通过GitHub托管和发布网页的服务，官方文档：<https://docs.github.com/en/pages>。本文使用GitHub Pages部署个人博客网站。

每个GitHub用户可以创建一个用户级网站，仓库名为`<username>.github.io`，发布地址为 `https://<username>.github.io`。GitHub Pages支持自定义域名，参考文档[About custom domains and GitHub Pages](https://docs.github.com/en/pages/configuring-a-custom-domain-for-your-github-pages-site/about-custom-domains-and-github-pages)。

在部署之前，检查_config.yml中的`url`是否正确配置为上述发布地址（或者自定义域名）。

> 注意：一般不需要配置`baseurl`。如果配置了，则文章中必须使用`relative_url`过滤器生成正确的URL，否则会导致404错误。参考[Jekyll’s site.url and baseurl](https://mademistakes.com/mastering-jekyll/site-url-baseurl/)。
{: .prompt-warning }

之后在GitHub上打开仓库设置，点击左侧导航栏 "Pages"，在 "Build and deployment" - "Source" 下拉列表选择 "GitHub Actions"。

![github-pages-deployment-source](/assets/images/creating-personal-blog-site/github-pages-deployment-source.png)

提交本地修改并推送至远程仓库，将会触发Actions工作流。在仓库的Actions标签页将会看到 "Build and Deploy" 工作流正在运行。构建成功后，即可通过配置的URL访问自己的博客网站。

<https://zzy979.github.io>

![personal-blog-site](/assets/images/creating-personal-blog-site/personal-blog-site.png)

### 4.6 评论系统
Jekyll生成的博客网站是静态的，没有后端和数据库，因此本身无法实现评论功能。然而，可以使用[disqus](https://disqus.com/)、[utterances](https://utteranc.es/)和[giscus](https://giscus.app/)等评论系统来实现评论功能。

本文使用giscus，它是利用[GitHub Discussions](https://docs.github.com/en/discussions)实现的评论系统，并且是开源、免费的。开启评论系统的步骤如下。

（1）安装[giscus app](https://github.com/apps/giscus)。

（2）在仓库设置页面 "Features" 一节中勾选 "Discussions"，开启仓库的GitHub Discussions功能。

![enable-github-discussions](/assets/images/creating-personal-blog-site/enable-github-discussions.png)

（3）在仓库的Discussions标签页，点击 "Categories" 旁边的编辑按钮，自定义用于博客评论的类别名称（例如 "Comments"）。

![edit-discussions-categories](/assets/images/creating-personal-blog-site/edit-discussions-categories.png)

（4）打开 <https://giscus.app/>，在页面上填写以下配置：
* Repository: `<username>/<username>.github.io`
* Page - Discussions Mapping：保持默认值 "Discussion title contains page pathname" 即可（URL为`https://<username>.github.io/posts/<title>`的文章将映射到标题为`/posts/<title>`的Discussion，即使用URL的pathname部分作为Discussion标题）
* Discussion Category：选择上一步创建的类别名称（例如 "Comments"）

之后找到 "Enable giscus" 一节，将自动生成的配置填写到_config.yml中`comments.giscus`的对应选项。

![giscus-config](/assets/images/creating-personal-blog-site/giscus-config.png)

```yaml
comments:
  active: giscus
  giscus:
    repo: ZZy979/zzy979.github.io
    repo_id: R_kgDOKOkhRA
    category: Comments
    category_id: DIC_kwDOKOkhRM4CZCpN
    mapping: pathname
```

（5）重启Jekyll服务器，在文章底部将会看到评论区，使用GitHub账号登录即可发表评论。

![comment-system-enabled](/assets/images/creating-personal-blog-site/comment-system-enabled.png)

### 4.7 写文章
至此，博客网站已经搭建完成，可以开始文章写作了。

要写一篇新的文章，在_posts目录下创建一个文件，命名格式为`YYYY-MM-DD-TITLE.md`（例如`2023-09-03-hello-world.md`），`TITLE`部分将作为文章的URL。

详细配置和语法参考Chirpy文档：
* [Writing a New Post](https://chirpy.cotes.page/posts/write-a-new-post/)
* [Text and Typography](https://chirpy.cotes.page/posts/text-and-typography/)

#### Front Matter
你需要在文章顶部填写Front Matter信息：

```yaml
---
title: TITLE
date: YYYY-MM-DD HH:MM:SS +/-TTTT
categories: [TOP_CATEGORIE, SUB_CATEGORIE]
tags: [TAG]
---
```

其中，`title`将展示为文章标题（不必与文件名的`TITLE`部分相同），`date`将展示为文章创建时间（时区写`+0800`）。

之后是正文内容。

#### 分类和标签
`categories`是文章的分类，支持至多两级分类。`tags`是文章的标签，可以有任意多个。例如：

```yaml
---
categories: [Animal, Insect]
tags: [bee]
---
```

#### 目录
默认情况下，目录将会自动生成并展示在文章右侧。如果想全局关闭，则将_config.yml中的`toc`变量设置为`false`。如果想对一篇特定的文章关闭，则在Front Matter中添加：

```yaml
---
toc: false
---
```

> 注意：Chirpy生成的目录只显示二级标题(##)和三级标题(###)，一级标题(#)不会显示在目录中。参考[issue #491](https://github.com/cotes2020/jekyll-theme-chirpy/issues/491#issuecomment-1015659620)。
{: .prompt-warning }
