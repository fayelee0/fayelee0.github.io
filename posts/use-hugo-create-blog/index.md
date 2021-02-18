# 使用 Hugo 自建个人博客

使用 Hugo 创建个人博客，并使用 Github Actions 来自动部署。

<!--more-->

## 准备阶段

1. 创建 [Github](https://github.com) 私有仓库 A 用来存储原始内容
2. 创建 [Github Pages](https://pages.github.com/) 公开仓库 B 用来发布
3. 生成 [SSH KEYGEN] 用来将 A 生成的内容发布到 B 中
4. 将私有 KEY 存储到仓库 A 的 **Settings -> Secrets**，并命名 **ACTIONS_DEPLOY_KEY**
5. 将共有 KEY 存储到仓库 B 的 **Settings -> Deploy keys**

## 开启 Hugo

生成 [Hugo](https://gohugo.io/getting-started/quick-start/) 项目并配置
```text
# 创建 Hugo 项目
hugo new site pursuetao.com

# 添加主题，可忽略该步骤使用默认主题
git submodule add https://github.com/dillonzq/LoveIt.git themes/LoveIt

# 根据该内容配置主题
https://hugoloveit.com/zh-cn/theme-documentation-basics/#basic-configuration

# 运行预览效果
hugo serve -D
```

控制台编译成功之后，在浏览器访问：
```text
http://localhost:1333/
```

将代码提交到 [Github](https://github.com) 仓库 A：
```text
git add .
git commit -S -m "feat: init project"
git push origin master
```

## 配置自动部署

在项目 A 中配置 [Github Actions](https://github.com/features/actions)，可直接在仓库中添加 .github/workflows/deploy-to-github-pages.yml 文件。将以下内容添加到项目中，并修改项目名称。

```yml
name: GithubPages

on:
  push:
    branches: [ master ]
  pull_request:
    branches: [ master ]

  # Allows you to run this workflow manually from the Actions tab
  workflow_dispatch:

jobs:
  deploy:
    runs-on: ubuntu-latest

    steps:
      # Checks-out your repository under $GITHUB_WORKSPACE, so your job can access it
      - uses: actions/checkout@v2
        with:
          submodules: recursive
          fetch-depth: 0

      - name: Setup Hugo
        uses: peaceiris/actions-hugo@v2
        with: 
          hugo-version: '0.80.0'
          extended: true

      - name: Build
        run: hugo --minify
   
      - name: Deploy
        uses: peaceiris/actions-gh-pages@v3
        with:
          deploy_key: ${{ secrets.ACTIONS_DEPLOY_KEY }}
          external_repository: <user>/<user>.github.io # TODO: 修改项目名
          publish_branch: master
          publish_dir: ./public
```

验证 Actions 是否正确执行：*在 Github 仓库 A 中找到 Actions 执行记录，如果是绿色表示执行成功*
![Actions Success](/images/use-hugo-create-blog-00.png)

配置成功之后访问 Github Pages 地址 `<user>.github.io` 查看效果。

## 
