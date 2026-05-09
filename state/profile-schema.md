# profile.json 数据契约

本文件定义 profile.json 的字段、类型、约束、文件路径与读写规约。LLM 在调用 openclaw 的 `read`/`write` 工具维护用户画像时，必须严格遵守本契约。

## 文件路径

- 路径：`~/.openclaw/workspace/fat-loss-agent/profile.json`
- 注：openclaw 工作区根目录 `~/.openclaw/workspace/` 由 openclaw 保证存在；插件子目录 `fat-loss-agent/` 不保证存在。

## 父目录创建责任

首次写盘前，LLM 必须确保 `~/.openclaw/workspace/fat-loss-agent/` 已存在（如不存在，使用 openclaw 提供的目录创建工具，或带 mkdir 语义的 write 工具创建）。否则首次 `write` 调用会失败。

## 完整 schema 示例

```json
{
  "schema_version": 1,
  "created_at": "2026-04-29T08:00:00Z",
  "updated_at": "2026-04-29T08:00:00Z",
  "name": "用户昵称（可选）",
  "age": 30,
  "city": "上海",
  "height_cm": 165,
  "weight_kg": 58.0,
  "body_fat_pct": 26.0,
  "goal": {
    "target_weight_kg": 53,
    "target_waist_cm": 70,
    "horizon_weeks": 12,
    "freeform": "改善腰腹线条"
  },
  "activity_level": "sedentary | light | moderate | active",
  "sleep_avg_hours": 6.5,
  "diet_pattern": {
    "homemade_pct": 40,
    "delivery_pct": 60,
    "preferences": "爱吃辣",
    "restrictions": "乳糖不耐 / 不吃香菜",
    "pantry_ingredients": "燕麦 / 鸡蛋 / 蓝莓 / 紫薯 / 鸡胸 / 西兰花 / 希腊酸奶",
    "favorite_takeout": "黄焖鸡米饭 / 西贝莜面 / 麻辣烫（清汤）/ 瑞幸燕麦拿铁"
  },
  "cycle": {
    "status": "regular",
    "last_period_start": "2026-04-15",
    "avg_cycle_days": 28,
    "avg_period_days": 5,
    "menopause_age": null
  },
  "training": {
    "history": "occasional",
    "preferred_modality": "cardio + yoga",
    "preferred_cardio": "跳绳 / 爬楼",
    "gym_access": "none",
    "home_equipment": "瑜伽垫 / 一对 5kg 哑铃",
    "target_muscles": "腰腹 / 臀腿",
    "preferred_time": "早晨 7-8 点早餐前"
  },
  "body_status": {
    "injury_history": "右膝半月板 2023 年扭伤过，不能完全下蹲到底",
    "chronic_issues": "颈肩僵硬 / 偶尔腰酸（久坐型）",
    "posture_concerns": "骨盆前倾 / 圆肩"
  }
}
```

## 字段必填性

