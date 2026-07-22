# 发布指南 · 多 Git 账号下推送到 GitHub

> 适用场景:本机已有**工作 Git 账号**(全局身份 `禹水 / yushui.zhy@alibaba-inc.com`,SSH key `~/.ssh/id_ed25519` 绑定工作/内网,且**未授权 github.com**)。
> 个人博客账号是 **`wolffossil`**,目标仓库 `wolffossil/wolffossil.github.io`。
> 核心思路:给个人账号配**独立 SSH key + SSH Host 别名 + 仓库级身份**,与工作配置完全隔离,互不干扰。

---

## 一、为什么不能直接 push

- 全局 `user.name/email` 是工作账号 → 直接提交会把**工作邮箱**写进个人博客的 commit 记录。
- 唯一的 `id_ed25519` 绑定工作账号,**在 github.com 上没有权限**(实测 `Permission denied (publickey)`)。

因此需要为个人账号单独准备一套认证 + 身份。下面是**一次性配置**,配完以后日常只需 `git push`。

---

## 二、一次性配置(只做一次)

### 1. 为个人账号生成独立 SSH key

```bash
ssh-keygen -t ed25519 -C "你的GitHub个人邮箱" -f ~/.ssh/id_ed25519_wolffossil
# 一路回车即可(passphrase 可留空)
```
> 用 `-f` 指定独立文件名,**不要覆盖**工作用的 `id_ed25519`。

### 2. 把公钥加到 GitHub 个人账号

```bash
pbcopy < ~/.ssh/id_ed25519_wolffossil.pub   # 复制公钥到剪贴板
```
浏览器登录 **wolffossil** 账号 → Settings → SSH and GPG keys → New SSH key → 粘贴 → 保存。

### 3. 配置 SSH Host 别名(关键)

编辑(没有就新建)`~/.ssh/config`,**追加**以下内容(不会影响工作 key):

```sshconfig
# 个人博客账号 wolffossil 专用
Host github-wolffossil
    HostName github.com
    User git
    IdentityFile ~/.ssh/id_ed25519_wolffossil
    IdentitiesOnly yes
```
- `Host github-wolffossil` 是**自定义别名**,后面 remote 要用它。
- `IdentitiesOnly yes` 强制只用这把 key,避免 SSH 误用工作 key。
- github.com 的默认行为保持不变,工作流程不受影响。

### 4. 验证连通

```bash
ssh -T git@github-wolffossil
# 期望输出:Hi wolffossil! You've successfully authenticated...
```

---

## 三、给博客仓库绑定个人身份 + 远程

在博客仓库目录里执行(**仓库级配置,不影响全局工作身份**):

```bash
cd /Users/zhy/Desktop/projects/wolffossil-blog

# 仓库级身份(只对本仓库生效)
git config user.name  "wolffossil"
git config user.email "你的GitHub个人邮箱"   # 或用 GitHub 隐私邮箱 <数字ID>+wolffossil@users.noreply.github.com

# 远程用 SSH 别名(注意冒号后是 用户名/仓库名)
git remote add origin git@github-wolffossil:wolffossil/wolffossil.github.io.git
# 若已存在 origin:git remote set-url origin git@github-wolffossil:wolffossil/wolffossil.github.io.git
```

**核对身份没写错**(避免把工作邮箱推上去):
```bash
git config user.email                 # 应显示个人邮箱
git log --format='%an <%ae>' -3       # 看已有提交的作者
```

---

## 四、推送

线上 `wolffossil.github.io` 目前只是旧的静态产物(可随时重新生成,且 `blog-file` 仓库还在做备份),直接用干净的新历史覆盖最清爽:

```bash
git branch -M main
git push -f origin main
```

> 如果你想保留线上旧历史而非覆盖,改用 `git pull origin main --allow-unrelated-histories` 合并后再 `git push`,但没必要——旧内容都是生成物。

---

## 五、开启 GitHub Actions 自动部署(只做一次)

浏览器打开:`https://github.com/wolffossil/wolffossil.github.io/settings/pages`
→ **Build and deployment → Source** 改为 **"GitHub Actions"**。

之后每次 `git push`,`.github/workflows/deploy.yml` 会自动用 Hugo 构建并部署,几分钟后 `https://wolffossil.github.io` 更新。构建进度在仓库 **Actions** 标签页查看。

---

## 六、日常发布(配置完成后)

```bash
cd /Users/zhy/Desktop/projects/wolffossil-blog
# ...写文章...
git add .
git commit -m "post: 新文章标题"
git push        # 触发自动部署,无需再手动 hugo 构建
```

---

## 附:HTTPS + PAT 备选方案(不想用 SSH 时)

1. GitHub(wolffossil)→ Settings → Developer settings → Personal access tokens → 生成 token(勾选 `repo`)。
2. remote 用 HTTPS:
   ```bash
   git remote set-url origin https://github.com/wolffossil/wolffossil.github.io.git
   ```
3. push 时用户名填 `wolffossil`,密码**粘贴 token**(不是账号密码)。
4. 想免输:`git config credential.helper osxkeychain`(macOS 钥匙串);多账号下注意钥匙串里区分不同 host 的凭据。

> SSH 别名方案对多账号更省心,推荐优先用它。

---

## 常见问题

| 现象 | 原因 / 解决 |
|------|------|
| `Permission denied (publickey)` | 公钥没加到 wolffossil 账号,或 remote 没用 `github-wolffossil` 别名 |
| commit 作者显示成工作邮箱 | 忘了在仓库里 `git config user.email`;用 `git commit --amend --reset-author` 修最近一次 |
| push 报 submodule 相关错 | 正常,主题是子模块;确保 `.gitmodules` 指向官方(已配好),CI 会自动拉取 |
| 别人 clone 缺主题 | 让对方 `git clone --recurse-submodules`,详见另一份《新机器更新博客》文档 |
