---
title: hexo博客搭建
thumbnail: https://www.msbgn.cn/msbgn/hexo001.png
categories: hexo
tags: [科技,hexo]
---


![hello hexo](https://www.msbgn.cn/msbgn/hexo002.png)

<!-- more -->
## 一、什么是 Hexo？


Hexo 是一个快速、简洁且高效的博客框架。`Hexo` 使用 `Markdown`（或其他渲染引擎）解析文章，在几秒内，即可利用靓丽的主题生成静态网页。

###  安装

 安装 `Hexo` 只需几分钟时间，若您在安装过程中遇到问题或无法找到解决方式，请提交问题，我会尽力解决您的问题。

### 安装前提
安装 Hexo 相当简单。然而在安装前，您必须检查电脑中是否已安装下列应用程序：

* [Node.js](https://nodejs.org/en/)
* [Git](https://git-scm.com/)

如果您的电脑中已经安装上述必备程序，那么恭喜您！接下来只需要使用 npm 即可完成 Hexo 的安装。
```
$ npm install -g hexo-cli
```
如果您的电脑中尚未安装所需要的程序，请根据以下安装指示完成安装。

### Mac 用户
您在编译时可能会遇到问题，请先到 `App Store` 安装 `Xcode`，`Xcode` 完成后，启动并进入 `Preferences` -> `Download` -> `Command Line Tools` ->` Install` 安装命令行工具。

## 安装 Git

* Windows：下载并安装 git.
* Mac：使用 Homebrew, MacPorts 或下载 安装程序 安装。
* Linux (Ubuntu, Debian)：`sudo apt-get install git-core`
* Linux (Fedora, Red Hat, CentOS)：`sudo yum install git-core`
##  安装 Node.js

安装 Node.js 的最佳方式是使用  [nvm](https://github.com/creationix/nvm)。

cURL:
```
$ curl https://raw.github.com/creationix/nvm/master/install.sh | sh
```

Wget:
```
$ wget -qO- https://raw.github.com/creationix/nvm/master/install.sh | sh
```

安装完成后，重启终端并执行下列命令即可安装 Node.js。
```
$ nvm install stable
```

或者您也可以下载 [安装程序](https://nodejs.org/en/)来安装。

##  安装 Hexo

所有必备的应用程序安装完成后，即可使用 `npm` 安装 `Hexo`。

```
$ npm install -g hexo-cli
```

安装 Hexo 完成后，请执行下列命令，Hexo 将会在指定文件夹中新建所需要的文件。
```
$ hexo init <folder>
$ cd <folder>
$ npm install
[root@vmware-130 ~]# rpm -qa|grep pcre
pcre-7.8-6.el6.x86_64
pcre-devel-7.8-6.el6.x86_64
```

新建完成后，指定文件夹的目录如下：

```
.
├── _config.yml
├── package.json
├── scaffolds
├── source
|   ├── _drafts
|   └── _posts
└── themes
```

###  _config.yml

网站的 配置 信息，您可以在此配置大部分的参数。

###  package.json 

应用程序的信息。EJS, Stylus 和 Markdown renderer 已默认安装，您可以自由移除。

###  scaffolds

[模板](https://hexo.io/zh-cn/docs/zh-cn/docs/writing.html) 文件夹。当您新建文章时，Hexo 会根据 scaffold 来建立文件。

###  source

资源文件夹是存放用户资源的地方。除` _posts` 文件夹之外，开头命名为 _ (下划线)的文件 / 文件夹和隐藏的文件将会被忽略。`Markdown` 和 `HTML` 文件会被解析并放到` public` 文件夹，而其他文件会被拷贝过去。

###  themes 
 
主题 文件夹。`Hexo` 会根据主题来生成静态页面。
您可以在 _config.yml 中修改大部份的配置。


###  网站


参数	描述
`title`网站标题
`subtitle`	网站副标题
`description`	网站描述
`author	`您的名字
`language`	网站使用的语言
`timezone`	网站时区。Hexo 默认使用您电脑的时区。时区列表。比如说：America/New_York, Japan, 和 UTC 。

###  网址


| 参数    |    描述	| 默认值|
| :-------- | --------:| :--: |
| `url`  | 网址	 |     |
| `root`	 | 网站根目录	|     |
| `permalink`	| 文章的 永久链接格式|  	:year/:month/:day/:title/   |
| `permalink_default`	| 永久链接中各部分的默认值|     |

网站存放在子目录

如果您的网站存放在子目录中，例如 https://yoursite.com/blog，则请将您的 url 设为 https://yoursite.com/blog 并把 root 设为 /blog/


| 参数	|    描述	| 默认值|
| :-------- | --------:| :--: |
| `source_dir	`  | 	资源文件夹，这个文件夹用来存放内容。|    source |
| `public_dir	`	 | 公共文件夹，这个文件夹用于存放生成的站点文件	|   public  |
| `archive_dir	`	| 归档文件夹|  	categories|
| `code_dir`	| Include code 文件夹|  downloads/code   |
| `i18n_dir`	| 国际化（i18n）文件夹|  lang   |
| `skip_render`	| 跳过指定文件的渲染，您可使用 glob 表达式来匹配路径。|    |

参数	描述	默认值
source_dir	资源文件夹，这个文件夹用来存放内容。	source
public_dir	公共文件夹，这个文件夹用于存放生成的站点文件。	public
tag_dir	标签文件夹	tags
archive_dir	归档文件夹	archives
category_dir	分类文件夹	categories
code_dir	Include code 文件夹	downloads/code
i18n_dir	国际化（i18n）文件夹	:lang
skip_render	跳过指定文件的渲染，您可使用 glob 表达式来匹配路径。	

##  init

```
$ hexo init [folder]

```

新建一个网站。如果没有设置 folder ，Hexo 默认在目前的文件夹建立网站。

##  new

```
$ hexo new [layout] <title>
```

新建一篇文章。如果没有设置 layout 的话，默认使用 _config.yml 中的 default_layout 参数代替。如果标题包含空格的话，请使用引号括起来。


##  generate

```
$ hexo generate
```

生成静态文件。

| 选项|    描述	|
| :-------- | --------:| :--: |
| `-d, --deploy`  | 文件生成后立即部署网站	 |     |
| `-w, --watch`	 | 监视文件变动|     |


##  publish
```
$ hexo publish [layout] <filename>
```

发表草稿


##  server

```
$ hexo server

```
启动服务器。默认情况下，访问网址为： http://localhost:4000/。


##  deploy

```
$ hexo deploy
```

部署网站。


##  render

```
$ hexo render <file1> [file2] ...
```
渲染文件。


##  migrate

```
$ hexo migrate <type>

```
从其他博客系统 迁移内容。

##  clean
```

$ hexo clean
```

##  list

```
$ hexo list <type>
```


##  主配置文件模板：


```
# Hexo Configuration
## Docs: https://hexo.io/docs/configuration.html
## Source: https://github.com/hexojs/hexo/

# Site
title: AlexZhou
subtitle: This is Aexl blog
description: msbgn blog
author: Rongfu Zhou
language: zh-CN
timezone: Asia/Shanghai

# URL
## If your site is put in a subdirectory, set url as 'http://yoursite.com/child' and root as '/child/'
url: /
root: /
permalink: :year/:month/:day/:title/
permalink_defaults:

# Directory
source_dir: source
public_dir: public
tag_dir: tags
archive_dir: archives
category_dir: categories
code_dir: downloads/code
i18n_dir: :lang
skip_render:

# Writing
new_post_name: :title.md # File name of new posts
default_layout: post
titlecase: false # Transform title into titlecase
external_link: true # Open external links in new tab
filename_case: 0
render_drafts: false
post_asset_folder: false
relative_link: false
future: true
highlight:
  enable: true
  line_number: true
  auto_detect: true
  tab_replace:

# Category & Tag
default_category: uncategorized
category_map:
tag_map:

# Date / Time format
## Hexo uses Moment.js to parse and display date
## You can customize the date format as defined in
## http://momentjs.com/docs/#/displaying/format/
date_format: YYYY-MM-DD
time_format: HH:mm:ss

# Pagination
## Set per_page to 0 to disable pagination
per_page: 3
pagination_dir: page

# Extensions
## Plugins: https://hexo.io/plugins/
## Themes: https://hexo.io/themes/
theme: icarus

#duoshuo_shortname: carl1990.duoshuo.com
#duoshuo: true
# Deployment
## Docs: https://hexo.io/docs/deployment.html
deploy:
  type: git
  repository: https://zhourongfu@github.com/zhourongfu/zhourongfu.github.io.git
  branch: master

```

主题模板： 我采用的是icarus主题，根据自己喜欢的主题配置，每个主题的配置方式有差异


```
# Menus
menu:
  首页: /
  python: /python
  归档: /archives
  关于: /about
# Customize
customize:
    logo:
        enabled: true
        width: 40
        height: 40
        url: images/logo.png
    profile:
        enabled: true # Whether to show profile bar
        avatar: css/images/avatar.png
        gravatar: # Gravatar email address, if you enable Gravatar, your avatar config will be overriden
        author: AlexZhou
        author_title: Time Flies(岁月如梭)
        location: shenzhen, China
        follow: https://github.com/zhourongfu/
    highlight: monokai
    sidebar:  right # sidebar position, options: left, right
    thumbnail: true # enable posts thumbnail, options: true, false
    favicon: /favicon.ico # path to favicon
    social_links:
        github: https://github.com/zhourongfu/
        weixin: http://mp.weixin.qq.com/s?__biz=MzI5NzMxMDQ1NA==&mid=2247483651&idx=1&sn=58ad9ee21c33243688628512624db18f&chksm=ecb6409adbc1c98c3561ce437435666ccd296f0d6a90999e3d2a9dc7d82cc7cbf435896be5a3#rd
        qq: http://wpa.qq.com/msgrd?v=3&uin=508095413&site=qq&menu=yes
    social_link_tooltip: true # enable the social link tooltip, options: true, false

# Widgets
widgets:
    - recent_posts
    - category
    - archive
    - tag
    - tagcloud
    - links
# Search
search:
    insight: true # you need to install `hexo-generator-json-content` before using Insight Search
    swiftype: # enter swiftype install key here
    baidu: true # you need to disable other search engines to use Baidu search, options: true, false

# Comment
comment:
    #disqus: hexo-theme-icarus # enter disqus shortname here
    duoshuo: carl1990  # enter duoshuo shortname here
    youyan: # enter youyan uid here

# Share
share:  bdshare  # options: jiathis, bdshare, addtoany, default
donate:
  enable: true
  text: 打赏我的人，据说都找到了女朋友~~
  wechat: http://www.msbgn.cn/msbgn/xpchatpay.png
  alipay: http://www.msbgn.cn/msbgn/wechatpay.png
  web: true
# Plugins
plugins:
    lightgallery: true # options: true, false
    justified-gallery: true # options: true, false
    google_analytics: # enter the tracking ID for your Google Analytics
    google_site_verification: # enter Google site verification code
    baidu_analytics: # enter Baidu Analytics hash key
# Miscellaneous
miscellaneous:
    open_graph: # see http://ogp.me
        fb_app_id:
        fb_admins:
        twitter_id:
        google_plus:
    links:
        百度: http://www.baidu.com
        同舟商城: http://www.10ytao.com
        铁匠运维: http://www.tiejiang.org
        51运维: http://www.51ou.com
        运维派: http://www.yunweipai.com
        Linux公社: http://www.linuxidc.cm

```
安装已经完成，下一节是修改主题相关操作、增加评论模块，打赏、分享、站内搜索。
