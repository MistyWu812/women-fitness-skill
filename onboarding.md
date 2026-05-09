# 对话式 Onboarding 流程

本文件由 LLM 在运行时查阅：当 `~/.openclaw/workspace/fat-loss-agent/profile.json` 不存在、或用户主动要求重置画像时，照此流程采集用户基础数据。Onboarding 是 **对话式而非表单式** —— 一轮一个字段，避免连环追问。流程必须以写出一份符合 `state/profile-schema.md` 的合法 profile.json 收尾。语气保持温暖、专业、零身体羞辱，把数据采集框定为"为了给你贴合身体节律的建议"，而不是"评估你"。

## 触发条件

- profile.json 不存在（路径见 `state/profile-schema.md` §文件路径）→ 自动进入 onboarding
- 用户主动说"重新开始 / 重置资料 / 我换了"等 → 备份旧 profile.json 为 `profile.json.bak.<timestamp>`（如存在）后进入 onboarding
- 用户主动说"修改 X" → 跳到对应字段单独修改，不重启全流程，遵循 profile-schema.md "读 → 改 → 刷新 updated_at → 写回"
- onboarding 进行中用户中途说"先不填了" → 不强写中间态，仅口头承诺"下次再聊"；下次进入仍判定 profile.json 不存在 → 从头开始（v0.0 极简策略，部分状态恢复推迟到 v0.1）
- 检测到 profile.json 存在但解析失败（JSON 损坏）→ 备份为 `.corrupt.<timestamp>` 后重走全流程，并向用户说明

## 采集策略：5 批次问完，不要逐个字段问

**总原则**：所有字段分成 **4 批**让用户一次性回答多项，节省时间、提升完成率。每一批用清晰的列表展示要问的字段，用户可以一段话答完，**不强制每项都答**（可选字段允许跳过）。

**第一轮回复必须 ≤ 80 字**：只一句简短问候 + 直接发出 Batch 1 的字段列表。**不要**在第一轮塞免责声明、自我介绍、流程说明、能力清单、隐私政策——这些一律延迟到全部字段采集完毕、profile.json 写入后的第一次普通对话再说。

### Batch 1 — 基本画像（6 项）

提问模板示例：

> "想给你做贴合身体节律的方案，先了解几个基本情况，下面任何一项不想说都可以跳过：
>
> - 怎么称呼你？
> - 年龄
> - 城市（用来推断时区和外卖品牌）
> - 身高 cm / 体重 kg
> - 体脂率 %（不知道没关系）"

涵盖字段：
- `name`（可选）
- `age`（必填，int 12-80）
- `city`（可选）
- `height_cm`（必填，num 140-200）
- `weight_kg`（必填，num 30-200）
- `body_fat_pct`（可选，num 10-50）

### Batch 2 — 生活节奏 + 身体节律（条件分支）

提问模板示例（接 Batch 1 用户答完后）：

> "好的我记下了。再了解下你的日常节奏和身体节律——这部分信息只本机存储，不上传任何地方：
>
> - 工作日活动量更接近：A 久坐 / B 轻度走动 / C 中度走动 / D 重体力？
> - 最近平均一天睡几小时？
> - 月经状态：A 规律 / B 不太规律但还有 / C 围绝经期（间隔不稳）/ D 已绝经 / E 哺乳期 / F 备孕或孕期 / G 医疗特殊
> - 如果你选 A 或 B，再说一下：上次月经哪天开始 / 平均周期多少天（默认 28）/ 经期一般几天？
> - 选 C/D 的话，方便告诉我大约什么时候开始的吗？（不强制）"

