# WorkBuddy 开发上下文记录

> 本文件由 WorkBuddy 对话上下文导出，用于核验本项目符合**麦当劳程序员创意开发大赛 · WorkBuddy 专项奖励**的联动条件（真实使用腾讯 WorkBuddy 智能体开发）。

---

## 一、项目信息

| 项目 | 内容 |
|---|---|
| 项目名称 | mcd-cn-assistant（麦当劳中国 MCP 智能助手技能） |
| 项目类型 | AI Agent 技能包（Skill） |
| 开发工具 | **腾讯 WorkBuddy** |
| 开发时间 | 2026 年 10 月 9 日 15:06 — 15:25（GMT+8） |
| 开发会话 | 单会话连续开发（任务模式 → Agent 模式） |

---

## 二、使用 WorkBuddy 的真实过程

本项目**全部开发过程均在 WorkBuddy 中完成**，未使用其他 AI 编码工具。具体使用的 WorkBuddy 能力如下：

### 2.1 MCP 连接器（Connector）

- 通过 WorkBuddy 的连接器管理（`连接器` → `自定义连接器`）配置麦当劳官方 MCP 服务 `mcd-mcp`；
- 配置文件写入 `~/.workbuddy/mcp.json`：
  ```json
  {
    "mcpServers": {
      "mcd-mcp": {
        "type": "streamablehttp",
        "url": "https://mcp.mcd.cn",
        "headers": { "Authorization": "Bearer YOUR_MCP_TOKEN" }
      }
    }
  }
  ```
- 依据 WorkBuddy 官方文档与官方赛事文档中给出的 WorkBuddy 专用接入四步流程完成配置。

### 2.2 Skill 系统（技能）

- 使用 WorkBuddy 的**用户级技能目录** `~/.workbuddy/skills/` 创建并落地本项目；
- 依据 WorkBuddy 技能规范编写 `SKILL.md`（含 `name` / `description` / `description_en` / `metadata` frontmatter）；
- 技能可通过对话自然触发，也可被自动化任务显式加载。

### 2.3 自动化任务（Automation）

- 通过 WorkBuddy 自动化能力创建**每日 09:00** 定时任务「麦当劳活动关注每日推送」（scheduleType: recurring）；
- 该任务在 prompt 中显式指定加载 `mcd-cn-assistant` 技能，并被约束为**纯只读**（禁止调用任何写操作工具）。

### 2.4 文件与代码工具

- 使用 WorkBuddy 的文件读写能力创建仓库全部文件（`SKILL.md`、`README.md`、`MCP_INTEGRATION.md`、`references/`、`docs/`、`config/`）；
- 使用命令执行能力完成哈希校验（确认 `CONTEST_DECLARATION.md` 与官方文件 MD5 一致：`91a7b32ec28098b609ca3f215cdf7147`）。

### 2.5 联网检索与可视化

- 使用联网检索获取并核对官方赛事规则与 MCP 文档；
- 使用联网检索能力复核官方仓库内容，确认 `M-China/mcd-mcp-server` 仅包含 MCP 接入文档、赛事规则实际位于 `M-China/mcd-developer-innovation-challenge`；
- 使用 WorkBuddy 的可视化能力生成架构图（三大能力与 MCP 调用链路）。

---

## 三、对话上下文摘录

| 轮次 | 用户诉求 | WorkBuddy 完成的工作 |
|---|---|---|
| 1 | 配置 `M-China/mcd-mcp-server`，并基于其内容做一个含「最新活动推送 / 关键词关注推送 / 最优惠点餐」三大能力的 skill | 检索并解读官方 MCP 文档 → 写入 `~/.workbuddy/mcp.json` → 创建 `mcd-cn-assistant` 技能（`SKILL.md` + `references/mcd-tools.md` + `config/keywords.json`）→ 创建每日 09:00 定时推送自动化 |
| 2 | 准备发布到 GitHub 公开仓库，需要项目介绍、参赛声明、MCP 接入说明 | 搭建仓库骨架 → 编写 `README.md` / `README_EN.md` / `docs/mcp-setup.md` / `docs/examples.md`；核实赛事规则的实际来源 |
| 3 | 提供赛事仓库地址 `M-China/mcd-developer-innovation-challenge` | 拉取官方规则全文 → **原样落地 `CONTEST_DECLARATION.md`（逐字节校验通过）** → 新增 `MCP_INTEGRATION.md`、`mcp-config.example.json` → 回改 `README.md` 补齐赛事要求的四大要素（项目介绍 / 安装方法 / 使用示例 / 目标用户） |

---

## 四、产出物清单

| 文件 | 是否赛事必交 | 说明 |
|---|---|---|
| `README.md` | 必须 | 项目介绍、安装方法、使用示例、目标用户 |
| `CONTEST_DECLARATION.md` | 必须 | 官方参赛声明，逐字节原样复制，未做任何改动 |
| `MCP_INTEGRATION.md` | 必须 | 实际使用的 MCP Server、Tool、调用流程、业务价值 |
| `mcp-config.example.json` | 必须 | 脱敏配置，仅含环境变量占位符 `${MCD_MCP_TOKEN}` |
| `workbuddy.md` | 专项奖励 | 本文件 |
| `SKILL.md` | 源代码主体 | 技能主文件，Agent 运行时加载 |
| `config/keywords.json` | 源代码 | 关键词关注配置 |
| `references/mcd-tools.md` | 源代码 | 36 个 MCP 工具速查表 |
| `docs/mcp-setup.md` | 文档 | 多客户端 MCP 接入教程 |
| `docs/examples.md` | 文档 | 真实对话示例 |
| `README_EN.md` | 文档 | 英文版项目介绍 |
| `LICENSE` | 文档 | MIT |
| `.gitignore` | 工程 | 排除本地凭证，防止 Token 泄露 |

---

## 五、可核验点

1. **Skill 结构符合 WorkBuddy 规范**：`SKILL.md` 位于技能根目录，含标准 frontmatter；`config/` 与 `references/` 为技能相对路径资源。
2. **MCP 接入符合 WorkBuddy 官方流程**：配置写在 `~/.workbuddy/mcp.json`，并在「自定义连接器」中启用。
3. **自动化真实创建**：每日 09:00 定时任务，任务 prompt 中显式要求加载本技能。
4. **无凭证泄露**：全仓库正则扫描无真实 Token；示例配置仅使用 `${MCD_MCP_TOKEN}` 占位符。
5. **官方声明未被改动**：`CONTEST_DECLARATION.md` 的 MD5 与官方仓库文件一致。

---

## 六、声明

本项目由参赛者使用**腾讯 WorkBuddy** 独立开发完成。本文件如实记录开发过程与使用到的 WorkBuddy 能力，供赛事方核验 WorkBuddy 专项奖励的联动条件。
