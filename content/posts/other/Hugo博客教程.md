+++
date = '2025-11-18T16:37:07+08:00'
title = 'Hugo博客教程'
categories = ["杂项"]
tags =  ["Hugo"]
+++
一套「Hugo + GitHub + 两分支（source/main）+ 自动部署」的完整流程，你照着做就能跑起来。

> 约定：
>
> + `source` 分支：Hugo 源码
> + `main` 分支：Hugo 生成的静态文件（给 GitHub Pages 用）
>

---

<h2 id="LMTYC">一、准备工作</h2>
1. **安装 Hugo**

```bash
hugo version
```

    - 看你系统自己装就行，确认一下版本：
2. **GitHub 上建仓库**
    - 新建一个 repo，比如：`my-blog`
    - 默认会有一个 `main` 分支，先保留（后面用来放静态文件）

---

<h2 id="VWZE4">二、本地初始化 Hugo 并推到 `source` 分支</h2>
1. 克隆仓库到本地：

```bash
git clone git@github.com:你的用户名/my-blog.git
cd my-blog
```

2. 创建并切到 `source` 分支：

```bash
git checkout -b source
```

3. 在当前目录初始化 Hugo 项目（以 `my-blog` 为例）：

```bash
hugo new site .
```

4. 加主题（以 Ananke 为例）：

```bash
git submodule add https://github.com/theNewDynamic/gohugo-theme-ananke.git themes/ananke
```

在 `config.toml` 里加上：

```toml
theme = "ananke"
baseURL = "https://你的用户名.github.io/"
```

5. 随便新建一篇文章测试：

```bash
hugo new posts/hello-world.md
```

然后去 `content/posts/hello-world.md` 把 `draft: true` 改成 `false`。

6. 提交到 `source` 分支：

```bash
git add .
git commit -m "init hugo site"
git push origin source
```

---

<h2 id="kUUA6">三、让 `main` 分支只放静态文件</h2>
我们让 Hugo 的输出目录指向一个单独的目录（比如 `public/`，默认就是），再用 GitHub Actions 帮你把 `public` 的内容同步到 `main` 分支，**你本地不需要切 main 分支手动搞**。

GitHub 这边需要两件事：

1. GitHub Actions workflow（自动构建）
2. GitHub Pages 设置（从 `main` 分支部署）

---

<h2 id="eUqYo">四、创建 GitHub Actions 自动构建</h2>
> 目标：当你 push 到 `source` 分支时：
>
> 1. 自动拉代码
> 2. 安装 Hugo
> 3. 执行 `hugo` 生成静态文件到 `public/`
> 4. 把 `public/` 的内容推送到 `main` 分支
>

1. 在本地 `source` 分支创建 workflow 文件夹：

```bash
mkdir -p .github/workflows
```

2. 新建文件 `.github/workflows/**<font style="color:rgb(31, 35, 40);">gh-pages</font>**.yml`，内容例如：

```yaml
name: GitHub Pages

on:
  push:
    branches:
      - source  # 当 source 分支有更新时触发构建

jobs:
  build-deploy:
    runs-on: ubuntu-20.04
    concurrency:
      group: ${{ github.workflow }}-${{ github.ref }}
    steps:
      - name: Checkout
        uses: actions/checkout@v3
        with:
          submodules: true
          fetch-depth: 0

      - name: Setup Hugo
        uses: peaceiris/actions-hugo@v2
        with:
          hugo-version: "latest"

      - name: Build
        run: hugo --minify

      - name: Deploy
        uses: peaceiris/actions-gh-pages@v3
        with:
          github_token: ${{ secrets.HUGO_BLOG }}  # 你的 GitHub token
          publish_branch: main  # 将构建的内容推送到 main 分支
          publish_dir: ./public  # public 目录是 Hugo 构建后的输出目录
```

3. 提交这个 workflow 文件：

```bash
git add .github/workflows/gh-pages.yml
git commit -m "add github actions for hugo deploy"
git push origin source
```

4. 这时候到 GitHub 仓库的 **Actions** 标签页里，应该会看到 workflow 在跑。
    - 如果成功，`main` 分支会被自动创建/更新，里面就是纯静态文件（`index.html` 等）。

---

<h2 id="hgqWQ">五、配置 GitHub Pages 使用 `main` 分支</h2>
1. 进入你的 GitHub 仓库页面 → Settings
2. 左侧找到 **Pages**
3. 配置：
    - **Source**：选 `Deploy from a branch`
    - **Branch**：选 `main` 分支，目录 `/ (root)`
4. 保存

GitHub 会给你一个地址，通常是：

+ `https://你的用户名.github.io/`  
或
+ `https://你的用户名.github.io/仓库名/`

几分钟后访问这个地址就能看到你的 Hugo 博客了。

---

<h2 id="pccaF">六、之后写博客的日常操作</h2>
以后你只需要动 **source 分支**：

1. 确保在 `source` 分支：

```bash
git checkout source
```

2. 新建文章：

```bash
hugo new posts/xxx.md
```

改好内容，把 `draft: true` 改成 `false`。

3. 本地预览（可选）：

```bash
hugo server -D
```

浏览器访问 `http://localhost:1313` 看效果。

4. 提交并推送：

```bash
git add .
git commit -m "add new post"
git push origin source
```

5. GitHub Actions 会自动执行构建和部署，把更新后的静态文件推送到 `main`，GitHub Pages 自动更新。

---

<h2 id="emrDt">七、常见坑 & 建议</h2>
1. **baseURL 写错**
    - 如果你仓库名不是 `你的用户名.github.io`，而是普通仓库，例如 `my-blog`，  
那 `config.toml` 里的 `baseURL` 应该是：

```toml
baseURL = "https://你的用户名.github.io/my-blog/"
```

2. **main 分支手动改动会被覆盖**
    - 不要手动在 `main` 分支改任何东西，因为每次 workflow 部署都会覆盖。
    - 所有内容都通过 `source` 分支的 Hugo 代码来维护。
3. **Actions 权限**
    - 一般默认的 `GITHUB_TOKEN` 有权限推到 `main`，不用额外设置。
    - 如果报权限错误，去仓库 → Settings → Actions → General  
确保 `Workflow permissions` 里勾选了：
        * `Read and write permissions`

---



