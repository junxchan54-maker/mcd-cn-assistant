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
| 开发时间 | **2026 年 10 月 9 日 15:06 起（10-09 持续开发，10-12 新增多人群点聚合能力）** |
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

- 使用 WorkBuddy 的可视化能力生成架构图与流程图（四大能力与 MCP 调用链路），并写入 README / MCP_INTEGRATION.md 的 mermaid 图。

---

## 三、对话上下文摘录

本项目在 WorkBuddy 中经过 **10 轮**连续对话完成，全程围绕同一项目推进：

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
| 10 | 提出高频缺口：多人聚餐群点（办公室下午茶 / 加班夜宵 / 宿舍 / 带娃局），要解决「算账难、下单乱、收钱烦」 | **新增能力四「多人群点聚合」**：SKILL.md 新增四阶段流程（开单→收单→算账→成单）与两条硬边界（严禁模拟登录他人账户、Skill 不代收款项）→ 新建 `references/group-order.md`（订单会话 JSON 结构、满减凑单判定式、券归属规则、配送费三种分摊算法、尾差兜底分摊总算法、账单模板）→ 同步更新 `README.md` / `README_EN.md` / `MCP_INTEGRATION.md` / `docs/examples.md`（能力四工具表、mermaid 调用流程、示例 9/10、边界刹车表扩至 8 项）→ 同步已安装技能副本 |
| 11 | 要求「把这个 skill 运行试下」，随后要求「修正和同步上去」 | **全链路实跑验收 + 按实测结果修正文档**。用真实 MCP 服务跑通四大能力：`now-time-info` → `campaign-calendar`（2026-10 全月活动）→ 关键词命中（麦当劳 X PEACEMINUSONE）→ `query-nearby-stores`（1420693）→ 盘券 11 张 → `query-store-coupons` → `query-meals` → `calculate-price`（券后 ¥19.00→¥9.90）→ 外送群点（门店 1421053，5 人，实付 ¥111.60）→ 分摊算账（Σ每人应付 = 实付，尾差兜底实测触发）。实跑发现并修正三处问题：① **券专享商品不带券算价返回 `price:0`/空 productList**（非报错、非下架）→ 写入 SKILL.md 要点与 FAQ；② **外送价 ≠ 到店价**（脆薯饼 ¥8.5/¥9.5 等）→ 写入能力三要点、阶段二、示例 6 与英文 README，明确禁止跨渠道复用价格；③ **打包费未纳入分摊定义**（`group-order.md` 只定义配送费，但外送单必有 `packingPrice`，会导致账单合计小于实付）→ 将 3.3/3.4 的 `F` 改为「公共费用 = 配送费 + 打包费」，`split_rules` 字段由 `delivery_fee` 改为 `public_fee`，账单列名同步为「券前小计 / 分摊券省 / 分摊公共费 / 应付」；④ 额外修正**小计口径易错点**（用券成员须用券前基准价，否则券省被重复扣减，差额恰等于券省额 ¥9.10）。README 能力四示例、`docs/examples.md` 示例 9 全部替换为**真实实测数据**。 |
| 12 | 提出「多人拼单需要用户自己提出来」是设计缺陷，要求**点餐前主动询问是否多人拼单** | **新增「点餐统一入口」交互层**：SKILL.md 新增第 3 节 *点餐入口：下单前必须主动询问「单人 / 多人拼单」*（红线 + 询问话术 + 4 种回答的路由表 + 免问例外 + 主动识别信号 + 3 条「忌」），并写入硬性规则第 6 条、摘要段与 frontmatter description；能力三第 1 步改为「几人份必须来自入口询问」，能力四触发改为「入口答复多人为主路径」并补一句不得退化为加购物车；FAQ 新增「没确认人数就按单人下单」条目。同步 `README.md`（解决什么问题表、1.3 四大能力加统一入口段、架构图新增入口节点、使用示例新增「入口 · 点餐前主动询问」、目标用户与痛点各加一条）、`README_EN.md`（能力表加 shared entry point 段 + Design principles 第 4 条）、`docs/examples.md`（新增 *示例 5：点餐时的入口询问*，原示例 5~10 顺移为 6~11，边界刹车表与小贴士各加一条）、`MCP_INTEGRATION.md`（新增 2.3 点餐统一入口，原 2.3~2.5 顺移为 2.4~2.6；3.3 流程图加 S0 入口节点；3.4 补入口说明；新增 5.4 交互入口规则，原 5.4 顺移为 5.5）、`references/group-order.md`（设计定位表新增「入口」行、成员话术表新增入口两行）→ 同步已安装技能副本 |
| 13 | 为提升浏览量与 Star 数，要求**在仓库顶部加可视化图**、**结尾加明确 CTA** | **新增视觉资产与转化路径**：新建 `assets/banner.svg`（1280×320 自绘动画 SVG：品牌深色渐变底 + 光晕装饰 + 主标题 + 四大能力 chip 依次点亮 + 右侧「群点会话 / 账单」双卡片示意 + 光带扫过；动画仅作点缀，静态帧同样完整）+ `assets/cta-star.svg`（1280×160 金色渐变 CTA 图 + 模拟 Star 按钮）。`README.md` / `README_EN.md` 顶部接入横幅并新增 `GitHub stars` 动态徽章，文末新增「如果这个技能有帮助，请点个 Star」CTA 段（图 + 三条理由（含「Star 数是唯一排名依据」）+ for-the-badge Star 按钮），仓库结构表与产出物清单同步登记 `assets/`。全部资产为手写 SVG，**未使用位图 / GIF**：矢量在任意缩放与高清屏下不糊、体积仅 ~8KB（GIF 同清晰度需数百 KB 且不可编辑）。⚠️ 过程教训：曾尝试用 Chrome 无头模式截图自检，Chrome 进程虽启动但渲染永不完成且不退出，叠加 stdout 管道导致命令永久阻塞 → **已确立规则：不启动任何浏览器 / GUI 进程，视觉校验改为确定性计算（XML 合法性 + 文本边界推算）** |

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
| `references/group-order.md` | 源代码 | 能力四：订单会话结构 + 分摊算法 + 账单模板 |
| `docs/mcp-setup.md` | 文档 | 多客户端 MCP 接入教程 |
| `docs/examples.md` | 文档 | 真实对话示例 |
| `README_EN.md` | 文档 | 英文版项目介绍 |
| `assets/banner.svg` | 视觉资产 | 顶部横幅（自绘动画 SVG，静态渲染同样完整） |
| `assets/cta-star.svg` | 视觉资产 | 结尾 Star CTA 图 |
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
