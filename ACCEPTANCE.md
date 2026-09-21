# Issue #152 验收记录

记录日期：2026-09-21。目标 Plugin：`0.1.1`；系统后端基线：`order_tracking` `main` `b1ee254b894c322689f0a2446b771ea1eb15ce7a`（生产 v1.0.18）；本机 Codex CLI：`0.149.1`。MCP 工具 schema 以该后端提交为准。本次 Plugin 验证未改动生产业务数据、真实通知或生产配置。

| 验收项 | 当前证据 | 状态 |
|---|---|---|
| 五项 MCP 前置 | #151/#153/#154/#155/#156 已合并到上述 `main`；服务端测试结果见各 Issue 实施记录 | 服务端代码已合并 |
| Plugin 结构与 Skill | plugin-creator `validate_plugin.py`、skill-creator `quick_validate.py`、JSON 解析与 `git diff --check` 通过 | 本地通过 |
| 本机 Codex 安装 | `codex plugin add` 已加载 `0.1.1`；`codex mcp list` 可见远程 HTTP 服务，认证状态为 OAuth。`codex mcp login order-tracking` 成功；此前已验证 Plugin 声明的 `client_id=kocotree-order-tracking-codex` 和正确的 resource 会传入登录请求 | 安装与本人 OAuth 登录通过 |
| 无登录生产入口 | 主任务反馈用户已完成生产 MCP 配置。外部回验：受保护资源元数据和授权服务器元数据均返回 200，resource 与 issuer 指向生产地址；未授权 `POST /mcp initialize` 返回 401。此前的 404 已解除；本任务未复查服务器配置文件 | 生产 MCP 入口通过 |
| 本人 OAuth 与 30 天共同授权 | 本机 Codex CLI 0.149.1 登录后，仅调用一次 `mcp__order_tracking__get_me`，成功返回角色 `admin`；响应未提供可独立确认账号活跃的字段。跨 Web/Plugin 的 30 天续期、统一退出和停用账号拒绝尚未实测 | 本人连接与只读工具通过；共享会话规则待验收 |
| 订单 456#、来源刷新、派工与合同 | 后端工具已合并；隔离 Codex 对话与业务回读 | 待验收 |
| 发货统计、逐箱收货、退回与清单 | 后端工具已合并；隔离 Codex 对话与 Web 对照 | 待验收 |
| `.xlsx` 本地引用、拖入、上传、预览、确认、下载 | 后端文件工具已合并；Codex 原生 `files` 注入和往返字节未验证 | 待验收 |
| 工厂、产品、人员、通知逐动作 | 后端工具已合并；隔离 Codex 对话、角色差异与 Web 对照 | 待验收 |
| 干净客户端安装与更新 | 需无系统源码、无作者本机配置的同事账号验证 | 待验收 |
| 工厂可见、通知送达与实际查看 | 需分别取得工厂端及通知渠道证据 | 待验收 |

## 完成验收所需实测

1. 在隔离测试环境部署与本后端提交匹配的 HTTPS `/mcp`、OAuth 元数据及文件通路；服务端配置 `ORDER_TRACKING_MCP_PUBLIC_URL`、与 Plugin `oauth.clientId` 一致的预注册公共客户端 ID、精确文件来源主机。先检查 `initialize`、`tools/list`、本人授权、统一退出和停用账号拒绝。测试包须使用单独的 Plugin 名称与测试地址。
2. 用无系统源码的干净 Codex 客户端安装；记录 Codex 版本、Plugin 版本、服务端提交与 MCP 工具目录。以普通管理员和最高管理员身份逐项核对 Web 权限及状态；工厂账号与停用账号应被拒绝。
3. 在隔离数据中执行 456#、发货、返修及资料人员各场景；保留目标 ID、预检差异、工具名、回读结果和 Web 对照。覆盖歧义、覆盖、重复提交、断线恢复、整批失败与显式部分执行。真实通知须单独授权。
4. 对同一标准 `.xlsx` 分别用本地引用和拖入附件完成上传、预览、确认与原件下载；记录输入文件与下载文件的字节大小、SHA-256、Codex 版本和工具输入形状。不得记录临时链接或凭据。如果客户端无法生成 `files` 参数，修订文件方案并复验后才能称完整交付。
5. 核对全部 Web 动作的工具覆盖和实际结果，分别确认保存、业务确认、工厂可见、通知送达与查看。完成前不得把本地结构校验或服务端测试写成业务验收通过。

## 生产 MCP 入口回验

主任务反馈用户已完成生产 MCP 配置。本任务于 2026-09-21 从外部验证：`GET /.well-known/oauth-protected-resource/mcp` 返回 200，resource 为 `https://order-tracking.kktree.cn/mcp`；`GET /.well-known/oauth-authorization-server` 返回 200，issuer 和授权、token、撤销端点均指向该域名；未授权 `POST /mcp initialize` 返回 401。随后本机 Codex 完成 OAuth 登录，并以本人身份成功调用一次只读 `get_me`。本任务没有读取或修改生产配置，也没有执行生产业务写入。

Codex 原生附件的实际下载主机尚未观测到，`ORDER_TRACKING_MCP_FILE_HOSTS` 的生效值未核验，上传与文件往返均待验收。须验证真实来源主机后按精确主机名配置，再验证同一 `.xlsx` 的上传、预览、确认与原件下载；生产业务写入和真实通知另需明确授权。
