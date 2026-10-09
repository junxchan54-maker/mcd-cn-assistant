# WorkBuddy 开发上下文记录

> 本文件为**麦当劳程序员创意开发大赛 · WorkBuddy 专项奖励**的核验材料，用于证明本项目**真实使用腾讯 WorkBuddy 智能体**完成开发。
> 赛事规则原文：*"workbuddy.md，使用 WorkBuddy 开发时的对话上下文，用于核验是否符合联动活动奖励条件，可自行导出"*。

---

## 一、项目信息

| 项目 | 内容 |
|---|---|
| 项目名称 | mcd-cn-assistant（麦当劳中国 MCP 智能助手技能） |
| 项目类型 | AI Agent 技能包（Skill） |
| 开发工具 | **腾讯 WorkBuddy**（全程使用，未使用其他 AI 编码工具） |
| 开发时间 | **2026 年 10 月 9 日 15:06 起（当日持续开发）** |
| 开发模式 | WorkBuddy 单会话连续开发（Agent 模式） |
| 仓库地址 | <https://github.com/junxchan54-maker/mcd-cn-assistant> |

---

## 二、使用 WorkBuddy 的真实过程

本项目**全部开发过程均在 WorkBuddy 中完成**，未使用其他 AI 编码工具。具体使用的 WorkBuddy 能力如下：

### 2.1 MCP 连接器（Connector）

- 通过 WorkBuddy 的**连接器管理**（`专家·技能·连接器` → `连接器` → 右上角 `自定义连接器`）配置麦当劳官方 MCP 服务 `mcd-mcp`；
- 配置文件写入 `~/.workbuddy/mcp.json`：
  ```json
  {
    "mcpServers": {
      "mcd-mcp": {
        "type": "streamablehttp",
        "url": "https://mcp.mcd.cn",
        "headers": { "Authorization": "Bearer ${MCD_MCP_TOKEN}" }
      }
    }
  }
  ```
- 依据 WorkBuddy 官方文档与官方赛事文档中给出的 WorkBuddy 专用接入四步流程完成配置，并在【自定义连接器】中信任启用。

### 2.2 Skill 系统（技能）

- 使用 WorkBuddy 的**用户级技能目录** `~/.workbuddy/skills/` 创建并落地本项目；
- 依据 WorkBuddy 技能规范编写 `SKILL.md`（含 `name` / `description` / `description_en` / `metadata` frontmatter）；
- 技能可通过对话自然触发，也可被自动化任务显式加载。

### 2.3 自动化任务（Automation）

- 通过 WorkBuddy 自动化能力创建**每日 09:00** 定时任务「麦当劳活动关注每日推送」（`scheduleType: recurring`）；
- 该任务在 prompt 中显式指定加载 `mcd-cn-assistant` 技能，并被约束为**纯只读**（禁止调用任何写操作工具）。

### 2.4 文件与命令工具

- 使用 WorkBuddy 的**文件读写能力**创建仓库全部文件（`SKILL.md`、`README.md`、`MCP_INTEGRATION.md`、`references/`、`docs/`、`config/`）；
- 使用 WorkBuddy 的**命令执行能力**完成：官方声明文件哈希校验（MD5 `91a7b32ec28098b609ca3f215cdf7147`）、Git 提交与推送、全仓库凭证脱敏扫描。

### 2.5 联网检索能力

- 使用联网检索获取并逐字核对官方赛事规则（`M-China/mcd-developer-innovation-challenge`）与官方 MCP 文档；
- 使用联网检索复核官方仓库内容，确认 `M-China/mcd-mcp-server` 仅包含 MCP 接入文档，赛事规则实际位于 `M-China/mcd-developer-innovation-challenge`；
- 使用联网检索比对 WorkBuddy / CodeBuddy 官方文档中的连接器与 MCP 配置说明。

### 2.6 可视化能力

- 使用 WorkBuddy 的可视化能力生成架构图与流程图（三大能力与 MCP 调用链路），并写入 README / MCP_INTEGRATION.md 的 mermaid 图。

---

## 三、对话上下文摘录

本项目在 WorkBuddy 中经过 **9 轮**连续对话完成，全程围绕同一项目推进：

