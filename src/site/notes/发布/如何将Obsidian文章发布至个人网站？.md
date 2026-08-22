---
{"dg-publish":true,"dg-path":"Others/ojeawd","dg-permalink":"ojeawd","permalink":"/ojeawd/","created":"2026-08-22T12:49:25.108+08:00","updated":"2026-08-22T17:29:45.324+08:00","dg-note-properties":{}}
---


# 如何将 Obsidian 文章发布至个人网站？

> - [插件：obsidian-digital-garden](https://github.com/oleeskild/obsidian-digital-garden)
> - [Digital Garden 插件官方文档](https://docs.forestry.md/)

## 准备

1. 准备 GitHub 模版仓库与 Token

	- Fork 模版：访问 GitHub 上的 [oleeskild/digitalgarden](https://github.com/oleeskild/digitalgarden) 仓库，点击右上角的 Fork，将其复制到你的个人 GitHub 账号下。
	- 创建 Personal Access Token (Classic)：
		- 进入 GitHub Settings -> Developer Settings -> Personal access tokens -> Tokens (classic)。
		- 点击 Generate new token (classic)，勾选 repo 权限。
		- 复制生成的 Token 备用（仅显示一次）。
		
2. 配置 Obsidian Digital Garden 插件

	- 安装插件：在 Obsidian 中打开 设置 -> 第三方插件 -> 浏览并搜索 Digital Garden，点击安装并启用。
	- 填写连接配置：
		- GitHub Repo Name：填入 Fork 的仓库名（例如 digitalgarden）。
		- GitHub Username：你的 GitHub 用户名。
		- GitHub Token：粘贴刚才生成的 Personal Access Token。
	- 选择发布基础设置：在插件设置中可根据偏好开启 Show Backlinks（反向链接）、Show Local Graph（局部关系图）或 Enable search（站内搜索）。

3. 在 Cloudflare Pages 部署网站

	- 登录 Cloudflare：进入 Cloudflare Dashboard，左侧导航栏选择 Workers & Pages -> Create application -> **Get started**（底部有个“Looking to deploy Pages”） -> Import an existing Git repository
	- 选择仓库：授权并选择刚才 Fork 的 digitalgarden 仓库，点击 Begin setup。
	- 配置构建参数：
		- Project Name：自定义网站项目名。
		- Production branch：main。
		- Framework preset：选择 None。
		- Build command：npm run build
		- Build output directory：dist
	- 部署：点击 Save and Deploy.

## 编写与发布

在笔记 yaml 部分添加属性：

- `dg-home: true`: 将该笔记作为网站的主页。只有一个页面可以设为 true
- `dg-publish: true`: 将笔记发布到网站。
- `dg-permalink: "aaa/bbb"`：自定义该页面的 URL 路径，比如 `yoursite.pages.dev/aaa/bbb`。注意！**如果你的笔记文件名为中文，那么必须自定义 URL 路径**。因为 URL 不支持中文。可以随便打点乱码，比如 awudhi。
- `dg-path: "Lecture/MIT6.5840/uhwdo"`：改变该文件在网站导航树中的虚拟文件夹层级结构。
- `title: "your-title"`：覆盖页面在网页端显示的标题（若不设置，默认使用文件名或第一级 `#` 标题）。
- `tags: [分布式系统, Go语言]`：页面标签，会在网站上生成对应的标签分类和筛选。也可以写为以下格式：

```yaml
---
tags:
  - summary
  - life
---
```

> [!NOTE]
> 如果要自定义文件夹层级结构，让笔记放在指定的目录下，需要注意一个小 bug：
> 例如，笔记 A 和笔记 B 都要放在目录 Other 下，那么不能全都设为 `dg-path: Others/`，而要分别设为`dg-path: Others/Ahiuhdcoi` 和`dg-path: Others/Bihjosid`（后面那个随便写什么都行），才能在目录树上正常渲染。
> 
> 如果不自定义，而是默认用 obsidian 自带的文件夹层级，则无需考虑这个 bug。

其她：

- `dg-pass: "your-password"`：为该单篇笔记设置访问密码。未输入正确密码前，访客无法阅读内容。
- `dg-pinned: "true"`：在侧边栏导航树中将该笔记置顶显示。
