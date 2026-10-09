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
| 鉴权方式 | 请求头 `Authorization: Bearer <你的 MCP Token>` |
| 支持版本 | MCP 协议 `2025-06-18` 及之前版本 |
| 工具数量 | 35 个（2026-10-09 对线上 `tools/list` 实测核对） |
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
        "Authorization": "Bearer ${MCD_MCP_TOKEN}"
      }
    }
  }
}
```

> 🔑 **`${MCD_MCP_TOKEN}` 是占位符，不是可以直接用的值**——必须替换成**你自己的** MCP Token（第一节申请到的那个）。也可以不在配置里写死，改为读取同名环境变量 `MCD_MCP_TOKEN`。

### 占位符速查

本文档与整个仓库中出现的以下写法**全都是占位符，需要替换成使用者你自己的信息**。本仓库**不含任何真实凭证**（已做脱敏处理）。

| 占位符 | 出现在 | 要替换成什么 |
|---|---|---|
| `${MCD_MCP_TOKEN}` | 所有 JSON 配置示例、命令行示例 | **你自己的**麦当劳 MCP Token；或直接设置同名环境变量 `MCD_MCP_TOKEN` |
| `<你的 MCP Token>` | 文档正文的字段说明 | **你自己的**麦当劳 MCP Token |
| `${input:mcd-token}` | VSCode `.vscode/mcp.json` 示例 | **无需手改**——这是 VSCode 的交互式输入语法，首次启动会弹窗请你填写 |

---

## 三、各客户端接入

### 3.1 WorkBuddy（官方推荐流程）

> **前置条件**：需先申请到麦当劳中国的 MCP Token（见第一节）。
> **参考文档**：[WorkBuddy 官方文档 · 连接器](https://www.workbuddy.cn/docs/workbuddy/From-Beginner-to-Expert-Guide/Function-Description/Connector)

1. 打开 WorkBuddy，在左侧边栏【**专家·技能·连接器**】，选中【**连接器**】页签；
2. 点击右上角【**自定义连接器**】→【**配置MCP**】；
3. 在打开的手动配置页面中填入以下 JSON 内容：

   ```json
   {
     "mcpServers": {
       "mcd-mcp": {
         "type": "streamablehttp",
         "url": "https://mcp.mcd.cn",
         "headers": {
           "Authorization": "Bearer ${MCD_MCP_TOKEN}"
         }
       }
     }
   }
   ```

   > ⚠️ **一定记得把 `${MCD_MCP_TOKEN}` 替换为「你自己的」实际 MCP Token，点击【保存】！**

4. 回到【**自定义连接器**】，将 `mcd-mcp`【**启用**】。接下来即可在对话框中输入需求，让 AI 调用相应工具。

**等价方式（手动编辑配置文件）**

也可以直接编辑 `~/.workbuddy/mcp.json`，写入同样内容。注意：**保存文件不会自动生效**，仍需回到【自定义连接器】将 `mcd-mcp` 启用。

#### 3.1.1 占位符与脱敏说明

本仓库**已做脱敏处理，不含任何真实 Token**。所有出现以下写法的位置，都需要**换成使用者你自己的信息**：

| 使用场景 | 占位符写法 | 是否要替换成真实 Token |
|---|---|---|
| **你自己的客户端配置**（WorkBuddy / Cursor / Cherry Studio …） | `${MCD_MCP_TOKEN}` | ✅ **必须替换成你自己的实际 Token**，然后保存 |
| 文档正文的字段说明 | `<你的 MCP Token>` | ✅ 需要替换成你自己的实际 Token |
| **仓库内的脱敏示例** [`mcp-config.example.json`](../mcp-config.example.json) | `${MCD_MCP_TOKEN}` | ❌ **不要**把真实 Token 提交回仓库——赛事规则要求示例文件"**只允许环境变量占位符**" |
| VSCode `.vscode/mcp.json` | `${input:mcd-token}` | ❌ 无需手改，VSCode 会弹窗请你填写 |

一句话：**仓库里出现的全都是占位符，照抄配置时记得把 `${MCD_MCP_TOKEN}` 换成你自己的 Token；但不要把真实 Token 提交回仓库。**

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
3. 粘贴上面的通用配置，**务必把 `${MCD_MCP_TOKEN}` 换成你自己的 Token**，点击确定；
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
  --header "Authorization: Bearer ${MCD_MCP_TOKEN}"
```

添加后用 `claude mcp list` 确认连接状态。

> 各客户端字段名与菜单位置可能随版本变化，如与本文档不一致，请以对应客户端官方文档为准。

---

## 四、验证连通性

接入完成后，对 Agent 说：

> 查一下现在的时间，再查一下这个月麦当劳的活动日历

- 如果返回了当前时间和活动列表 → **接入成功**；
- 如果报 `401`，或报 **`403` 且响应体含 `校验鉴权authToken必填!`** → 见下节排查；
- 如果报 `429` → 触发了限流，降低调用频率后重试。

也可以直接检查工具是否可见：在客户端的 MCP 面板中，`mcd-mcp` 下应列出 35 个工具（`campaign-calendar`、`query-meals`、`calculate-price` 等）。

---

## 五、错误码与排查

| 错误码 | 原因 | 处理建议 |
|---|---|---|
| `401` | MCP Token 无效、已过期 | 确认 Token 未失效；重新复制 Token 更新配置 |
| **`403` + `校验鉴权authToken必填!`** | **`Authorization` 头的值缺少 `Bearer ` 前缀** | 值必须是 `Bearer <你的 MCP Token>`（注意 Bearer 后有**一个空格**）。只填 token 本身会被服务端判定为"未提供鉴权"——**这是本项目实际踩过的坑，见下方常见坑第 1 条** |
| `429` | 触发限流（超过 600 次/分钟） | 降低请求频率，复用已获取的结果，避免同时发起大量查询 |
| 找不到工具 / 服务未连接 | 配置未生效或未被启用 | WorkBuddy 需在「自定义连接器」中点击信任；其他客户端需打开启用开关或重启客户端 |
| 客户端不支持 | 客户端要求 stdio 传输 | 需选择支持 **Streamable HTTP** 的客户端 |

**常见坑**

1. **`Authorization` 只填了 Token，漏了 `Bearer ` 前缀** → 服务端返回 **`403`（不是 `401`）**，响应体为 `{"code":"400003","msg":"校验鉴权authToken必填!"}`。很多人只盯着 `401` 排查，反而会漏掉这一条。正确写法：
   ```json
   "headers": { "Authorization": "Bearer ${MCD_MCP_TOKEN}" }
   ```
   > 小技巧：改这个请求头**不会导致连接器掉信任**——WorkBuddy 的信任记录绑定的是服务 URL（`sha256(url)`），与 headers 无关。
2. 配置里的 `${MCD_MCP_TOKEN}` 忘了替换成自己的 Token → 401；
3. 复制 Token 时带了多余空格或引号 → 401；
4. WorkBuddy 中只写了 `mcp.json` 但没去「自定义连接器」信任 → 工具不出现；
5. 多个技能同时高频查询菜单/门店 → 容易触发 429。

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
