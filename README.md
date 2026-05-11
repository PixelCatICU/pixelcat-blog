<p align="center">
  <img src="source/images/pixel-cat.svg" alt="像素猫 Logo" width="160" />
</p>

# 像素猫 - 科学上网ICU

「像素猫 - 科学上网ICU」YouTube 频道的中文教程博客，使用 Hexo 构建，主题为本项目内置的 `pixel-cactus`，风格参考 `hexo-theme-cactus` 的极简博客布局。

博客内容主要覆盖科学上网、网络工具、代理协议、服务器搭建、节点部署、路由分流、隐私安全和实用软件教程。

## 技术栈

- Hexo 7
- EJS 模板
- 自定义主题 `themes/pixel-cactus`
- `hexo-generator-search` 本地搜索
- `hexo-generator-feed` Atom/RSS 订阅

## 本地开发

安装依赖：

```bash
npm install
```

启动本地服务：

```bash
npm run server
```

默认访问：

```text
http://localhost:4000/
```

生成静态文件：

```bash
npm run build
```

清理生成文件：

```bash
npm run clean
```

## 部署到 Cloudflare Pages

Cloudflare Pages 构建设置：

- Framework preset: `Hexo`
- Build command: `npm run build`
- Build output directory: `public`
- Node.js version: `20` 或 `22`

部署流程：

1. 将仓库推送到 GitHub。
2. 在 Cloudflare Dashboard 进入 `Workers & Pages`。
3. 创建 Pages 项目并连接该 GitHub 仓库。
4. 填写上面的构建设置。
5. 部署完成后绑定自定义域名。

## 常用目录

```text
source/_posts/                 文章
source/about/                  关于页面
source/search/                 搜索页面
source/tags/                   标签页面
source/images/                 图片资源
themes/pixel-cactus/layout/    主题模板
themes/pixel-cactus/source/    主题静态资源
```

## 链接

- YouTube: https://www.youtube.com/@PixelCatICU
- GitHub: https://github.com/PixelCatICU
- X: https://x.com/PixelCatICU
- RSS: `/atom.xml`

## 说明

文章默认用于合法合规的学习、研究、远程办公与公开信息访问场景。请遵守所在地法律法规、学校或公司的网络使用政策。
