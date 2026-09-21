# 跟单管理系统 Codex Plugin

Codex 管理员对话入口。Plugin 只包含远程 MCP 地址和业务操作说明；业务工具、权限、数据和文件均由跟单管理系统现有服务提供。业务上传仅支持 `.xlsx`。

## 安装与更新

本仓库公开可读。使用已支持 Plugin 的 Codex 客户端执行：

```sh
codex plugin marketplace add kocotree/order-tracking-plugin --ref main
codex plugin add order-tracking@order-tracking
```

安装后开启新的 Codex 任务，在插件目录确认 **跟单管理系统** 已启用，再请求“用跟单管理系统查询我的账号”。首次调用会打开浏览器；浏览器已有有效管理员网页登录时复用该登录，否则完成飞书登录后返回 Codex。每位管理员须用本人身份连接。不要分享账号、Cookie、token 或授权回调地址。

更新时执行 `codex plugin marketplace upgrade order-tracking`，再执行 `codex plugin add order-tracking@order-tracking` 并开启新任务。插件版本见 `plugins/order-tracking/.codex-plugin/plugin.json`；更新 Plugin 不会部署服务端。

## 会话与退出

Web 与 Plugin 共用 30 天活动期限。任一端有效业务使用会续期；两端均无有效活动满 30 天、账号停用或统一退出后，均需本人重新登录。令牌自动刷新本身不续期。对话中调用 `logout_shared_session` 会同时退出关联的 Web 与 Plugin 会话；Web 主动退出也具有同样效果。若 Codex 凭据丢失，重新调用工具并按浏览器授权流程连接。

## 文件与故障定位

返修上传使用 `upload_repair_workbook` 的原生文件参数 `files`；每次最多 20 份，每份最多 20 MiB，必须是有效 `.xlsx`。合同、清单和返修原件下载使用工具返回的 `downloadUrl`，链接要求本人 Web 登录。下载后核对 `sizeBytes` 和 `sha256`。Codex 原生附件注入的兼容性仍须按 `ACCEPTANCE.md` 实测；上传失败时记录客户端版本、工具名、错误与输入形状，勿记录附件临时 URL。

若工具无法出现，先检查插件是否启用、是否已开启新任务、`codex plugin list` 和 `codex plugin marketplace list`。若返回 401，完成浏览器登录；若返回 403，核对本人管理员账号状态；409 表示对象版本或状态已变化，回读后重新确认。连接失败时只读检查 `/mcp` 与 OAuth 元数据是否由目标环境提供。生产连接固定为 `https://order-tracking.kktree.cn/mcp`；测试环境必须使用独立 Plugin 名称、地址与账号。

业务动作遵循 `plugins/order-tracking/skills/order-tracking/SKILL.md`。实施与验收状态见 `ACCEPTANCE.md`。本仓库不含系统源码、服务器配置、内部业务数据或凭据。
