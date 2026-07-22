# wolffossil.github.io

基于 [Hugo](https://gohugo.io/) + [hugo-theme-stack](https://github.com/CaiJimmy/hugo-theme-stack) 的个人博客。
源码与部署统一在本仓库，通过 GitHub Actions 自动构建并发布到 GitHub Pages。

## 日常写作

```bash
# 新建一篇文章
hugo new content post/my-post/index.md

# 本地预览(含草稿)
hugo server -D

# 发布：写完 push 即可，Actions 会自动构建部署
git add . && git commit -m "post: my-post" && git push
```

无需再手动执行 `hugo` 构建，也无需维护单独的成品仓库。

## 说明

- 文章放在 `content/post/` 下。
- 主题以 git submodule 引入（`themes/hugo-theme-stack`）。克隆需带 `--recurse-submodules`。
- 构建部署逻辑见 `.github/workflows/deploy.yml`，Hugo 版本锁定在 `0.121.2`。
