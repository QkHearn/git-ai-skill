# Git AI 参考

指南原文：`https://gitai.matrix.openharmony.cn/guide/README.md`（先登录，或由已登录客户端换 guide session 后再取）。下面是本机 `git-ai` help / config 与指南 FAQ 的压缩对照。

## 命令

`git ai <command>` 与 `git-ai <command>` 相同。常用：

| 命令 | 作用 |
|---|---|
| `checkpoint` | 手动记一笔归属。预设含 cursor、opencode、deveco、codearts、codemate、human 等 |
| `log` | 带 AI 统计的 log。`--raw` / `--notes` 看原始 note |
| `blame <file>` | 带 AI 归属的 blame |
| `diff <commit\|a..b>` | 带归属的 diff。`--json`、`--include-stats`、`--all-prompts` |
| `stats [commit\|range]` | 比例。`--json` |
| `usage --period 1d\|3d\|7d\|30d` | 本机用量，默认 30d |
| `status` | 未提交归属。`--json`、`--diff-only` |
| `show <rev\|range>` | authorship log |
| `show-prompt <id>` | 按 prompt id 查。`--commit` 或 `--offset` |
| `config` | 见下节 |
| `debug` | 支持诊断 |
| `bg` | 后台服务。`status` / `shutdown` / `shutdown --hard` / `restart` |
| `install-hooks` | `--env` 同时改 shell PATH；`--skills` 装 agent skill |
| `uninstall-hooks` | 卸掉全部工具 hooks |
| `upgrade [--force]` | 在线升级 |
| `fetch-notes [remote]` | 拉归属 notes。`--commits`、`--remote`、`--json` |
| `login` / `logout` / `whoami` | 登录态 |
| `git-path` | 底层真实 git 路径 |

没有单独的 `config --help` 子命令；`git ai config` 无参数即打印帮助式 JSON，带未知参数时打印键说明。`install-hooks --help` 会真的去装 hooks，不要用它探帮助。

## 配置键

`git ai config set <key> <value>`。数组用 `set` 会变成单元素，追加用 `--add`。

| 键 | 含义 |
|---|---|
| `git_path` | 底层 git |
| `log_level` | trace/info/warn/error/off |
| `api_base_url` | 内部固定 `https://gitai.matrix.openharmony.cn` |
| `api_key` | `X-API-Key`。蓝区正常走 login，不配 key |
| `author.name` / `author.email` | 覆盖 git-ai 作者展示，不替代 `git config user.*` |
| `allow_repositories` / `exclude_repositories` | 仓库白/黑名单 |
| `exclude_prompts_in_repositories` / `include_prompts_in_repositories` | prompt 存储范围 |
| `prompt_storage` | default/notes/local |
| `default_prompt_storage` | 未列入 include 的仓库的回退模式 |
| `notes_backend.kind` | git_notes/http/both。内部用 both |
| `notes_backend.backend_url` | 默认同 api_base_url。路径前缀会拼到 `/worker/notes/...` |
| `quiet` | 提交后不打印比例条 |
| `telemetry_oss` | on/off |
| `disable_version_checks` / `disable_auto_updates` | 关版本检查 / 自动更新 |
| `update_channel` | latest/next |
| `feature_flags` | 对象。`--add feature_flags.<name> true` |
| `allow_superuser` | 允许 root。日常保持 false |
| `custom_attributes` | 遥测 string→string |
| `git_ai_hooks` | hook 名 → shell 命令 |
| `codex_hooks_format` | config_toml/hooks_json |
| `untraced_fixup_ignored_paths` | untraced commit fixup 跳过的 glob |
| `transcript_streaming_lookback_days` | 扫 transcript 的回看天数，0 为不限 |
| `max_checkpoint_file_size_bytes` | 单文件 checkpoint 上限 |
| `max_checkpoint_total_size_bytes` / `max_checkpoint_total_lines` | 单次 checkpoint 总量上限 |
| `daemon_memory_limit_mb` / `daemon_startup_timeout_secs` | daemon 内存与启动等待 |

