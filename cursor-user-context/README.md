# cursor-user-context

本目录集中管理 Cursor 的用户规则、记忆与偏好，解决两个问题：

1. **换账号迁移** — Cursor 换号后云端 Memories / User Rules 不会自动跟随，本目录提供本地备份与恢复流程。
2. **跨项目一致性** — 将规则和偏好纳入仓库，在任何设备 clone 后即可由项目规则自动加载。

## 文件索引

| 文件 | 内容 |
|------|------|
| [`用户上下文汇总.md`](用户上下文汇总.md) | **主文件**：换账号清单、记忆条目、个人偏好。 |
| [`USER_RULES.md`](USER_RULES.md) | User Rules 全文备份（Cursor 设置 → Rules → User Rules）。 |

## 相关项目规则

`.cursor/rules/user-context-from-repo.mdc`（`alwaysApply: true`）让助手在每次对话中自动参考本目录内容。

## 日常维护

- 在 Cursor 设置里修改了 User Rules → 同步更新 `USER_RULES.md`。
- 有新的重要事实或偏好 → 更新 `用户上下文汇总.md` 的记忆条目或偏好部分。
- **不要**在本目录写入密钥、Token 等敏感信息。
