# 本地

## 安装

1. 安装 Node.js。[官网下载](https://nodejs.org/en/)
2. 安装 Git。[官网下载](https://git-scm.com/downloads)
3. 使用 npm 或者 cnpm，进行 hexo 的安装。
```shell
npm install -g hexo-cli
```
npm太慢就用cnpm，设置为淘宝源。
```shell
npm install -g cnpm --registry=https://registry.npm.taobao.org
cnpm install -g hexo-cli
```

## 使用

1. 初始化 hexo。

先建立存放日后博客的文件夹，在文件夹路径下打开 bash 窗口，进行 hexo 安装。
```shell
hexo init
```
2. 新建文章
```shell
// 在 source/_posts 目录下生成对应的 md 文件
hexo new "文章名称"
```
3. 清理缓存
```shell
hexo clean
```
4. 生成静态文件
```shell
hexo g
```
5. 启动博客
```shell
hexo s
```

注：每次修改主题或修改文章，强力建议都要`hexo clean` 清理下，再进行生成和重新启动。

# 部署

## `github.io` 白嫖
1. 生成github.io仓库

首先注册并登录 GitHub，创建新的 public 仓库，仓库名称一定要是：
`YourGitHubName.github.io`

注：**YourGitHubName是你的GitHub昵称，大小写敏感**！部署好后，链接都是小写。

2. 本地安装 Hexo 的 git 部署插件
```yml
npm install --save hexo-deployer-git
```

3. 本地修改 _config.yaml 文件

在 Hexo 目录下的 _config.yaml 文件中，对 #Deployment 做如下修改：

```yml
# Deployment
## Docs: https://hexo.io/docs/deployment.html
deploy:
  type: git
  repo: https://github.com/YourGitHubName/YourGitHubName.github.io
  branch: master
```
注：type、repo 和 branch 冒号后都有一个空格。


4. 部署
```shell
hexo d
```
部署成功后，浏览器输入 `YourGitHubName.github.io` 即可访问。

# 美化
hexo 有很多主题能进行页面美化，常见的有 next、Yilia、Melody。

我使用的是国人制作的 Fluid，功能多样，文档详实，还有微信群...

我的[hexo博客](https://barrybean.github.io/)

Fluid 的 [GitHub](https://github.com/fluid-dev/hexo-theme-fluid) 和 [官方文档](https://hexo.fluid-dev.com/docs/guide/)。

