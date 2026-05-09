# 🍑 女性减脂塑形 AI 教练 Skill

> [English version](./README_EN.md)
作为一个 30+、想保持身材但找私教时间不够、市面上的健身 App 都不考虑月经周期的女性——我把自己 X 年踩坑的减脂方法论喂给 Claude 做成了 AI 教练。每周帮我省 30分钟做训练计划，会按周期调热量目标。
这是一套基于女性月经周期 × 4 阶段（月经 / 卵泡 / 排卵 / 黄体）+ 围绝经期 / 绝经后路径的减脂塑形方法论。
作为 Markdown 知识包提供给任意 LLM（Claude / GPT / Qwen 等），让它扮演一个懂周期、懂中国便利店外卖、懂居家训练的女性私人教练。

> ⚠️ 本项目不是医疗建议。月经异常 / 出血异常 / 慢性病 / 孕哺乳期请就医。

## 这是什么

11 份按主题切分的 Markdown 文件，定义了：

- **谁来用**：默认中国大陆城市成年女性（18+）、外卖 + 便利店饮食、无健身房会员、办公室久坐
- **怎么聊**：5 个 batch 的 onboarding 流程，5 屏内完成画像采集
- **吃什么**：BMR / TDEE / 三大营养素计算 + 训练日 vs 静息日热量分段 + 便利店即食 / 外卖搭配示例
- **怎么练**：3 日分化（臀腿 / 胸肩 / 背）+ 上下半身切分 + 全身环路 3 种模板，含周期阶段强度调整
- **周期怎么算**：基于 last_period_start / avg_cycle_days / avg_period_days 推算今日 phase，按 phase 给训练 + 饮食 + 体感建议
- **3 种固定报告**：① 目标可行性评估 ② 本周计划 ③ 当日计划（Section A-G 结构化模板）

## 文件结构

```
SKILL.md                  # 主入口 + 路由逻辑 + 5-batch onboarding 模板（verbatim）
state/profile-schema.md   # 数据契约 + 周期推算算法
onboarding.md             # 首次采集流程
cycle-rules.md            # 4 阶段 × 3 维度（训练 / 饮食 / 体感）建议库
non-cyclic-rules.md       # 围绝经期 / 绝经后 / 哺乳期路径
lifestyle-tips.md         # 便利店 / 外卖 / 居家训练库
weekly-split.md           # 3-day 分化 / 上下分化 / 全身环路
kcal-targets.md           # BMR / TDEE / 蛋白脂肪碳水 / 饮水 / 训练日热量加成
exercises.md              # 动作库 + 禁忌动作映射 + 器械替换逻辑
reports.md                # 3 种固定报告模板
progress-checkins.md      # 4 类反馈（饮食 / 训练 / 围度 / 体感）
```

## 怎么用

### A. 作为系统提示词喂给任何 LLM（最简单）

把所有 .md 拼成单一 system prompt：

```bash
cat SKILL.md state/profile-schema.md cycle-rules.md non-cyclic-rules.md \
    lifestyle-tips.md weekly-split.md kcal-targets.md exercises.md \
    reports.md progress-checkins.md onboarding.md > system-prompt.md
```

任何长 context LLM 都行（建议 Claude Sonnet / GPT-5 / qwen3-max 等）。

### B. 作为 Claude Code skill

```bash
mkdir -p ~/.claude/skills/fitness
cp -r * ~/.claude/skills/fitness/
```

### C. 自建 web chat

仓库里没包含 web 实现，但你可以：
- 用任意 OpenAI-compatible API
- 把 11 份 .md 拼成 system prompt
- 加一个 profile JSON 持久化 + 历史消息存储

## Adapt 到你的 LLM 平台

文件里有几处 platform-specific 引用（如 `~/.openclaw/workspace/fat-loss-agent/profile.json` 路径、`openclaw read/write` 工具）—— 这些是原作者运行环境。Adapt 到其他平台：

- 把 `~/.openclaw/workspace/fat-loss-agent/profile.json` 替换成你的 profile 存储路径
- 把 `openclaw 的 read / write 工具` 替换成你 LLM 平台的文件读写能力（Claude Code 自带 Read/Write tool；OpenAI Assistants 用 file_id；自建 web 用数据库）
- 把 `<UPDATE_PROFILE>{...}</UPDATE_PROFILE>` 这种 marker 协议改成你后端能解析的格式

## 数据原则

- **本地优先**：profile.json 默认本机存储；不上报 / 不打点
- **可删除**：用户随时删本地文件即可清空
- **不医疗化**：明确避免诊断式语言；月经异常 / 持续无月经 / 异常出血 / 慢病 / 怀孕哺乳 → 一律建议就医

## 方法论范式

skill 借鉴了中文减脂内容圈普遍流行的 5 种范式（不点名具体个人）：

| 范式 | 核心 |
|---|---|
| 不节食派 | 粗略热量 + 关注饮食关系，反对过度精确 |
| 力量增肌派 | 复合动作（深蹲 / 硬拉 / 卧推）+ 1.6-2.0 g/kg 蛋白 |
| 薄肌审美派 | 普拉提 / 中等重量 + 控糖 + 早睡 |
| 体态修正派 | 假胯宽 / 圆肩等 → 臀中肌激活 + 拉伸松解 |
| 周期性放纵派 | 每周 1 顿 cheat meal 维持心理可持续 |

skill 不替任何具名个人代言，所有方法论都做了二次提炼 + 周期化封装 + 中国大陆生活场景适配。

## License

MIT — 可商用，可改，要求保留版权声明。详见 [LICENSE](LICENSE)。

## Disclaimer

本项目不是医疗建议、不是营养师、不是康复治疗师。
所有训练 / 饮食 / 周期建议仅基于公开方法论的二次整理，不替代专业医疗。
经期异常、停经、剧痛、异常出血、骨折后康复、孕期哺乳期 → 请去医院 / 找注册营养师 / 找专业康复师。
