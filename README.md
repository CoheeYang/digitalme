<div align="center">

<img src="docs-banner.png" alt="digitalme · AI 数字人录播课生成" width="880">

# digitalme

### AI 数字人录播课生成 · 讲稿一键变视频课程

把一份讲稿 / 文档 / 大纲，变成一节有数字人讲师、有声音克隆配音、有自动生成课件的**完整录播课视频**。

[官网 digitalme.com.cn](https://digitalme.com.cn) · 网页版开箱即用 · CLI + Agent 技能自动化

**AI 数字人视频** · **讲稿转视频** · **声音克隆** · **自动课件** · **一键成课**

</div>

---

## 这是什么

**digitalme** 是一个面向内容创作者与培训者的 AI 课程视频生产平台：

- 🎬 **数字人讲师**：上传一张你的正面照，视频里就是你本人在讲课（口型同步）
- 🎙️ **声音克隆**：录一段 15 秒的声音样本，整节课都用"你的声音"配音；也可选用平台预制音色
- 📊 **自动课件**：讲稿自动生成配套 HTML 课件（全屏课件 + 数字人画中画布局）
- ✍️ **两种起点**：给成稿直接做课；或只丢一堆材料（docx/pdf/pptx/md/txt）+ 一句"做成一节 20 分钟的入门课"，平台写稿做课
- 👀 **全程可审**：写稿、课件、成片 3 个人工审阅点，不满意就提意见重做，通过才进入下一步
- 🤖 **Agent 友好**：全套能力封装成 `dm` 命令行 + Agent 技能，让 Claude Code / Codex / Cursor / OpenCode 等 AI 助手替你跑完整个流程

**适合谁**：想做网课但不想出镜/不想剪辑的知识博主、需要批量生产培训视频的企业内训师、用 AI 助手自动化内容生产的开发者和效率玩家。

## 30 秒开始

### 方式一：网页版（零门槛）

打开 **[digitalme.com.cn](https://digitalme.com.cn)** → 手机号注册 → 上传形象照 + 声音样本 → 写下你的课程意图 → 走完 3 个审阅点 → 下载成片。新用户有注册赠送积分，每天有免费补给。

### 方式二：在你的 AI 编程助手里（推荐进阶）

```bash
npx skills add CoheeYang/digitalme
```

然后对助手说：

> 帮我把桌面上的《产品介绍.docx》做成一节数字人课程视频，用平台第一个预制音色，课件审阅过了之后成片直接下载到 output.mp4

助手会自己完成：上传/复用素材 → 生成讲稿 → 报价并征得你同意 → 过审阅点 → 下载成片。

<details>
<summary><b>各框架安装明细</b>（<code>npx skills add</code> 会自动识别，以下为手动路径）</summary>

| 框架 | 安装方式 |
|---|---|
| Claude Code | `npx skills add CoheeYang/digitalme`（或把 `skills/digitalme/` 复制到 `~/.claude/skills/`） |
| Codex | 复制 `skills/digitalme/` 到 `~/.codex/skills/` |
| OpenCode | 复制 `skills/digitalme/` 到 `~/.opencode/skills/` |
| Cursor | 复制 `skills/digitalme/` 到 `~/.cursor/skills/` |
| Kiro | 复制 `skills/digitalme/` 到 `~/.kiro/skills/` |
| 其他任意框架 | `npx digitalme skill install --dir <你的 skills 目录>`；或直接把本仓库的 [SKILL.md](skills/digitalme/SKILL.md) 内容贴进你的助手对话框，一样能用 |

</details>

### 方式三：直接用命令行

```bash
npm i -g digitalme          # 或每次 npx digitalme
npx digitalme login --url https://digitalme.com.cn --token <dmt_令牌>   # 令牌在官网「设置」页签发
```

```bash
dm resource ls --kind voice                     # 复用：平台预制音色 / 历史素材
dm upload 形象照.png --kind avatar              # 上传你的正面照
dm project create --title "我的第一节课"
dm run create --project <pid> --avatar-key <cosKey> \
              --voice-key <声音cosKey> --voice-upload <声音id> \
              --intent "把这份材料做成一节入门课" --materials <材料上传id>
dm run confirm <runId>                          # 确认报价（冻结积分，先给用户看报价！）
dm run watch <runId>                            # 阻塞到审阅点（退出码 3）或终态
dm run review <runId> --node script_polish --approve
dm run auto <runId> --approve-all --out 课程.mp4 # 获用户授权后：自动过审 + 下载成片
```

所有命令输出 JSON、退出码即状态机——天然适合脚本与 Agent 消费。完整命令参考 `dm help`。

## 生成一节课的完整流程

```
素材（形象照 + 声音样本 + 讲稿/材料）
   ↓
讲稿润色（成稿模式）或 智能体写稿（意图模式）     ← 审阅点 1：口播稿
   ↓
自动课件（HTML，随成片嵌入）                    ← 审阅点 2：课件
   ↓
数字人合成（声音克隆配音 + 形象口型）
   ↓
成片合成（全屏课件 + 数字人画中画）              ← 审阅点 3：成片
   ↓
下载 MP4（1080p）
```

每个审阅点都可以：**通过**（approve）或 **提修改意见重做**（revise）。

## 计费

积分制，按用量计费：字数越多、时长越长、画质档位越高，消耗越多。**每次创建任务先生成报价，你确认后才会冻结积分**；任务失败或取消自动全额退还。新用户注册赠送积分，每日有免费补给，充值请在官网进行。

## English Section

**digitalme** turns scripts, documents, or outlines into complete **AI-avatar course videos** — a digital human presenter (lip-synced to your photo), voice cloning from a 15-second sample, auto-generated slide deck, and a 1080p final MP4. Three human review checkpoints (script → slides → final cut) keep you in control.

Full automation is available through the `dm` CLI (`npm i -g digitalme`) and a ready-made agent skill (`npx skills add CoheeYang/digitalme`) for Claude Code, Codex, Cursor, OpenCode and 20+ other agents: upload/reuse assets, generate the script from materials, approve reviews, and download the final video — all from your AI assistant.

Service: [digitalme.com.cn](https://digitalme.com.cn) (credit-based billing, free daily allowance, quote before any charge, auto-refund on failure).

## 链接

- 官网（网页版）：https://digitalme.com.cn
- Agent 技能文档：[skills/digitalme/SKILL.md](skills/digitalme/SKILL.md)
- 问题反馈：[Issues](https://github.com/CoheeYang/digitalme/issues)

## License

本仓库的技能文档以 [MIT](LICENSE) 发布。平台服务条款见官网。

<div align="center">

**让每个有知识的人，都拥有自己的数字人课堂。**

</div>
