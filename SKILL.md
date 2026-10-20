---
name: git-ai
description: >-
  安装、登录、配置并排障蓝区 Git AI（git-ai / git ai）：内部安装脚本、公司邮箱、授权码、
  git-ai config、hooks、AI 产出统计与升级。Use when the user mentions git-ai, git ai,
  Git AI, gitai.matrix.openharmony.cn, 授权码, user_code, AI 产出, Co-authored-by,
  install-hooks, or configuring git-ai identity and server.
---

# 蓝区 Git AI

内部定制的 git 代理，把 AI 改动归属记到提交上，数据只上报 `https://gitai.matrix.openharmony.cn`。指南：<https://gitai.matrix.openharmony.cn/guide/>（需用 Git 邮箱 + 授权码登录；正文是 `/guide/README.md`）。命令细节与 FAQ 见 [reference.md](reference.md)。

入口优先级：PATH 里的 `git ai` / `git-ai`；没有则用 `$HOME/.git-ai/bin/git-ai`。`git` 必须解析到 `~/.git-ai/bin` 里的 shim，系统 git 在前则采集不到数据。

## 安装

先确认 `git --version` ≥ 2.27。邮箱不是公司邮箱时，钩子步骤会失败并提示 `请使用公司邮箱注册`。只有用户明确给出公司邮箱时才改：

```bash
git config --global user.email '<公司邮箱>'
```

按系统执行官方脚本（不要改写成别的安装方式）：

```bash
# macOS / Linux / Windows WSL
curl -ksSL https://gitai.matrix.openharmony.cn/release/install.sh | bash

# 鸿蒙 PC：先在应用市场安装 GitNext，终端是 zsh
curl -ksSL https://gitai.matrix.openharmony.cn/release/install.sh | zsh

# Windows（非 WSL、非 Git Bash）
powershell -NoProfile -ExecutionPolicy Bypass -Command "irm https://gitai.matrix.openharmony.cn/release/install.ps1 | iex"
```

脚本会：下载校验二进制到 `~/.git-ai/bin`，链到 `~/.local/bin/git-ai`，执行 `install-hooks --env`（改 shell PATH、给已安装的 Agent 装 hooks），然后 `git ai login`。

装完后：

1. 当前 shell 执行 `source ~/.zshrc`（bash 用 `~/.bashrc`），或新开终端。macOS 上 `~/.bash_profile` 权限失败可忽略，zsh 不受影响。
2. 重启正在运行的 IDE / Agent（Cursor、VS Code、DevEco 等）。不重启则此前会话仍算人工。
3. 校验：`git ai --version`、`git ai whoami`、`which git` 指向 `~/.git-ai/bin/git`。

重装会删除 `~/.git-ai/config.json`。要保留自定义配置时加 `GIT_AI_SKIP_CONFIG_RESET=1`。不要用 root/sudo 安装；提权 daemon 会导致普通进程连不上。

## 身份与配置

两层身份，不要混用：

| 层 | 作用 | 命令 |
|---|---|---|
| Git 全局身份 | 钩子校验、网站登录邮箱、提交 Author | `git config --global user.name` / `user.email` |
| git-ai author 覆盖 | 只覆盖 git-ai 展示名和邮箱，可空 | `git ai config set author.name` / `author.email` |

公司邮箱是 Git 全局 `user.email`。未要求时不要改 git config。

查看与修改 git-ai 配置（写在 `~/.git-ai/config.json`）：

```bash
git ai config                         # 全部 JSON
git ai config <key>                   # 支持点号，如 author.email、notes_backend.kind
git ai config set <key> <value>
git ai config --add <key> <value>     # 数组追加，或对象字段 upsert
git ai config unset <key>             # 恢复默认
```

内部环境必须保持：

- `api_base_url` = `https://gitai.matrix.openharmony.cn`
- `notes_backend.backend_url` 同上，`notes_backend.kind` = `both`

不要改回上游 `https://usegitai.com`。常用键：`log_level`、`quiet`、`exclude_repositories`、`allow_repositories`、`prompt_storage`、`feature_flags`、`disable_auto_updates`。完整键表见 [reference.md](reference.md)。

仓库级排除示例：

```bash
git ai config set exclude_repositories .
git ai config --add exclude_repositories "private/*"
```

## 登录与授权码

`git ai whoami` 看状态。已登录时再执行 `git ai login` 会直接报 `Already logged in`，先 `git ai logout` 才能重登。

首次 `git ai login`：

1. 终端打印一次性 `user_code`（授权码，形如 `XXXX-XXXX`）。这是网站登录密码，只显示一次。忘记后找部门管理员重置。
2. 浏览器打开 `https://gitai.matrix.openharmony.cn/verify?code=...` 完成设备授权。
3. 看到 `Successfully logged in!` 后再 `git ai whoami`：`Login: logged in`，邮箱为公司邮箱。

再次登录时，网页授权要填这个授权码。网站 <https://gitai.matrix.openharmony.cn> 使用 **Git 提交邮箱 + 授权码** 登录。

未登录只影响上报：本机仍采集，存在 Local metrics database，登录后自动补传。不要把授权码、access token 写进仓库、skill 或对话摘要。

## 验证与查看产出

在 git 仓库里让 Agent 改文件并提交。`git log -n 1` 应出现统计行：

```text
AI[100%] Human Fixed[0%] Human[0%] AI Adopted[0%]
```

AI 占比 **大于 70%** 才追加 `Co-authored-by: <Agent> (<model>) <ai@local>`。没有这一行不代表没采集。

```bash
git ai stats                  # 当前提交的比例条
git ai stats --json
git ai stats <sha>
git ai stats <a>..<b> --json
git ai log
git ai blame <file>
git ai status                 # 未提交归属
git ai show <rev>
```

直接在 git 仓库目录里用 Agent 改代码。仓库外改完再拷贝进来，统计会是 0%。提交前不要用格式化或会改文件的 pre-commit hook 覆盖同一批 AI 改动，否则会记成人工。

## 升级与 hooks

```bash
git ai upgrade                # 在线升到 latest
git ai upgrade --force        # 已是最新也重装
git ai install-hooks --env    # 重装 hooks 并写 shell PATH
```

`git commit` / `git push` 发现新版本会提示升级。hooks 按本机已安装的 Agent 自动配置；TraeCode 还必须在 UI 里打开 Hooks，并设为本地自动运行。TraeWork 和 Trae CLI 不支持。

## 排障顺序

1. `git ai --version` 与 `git ai whoami`。
2. `which git` / `git --exec-path`：git 必须走 `~/.git-ai/bin`。
3. IDE 是否在装 hooks 之后重启。
4. `git ai bg status`；锁冲突时 `git ai bg shutdown` 后新开终端。提权 daemon 用管理员终端执行 `git ai bg shutdown --hard`，再在普通终端跑一次 `git status`。
5. 大提交被跳过时终端会写 `Skipped git-ai stats for large commit`，归属还在，用 `git ai stats <sha>` 补算。
6. cherry-pick 后归属偏低：`git ai fetch-notes --commits <源sha>`，再用 `git ai blame <file>`。

现象对照表在 [reference.md](reference.md)。指南更新后以 `/guide/README.md` 为准，不要凭本 skill 否定线上新条款。