涵盖字段：
- `activity_level`（必填，enum：A→sedentary / B→light / C→moderate / D→active）
- `sleep_avg_hours`（可选，num 3-12）
- `cycle.status`（必填，enum：A→regular / B→irregular / C→peri_menopause / D→menopause / E→breastfeeding / F→pregnant / G→medical）
- `cycle.last_period_start`（**仅 status ∈ {regular, irregular} 时必填**，date YYYY-MM-DD ≤ today；其它 status 省略或 null）
- `cycle.avg_cycle_days`（**仅 status=regular 时必填**，int 21-40）
- `cycle.avg_period_days`（**仅 status=regular 时必填**，int 2-10）
- `cycle.menopause_age`（仅 status=menopause 时可选，年龄数字）

### Batch 3 — 训练偏好（6 项）

提问模板示例（接 Batch 2 用户答完后）：

> "再聊聊运动这块：
>
> - 之前有没有规律运动习惯？（无 / 偶尔 / 每周 1-2 次 / 每周 3 次以上）
> - 喜欢的训练类型？（力量 / 有氧 / 瑜伽 / 普拉提 / 跳舞 / 都试过没明显偏好；多选都行）
> - 有氧偏好哪种？（跑步 / 爬楼 / 游泳 / 骑行 / 跳绳 / 都不太喜欢）
> - 健身房？（有月卡 / 没办过 / 偶尔去 / 不打算去）
> - 家里有什么器械？（哑铃几公斤 / 壶铃几公斤 / 弹力带 / 瑜伽垫 / 跳绳 / 都没有）
> - 一般什么时间训练方便？（早晨早餐前 / 早餐后 / 中午午休 / 下班后晚饭前 / 晚饭后 1 小时 / 周末灵活）"

涵盖字段：
- `training.history`（可选，enum：none / occasional / regular_1-2x / regular_3+x）
- `training.preferred_modality`（可选，自由文本，建议归一化到 力量 / 有氧 / 瑜伽 / 普拉提 / 跳舞 / 综合）
- `training.preferred_cardio`（可选，自由文本，可多项 / 分隔，如 "跳绳 / 爬楼"）
- `training.gym_access`（可选，enum：membership / none / occasional / not_planned）
- `training.home_equipment`（可选，自由文本，例 "一对 5kg 哑铃 + 瑜伽垫" / "壶铃 12kg" / "无"）

### Batch 4 — 训练安全筛查 + 目标肌群（4 项，全可选；都没有让用户答"无"）

提问模板示例（接 Batch 3 用户答完后）：

> "再问几个跟身体相关的，避免推到不合适的动作（都没有就答'无'）：
>
> - 受过伤吗？比如哪个膝盖、哪侧腰、哪个肩？现在哪些动作完全做不了？
> - 久坐打工人常见的：颈肩僵硬 / 肩周炎 / 腱鞘炎 / 腰椎间盘 / 膝盖痛 —— 有任何一项请直接说。
> - 体态自评（不知道也行）：膝超伸 / O 型腿 / X 型腿 / 骨盆前倾 / 圆肩驼背 / 头前引？
> - 重点想练哪块？腰腹 / 臀腿 / 手臂 / 背 / 肩 / 全身均衡（多选都行）。"

涵盖字段：
- `body_status.injury_history`（可选，自由文本）
- `body_status.chronic_issues`（可选，自由文本）
- `body_status.posture_concerns`（可选，自由文本）
- `training.target_muscles`（可选，自由文本）

### Batch 5 — 饮食习惯 + 目标（2 个对象，多子项）

提问模板示例（接 Batch 4 用户答完后）：

> "最后聊聊吃饭和目标：
>
> - 工作日吃饭做饭 vs 外卖大致几比几？有不吃 / 过敏 / 特别爱吃的食物吗？
> - 家里常备哪些食材？（如 "燕麦 / 鸡蛋 / 紫薯 / 鸡胸 / 西兰花"，没有就答"基本不囤"）
> - 常点的外卖或常去的店？（如 "黄焖鸡 / 西贝 / 瑞幸"，方便我推你吃过的）
> - 想达到什么目标？可以是数字（比如「3 个月减 5kg、腰围 70」）也可以是自由表述（比如「改善腰腹线条」「拍婚纱照前瘦点」）。"

