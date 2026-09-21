# Issue #152 验收记录

记录日期：2026-09-21。目标 Plugin：`0.1.1`；系统后端基线：`order_tracking` `main` `b1ee254b894c322689f0a2446b771ea1eb15ce7a`（生产 v1.0.18）；本机 Codex CLI：`0.149.1`。MCP 工具 schema 以该后端提交为准。生产业务数据、真实通知和共享配置均未改动。

| 验收项 | 当前证据 | 状态 |
|---|---|---|
| 五项 MCP 前置 | #151/#153/#154/#155/#156 已合并到上述 `main`；服务端测试结果见各 Issue 实施记录 | 服务端代码已合并 |
| Plugin 结构与 Skill | plugin-creator `validate_plugin.py`、skill-creator `quick_validate.py`、JSON 解析与 `git diff --check` 通过 | 本地通过 |
| 本机 Codex 安装 | `codex plugin add` 已加载 `0.1.1`；`codex mcp list` 可见远程 HTTP 服务。Plugin 声明公开 `oauth.clientId=kocotree-order-tracking-codex`，格式按 Codex 官方 MCP 文档核对 | 本机加载通过；生产 404 导致 OAuth 调用未验收 |
| 无登录生产入口 | `POST /mcp` 与 OAuth 两项元数据在 2026-09-21 返回应用 JSON 404。SSH 只读检查确认 v1.0.18 API 和前端容器健康、Nginx 有代理规则，但生产受保护配置及 API 容器均无 `ORDER_TRACKING_MCP_PUBLIC_URL`、`ORDER_TRACKING_MCP_CLIENT_ID`、`ORDER_TRACKING_MCP_FILE_HOSTS`；`/health/ready` 的 200 是前端 HTML | 已定位 MCP 未启用，阻塞真实连接 |
| 本人 OAuth 与 30 天共同授权 | #151 曾在隔离环境以 CLI 0.149.1 验证登录；本 Plugin 尚未完成真实连接 | 待验收 |
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

## 生产 MCP 启用待确认变更

只读证据：GitHub v1.0.18 发布工作流成功，生产 API 与前端镜像均标记为 `b1ee254b894c322689f0a2446b771ea1eb15ce7a` 且容器健康。前端运行中 Nginx 已转发 `/mcp` 和 OAuth 路径。生产受保护文件 `DEPLOY_DIR/deploy/.env.production` 权限为 600，当前文件及 API 容器均无三项 `ORDER_TRACKING_MCP_*` 配置。后端 `app/main.py` 仅在 `mcp_public_url` 非空时注册 MCP 和 OAuth，故该配置缺失可解释当前应用 JSON 404。

经生产配置变更授权后，在受保护环境文件中只增加以下两项，保留原有键值及权限；OAuth ID 与本 Plugin 的 `oauth.clientId` 精确相同：

```dotenv
ORDER_TRACKING_MCP_PUBLIC_URL=https://order-tracking.kktree.cn/mcp
ORDER_TRACKING_MCP_CLIENT_ID=kocotree-order-tracking-codex
```

`ORDER_TRACKING_MCP_FILE_HOSTS` 目前仍须为空：Codex 原生附件的实际下载主机尚未观测到，服务端会拒绝上传。得到真实附件主机并验证来源后，另按精确主机名加入该配置，再验收文件往返；不能用通配符或猜测主机。生产环境的完整业务验收在此之前仍为待完成。

保存配置后，以当前发布目录 `DEPLOY_DIR/releases/b1ee254b894c322689f0a2446b771ea1eb15ce7a/deploy` 执行 `docker compose --env-file .env.production -f compose.production.yaml config --quiet`，再以同一配置执行 `docker compose --env-file .env.production -f compose.production.yaml up -d --no-deps --no-build --wait --wait-timeout 180 api`。这会重建 API 容器，Web 与小程序的 API 请求可能短暂中断；worker 与前端容器不需重建。未获生产配置和重建授权前不执行。

只读回验：API 容器健康；`GET /.well-known/oauth-protected-resource/mcp` 与 `GET /.well-known/oauth-authorization-server` 返回 200 且 resource/issuer 与生产地址一致；未授权 `POST /mcp` 返回 401 与 OAuth 挑战；再用 Codex 本人登录查询 `get_me`。不把状态码检查当作本人授权、附件或业务验收。
