<p align="center">
  <img src="source/images/pixel-cat.svg" alt="像素猫 Logo" width="160" />
</p>

# PixelCat Blog

「像素猫 - 科学上网ICU」的中文教程站点，使用 Hexo 7 构建，当前主要承载 PixelCat Proxy 安装指南。

站点内容聚焦 PixelCat Proxy 一键部署脚本，覆盖 NaiveProxy、Hysteria2、服务器准备、域名解析、防火墙放行、客户端配置、节点诊断和常见故障排查。

## 当前内容

- 安装指南：`source/_posts/pixelcat-proxy-install-guide.md`
- 首页精选专题：`source/_data/projects.json`
- 关于页面：`source/about/index.md`
- 搜索页面：`source/search/index.md`
- 标签页面：`source/tags/index.md`

## 技术栈

- Hexo 7.3
- EJS 模板
- 自定义主题 `themes/pixel-cactus`
- `hexo-generator-search` 本地搜索
- `hexo-generator-feed` Atom 订阅
- `hexo-generator-sitemap` sitemap
- `hexo-generator-robotstxt` robots.txt

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

## 配置入口

站点主配置在 `_config.yml`：

- `url`: 生产站点地址，当前为 `https://pixelcat.icu`
- `theme`: 当前主题，固定为 `pixel-cactus`
- `theme_config.nav`: 顶部导航
- `theme_config.profile`: 首页简介和头像
- `theme_config.social_links`: 页脚和结构化数据使用的社交链接
- `feed`: Atom 订阅配置
- `sitemap`: sitemap.xml / sitemap.txt 配置
- `robotstxt`: robots.txt 配置

SEO 模板在 `themes/pixel-cactus/layout/_partial/head.ejs`，会生成：

- canonical URL
- meta description / keywords / author / robots
- Open Graph
- Twitter Card
- JSON-LD 结构化数据
- sitemap link
- Google Analytics

## 常用目录

```text
source/_posts/                 文章
source/_data/projects.json      首页精选专题数据
source/about/                  关于页面
source/search/                 搜索页面
source/tags/                   标签页面
source/images/                 站点图片资源
themes/pixel-cactus/layout/    主题模板
themes/pixel-cactus/source/    主题静态资源
public/                        Hexo 生成结果
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

## 代码审查记录

本次全量检查覆盖了 Hexo 配置、主题模板、样式、搜索页脚本、文章内容入口和构建脚本。

- `npm run build` 可正常完成。
- README 已同步当前项目定位、依赖、SEO 能力和目录结构。
- 未发现需要立即修复的构建阻塞问题。

## 链接

- 官网：https://pixelcat.icu
- YouTube：https://www.youtube.com/@PixelCatICU
- GitHub：https://github.com/PixelCatICU
- X：https://x.com/PixelCatICU
- RSS：`/atom.xml`

## 合规说明

文章默认用于合法合规的学习、研究、远程办公与公开信息访问场景。请遵守所在地法律法规、服务商条款以及学校或公司的网络使用政策。
