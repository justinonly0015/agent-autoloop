# 本 fork 的本地约定

> 这个文件只属于本 fork，不提交给上游，也不随上游更新。上游没有它。
> 上游：https://github.com/KKKKhazix/agent-autoloop

## 定位

只做两件事：**同步上游**、**加自己的改动**。不往上提交 PR。

## 分支

| 分支 | 用途 | 规则 |
| --- | --- | --- |
| `main` | 镜像上游 | 只允许 `git merge --ff-only upstream/main`；不放自己的提交，否则快进会失败 |
| `local` | 日常定制 | 从 `main` 切出；上游更新后把 `main` 合进来 |

日常检出 `local`。它的 HEAD 就是安装时被克隆的那一份。

一次性的、想单独试的改动，从 `local` 再开 `feature/<名字>`，验完合回 `local` 并删掉。

## 同步上游

```bash
git fetch upstream --tags
git checkout main
git merge --ff-only upstream/main
git push origin main --tags
git checkout local
git merge main
```

`--tags` 只在第一次需要：fork 不复制 tag，普通 `fetch` 只自动跟随（auto-follow）指向新提交的 tag，补不齐历史。补齐之后普通 `git fetch upstream` 就够。

## 环境

| 工具 | 版本 | 说明 |
| --- | --- | --- |
| Node | 24.9.0 | 项目 `engines.node >= 24` |
| Git | 2.55.0.windows.5 | `plugins/delivery/review.ts` 的 `--diff-merges=remerge` 要 ≥ 2.36；**不要用 2.36**，16 路并发跑测试会卡死 |
| codex | 0.169.0 | 当工人；装在 `D:\nodejs\node_global` |

## 安装与使用：两者兼顾

用本地仓库装，安装器克隆的是当前 HEAD 而不是 Release，所以改完代码 `update` 就能同步过去。

```powershell
$env:AGENT_AUTOLOOP_REPO = "D:\justin\Agent-AutoLoop"
cd D:\justin\Agent-AutoLoop
.\install.ps1 --no-service
```

- 日常用 `agent-autoloop` 命令，程序在 `~/.agent-autoloop/app`，数据在 `~/.agent-autoloop`
- 改代码改 `D:\justin\Agent-AutoLoop`，提交后 `agent-autoloop update` 就同步过去
- 排查时 `agent-autoloop why <轮次>`、`status --json` 都在，日志和事实日志都能查

要点：

- 安装器是 `--depth 1` 克隆**当前检出的分支**，所以装之前确认在 `local` 上。
- 它克隆的是 HEAD，**未提交的改动不会进安装副本**。要生效先提交。
- 去掉 `--no-service` 会注册 Windows 登录任务，登录后后台常驻。第一次先别加，确认链路通了再开。

## 跑测试

只用 `npm run test:windows`。它按 `scripts/test-windows.ts` 的排除集跳过 27 个 POSIX-only 文件（含 `desktop`）。

不要自己拼文件列表全量跑：`npm run test:quick` 的排除项只有 `simulation`，在 Windows 上会跑到只适配 POSIX 的用例，其中 `tests/desktop.test.ts` 会真的弹系统通知。

本机基线：549 通过 / 1 失败 / 3 跳过。唯一失败是 `windows-admin.test.ts:75`，需要「管理员 + 已登录桌面会话」，测试自带 `skip: CI`，本机过不了，属正常。
