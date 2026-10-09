# MCP 接入说明

本文档说明如何把**麦当劳中国 MCP 服务**接入到你使用的 AI 客户端，并让 `mcd-cn-assistant` 技能正常工作。

---

## 一、前置条件：申请 MCP Token

麦当劳中国 MCP 是**远程托管服务**，不需要本地安装任何进程，但需要一枚身份凭证（MCP Token）。

1. 打开麦当劳 MCP Server 开放平台：<https://open.mcd.cn/mcp>
2. 点击右上角 **【登录】**，使用**手机号 + 验证码**登录；
3. 登录成功后，右上角按钮变为 **【控制台】**，点击打开控制台弹窗；
4. 点击 **【激活】** 申请 MCP Token；
5. 阅读并 **同意服务协议**；
6. Token 申请成功后，点击 **一键复制** 保存。

> ⚠️ Token 是访问你麦当劳账户权益的凭证，等同于账号权限。请勿提交到公开仓库、勿分享给他人。本项目已在 `.gitignore` 中排除本地凭证文件。

---

## 二、服务基本信息

| 项目 | 值 |
|---|---|
| 接入地址 | `https://mcp.mcd.cn` |
| 传输协议 | **Streamable HTTP**（不支持 WebSocket） |
| 鉴权方式 | 请求头 `Authorization: Bearer <MCP_TOKEN>` |
| 支持版本 | MCP 协议 `2025-06-18` 及之前版本 |
| 工具数量 | 36 个（服务版本 1.0.9） |
| 限流 | 每 Token **600 次/分钟**，超限返回 `429` |
| 服务范围 | 中国大陆（不含港澳台） |

**通用配置示例**（绝大多数客户端通用）：

```json
{
  "mcpServers": {
    "mcd-mcp": {
      "type": "streamablehttp",
      "url": "https://mcp.mcd.cn",
      "headers": {
        "Authorization": "Bearer YOUR_MCP_TOKEN"
      }
    }
  }
}
```

把 `YOUR_MCP_TOKEN` 替换为第一节拿到的真实 Token 即可。

---

## 三、各客户端接入

### 3.1 WorkBuddy

1. 打开配置文件 `~/.workbuddy/mcp.json`（若不存在则新建），写入上面的通用配置；
2. **保存后不会自动生效**——进入**连接器管理页**，点击右上角的 **「自定义连接器」** 入口；
3. 在自定义连接器列表中找到新出现的 `mcd-mcp`，点击 **信任 / 启用**；
4. 启用后在对话中即可直接使用。

### 3.2 Cursor

1. 顶部菜单 **设置 → Tools & MCP**；
2. 在 Installed MCP Servers 中点击 **Add Custom MCP**；
3. 在打开的 `mcp.json` 中粘贴上面的通用配置，替换 Token；
4. 关闭并保存。回到设置页应看到麦当劳工具列表，服务状态显示 **已连接**；
5. 按 `Ctrl/Cmd + L` 打开右侧 Agent 对话框开始使用。

> 项目级配置放在 `.cursor/mcp.json`，全局配置放在 `~/.cursor/mcp.json`。

### 3.3 Cherry Studio

1. 打开设置 → 选择 **MCP** 选项卡；
2. 点击 **添加** → 在下拉框中选择 **从 JSON 导入**；
3. 粘贴上面的通用配置，**务必替换 `YOUR_MCP_TOKEN`**，点击确定；
4. 添加完成后打开该服务的 **启用开关**。

### 3.4 Trae

1. **设置 → MCP → 手动添加**；
2. 在手动配置页填入通用配置（替换 Token），点击 **确认**；
3. 回到 MCP 页面，服务状态应显示 **已连接**；
4. 回到对话框，选择 **Builder with MCP** 模式后使用。

### 3.5 VSCode（GitHub Copilot）

VSCode 使用 `servers` 字段（不是 `mcpServers`），且类型写 `http`。建议用 `inputs` 避免 Token 落盘明文：

```json
{
  "inputs": [
    {
      "type": "promptString",
      "id": "mcd-token",
      "description": "麦当劳 MCP Token",
      "password": true
    }
  ],
  "servers": {
    "mcd-mcp": {
      "type": "http",
      "url": "https://mcp.mcd.cn",
      "headers": {
        "Authorization": "Bearer ${input:mcd-token}"
      }
    }
  }
}
```

配置文件位置：项目内 `.vscode/mcp.json`，或全局用户配置。

### 3.6 Claude Code

```bash
claude mcp add --transport http mcd-mcp https://mcp.mcd.cn \
  --header "Authorization: Bearer YOUR_MCP_TOKEN"
```

添加后用 `claude mcp list` 确认连接状态。

> 各客户端字段名与菜单位置可能随版本变化，如与本文档不一致，请以对应客户端官方文档为准。

---

## 四、验证连通性

接入完成后，对 Agent 说：

> 查一下现在的时间，再查一下这个月麦当劳的活动日历

- 如果返回了当前时间和活动列表 → **接入成功**；
- 如果报 `401` → 见下节排查；
- 如果报 `429` → 触发了限流，降低调用频率后重试。

也可以直接检查工具是否可见：在客户端的 MCP 面板中，`mcd-mcp` 下应列出约 36 个工具（`campaign-calendar`、`query-meals`、`calculate-price` 等）。

---

## 五、错误码与排查

| 错误码 | 原因 | 处理建议 |
|---|---|---|
| `401` | MCP Token 无效、已过期或未提供 | 检查请求头是否为 `Authorization: Bearer <token>`；确认 Token 未失效；重新复制 Token 更新配置 |
| `429` | 触发限流（超过 600 次/分钟） | 降低请求频率，复用已获取的结果，避免同时发起大量查询 |
| 找不到工具 / 服务未连接 | 配置未生效或未被启用 | WorkBuddy 需在「自定义连接器」中点击信任；其他客户端需打开启用开关或重启客户端 |
| 客户端不支持 | 客户端要求 stdio 传输 | 需选择支持 **Streamable HTTP** 的客户端 |

**常见坑**

- 配置里 `YOUR_MCP_TOKEN` 忘了替换 → 必然 401；
- 复制 Token 时带了多余空格或引号 → 401；
- WorkBuddy 中只写了 `mcp.json` 但没去「自定义连接器」信任 → 工具不出现；
- 多个技能同时高频查询菜单/门店 → 容易触发 429。

---

## 六、安全建议

1. **不要把 Token 提交到 Git**。本仓库 `.gitignore` 已排除常见凭证文件；如果你的客户端配置写在仓库目录内，请务必确认它被忽略。
2. **优先使用客户端提供的密钥输入机制**（如 VSCode 的 `inputs`、Claude Code 的交互式添加），避免明文落盘。
3. **定期轮换 Token**：在开放平台控制台重新激活 / 更换。
4. **只从官方地址接入**：`https://mcp.mcd.cn`，不要相信任何第三方代理地址。
5. **写操作谨慎**：下单、领券、取消订单会真实产生费用或消耗权益，执行前务必核对明细。

---

## 七、参考链接

- 麦当劳 MCP Server 开放平台：<https://open.mcd.cn/mcp>
- 官方 MCP 接入文档：<https://github.com/M-China/mcd-mcp-server>
- 工具清单与调用顺序：[../references/mcd-tools.md](../references/mcd-tools.md)