| 字段 | 必填 | 默认值 | 说明 |
|---|---|---|---|
| `schema_version` | 自动 | `1` | 由 LLM 写入，始终为 1（v0.0） |
| `created_at` | 自动 | 当前 UTC 时间 | 仅首次创建时写入 |
| `updated_at` | 自动 | 当前 UTC 时间 | 每次 `write` 时刷新 |
| `name` | 可选 | `null` 或省略 | 用户昵称 |
| `age` | 必填 | — | 整数年龄 |
| `city` | 可选 | `null` 或省略 | 用户所在城市，用于时区推断 |
| `height_cm` | 必填 | — | 身高（厘米） |
| `weight_kg` | 必填 | — | 当前体重（公斤） |
| `body_fat_pct` | 可选 | `null` 或省略 | 体脂率百分比 |
| `goal.target_weight_kg` | 可选 | `null` 或省略 | 目标体重 |
| `goal.target_waist_cm` | 可选 | `null` 或省略 | 目标腰围 |
| `goal.horizon_weeks` | 可选 | `null` 或省略 | 目标达成周数 |
| `goal.freeform` | 可选 | `null` 或省略 | 自由文字目标描述 |
| `activity_level` | 必填 | — | 枚举见下 |
| `sleep_avg_hours` | 可选 | `null` 或省略 | 平均睡眠时长 |
| `diet_pattern.homemade_pct` | 可选 | `null` 或省略 | 自做饭占比（0-100） |
| `diet_pattern.delivery_pct` | 可选 | `null` 或省略 | 外卖占比（0-100） |
| `diet_pattern.preferences` | 可选 | `null` 或省略 | 饮食偏好（自由文字） |
| `diet_pattern.restrictions` | 可选 | `null` 或省略 | 饮食禁忌（自由文字） |
| `diet_pattern.pantry_ingredients` | 可选 | `null` 或省略 | 家里常备食材（自由文字，分隔符 / 或 顿号都可），LLM 推餐时**优先使用这些食材**避免推不可达 |
| `diet_pattern.favorite_takeout` | 可选 | `null` 或省略 | 常点的外卖品牌或菜系，LLM 推外卖时**优先使用用户已经习惯的**而不是泛推 |
| `cycle.status` | 必填 | `regular` | 月经状态枚举：`regular`（规律）/ `irregular`（不规律但有）/ `peri_menopause`（围绝经期，cycles 不稳）/ `menopause`（绝经 ≥ 12 个月无月经）/ `breastfeeding` / `pregnant` / `medical`（子宫切除 / 内分泌治疗等）。**决定走 cycle-rules.md（status=regular）还是 non-cyclic-rules.md（其他状态）。** |
| `cycle.last_period_start` | 条件必填 | — | 最近一次月经起始日，`YYYY-MM-DD`。**仅当 `status=regular` 或 `irregular` 时必填**；其它 status 省略或为 null。 |
| `cycle.avg_cycle_days` | 条件必填 | — | 平均月经周期天数。**仅 `status=regular` 时必填**。`irregular` 用户可选，填了用作粗略参考。 |
| `cycle.avg_period_days` | 条件必填 | — | 平均月经持续天数。**仅 `status=regular` 时必填**。 |
| `cycle.menopause_age` | 可选 | `null` | 绝经发生年龄（仅 `status=menopause` 用户可选填，用于计算绝经年限） |
| `training.history` | 可选 | `null` 或省略 | 运动历史枚举：`none` / `occasional` / `regular_1-2x` / `regular_3+x` |
| `training.preferred_modality` | 可选 | `null` 或省略 | 偏好训练类型自由文字（如 "力量" / "有氧" / "瑜伽" / "普拉提" / "都试" / 多选用空格或 / 分隔） |
| `training.preferred_cardio` | 可选 | `null` 或省略 | 偏好有氧自由文字（如 "跑步" / "爬楼" / "游泳" / "骑行" / "跳绳" / "都不太喜欢"） |
| `training.gym_access` | 可选 | `null` 或省略 | 健身房状况枚举：`membership` / `none` / `occasional` / `not_planned` |
| `training.home_equipment` | 可选 | `null` 或省略 | 家中可用器械自由文字（如 "一对 5kg 哑铃 / 瑜伽垫 / 弹力带 / 12kg 壶铃"），无 → 写 "无" 或 "全徒手"。LLM 推荐负重动作前必须读 + `exercises.md §选动作的器械逻辑` 决定走 bodyweight 还是负重组。 |
| `training.target_muscles` | 可选 | `null` 或省略 | 重点想练的部位自由文字（如 "腰腹 / 臀腿 / 手臂 / 背 / 肩 / 全身均衡"），多选用 / 分隔。LLM 设计训练计划时，**优先**选目标肌群对应的动作。 |
| `training.preferred_time` | 可选 | `null` 或省略 | 一般训练时间段自由文字（如 "早晨 7-8 点早餐前 / 中午午休 / 下班后 19 点 / 晚饭后 1 小时 / 周末灵活"）。LLM 据此安排：餐前餐后窗口、补蛋白时机、饮水分时段、空腹有氧适用性。 |
| `body_status.injury_history` | 可选 | `null` 或省略 | 受伤史 + 当前禁动作。例 "右膝半月板 2023 年扭伤过，不能完全下蹲到底" / "腰椎间盘突出，禁止仰卧起坐 / 硬拉" / "无"。**LLM 推荐每个动作前必须查这个字段防 contraindicate。** |
| `body_status.chronic_issues` | 可选 | `null` 或省略 | 慢性问题自由文字（颈肩僵硬 / 肩周炎 / 腱鞘炎 / 腰椎间盘 / 膝盖痛 / 痛经剧烈 / 偏头痛 / 等），无 → "无"。 |
| `body_status.posture_concerns` | 可选 | `null` 或省略 | 体态自评自由文字（膝超伸 / O 型腿 / X 型腿 / 骨盆前倾 / 圆肩驼背 / 头前引 / 不知道），多选用 / 分隔。LLM 据此调整动作侧重（如骨盆前倾 → 强化臀肌 + 松髂腰肌）。 |

