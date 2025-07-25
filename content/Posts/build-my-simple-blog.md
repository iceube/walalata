+++
date = '2025-07-23T16:48:34+08:00'
draft = false
title = 'Build My Simple Blog'
+++

## Hugo 介绍
Hugo 是一个用 Go 语言编写的静态网站生成器，可以在几秒钟内渲染出完整的网站。
Hugo有三种版本：
	标准版（standard）、扩展版（extended）和扩展/部署版（extended/deploy）。标准版提供核心功能，而扩展版和扩展/部署版提供高级功能。
	除非特定部署需求需要扩展/部署版，官方推荐使用扩展版。
## 开始-本地调试
- 下载 Git 。
- 下载 Hugo ,地址：[Hugo](https://github.com/gohugoio/hugo/releases) ，解压缩到目的目录，编辑环境变量。
- 初始化站点
~~~ cmd
hugo new site myblog  # 生成站点目录，myblog 可以是任何站点名称
cd myblog # 进入创建的站点
~~~
- 下载主题，hugo 提供很多主题[Hugo themes](https://themes.gohugo.io/)，放到themes文件夹下 
	需要在`hugo.toml`中指定主题名（同主题文件夹名称一致），例如：
``` toml
baseURL = 'https://example.org/'
languageCode = 'en-us'
title = 'My New Hugo Site'
theme = 'PaperMod'
```

- 生成第一篇文档
	记得修改 draft = false，默认不渲染草稿（draft = true）.
  ~~~ cmd
	myblog>hugo new content/posts/first-post.md  # 生成 Markdown 文件
	~~~
	发现换行存在问题，查询相关资料，发现 Hugo 中的换行需要两个回车，但是可以通过修改`hugo.toml`保持一致。添加如下代码段：
	~~~ toml	
	[markup.goldmark.renderer]
	  hardWraps = true
	~~~

## 部署-Netlify
1. 将项目托管到 Github 上
	创建一个公开的仓库用来存放项目。
	~~~ git
	git init
	git add.
	git commit -m "first commit"
	git branch -M main
	git remote add origin https://github.com/iceube/walalata.git
	git push -u origin main
	~~~
	如果有**网络问题**，可以看看自己科学上网的代理端口是什么，修改成对应的
	~~~ git
	# 临时使用代理（示例）
	export https_proxy=http://127.0.0.1:7897 http_proxy=http://127.0.0.1:7897
	git push
	~~~
2. 在 Netlify 上部署 
	参考：[在 Netlify 上托管 | Hugo官方文档](https://hugo.opendocs.io/hosting-and-deployment/hosting-on-netlify/)
	可是使用 GitHub 账号登陆。
	- 一些注意点：
		1. Build command: hugo
		2. Publish directory: public
		3. 添加环境变量：
			key: HUGO_VERSION
			value: 版本号（例如：0.148.1）
		4. 修改 `hugo.toml`中的`baseURL`为分配给你的站点
## 后续问题
1. md 不能显示内嵌的 html
	修改`hugo.toml`，补充`unsafe = true`，但是会和 `hardWraps` (解决两个回车换行)冲突，看自己的需求进行取舍：
	~~~ toml
	[markup.goldmark.renderer]
	  hardWraps = false
	  unsafe = true