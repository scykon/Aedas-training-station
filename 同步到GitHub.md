# 把训练系统同步到 GitHub 私人仓库

## 一、你这台机器的现状（已实测确认）

| 检查项 | 结果 | 影响 |
|---|---|---|
| Git | **未安装**（`Program Files`、`Program Files (x86)`、`AppData\Local\Programs` 三处常见位置都查过，均不存在） | 想用命令行就必须先装 |
| GitHub CLI（`gh`） | 未安装 | 建仓库得去网页操作 |
| SSH 密钥 | 没有（`~/.ssh/id_ed25519`、`~/.ssh/id_rsa` 都不存在） | 首次推送需要走 Token 或新建密钥 |
| winget | **可用** | 可以用一行命令装好 Git |
| 待上传文件 | 6 个文件 + 1 个 `uploads` 目录，合计约 124 KB | 体积很小，三条路线都毫无压力 |

---

## 二、三条路线怎么选

| 路线 | 要装什么 | 适合谁 | 代价 |
|---|---|---|---|
| **A 网页拖拽上传** | 什么都不装 | 只想把现在这批文件存一份 | 每次更新都要手动拖，容易漏文件，且无法保留空目录 |
| **B Git 命令行** ← 推荐 | 装 Git（约 60MB） | 长期用，每天记录训练成果 | 需要记 3 条命令 |
| **C GitHub Desktop** | 装 GitHub Desktop（自带 Git） | 不想碰命令行，但要长期用 | 多一个常驻 GUI 软件 |

**我的建议是 B。** 你要连续记录 12 周、每天传手绘图纸，路线 A 手工拖很快就会烦到放弃；而命令行日常同步只需要固定 3 条命令。

---

## 三、路线 A：网页拖拽上传（零安装，5 分钟）

1. 打开 github.com，登录 → 右上角 `+` → **New repository**
2. **Repository name** 填：`aedas-plan-growth`
3. 可见性选 **Private**（私人仓库）
4. 下面的 Add a README file / Add .gitignore / Choose a license **全部不要勾**（避免和本地文件冲突）
5. 点 **Create repository**
6. 在新仓库页面点 **uploading an existing file** 链接
7. 把 `aedas-plan-growth` 文件夹里**除 `uploads` 外的所有文件**拖进上传区
   - 网页上传不支持空文件夹，`uploads` 文件夹放进 `.gitkeep` 后即可正常上传
8. 底部 Commit changes 填一句说明 → **Commit changes**

> 以后更新：进仓库 → Add file → Upload files → 拖入改动过的文件（同名会覆盖）→ Commit。

---

## 四、路线 B：Git 命令行（推荐，一次配好长期用）

### 第 1 步 · 安装 Git

在 PowerShell 里执行：

```powershell
winget install --id Git.Git -e --source winget
```

装完后 **关掉并重新打开终端**（否则 PATH 不生效），然后验证：

```powershell
git --version
```

### 第 2 步 · 配置身份（只做一次）

```powershell
git config --global user.name "你的名字或GitHub用户名"
git config --global user.email "你的GitHub注册邮箱"
git config --global core.quotepath false
git config --global init.defaultBranch main
```

其中 **`core.quotepath false` 对你特别重要** —— 你的文件名全是中文，不设这条的话 `git status` 里中文会显示成 `\346\226\207...` 这样的转义码，很难认。

### 第 3 步 · 本地初始化并提交

```powershell
cd "C:\Users\quxz\WorkBuddy\2026-09-15-11-10-34\aedas-plan-growth"
git init
git add .
git commit -m "初始化：商业综合体平面设计成长训练系统"
```

### 第 4 步 · 在 GitHub 网页建私人仓库

github.com → 右上角 `+` → New repository：

- **Repository name**：`aedas-plan-growth`
- 选 **Private**
- **不要**勾选任何初始化选项
- Create repository

### 第 5 步 · 关联远程仓库并推送

把下面两处的 `你的用户名` 换成你的 GitHub 用户名：

```powershell
git remote add origin https://github.com/你的用户名/aedas-plan-growth.git
git branch -M main
git push -u origin main
```