## 字段语义与合法值

- `activity_level`：枚举，必须为以下四个字符串之一：
  - `sedentary`：久坐（每日步数 < 5000，几乎无运动）
  - `light`：轻度走动（5000–8000 步/天，偶尔轻运动）
  - `moderate`：中度走动（8000–12000 步/天，每周 2–3 次中强度运动）
  - `active`：重体力（>12000 步/天 或 每周 4 次以上中高强度运动 / 体力劳动）
- `height_cm`：数字，合法范围 140–200。
- `weight_kg`：数字，合法范围 30–200，允许保留 1 位小数。
- `body_fat_pct`：数字，合法范围 10–50，可缺省；女性低于 18 时 LLM 应给出健康警示。
- `age`：整数，合法范围 12–80（v0.0 强约束女性可孕年龄 + 缓冲，超出范围请用户确认或拒绝）。
- `cycle.avg_cycle_days`：整数，合法范围 21–40。
- `cycle.avg_period_days`：整数，合法范围 2–10。
- `cycle.last_period_start`：日期字符串，格式严格为 `YYYY-MM-DD`，且必须不晚于今天（不允许未来日期）。若 `today - last_period_start > 90 天`（远超 1.5 × cycle），视为初始化失误，要求用户重填，而非走陈旧数据流程。
- 时间戳字段（`created_at` / `updated_at`）：ISO 8601 UTC 字符串，格式严格为 `YYYY-MM-DDTHH:MM:SSZ`（必须以 `Z` 结尾，秒级精度），例：`2026-04-29T08:00:00Z`。
- 日期字段（`cycle.last_period_start`）：仅日期，格式严格为 `YYYY-MM-DD`。
- `goal.target_weight_kg`：数字，合法范围 30–200；建议不低于 BMI 18 对应体重（BMI 18 体重 = 18 × (height_cm/100)²，单位 kg）；若用户目标低于此值，LLM 应主动提示风险。
- `goal.horizon_weeks`：整数，合法范围 4–52。
- `goal.target_waist_cm`：数字，合法范围 40–150（参考值，超出请用户确认）。
- `diet_pattern.homemade_pct` 与 `diet_pattern.delivery_pct`：分别为 0–100 整数；不强制相加 = 100（剩余可能是外食、聚餐等）。上限约束：`homemade_pct + delivery_pct ≤ 100`；若超过，视为输入错误，要求用户重填。
- `sleep_avg_hours`：数字，合法范围 3–12。
- `name` / `city` / `goal.freeform` / `diet_pattern.preferences` / `diet_pattern.restrictions`：自由文字，无固定长度限制，但应保持简洁（建议 ≤ 100 字）。

## 读写规约

- **创建**：首次 onboarding 完成且用户确认无误后，LLM 调用 openclaw `write` 工具一次性写入完整 JSON。`schema_version` 设为 `1`，`created_at` 与 `updated_at` 同时填当前 UTC 时间（精确到秒，格式 `YYYY-MM-DDTHH:MM:SSZ`）。
- **UTC 时间来源**：LLM 必须从 openclaw 提供的当前时间能力（如 `system.now()` 或对话上下文的 `当前时间`）获取 UTC 时间戳，禁止凭训练数据猜测或复用对话开始的旧时间。
- **更新**：禁止整文件覆盖。流程必须严格按以下四步执行：
  1. `read` 读取当前 profile.json 全文并解析为对象。
  2. 仅修改本次需要变更的字段（保留其他字段原值）。
  3. 同步刷新 `updated_at` 为当前 UTC 时间。
  4. `write` 写回完整对象。
