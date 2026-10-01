---
name: digitalme
description: >-
  把讲稿/文稿/大纲生成为 AI 数字人录播课视频（数字人讲解 + 声音克隆配音 + 自动课件 + 成片合成，全程 3 个人工审阅点）。
  当用户要求"把这份文稿/课程做成视频""生成数字人课程/录播课""做一节视频课""讲稿转视频"，或需要查询、继续、审阅、取消
  digitalme 平台上已有的课程生成任务时使用。依赖 dm 命令行（npx digitalme-cli）。
---

# digitalme：AI 数字人课程生成

把一份讲稿变成成片视频的完整流水线。核心心智模型：**备素材（可复用）→ 创建 run → 确认报价 → 循环（watch 等审阅点 → 查产物 → approve/revise）→ 下载成片**。积分是真实计费，confirm 与审阅决定是 human-in-the-loop 动作，必须征得用户同意。

## 第一步：环境自检（每次会话开始做一次）

运行 `dm me`：
- 正常返回用户与积分 → 就绪
- 退出码 2 / 提示未配置 → 指导用户：打开 https://digitalme.com.cn 注册登录 → 「设置」页签发 API Token → 执行
  `npx digitalme-cli login --url https://digitalme.com.cn --token <dmt_开头的令牌>`，然后重试
- 503 / 网络错误 → 平台冷启动（首次访问从 0 拉起约 1 分钟），等 15 秒重试一次即可，不必报告为故障

## 第二步：备素材（优先复用，避免重传）

先查已有资源：`dm resource ls`（或 `--kind voice|avatar|script` 过滤）。
- **声音（必传）**：平台预制音色（`isPreset: true`，拿来即用）或用户历史自录样本；返回的 `id`+`cosKey` 直接进 run create。新录样本须在网页端照固定朗读文案录制（照读样本克隆效果最好）
- **形象照（必传）**：用户正面照；没有可复用的就 `dm upload <file> --kind avatar`
- **讲稿/材料（可选）**：成稿直传 `dm upload 讲稿.docx --kind script`（支持 txt/md/docx/pdf/pptx）；或走意图模式让平台从材料直接写稿
- 新上传：`dm upload <file> --kind avatar|voice|script` → 记下 `cosKey` / `id`

## 命令速查

| 动作 | 命令 |
|---|---|
| 报价预估（不建任务） | `dm pricing --chars <字数> [--model lite\|pro]` |
| 我的资源（复用） | `dm resource ls [--kind voice\|avatar\|script]` |
| 建项目 | `dm project create --title <标题>` → 记下 `id` |
| 创建 run（拿报价，不扣款） | `dm run create --project <pid> --avatar-key <cosKey> --voice-key <cosKey> --voice-upload <id> (--text "<全文>" \| --text-file <路径> \| --script-upload <上传id> \| --intent "<自然语言意图>" [--materials <上传id,逗号分隔>]) [--model lite\|pro]` |
| 确认报价（冻结积分+入队） | `dm run confirm <runId>` |
| 跟进到需要动作 | `dm run watch <runId>`（阻塞；退出码见下） |
| 自动驾驶（须用户明确授权） | `dm run auto <runId> --approve-all [--out 成片.mp4]`（自动过审阅点并下载成片） |
| 审阅 | `dm run review <runId> --node <节点> --approve` 或 `--revise --feedback "<具体意见>"` |
| 取消（全额退冻结） | `dm run cancel <runId>`（仅 queued/awaiting_review） |
| 产物 | `dm artifact ls <runId>` → `dm artifact cat <id>`（文本）/ `dm artifact get <id> -o <文件>`（音视频） |

**run create 两种模式**：成稿模式（--text/--text-file/--script-upload）直接润色成稿；**意图模式**（--intent + --materials）由平台智能体分析材料写稿——用户只给大纲/材料/想法时选它（平台可能向用户提问澄清，注意转述）。

所有命令 stdout 输出 JSON（watch/auto 为 NDJSON 流）；错误在 stderr 也是 JSON。详细 flag 用 `dm help <命令>` 查。

## 退出码契约（watch / auto 的状态机）

| 码 | 含义 | 你该做什么 |
|---|---|---|
| 0 | run 已完成 | `dm artifact ls` 找 final_video 并交付 |
| 1 | 一般错误 | 读 stderr JSON，向用户如实转述 |
| 2 | 未认证/未配置 | 引导用户 login（见环境自检） |
| 3 | **到达审阅点** | 进入下方审阅循环 |
| 4 | run 失败 | 冻结积分已自动全额退；末尾 run/error-event 行有原因，转述即可 |
| 5 | 已取消 | 确认是否符合预期 |
| 6 | 超时 | run 仍在跑，再次 watch/auto |

## 审阅循环（3 个审阅点，按序出现）

`watch` 退出码 3 时，末行 `watch/end` JSON 的 `pendingReview.node` 告诉你卡在哪个节点：

| 节点 | 产物（artifact kind） | 检查什么 | 建议姿势 |
|---|---|---|---|
| script_polish | polished_script（文本，cat 直读） | 口语化、断句、术语、无事实错误 | 可自主判断；不满意就 revise 并给具体意见 |
| slides | slides_html（HTML，cat 直读） | 章节结构、图文对应、错别字 | 可自主判断 |
| compose | segment / final_video（成片） | 整体效果（配音与数字人形象在成片里一并体现） | **最后关口：请用户拍板** |

- 查产物：`dm artifact ls <runId>`（按 kind 找 id）→ 文本 `dm artifact cat <id>`，音视频 `dm artifact get <id> -o 文件名` 后把路径告诉用户（不要试图读音视频内容）
- 通过：`dm run review <runId> --node <节点> --approve`
- 修改：`dm run review <runId> --node <节点> --revise --feedback "<具体、可执行的修改意见>"`（"不好"不行，要"第二段口吻改得更口语化，删掉英文缩写"）
- 审阅后 run 回到队列 → 再次 `dm run watch <runId>` 等下一个节点
- **用户明确授权"全自动、不用每步问我"时**：`dm run auto <runId> --approve-all --out 成片.mp4` 一条命令跑到底（含全部审阅通过与成片下载）

## 铁律

1. **confirm 前必须向用户报告报价**（`run create` 返回的 `quote.quoteCredits`，即冻结积分数）并征得同意——这是真实计费动作
2. `--approve-all` 自动过审 = 你替用户做了全部审阅决定，**必须先获得用户明确授权**再用；默认逐点人工审
3. revise 的 feedback 必须具体可执行
4. run 失败/取消会自动全额退冻结积分，如实转述即可，不要谎报损失
5. 并发上限 2 个活跃 run；同一文稿重跑前确认旧 run 已终态或已取消
6. 积分余额在 `dm me` 的 `credits.balance`；新用户有注册赠送，每天有免费补给，不足时引导用户领取或充值（https://digitalme.com.cn）