涵盖字段：
- `diet_pattern.homemade_pct` + `diet_pattern.delivery_pct`（可选，0-100，sum ≤ 100）
- `diet_pattern.preferences` + `diet_pattern.restrictions`（可选，自由文本）
- `diet_pattern.pantry_ingredients`（可选，自由文本，家里常备食材）
- `diet_pattern.favorite_takeout`（可选，自由文本，常点外卖品牌/菜系）
- `goal.target_weight_kg` + `goal.target_waist_cm` + `goal.horizon_weeks`（全部可选）
- `goal.freeform`（可选，建议至少这一项）

### 字段可选 / 必填判定依据

必填字段是后续 SKILL.md 路由 + cycle-rules.md 周期阶段判断必须依赖的最小集：`age` / `height_cm` / `weight_kg` / `activity_level` / `cycle.last_period_start` / `cycle.avg_cycle_days` / `cycle.avg_period_days`。其它字段缺失不影响基础建议生成，可逐步在后续对话中补全。

### activity_level 字母 → 枚举映射

- A → `sedentary`（每天坐 8+ 小时，几乎不走动）
- B → `light`（一天会走 30-60 分钟，无规律训练）
- C → `moderate`（每周 2-3 次中等强度训练，或日常通勤步行较多）
- D → `active`（每周 4+ 次训练，或体力劳动）

## 提问范式

- **3 批一次发完一批的所有字段**，不要拆成多轮问；用户回完一批再发下一批
- 用户某批只答了一部分必填字段：先收集已答的，礼貌补问缺失的必填项，**不要重发整批**
- 用户在 Batch 2/3 之前已主动给过某字段（如 Batch 1 时就说了体脂率）：直接记录，下批跳过该字段
- 给出枚举可选项时用 A/B/C/D 字母标号
- 用户给的单位不明（"我 60"）→ 复问"60 是 kg 吗？"
- **必填字段被跳过**：温柔解释为什么必填，再请一次；仍拒绝则放弃写盘
- **可选字段**：明确"不知道 / 不想说就跳过"，不追问第二次
- 用户给出明显异常值（年龄 2 岁 / 体重 500 kg / 月经日期在未来）：温柔复问"这个数值看着有点不太对，是不是写错了？"
- 不在 onboarding 中夹带建议或评价（如"你这体重有点高"），等 profile 写完后由 SKILL.md 路由再给建议
- Batch 之间不解释"现在进入第 X 批"，自然过渡即可

## 复述确认步骤

14 项采集完毕（含已标记跳过的可选字段）后，LLM **必须** 用一段话复述用户的语义画像，再请用户确认或指出错处。

模板示例：

> "好的，我整理一下：你叫 [name]，[age] 岁住在 [city]，身高 [height_cm] / 体重 [weight_kg]（体脂率 [body_fat_pct]% 或 未填），平均睡眠 [sleep_avg_hours]h（或 未填），活动量是 [activity_level]，平均周期 [cycle.avg_cycle_days] 天 / 经期 [cycle.avg_period_days] 天 / 上一次月经 [cycle.last_period_start]，主要 [homemade_pct vs delivery_pct] 自己做 vs 外卖（饮食偏好/禁忌：[preferences] / [restrictions]），目标是 [goal.freeform]（数字目标：[target_weight_kg] kg / [horizon_weeks] 周，如有）。哪里需要修改？"

- 用户回 "OK / 没问题 / 对" 等 → 进入写盘
- 用户指明某项错 → 单独改那一项 → 再复述一遍 → 直到用户确认
- 复述时被跳过的字段以"（未填）"标注，不要替用户编造默认值（除非该字段在 profile-schema.md 中明确规定有 default）
- 复述长度控制在 3-5 句，过长用户反而不读

## 写盘动作