- **读取**：每轮对话开始时由 SKILL.md 触发 `read`；如文件不存在，路由到 onboarding 流程；如文件存在但 JSON 解析失败或必填字段缺失，提示用户并按需进入修复流程。
- **删除**：v0.0 不提供删除接口（用户想重置时手动 `rm ~/.openclaw/workspace/fat-loss-agent/profile.json` 即可）。
- **null vs 省略**：写入时统一**省略未填写的可选字段**，不要写显式 `null`，以减小文件体积与 diff 噪声；读取时**缺失键与 `null` 等价**处理。

## schema_version 与未来迁移

- v0.0 当前 `schema_version = 1`。
- 未来升级（如新增字段、改变语义）时：在本文件追加「迁移片段」小节，描述 v1→v2 的字段变化与默认值；LLM 读取到旧版本文件时，按迁移片段就地升级，并将 `schema_version` 更新到新版本，刷新 `updated_at`，然后写回。
- v0.0 不需要写迁移代码。

## 周期阶段计算

本节为周期阶段算法的**唯一规范源**。下游 `cycle-rules.md`、`SKILL.md`、`onboarding.md` 等只允许引用本算法，禁止重定义。

时区：优先按 `city` 推断（如"上海" → `Asia/Shanghai`）；若 `city` 缺失，默认 `Asia/Shanghai`（v0.0 自用者主要在中国大陆）。`cycle.last_period_start` 视为该时区下的本地日期；`today` 必须在同一时区下取整天数。

算法（文字 + 伪码）：

```
设 today          = 当前日期（时区处理见下方"时区"说明）
设 days_since     = (today - cycle.last_period_start) 的整天数（向下取整，非负整数）
设 day_in_cycle   = days_since mod cycle.avg_cycle_days

如 day_in_cycle <  cycle.avg_period_days                 → 月经期 (menstrual)
否则如 day_in_cycle <  (cycle.avg_cycle_days / 2) - 2     → 卵泡期 (follicular)
否则如 day_in_cycle <= (cycle.avg_cycle_days / 2) + 2     → 排卵期 (ovulatory)
否则                                                       → 黄体期 (luteal)
```

说明：

- 上式中 `cycle.avg_cycle_days / 2` 使用浮点除法；与 `day_in_cycle`（整数）比较时按数值大小判定。
- 边界语义（与伪码中运算符严格一致）：卵泡期判定使用严格小于 `<`，排卵期判定使用 `<=`。因此：
  - 恰好等于 `avg_period_days` 时**进入卵泡期**（月经期判定 `day_in_cycle < avg_period_days` 为假）。
  - 恰好等于 `cycle/2 - 2` 时**进入排卵期**（卵泡期判定 `day_in_cycle < cycle/2 - 2` 为假）。例如 28 天周期、`day_in_cycle = 12` 时，卵泡期判定 `12 < 12` 为假，落入排卵期判定 `12 <= 16` 为真。
  - 恰好等于 `cycle/2 + 2` 时**仍属排卵期**（排卵期判定 `day_in_cycle <= cycle/2 + 2` 为真）。

实际示例：

- `last_period_start = 2026-04-15`，`avg_cycle_days = 28`，`avg_period_days = 5`，`today = 2026-04-29`
- `days_since = 14`，`day_in_cycle = 14 mod 28 = 14`
- 判定（按伪码顺序逐条比对）：
  - 月经期：`14 < 5` 为假 → 不进入。
  - 卵泡期：`14 < 28/2 - 2 = 12` 为假（严格小于） → 不进入。
  - 排卵期：`14 <= 28/2 + 2 = 16` 为真 → 进入。
- 结果：**排卵期 (ovulatory)**

数据陈旧检测：

- 若 `(today - cycle.last_period_start) > 1.5 × cycle.avg_cycle_days`（按整天数比较），LLM 应主动提示用户更新 `cycle.last_period_start`。
- 在用户更新前，LLM 暂时按「陈旧数据」处理：不假设当前阶段，只提供与周期阶段无关的保守建议（如基础饮食结构、轻度活动），并在回复中说明「周期数据已过期，建议先更新」。

### 阶段标识符（key）

在所有结构化引用中（如 `cycle-rules.md` 的章节锚点、内部传值），统一使用以下小写英文 key：

| 中文 | key |
|---|---|
| 月经期 | `menstrual` |
| 卵泡期 | `follicular` |
| 排卵期 | `ovulatory` |
| 黄体期 | `luteal` |
