# 发布指南 · 多 Git 账号下推送到 GitHub(零影响工作账号)

> 本机现状:全局身份是**工作账号** `禹水 / yushui.zhy@alibaba-inc.com`,SSH key `~/.ssh/id_ed25519` 绑定工作/内网(且**未授权 github.com**),无 `~/.ssh/config`。
> 个人博客账号 **`wolffossil`**,目标仓库 `wolffossil/wolffossil.github.io`。
> 目标:配好个人账号推送,同时**完全不影响工作账号**。

---

## 零、三条铁律(不误伤工作账号的关键)

1. **永远不加 `--global`** —— 博客身份只用**仓库级** `git config`,写进博客 `.git/config`,与全局工作身份物理隔离。
2. **生成 key 必带 `-f` 指定新文件名** —— 绝不覆盖工作的 `~/.ssh/id_ed25519`。
3. **用仓库级 `core.sshCommand` 指定专用 key** —— 连 `~/.ssh/config` 都不改,工作 SSH 走向完全不受影响。

> 记住:博客的每一项设置,要么是**新建文件**,要么是**只对博客仓库生效的本地配置**。

---

## 一、方法 A(推荐):零全局足迹

`core.sshCommand` 写在博客仓库自己的 `.git/config` 里,只在这个仓库生效;`~/.ssh/config`(所有 SSH 共享)完全不动。

### 1. 生成博客专用 key(新文件,不碰工作 key)
```bash
ssh-keygen -t ed25519 -C "a1048077729@outlook.com" -f ~/.ssh/id_ed25519_wolffossil
# 一路回车即可
```

### 2. 公钥加到 GitHub 个人账号
```bash
pbcopy < ~/.ssh/id_ed25519_wolffossil.pub
```
浏览器登录 **wolffossil** → Settings → SSH and GPG keys → New SSH key → 粘贴保存。

### 3. 只在博客仓库做本地配置(全部不带 --global)
```bash
cd /Users/zhy/Desktop/projects/wolffossil-blog

# 让本仓库专用博客 key —— 不改 ~/.ssh/config,工作 SSH 零影响
git config core.sshCommand "ssh -i ~/.ssh/id_ed25519_wolffossil -o IdentitiesOnly=yes"

# 仓库级身份 —— 不动全局工作身份
git config user.name  "wolffossil"
git config user.email "a1048077729@outlook.com"    # 或隐私邮箱 <数字ID>+wolffossil@users.noreply.github.com

# remote 用普通 github.com 即可(认证由 core.sshCommand 接管)
git remote add origin git@github.com:wolffossil/wolffossil.github.io.git
# 已存在则:git remote set-url origin git@github.com:wolffossil/wolffossil.github.io.git
```

### 4. 验证走的是博客 key
```bash
GIT_SSH_COMMAND="ssh -i ~/.ssh/id_ed25519_wolffossil -o IdentitiesOnly=yes" ssh -T git@github.com
# 期望:Hi wolffossil! You've successfully authenticated...
```

### 5. 推送
```bash
git branch -M main
git push -f origin main
```
> 线上仅是旧的静态产物(可重新生成、`blog-file` 亦为备份),`-f` 覆盖最干净。想保留旧历史则改用 `git pull origin main --allow-unrelated-histories` 后再 push。

---

## 二、方法 B(备选):SSH Host 别名

如果你更习惯 ssh 别名、或同一账号要在多个仓库复用,可改用 `~/.ssh/config` 别名。**注意:这会创建一个所有 SSH 共享的配置文件**,但只要遵守"只追加、不写 `Host *`、不碰 `github.com` 默认项",工作账号依然不受影响。

`~/.ssh/config` **追加**(文件不存在则新建):
```sshconfig
# 个人博客账号 wolffossil 专用(独立别名,不影响其他 host)
Host github-wolffossil
    HostName github.com
    User git
    IdentityFile ~/.ssh/id_ed25519_wolffossil
    IdentitiesOnly yes
```
然后 remote 用别名:
```bash
git remote set-url origin git@github-wolffossil:wolffossil/wolffossil.github.io.git
git config user.name  "wolffossil"
git config user.email "a1048077729@outlook.com"
ssh -T git@github-wolffossil     # 验证:Hi wolffossil!
```

> 两种方法二选一即可。**只想影响面最小 → 用方法 A。**

---

## 三、开启 GitHub Actions 自动部署(只做一次)

浏览器打开 `https://github.com/wolffossil/wolffossil.github.io/settings/pages`
→ **Build and deployment → Source** 改为 **"GitHub Actions"**。
之后每次 `git push` 自动构建部署;进度在仓库 **Actions** 标签页看。

---

## 四、事后自查:证明工作账号毫发无损

```bash
git config --global user.email        # 仍是 yushui.zhy@alibaba-inc.com ✓
git config --global user.name         # 仍是 禹水 ✓
ls -la ~/.ssh/id_ed25519              # 工作 key 还在、时间戳未变 ✓
git -C /Users/zhy/Desktop/projects/wolffossil-blog config user.email   # 博客=个人邮箱 ✓
git -C /Users/zhy/Desktop/projects/wolffossil-blog config core.sshCommand  # 指向博客 key ✓
```
再正常做一次工作 git 操作(内网 push/pull)确认无异常即可。

---

## 五、日常发布(配置完成后)

```bash
cd /Users/zhy/Desktop/projects/wolffossil-blog
# ...写文章...
git add .
git commit -m "post: 新文章标题"
git push        # 触发自动部署,无需手动 hugo 构建
```

---

## 附:HTTPS + PAT 备选(不想用 SSH 时)

⚠️ 多账号下走 HTTPS 要小心 macOS 钥匙串把凭据和工作账号搞混,**优先用方法 A 的 SSH**。若确需:
1. wolffossil → Settings → Developer settings → Personal access tokens → 生成(勾 `repo`)。
2. `git remote set-url origin https://github.com/wolffossil/wolffossil.github.io.git`
3. push 时用户名 `wolffossil`、密码粘贴 **token**。
4. **不要**全局开 `credential.helper`;要用就在本仓库级设,避免与工作凭据混淆。

---

## 常见问题

| 现象 | 原因 / 解决 |
|------|------|
| `Permission denied (publickey)` | 公钥没加到 wolffossil;或 `core.sshCommand`/remote 没配对 |
| commit 作者是工作邮箱 | 忘了在仓库里 `git config user.email`;`git commit --amend --reset-author` 修最近一次 |
| 担心影响工作 key | 方法 A 全程不碰 `~/.ssh/id_ed25519` 和 `~/.ssh/config`,按第四节自查即可 |
| 别人 clone 缺主题 | `git clone --recurse-submodules`,见《新机器更新博客》 |
| 换机器后 push 又要认证 | `core.sshCommand` 是仓库级、不随 push 上传,新克隆需重设一次(见另一份文档) |