写盘前 LLM 必须按顺序执行：

1. 确保 `~/.openclaw/workspace/fat-loss-agent/` 已存在（用 openclaw 提供的目录创建工具，mkdir -p 语义；缺失父目录 `~/.openclaw/workspace/` 由 openclaw 工作区根保证不会发生）
2. 从 openclaw 当前时间能力获取 UTC 时间戳，填入 `created_at` 与 `updated_at`
3. 设置 `schema_version = 1`，可选字段未填的按 profile-schema.md "null vs 省略" 约定省略对应 key
4. 调用 openclaw `write` 工具一次性写入完整 JSON 到 `~/.openclaw/workspace/fat-loss-agent/profile.json`
5. 写盘成功后，发送一句温暖过渡："profile 我记住啦。先给你看一下我打算怎么帮你达成目标——咱们对齐一下逻辑，避免之后每次都从头解释。"
6. **立即按 `./reports.md` §Report 1（目标可行性评估）输出固定 6 段报告**（画像 / 目标解读 + 可行性 / 分期重点 / 执行框架 / 沟通规则 / 求确认）。不等用户问。等用户回复"OK / 同意"或"我想调 X"后才进入正常对话路由。

**字段冲突处理**：若复述阶段用户回头改某项导致已写入字段语义冲突（如把 age 从 25 改到 55、改变了周期阶段判断），不需要重新走完整流程，只更新冲突项 + 刷新 updated_at（参考 profile-schema.md 单字段更新规约）。

## 错误恢复

- **数值不合法**（超出 schema 范围）：温柔复问，给出参考区间（如"身高一般在 140-200 cm 这个范围"）
- **日期格式错**（如说"4 月 15"省了年）：默认补今年，再请用户确认
- **末次月经日期 > 今天**：必须复问，不接受未来日期
- **末次月经日期 > 90 天前**：按 `state/profile-schema.md` "陈旧数据"约束处理，温和提醒用户更新
- **用户问元问题**（"你怎么知道这些 / 我的数据安全吗"）：暂停字段采集，回答后再接着问；强调本地存储、无外发
- **用户表达情绪**（"我已经试过很多次都失败了 / 我恨自己的身体"）：暂停 onboarding，先共情；必要时引用 `cycle-rules.md` §跨阶段共通原则的"暴食催吐"警示，建议心理咨询；待用户表态愿意继续再回到 onboarding
- **用户提到怀孕 / 哺乳 / 已知慢性病 / 服药**：先记录到备注，写盘前增加一句免责，引用 `cycle-rules.md` §跨阶段共通原则的"必须先咨询主治医生"
- **用户给出明显 ED 倾向数据**（如 BMI 已 < 17 仍要求"再瘦 10 kg"、明确说"不吃饭就行"）：立即暂停采集，参考 `cycle-rules.md` §跨阶段共通原则之 ED 提示语，建议先与心理 / 医疗专业人员沟通。**不写盘**（profile.json 维持不存在状态），把控制权交回 SKILL.md §安全与免责
- **用户切换语言**（中途改用英文 / 方言 / 表情包回复）：保持 LLM 自身一致语气，按用户语言复读关键术语一次确认理解

## 与其他文件的关系

- profile.json 字段语义、合法范围、写入规约：完全遵循 `state/profile-schema.md`
- onboarding 完成后，下一轮对话由 `SKILL.md` 路由（参考 `cycle-rules.md` + `lifestyle-tips.md` 给建议）
- 修改资料的单字段更新必须遵循 `state/profile-schema.md` "读 → 改单字段 → 刷新 updated_at → 写回" 规约
- 用户在 onboarding 中表现出 ED 信号 / 极端心理状态时的处理同步至 `cycle-rules.md` §跨阶段共通原则
- 写盘后是否触发"本周计划"建议依赖 `SKILL.md` 的路由策略，本文件仅负责把控制权交回
