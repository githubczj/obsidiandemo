# Obsidian Git 插件安装与配置完全指南

> **适用场景**：将 Obsidian 笔记库通过 Git 同步到 GitHub，实现云端备份与多端同步。  
> **核心前提**：Obsidian Git 插件只是"指挥官"，系统级 Git 软件才是"干活的"，两者缺一不可。

---

## 一、理解两个关键概念

| 概念 | 类型 | 作用 |
|------|------|------|
| **Obsidian Git 插件** | Obsidian 插件 | 提供图形界面，发出同步指令 |
| **Git 软件（Git for Windows）** | 系统级工具 | 真正执行提交、推送等操作 |

**常见报错根因**：`Git is not ready` → 插件找不到系统级 Git，或笔记库未初始化为 Git 仓库。

---

## 二、解决 `Git is not ready` 报错

### 第一步：确认系统是否已安装 Git

按 `Win + R`，输入 `cmd`，回车，执行：

```bash
git --version
```

- **返回版本号**（如 `git version 2.45.0`）→ 已安装，继续下一步
- **返回"不是内部或外部命令"** → 未安装，前往 [https://git-scm.com](https://git-scm.com) 下载安装，**安装时务必勾选 `Add to PATH`**

---

### 第二步：获取 Git 可执行文件的完整路径

打开 PowerShell，执行：

```bash
where git
```

复制返回的完整路径，例如：

```
C:\Program Files\Git\cmd\git.exe
```

> ⚠️ **注意**：需要的是 `git.exe` 结尾的完整路径，不是文件夹路径。

---

### 第三步：配置 Obsidian Git 插件路径

进入 Obsidian → `设置` → `Obsidian Git` → 找到 **Advanced** 区域：

| 配置项 | 填写内容 |
|--------|----------|
| `Custom Git binary path` | 粘贴上一步复制的完整路径，如 `C:\Program Files\Git\cmd\git.exe` |
| `Custom base path` | **清空，保持空值**（99% 用户无需修改） |

配置完成后，点击 **`Reload with new environment variables`** 按钮使其生效。

---

### 第四步：初始化本地 Git 仓库

如果笔记库从未被 Git 管理过，插件依然无法识别。执行以下操作：

1. 找到你的笔记库文件夹（含 `.obsidian` 文件夹的那一层）
2. 在地址栏输入 `cmd` 回车，打开终端
3. 依次执行：

```bash
git init
git add .
git commit -m "初始化笔记仓库"
```

执行完毕后，笔记库根目录出现 `.git` 隐藏文件夹，即代表初始化成功。

---

### 第五步：验证配置生效

1. 点击 `Reload with new environment variables`
2. 完全关闭 Obsidian，重新打开
3. 顶部 `Git is not ready` 提示消失 → **本地配置完成**

---

## 三、配置远程仓库（同步到 GitHub）

### 1. 在 GitHub 创建空仓库

- 登录 GitHub → 右上角「+」→「New repository」
- 填写仓库名，**不要勾选 `Initialize this repository with a README`**，直接点创建

### 2. 关联本地仓库与远程仓库

在笔记库终端执行（替换为你自己的 GitHub 用户名和仓库名）：

```bash
git remote add origin https://github.com/你的用户名/你的仓库名.git
```

### 3. 推送第一次提交

```bash
git branch -M main
git push -u origin main
```

---

## 四、解决 `getaddrinfo() thread failed to start` 网络报错

**问题原因**：Git 无法解析 GitHub 域名，属于网络层问题，与 Obsidian 无关。

### 方法一：清除 Git 代理配置

```bash
git config --global --unset http.proxy
git config --global --unset https.proxy
git config --global http.proxy ""
git config --global https.proxy ""
```

---

### 方法二：手动修改 hosts 文件（绕过 DNS）

1. 以**管理员身份**打开 `C:\Windows\System32\drivers\etc\hosts`
2. 在文件末尾追加：

```
140.82.114.4      github.com
199.232.69.194    github.global.ssl.fastly.net
```

3. 刷新 DNS 缓存：

```bash
ipconfig /flushdns
```

4. 重试 `git push`

---

### 方法三：改用 SSH 协议（推荐，彻底避开 HTTPS 网络问题）（亲测有效）

**Step 1**：生成 SSH 密钥

```bash
ssh-keygen -t ed25519 -C "你的GitHub绑定邮箱"
```

一路回车，完成后在 `~/.ssh/` 目录生成 `id_ed25519.pub` 文件。

**Step 2**：将公钥添加到 GitHub

登录 GitHub → `Settings` → `SSH and GPG keys` → `New SSH key`，将 `id_ed25519.pub` 的内容粘贴进去。

**Step 3**：修改远程仓库地址为 SSH 协议

```bash
git remote set-url origin git@github.com:你的用户名/你的仓库名.git
```

**Step 4**：重新推送

```bash
git push -u origin main
```

---

## 五、GitHub 账号配置说明

### 查看当前 Git 关联邮箱

```bash
git config --global user.email
```

### 修改为 GitHub 绑定邮箱（保持一致，避免提交记录混乱）

```bash
git config --global user.name "你的GitHub用户名"
git config --global user.email "你的GitHub绑定邮箱"
```

### GitHub `Confirm access` 验证说明

在 GitHub 添加 SSH Key 等敏感操作时，会触发二次验证（sudo mode），直接输入 GitHub 账号密码点击 `Verify` 即可，验证后数小时内免重复验证。

---

## 六、Obsidian Git 插件推荐设置

| 配置项 | 推荐值 | 说明 |
|--------|--------|------|
| `Automatically refresh source control view on file changes` | 开启 | 文件变更时自动刷新状态栏 |
| `Source control view refresh interval` | `7000ms` | 刷新间隔，卡顿时可适当调高 |
| `Diff view style` | `Split` | 左右分栏对比修改，直观 |
| 定时自动提交/同步 | 按需开启 | 也可使用命令面板手动提交 |

---

## 七、常见问题速查

| 报错 / 现象 | 根本原因 | 解决方向 |
|-------------|----------|----------|
| `Git is not ready` | Git 未安装 或 路径未配置 | 参考第二章 |
| `Git is not ready`（路径已填） | 笔记库未初始化为 Git 仓库 | 执行 `git init` |
| `getaddrinfo() thread failed to start` | DNS 解析 GitHub 失败 | 参考第四章 |
| push 成功但 GitHub 无提交记录 | 本地 Git 邮箱与 GitHub 邮箱不一致 | 参考第五章 |
| SSH 推送提示 `Permission denied` | 公钥未添加到 GitHub | 重新执行方法三 Step 2 |

---

> **小结**：整个流程的核心就三件事——**装好 Git → 初始化仓库 → 配置插件路径**。网络问题优先用 SSH 协议彻底解决，比修改 hosts 更稳定。
