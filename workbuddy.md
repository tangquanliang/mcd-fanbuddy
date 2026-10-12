# WorkBuddy 开发对话上下文（workbuddy.md）

> 本文件记录「麦麦饭搭子」使用腾讯 WorkBuddy 辅助开发的真实过程，用于参与麦当劳程序员创意开发大赛的 WorkBuddy 专项奖励评审。开发持续进行中，本文件随开发进度追加更新。

---

## 开发环境

- **开发工具**：腾讯 WorkBuddy（AI 智能体，workbuddy.cn）
- **开发者**：tangquanliang（GitHub 个人开发者）
- **开发方式**：与 WorkBuddy 对话式协作，由 WorkBuddy 完成市场调研、方案设计、Skill 编写、文档生成与代码校验，开发者做创意决策与验收

---

## 第 1 阶段：赛事调研（2026-10-11）

**开发者**：把 McDonald's China（M-China）的比赛规则和 MCP 能力摸清楚。

**WorkBuddy 执行**：
1. 在 GitHub 定位官方组织 `M-China`，找到赛事仓库 `mcd-developer-innovation-challenge` 与 `mcd-mcp-server`；
2. 通过 GitHub API 读取《活动规则》全文，提炼关键点：按 GitHub Star 数排名、需提交 README/CONTEST_DECLARATION/MCP_INTEGRATION/mcp-config.example.json/workbuddy.md 五个文件、10月25日23:59截止；
3. 调研 MCP 开放能力：活动日历查询、麦麦省券列表查询、麦麦省一键领券、我的优惠券查询、当前时间信息查询共 5 个工具；
4. 分析已有 445+ 份参赛申请，发现「省钱助手」「点餐助手」同质化严重，建议差异化定位。

## 第 2 阶段：创意决策（2026-10-11）

**WorkBuddy 提案**：给出「麦麦饭搭子 / 省钱精算师 / 羊毛日历」三个方案及差异化分析。

**开发者决策**：选定「麦麦饭搭子」——饭点场景切入，多工具编排做决策推荐而非信息罗列，配合 WorkBuddy 自动化能力做定时推送，形成「Skill + 自动化」的完整答卷。

## 第 3 阶段：项目骨架（2026-10-11）

**WorkBuddy 执行**：搭建仓库 `mcd-fanbuddy`，完成以下文件初稿：
- `SKILL.md`：麦麦饭搭子 Skill 本体（5 大能力：饭点推荐 / 活动速览 / 券包管家 / 一键薅券 / 饭点自动化）
- `README.md`：项目介绍、安装方法、使用示例、目标用户
- `CONTEST_DECLARATION.md`：原创性 / 合规性 / 敏感信息声明
- `MCP_INTEGRATION.md`：MCP Server、Tool 清单、多工具编排调用流程
- `mcp-config.example.json`：脱敏配置示例（仅 `YOUR_MCP_TOKEN` 占位符）

## 第 4 阶段：MCP 接入与真机联调（2026-10-11，已完成）

**开发者**：申请到 MCP Token 并发给我，让我接上去实测。

**WorkBuddy 执行**：
1. 用 Streamable HTTP（JSON-RPC 2.0，protocolVersion 2025-06-18）连上 `https://mcp.mcd.cn/mcp-servers/mcd-mcp`，握手成功（serverInfo: mcd-mcp 1.0.0）；
2. `tools/list` 发现**实际开放 35 个工具**（远超公开资料流传的 5 个），据此**升级方案**：新增"热量预算搭餐"（`list-nutrition-foods`）与"算真账"（`calculate-price`）两大差异化能力；
3. 实测并记录真实返回：
   - `now-time-info` → 2026-10-11 19:36:17（SUNDAY）
   - `campaign-calendar` → 超值 9.9 元早餐两件套（10/8–10/21）、韩式风味蘸酱上新、麦当劳 × PEACEMINUSONE 等
   - `query-my-coupons` → 2 张新人券（中杯美式 7 折 ¥13.3 / 中杯奶铁 7 折 ¥14.7，到店专用，有效期至 11/9）
   - `available-coupons` → 麦麦省可领券列表
   - `list-nutrition-foods` → 160 款餐品营养数据（成功解析全部 160 条）
4. 写脚本完成「饭点推荐」端到端真实运行（只调只读工具，未领券未下单），产出真实决策结果与营养对比表，存入 `docs/demo-真实输出.md` 并写入 README。

- [x] 麦当劳开放平台 MCP Token 申请（open.mcd.cn/mcp）
- [x] 连通性验证（initialize + tools/list 成功）
- [x] 关键 Tool 逐一实测（真实返回已记录）
- [x] 「饭点推荐」多工具编排实测（真实输出已归档）
- [x] 「热量预算搭餐」实测（600 kcal 内高蛋白 Top8 已产出）
- [ ] WorkBuddy 端配置 mcd-mcp 并在对话中实测 Skill
- [ ] WorkBuddy 自动化饭点推送实测

## 第 5 阶段：开源发布与报名（待开始）

- [ ] 推送至开发者 GitHub 公开仓库
- [ ] 在 M-China/mcd-developer-innovation-challenge 按模板提交参赛 Issue

---

*本文件由 WorkBuddy 在开发过程中持续更新。*
