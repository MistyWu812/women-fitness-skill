# 🍑 女性减脂塑形 AI 教练 Skill

一套基于女性月经周期 × 4 阶段（月经/卵泡/排卵/黄体）+ 围绝经期/绝经后路径的减脂塑形方法论。
作为 Markdown 知识包提供给任意 LLM（Claude / GPT / Qwen 等）使用，让它扮演一个懂周期、懂中国便利店外卖、懂居家训练的女性私人教练。

> 不是医疗建议；月经异常 / 出血异常请就医。

## 这是什么

12 份按主题切分的 Markdown 文件，定义了：

- **谁来用**：默认中国大陆城市成年女性（18+）、外卖+便利店饮食、无健身房会员、办公室久坐
- **怎么聊**：5 个 batch 的 onboarding 流程，最多问到 5 屏完成画像采集
- **吃什么**：BMR/TDEE/三大营养素计算 + 训练日 vs 静息日热量分段 + 便利店即食 / 外卖搭配示例
- **怎么练**：3 日分化（臀腿/胸肩/背）+ 上下半身切分 + 全身环路 3 种模板，含周期阶段强度调整
- **周期怎么算**：基于 last_period_start / avg_cycle_days / avg_period_days 推算今日 phase，按 phase 给出训练 + 饮食 + 体感建议
- **3 种固定报告**：① 目标可行性评估 ② 本周计划 ③ 当日计划（Section A-G 结构化模板）

## 文件结构

```
SKILL.md                  # 主入口 + 路由逻辑 + 5-batch onboarding 模板（verbatim）
state/profile-schema.md   # 数据契约 + 周期推算算法
onboarding.md             # 首次采集流程
cycle-rules.md            # 4 阶段 × 3 维度（训练/饮食/体感）建议库
non-cyclic-rules.md       # 围绝经期 / 绝经后 / 哺乳期路径
lifestyle-tips.md         # 便利店 / 外卖 / 居家训练库
weekly-split.md           # 3-day 分化 / 上下分化 / 全身环路
kcal-targets.md           # BMR / TDEE / 蛋白脂肪碳水 / 饮水 / 训练日热量加成
exercises.md              # 动作库 + 禁忌动作映射 + 器械替换逻辑
reports.md                # 3 种固定报告模板
progress-checkins.md      # 4 类反馈（饮食/训练/围度/体感）
influences.md             # 方法论灵感来源（小红书博主清单 + 取舍说明）
```

## 怎么用

### A. 作为系统提示词喂给 LLM（最简单）

把 `SKILL.md` 拼上其他 .md 文件作为 system prompt 发给 LLM，然后用户消息走对话。
任何支持长 context 的 LLM 都行（建议 Claude / GPT-5 / Qwen3-Max 等）。

```bash
# 拼接成单一 system prompt
cat SKILL.md state/profile-schema.md cycle-rules.md non-cyclic-rules.md \
    lifestyle-tips.md weekly-split.md kcal-targets.md exercises.md \
    reports.md progress-checkins.md onboarding.md influences.md > system-prompt.md
```

### B. 作为 Claude Code skill

```bash
mkdir -p ~/.claude/skills/fitness
cp -r * ~/.claude/skills/fitness/
```

### C. 作为 openclaw skill pack

文件结构兼容 [openclaw](https://github.com/openclaw) 风格，frontmatter 已设好。
直接放到 openclaw skills 目录即可。

### D. 自建 web chat（参考实现）

仓库里没包含 web 实现，但你可以：
- 用任意 OpenAI-compatible API（推荐通义 qwen3-max / Claude Sonnet）
- 把 12 份 .md 拼成 system prompt
- 加一个 profile JSON 持久化 + 历史消息存储

我自己跑的版本基于 Hono + Vercel + Upstash Redis，后续可能单独开源。

## 数据原则

- **本地优先**：profile.json 默认存放在用户本机（`~/.openclaw/workspace/fat-loss-agent/profile.json`）；不上报、不打点
- **可删除**：所有数据用户随时删本地文件即可清空
- **不医疗化**：明确避免诊断式语言；月经异常 / 持续无月经 / 异常出血 / 慢病 / 怀孕哺乳 → 一律建议就医

## 方法论致谢

`influences.md` 列了主要灵感来源（小李不吃 / 好人松松 / 欧阳春晓 / 张俪 / 修贤 / 体态大师 / 植女等），都是公开内容。我做了二次取舍 + 周期化封装 + 中国大陆生活场景适配。

## License

MIT — 可商用，可改，要求保留版权声明。详见 [LICENSE](LICENSE)。

## Disclaimer

本项目不是医疗建议、不是营养师、不是康复治疗师。
所有训练 / 饮食 / 周期建议仅基于公开方法论的二次整理，不替代专业医疗。
经期异常、停经、剧痛、异常出血、骨折后康复、孕期哺乳期 → 请去医院 / 找注册营养师 / 找专业康复师。
