# WitX Plugins

[WitX](https://github.com/FoLAWy-py/WitX) 的官方 marketplace。Plugin 是一组 Skill：某类任务的已验证做法，Agent 在合适时加载。Plugin 只有文字，不运行代码，也不授予任何权限：付款仍然要你批准，Skill 不能改变这一点。

## 安装

在运行 WitX Worker 的机器上：

```sh
witx plugin install basics            # 官方 marketplace（witx）里的 basics；先显示全文，确认后才装
witx plugin list                      # 已安装的 Plugin，以及每个是否被加载
witx plugin update basics             # 先显示改了什么，确认后才更新
witx plugin remove basics

witx plugin marketplace add https://github.com/someone/their-plugins.git
witx plugin install their-plugin@their-marketplace
```

Plugin 装在 `WITX_PLUGINS_DIR`（默认 `~/.witx/plugins`）里，锁定到当时的 commit 和文件的 SHA-256。任务运行时不会去拉新版本；文件被改动、和锁文件对不上的 Plugin 不会被加载。

## 写一个 Plugin

```
plugins/<plugin>/
  .witx-plugin/plugin.json        {"name", "version", "description"}
  skills/<skill>/SKILL.md
  README.md, LICENSE              可选
```

Plugin 里只能有这些文件。脚本、钩子、MCP 服务以及任何其他文件，都会被拒绝。

`SKILL.md` 的开头是 `---` 包起来的头部：

| 字段 | 含义 |
| --- | --- |
| `name` | 与目录同名，小写加短横线。Agent 看到的名字是 `<plugin>:<name>` |
| `description` | 一句话，最多 300 字符，Agent 靠它判断要不要加载 |
| `apps` | 可选，逗号分隔的包名 |
| `tools` | 正文里用到的 WitX 工具，如 `launch_app, type_text` |
| `effects` | 正文里提到的效果，如 `delete_data` |

正文写方法、注意事项、怎样算完成、出错时怎么恢复。不写固定坐标，也不写代码。

- 正文里用反引号写出的小写标识符，都必须在 `tools` 或 `effects` 里声明。
- 声明的工具和效果必须在当前的 WitX 里存在，否则这个 Plugin 不会被加载。所以 WitX 改了工具名，会马上发现哪些 Skill 需要跟着改。
- 改动任何文件都要提升 `plugin.json` 的 `version`。

提交前在 WitX 的仓库里运行：

```sh
uv run witx plugin check /path/to/witx-plugins
```

## 列入别人仓库里的 Plugin

`.witx-plugin/marketplace.json` 里的条目可以指向另一个仓库，但必须锁定到一个完整的 commit：

```json
{ "name": "their-plugin", "source": { "git": "https://github.com/someone/repo.git", "commit": "<40 位 SHA>", "path": "plugins/their-plugin" } }
```

更新这类 Plugin，就是提一个 PR 把 `commit` 改成新的 commit，评审时看两个 commit 之间的差异。

## CI

每个 PR 都会用 WitX 最新的 main 运行 `witx plugin check`。WitX 自己的 CI 也会对这个仓库的 main 运行同样的检查：WitX 改名或删除一个工具时，在合并之前就会知道哪些 Skill 会坏。