仓库模式可以是 glob（`https://github.com/org/*`）、git URL，或路径（`.` 解析成当前仓库 remote）。

## 统计行

提交说明里的一行：

```text
AI[n%] Human Fixed[n%] Human[n%] AI Adopted[n%]
```

`Co-authored-by` 只在 AI 占比 > 70% 时出现。

内部大提交跳过自动统计的阈值（达到任一即跳过，归属 note 仍在）：

| 指标 | 内部阈值 |
|---|---|
| hunk | ≥ 3000 |
| 新增行 | ≥ 18000 |
| 有新增的文件数 | ≥ 600 |
| 删除行 | ≥ 18000 |

提示文案：`Skipped git-ai stats for large commit`。补算：`git ai stats <sha>`。

## 支持的 Agent

安装脚本只配置本机已安装的工具。

通用：Claude Code、Codex、Cursor、VS Code Chat Hook、GitHub Copilot、OpenCode、Windsurf、Gemini CLI、Cline、Amp、Droid、Pi、Firebender、JetBrains 插件。Visual Studio 暂不支持。

内部额外：OpenCode v2、TraeCode（UI 手动开 Hooks；TraeWork / Trae CLI 不行）、ZCode、CodeArts、OpenClaw、WorkBuddy（5.30.1+）、DeepSeek Harness `dsh`（5.30.2 之后）、CodeAgentGUI（原 CodeMate）、CodeAgentOc、TestAgent（CLI 已适配）、MatrixAssistant、CodeAgentCLI、CodeGenie、DevEco（Studio 与 CLI）、灵犀 Chrys（5.29+）。

平台：Windows x64、macOS x64/arm64、Linux x64/arm64、HarmonyOS arm64。鸿蒙 PC 需先装 GitNext。

## FAQ 对照

| 现象 | 处理 |
|---|---|
| 钩子失败：`请使用公司邮箱注册` | `git config --global user.email` 改为公司邮箱后重跑安装脚本 |
| 找不到 `git ai` | 新开终端，或 `source ~/.zshrc` / `~/.bashrc` |
| Windows `Git\cmd` 排在 git-ai 前 | 把 `%USERPROFILE%\.git-ai\bin` 放到系统 PATH 最前，重开终端 |
| Git Bash 里 git-ai 不生效 | `~/.bashrc` 加 `export PATH="$HOME/.git-ai/bin:$PATH"` |
| Git Bash 跑 install.sh 报不支持的操作系统 | Windows 改用 install.ps1 |
| 钩子退出码 -1073741515 或缺少 VCRUNTIME140.dll | 装 https://aka.ms/vc14/vc_redist.x64.exe |
| VS Code 扩展安装失败 | 关掉 VS Code 后重跑安装脚本 |
| `daemon startup blocked: lock held` | `git ai bg shutdown`，新开终端 |
| 普通进程连不上提权 daemon / os error 5 | 管理员终端 `git ai bg shutdown --hard`，普通终端再跑 `git status` |
| 装完仍无统计 | 彻底退出 IDE 和 Agent 进程后重开。Cursor 必须重启 |
| AI 显示 0% 且刚格式化过 | 格式化或改文件的 hook 覆盖了 AI 归属。不要在 AI 改动后再人工格式化 |
| 仓库外编写再拷贝进仓库 | 在仓库目录内让 Agent 直接改 |
| 无统计行 | PATH 未把 git-ai shim 放在最前。升级到 5.29.1408+ 并重跑安装 |
| git < 2.27 | 升级 git 后重装 |
| cherry-pick 后 AI 偏低 | `git ai fetch-notes --commits <源sha>` |

已在新版本修复、只需 `git ai upgrade` 的旧问题：无 stat bar（5.29.1408+）、git 过低静默失败（2.18.936+ 起安装前检查）、Windows CRLF 导致 0%（5.14.1201+）、cherry-pick 归属提醒偶发丢失（5.30.1+）。