| 轮次 | 用户诉求 | WorkBuddy 完成的工作 |
|---|---|---|
| 1 | 配置 `M-China/mcd-mcp-server`，并基于其内容做一个含「最新活动推送 / 关键词关注推送 / 最优惠点餐」三大能力的 Skill | 检索并解读官方 MCP 文档 → 写入 `~/.workbuddy/mcp.json` → 创建 `mcd-cn-assistant` 技能（`SKILL.md` + `references/mcd-tools.md` + `config/keywords.json`）→ 创建每日 09:00 定时推送自动化 |
| 2 | 准备发布到 GitHub 公开仓库，需要项目介绍、参赛声明、MCP 接入说明 | 搭建仓库骨架 → 编写 `README.md` / `README_EN.md` / `docs/mcp-setup.md` / `docs/examples.md`；核实赛事规则的实际来源（确认不在 `mcd-mcp-server` 仓库中） |
| 3 | 提供赛事仓库地址 `M-China/mcd-developer-innovation-challenge` | 拉取官方规则全文 → **原样落地 `CONTEST_DECLARATION.md`（逐字节校验通过）** → 新增 `MCP_INTEGRATION.md`、`mcp-config.example.json` → 回改 `README.md` 补齐赛事要求的四大要素（项目介绍 / 安装方法 / 使用示例 / 目标用户） |
| 4 | 提供 GitHub 用户名与邮箱 | 配置 Git 身份 → 创建首次提交（14 个文件 / 1402 行） |
| 5 | 复查规则，指出官方 WorkBuddy 接入步骤与「替换 Token 并保存」提示缺失 | 逐字复读规则 → 修正 `docs/mcp-setup.md` §3.1 为官方四步流程 → `README.md` 的「安装方法」同步官方流程并加 ⚠️ 替换 Token 提示 |
| 6 | 远程仓库已创建 | 排查推送授权（GitHub PAT 权限最小化定位）→ 成功推送 `main` 分支 → 建立分支跟踪 → 远程一致性核验 |
| 7 | 新添加自定义连接器 `mcd-mcp` | 连接器排障：定位并修复 `Authorization` 缺失 `Bearer ` 前缀导致的 `403` → 实测 `tools/list` 拉取 **35 个工具** → 据此**校准文档中的工具清单**（总数 36→35、修正 2 个工具名连字符错误、补录 2 个遗漏工具） |
| 8 | 要求上传前对 Token 与私人信息脱敏 | 全仓库 + Git 历史全量扫描（确认无真实凭证入库）→ 统一占位符写法（6 种 → 2 种）→ 新增「占位符速查」表 |
| 9 | 重新检查规则，核对改动是否符合「开发 Skill」要求 | 逐条比对官方必交文件清单与内容约束 → 全量合规自检（见第五节）→ 补全本文件的完整开发记录 |

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
| `references/mcd-tools.md` | 源代码 | 35 个 MCP 工具速查表 |
| `docs/mcp-setup.md` | 文档 | 多客户端 MCP 接入教程 |
| `docs/examples.md` | 文档 | 真实对话示例 |
| `README_EN.md` | 文档 | 英文版项目介绍 |
| `LICENSE` / `.gitignore` / `.gitattributes` | 工程 | MIT 许可、凭证排除、行尾锁定 |

---

## 五、可核验点

1. **Skill 结构符合 WorkBuddy 规范**：`SKILL.md` 位于技能根目录，含标准 frontmatter；`config/` 与 `references/` 为技能相对路径资源。
2. **MCP 接入符合 WorkBuddy 官方流程**：配置写在 `~/.workbuddy/mcp.json`，并在「自定义连接器」中信任启用。
3. **MCP 服务真实可用**：对 `https://mcp.mcd.cn` 实测 `tools/list` 返回 35 个工具（`campaign-calendar`、`query-meals`、`calculate-price` 等）。
4. **自动化真实创建**：每日 09:00 定时任务，任务 prompt 中显式要求加载本技能，并限定为纯只读。
5. **官方声明未被改动**：`CONTEST_DECLARATION.md` 的 MD5 与官方仓库文件一致（`91a7b32ec28098b609ca3f215cdf7147`）。
6. **无凭证泄露**：全仓库正则扫描无真实 Token / 邮箱 / 手机号；示例配置仅使用 `${MCD_MCP_TOKEN}` 占位符。

---

## 六、关于本文件的导出说明

- 本文件为上述 WorkBuddy 开发过程的**结构化记录**（含使用的 WorkBuddy 能力、逐轮对话摘要、产出物与可核验点）。
- WorkBuddy 支持**导出完整对话**：在对话页面右上角菜单选择「导出」（支持 TXT / Markdown / PDF），或在设置中选择「导出全部数据」。
- 如需更完整的原始记录用于核验，可直接使用 WorkBuddy 的对话导出功能，将导出的原始对话一并附上（本文件可与原始导出互补使用）。

---

## 七、声明

本项目由参赛者使用**腾讯 WorkBuddy** 独立开发完成。本文件如实记录开发过程与使用到的 WorkBuddy 能力，供赛事方核验 WorkBuddy 专项奖励的联动条件。
