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
- 主题以 git submodule 引入，**直接跟随官方** [`CaiJimmy/hugo-theme-stack`](https://github.com/CaiJimmy/hugo-theme-stack)（当前 `v4.0.3`），不再维护个人 fork。克隆需带 `--recurse-submodules`。
- 更新主题：`git submodule update --remote themes/hugo-theme-stack` 后 commit。
- 自定义样式统一放在 `assets/scss/custom.scss`（覆盖主题占位），改主题外观不用动 submodule。
- 构建部署逻辑见 `.github/workflows/deploy.yml`，Hugo 锁定 `0.164.0`。

> 注：代码块「复制」按钮为主题内置英文 `Copy`（旧 fork 曾汉化，为不再维护 fork 已放弃这处汉化）。
