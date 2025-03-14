---
title: Obsidian Lastmod Updater
date: 2025-03-12
slug: obsidian-lastmod-updater
categories:
  - Tool
description: 使用 AI 工具开发一个Obsidian插件，实现在保存时，Obsidian 将自动更新 Front Matter 中的 lastmod 字段值
draft: false
lastmod: 2025-03-14
---
## 需求分析

在完成编写或修改博客文章后，我常需要手动去修改笔记属性中的最后修改时间（下称 lastmod）字段的值，起初博文只有几篇，能够记得修改 lastmod，但如今成篇加上尚未发布的草稿共有二十多篇，每次修改或想发布草稿，手动修改 lastmod 都让我十分难受，尤其是修改了多篇博文时。

在最近一次修改了博文却忘记修改 lastmod 后，我终于决定寻求解决方案。

## 功能设计

经过数分钟的思考后，我认为该插件应该具备功能是，在我按下 Ctrl + S 保存时，Obsidian 能自动修改 Front Matter 中 lastmod 值为当前日期。

由此场景，再考虑到插件的扩展性，功能设计如下：

- 保存时，自动设置 lastmod 字段值为当前日期。
- 支持用户自定义 Front Matter 的分隔符。
- 支持用户自定义 lastmod 日期格式。
- 支持用户自定义 lastmod 字段名称。

## 开发工作
### 环境准备

- Node.js [**必须**]
- Git [推荐]
- Visual Studio Code [推荐]
- 一个空 Obsidian 仓库 [推荐]

### 创建插件
依据 [官方插件构建文档](https://docs.obsidian.md/Plugins/Getting+started/Build+a+plugin#:~:text=Plugins%20let%20you%20extend%20Obsidian%20with%20your%20own,from%20source%20code%20and%20load%20it%20into%20Obsidian.)，可去往此链 [obsidianmd/obsidian-sample-plugin](https://github.com/obsidianmd/obsidian-sample-plugin)，据模板创建插件框架。

可通过右上角的 **Use this template** 按钮在 GitHub 创建自己插件仓库后，clone 到本地进行开发，也可直接 clone 模板到本地进行开发。

若无 Git，还可直接下载 Zip 后解压进行开发。

进入插件目录（本文中为 `obsidian-lastmod-updater`），安装依赖：
```shell
npm i # npm install
npm run dev  # 构建插件，生成 main.js
```

### 修改插件属性

在根目录中的 `manifest.json` 和 `package.json` 中编辑自己的插件属性，参考如下：

***manifest.json***
```json
{
    "id": "obsidian-lastmod-updater",
    "name": "Lastmod Updater",
    "version": "1.0.2",
    "minAppVersion": "0.15.0",
    "description": "Automatically updates the YAML front matter's lastmod field to the current date each time you save a file.",
    "author": "Hilariouhiss",
    "authorUrl": "https://blog.frosted.top",
    "fundingUrl": "",
    "isDesktopOnly": false
}
```

其中需要修改的有：
- `id`：插件唯一表示符，对应 `package.json` 中的 `name` 字段
- `name`：可读性强的插件名称
- 修改插件目录名称以匹配 `id`

可选修改的有：
- `version`：插件版本，对应 `package.json` 同字段
- `description`：对插件功能的简单描述，对应 `package.json` 同字段
- `author`：作者
- `authorUrl`：作者的个人网站
- `fundingUrl`：赞助作者的网站

### 功能实现

在根目录的 `main.ts` 中开发功能。

此部分使用 VS Code 的 ChatGPT Copilot 完成，步骤如下：
- Copilot 功能区最上方，选择 Copilot 编辑
- 聊天框中，选择 `main.ts` 文件
- 输入想要实现的功能，需要描述准确，先实现基础功能
- 在测试基础功能正常后，再命令 Copilot 逐条实现其他扩展功能

### 自动构建与发布

借助 Github Action 实现，步骤如下：

1. 在根目录中创建 `.github/workflows/release.yml`，贴入以下内容：

```yml
name: Release Obsidian plugin

on:
  push:
    tags:
      - "*"
env:
  PLUGIN_NAME: obsidian-lastmod-updater

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - name: Use Node.js
        uses: actions/setup-node@v1
        with:
          node-version: "14.x"
      - name: Build
        id: build
        run: |
          npm install
          npm run build
          mkdir ${{ env.PLUGIN_NAME }}
          cp main.js manifest.json styles.css ${{ env.PLUGIN_NAME }}
          zip -r ${{ env.PLUGIN_NAME }}.zip ${{ env.PLUGIN_NAME }}
          ls
          echo "tag_name=$(git tag --sort version:refname | tail -n 1)" >> $GITHUB_ENV
      - name: Create Release
        id: create_release
        uses: actions/create-release@v1
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
          VERSION: ${{ github.ref }}
        with:
          tag_name: ${{ github.ref }}
          release_name: ${{ github.ref }}
          draft: false
          prerelease: false
      - name: Upload zip file
        id: upload-zip
        uses: actions/upload-release-asset@v1
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        with:
          upload_url: ${{ steps.create_release.outputs.upload_url }}
          asset_path: ./${{ env.PLUGIN_NAME }}.zip
          asset_name: ${{ env.PLUGIN_NAME }}-${{ env.tag_name }}.zip
          asset_content_type: application/zip
      - name: Upload main.js
        id: upload-main
        uses: actions/upload-release-asset@v1
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        with:
          upload_url: ${{ steps.create_release.outputs.upload_url }}
          asset_path: ./main.js
          asset_name: main.js
          asset_content_type: text/javascript
      - name: Upload manifest.json
        id: upload-manifest
        uses: actions/upload-release-asset@v1
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        with:
          upload_url: ${{ steps.create_release.outputs.upload_url }}
          asset_path: ./manifest.json
          asset_name: manifest.json
          asset_content_type: application/json
      - name: Upload styles.css
        id: upload-css
        uses: actions/upload-release-asset@v1
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        with:
          upload_url: ${{ steps.create_release.outputs.upload_url }}
          asset_path: ./styles.css
          asset_name: styles.css
          asset_content_type: text/css
```

需将 `env.PLUGIN_NAME` 修改为插件 `id`

2. 将仓库推送至 Github   
    注意：在完整修改或开发时，应执行 `npm run build` 命令进行项目构建，否则应用不生效
3. 执行如下命令：
```shell
git tag -a 1.0.2 -m "1.0.2"
git push origin 1.0.2
```

注意：版本号应该与 `manifest.json` 和 `package.json` 中的 `version` 对应

4. 访问 Github 仓库，可见对应版本的 Release，将 zip 文件解压提取后，放在 `.obsidian/plugins` 目录下，并开方第三方插件即可使用。