首次推送会弹窗要求认证。**不要用账号密码**（GitHub 早已停用密码推送），二选一：

**(a) Personal Access Token（推荐，最简单）**

1. GitHub → 右上角头像 → **Settings** → 左侧拉到底 **Developer settings**
2. **Personal access tokens** → **Fine-grained tokens** → **Generate new token**
3. **Repository access**：选 *Only select repositories* → 只勾 `aedas-plan-growth`
4. **Permissions** → Repository permissions → **Contents** 设为 **Read and write**
5. 有效期建议 90 天
6. 生成后**立刻复制**（只显示一次）。推送弹窗里用户名填 GitHub 用户名，密码框粘贴这个 Token

**(b) SSH 密钥（配一次以后免密）**

```powershell
ssh-keygen -t ed25519 -C "你的GitHub邮箱"
# 一路回车即可，然后查看公钥：
type $env:USERPROFILE\.ssh\id_ed25519.pub
```

复制输出内容 → GitHub → Settings → **SSH and GPG keys** → New SSH key → 粘贴保存。
然后远程地址改用 SSH 形式：

```powershell
git remote set-url origin git@github.com:你的用户名/aedas-plan-growth.git
```

### 日常更新（以后只需要这 3 条）

```powershell
cd "C:\Users\quxz\WorkBuddy\2026-09-15-11-10-34\aedas-plan-growth"
git add .
git commit -m "第 N 周：动线抄绘 + 得铺率计算"
git push
```

---

## 五、路线 C：GitHub Desktop（不碰命令行）

1. 到 desktop.github.com 下载安装（安装包会**自带 Git**，不用单独装）
2. 打开后用 GitHub 账号登录（图形化授权，不需要手动配置 Token）
3. 菜单 **File → Add local repository** → 选择
   `C:\Users\quxz\WorkBuddy\2026-09-15-11-10-34\aedas-plan-growth`
   - 如果提示这不是 Git 仓库，点它给的 **create a repository** 就地创建
4. 左下角 Summary 填一句说明 → 点 **Commit to main**
5. 顶部点 **Publish repository** → **勾选 Keep this code private** → 发布完成

以后每次：打开 GitHub Desktop → 左侧会列出改动 → 写一句 Summary → **Commit** → 右上角 **Push origin**。

---

## 六、五个必须注意的坑

1. **仓库根目录一定要设在 `aedas-plan-growth`，不要设在上一层的 `2026-09-15-11-10-34`。**
   上层目录里有 `.workbuddy\`（存放本地记忆与工作数据），那个目录不该进 Git。
   —— 如果因为特殊原因必须传上层，本目录的 `.gitignore` 模板里已经预留了 `.workbuddy/` 的忽略规则，照着放到上层即可。

2. **单文件 100MB 硬限制。** 以后手绘照片如果是手机原图（可能十几 MB），长期累积会让仓库迅速膨胀。建议拍照后压缩到 1MB 以内再放 `uploads`；真需要保留原图，就放到网盘，仓库里只放压缩版。

3. **私人仓库 ≠ 加密保险箱。** 任何拿到你 Token 的人、以及你邀请的协作者都能看到全部内容。Token 泄露要立刻在 GitHub 上 Revoke。

4. **上传前先自查有没有涉及公司保密内容。** 这套文件里我写的都是公开可查的行业指标与公开项目案例（苏州中心、苏州 HLCC Mall、上海 Landmark Center 等均为公开发布的项目），本身没有保密风险。但**你第 11 周要做的「用自己参与过的真实项目反推平面」会涉及公司项目名称、甲方信息、未公开图纸** —— 那部分要不要进仓库，先确认你和公司的保密约定，必要时用代号替代真实项目名。

5. **`.gitignore` 已随本目录提供**，内容是系统垃圾文件 + `.workbuddy/` + 可选的图纸忽略规则。不需要可以删，但建议留着。

---

## 七、可选：加一份仓库说明

如果想让仓库首页好看一点，可以在 GitHub 仓库页点 **Add file → Create new file**，命名为 `README.md`，内容直接复用本目录已有的 `README.md`（它已经是现成的索引页）。
