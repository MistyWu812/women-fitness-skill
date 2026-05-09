# 动作清单 + 禁忌映射 + 器械逻辑

本文件给 LLM 在生成训练菜单时三件事的依据：
1. **可用动作清单**（按器械分组）
2. **禁忌动作映射**（受伤 / 体态 / 慢性问题 → 禁动作 + 替代）
3. **选动作的器械逻辑**（按 profile.training.gym_access / home_equipment 决定推什么）

**动作输出仅给文字（动作名 + 组数 × 次数），不附图、不附 URL**。用户需要看动作示范时，引导她搜 B 站 / 抖音 / Keep / YouTube 上的演示视频。

## 禁忌动作映射（必读，推每个动作前先过一遍）

读 `profile.body_status` 三个字段（injury_history / chronic_issues / posture_concerns）+ `training.target_muscles`，先过滤再选动作。

### 受伤 / 慢性问题

| 主诉关键词 | 禁动作 | 替代 / 调整 |
|---|---|---|
| 膝盖痛 / 半月板 / 髌骨软化 | 深蹲跳、波比跳、弓步深下蹲、登山者快速版、跳箱 | 浅蹲（仅至椅高）、靠墙静蹲、桥式、死虫子、鸟狗式、侧躺抬腿 |
| 膝超伸 | 站立时锁死膝盖的所有动作（深蹲到底、空中蹬车伸直腿） | 深蹲时膝盖保持微屈 5°、桥式、靠墙静蹲限制角度 |
| 腰椎间盘 / 椎间盘突出 / 腰肌劳损 | 仰卧起坐 / 标准卷腹（屈髋类）、硬拉变式、负重体前屈、负重侧弯 | 死虫子、鸟狗式、桥式（轻强度）、侧平板 |
| 腰肌酸痛（无突出） | 高强度核心、长时间平板、负重深蹲 | 桥式、死虫子、温和瑜伽流（猫牛 / 婴儿式） |
| 肩周炎 / 肩痛 / 旋肩袖损伤 | 标准俯卧撑、肩推、引体、波比跳（含俯撑） | 跪姿俯卧撑（仅在不痛时）、墙俯卧撑、下肢主导动作（深蹲 / 弓步 / 桥式）、弹力带轻拉 |
| 腱鞘炎 / 手腕痛 | 撑地动作（俯卧撑、平板、波比跳）、抓握类（壶铃摆动） | 前臂支撑平板、拳支撑俯卧撑（如不加重痛感）、下肢 + 核心动作 |
| 颈椎病 / 颈僵 | 倒立、撑头卷腹（手压脖子）、长时间俯卧（如标准平板低头版） | 仰卧动作（死虫子 / 桥式）、瑜伽流避免颈部承重 |
| 痛经剧烈（经期） | 全部高强度（HIIT / 力量 / 核心） | 散步、阴瑜伽、热敷拉伸；详见 `cycle-rules.md` §menstrual |
| 偏头痛 | 倒立、低头长时间 | 直立 / 仰卧动作 |

### 体态问题（强化方向：体态修正派常用 6 动作清单）

| 体态 | 重点强化 | 重点拉伸 / 松解 |
|---|---|---|
| 骨盆前倾 | 臀大肌（桥式 / 单腿臀桥）、腹横肌（死虫子 / 鸟狗式） | 髂腰肌（弓步拉伸）、竖脊肌 |
| 骨盆后倾 | 竖脊肌（鸟狗式背伸版）、髂腰肌（抬腿） | 臀肌、腘绳肌 |
| 圆肩 / 驼背 | 背肌（哑铃划船 / 弹力带反向飞鸟） | 胸肌、前三角 |
| 头前引 | 颈深屈肌（仰卧点头）、菱形肌 | 上斜方肌、胸锁乳突肌 |
| 膝超伸 | 腘绳肌、臀肌 | 股四头（小心拉伸防过度）、避免锁膝习惯 |
| O 型腿（膝外翻） | 臀中肌（侧平板抬腿、蚌式） | 髂胫束 |
| X 型腿（膝内扣） | 臀中后肌（侧抬腿、蚌式）、足底稳定 | 内收肌 |

### 目标肌群优先动作

| `training.target_muscles` 关键词 | 优先选 |
|---|---|
| 腰腹 / 核心 | 平板、死虫子、卷腹（无腰伤前提）、鸟狗式 |
| 臀腿 / 翘臀 | 桥式、单腿臀桥、深蹲（无膝伤前提）、弓步、相扑深蹲 |
| 手臂 | 哑铃二头弯举（如有哑铃）、跪姿俯卧撑（如无肩腕伤） |
| 背 | 哑铃俯身划船（如有哑铃）、弹力带反向飞鸟、超人式 |
| 肩 | 哑铃肩推（如有哑铃 + 无肩伤）、弹力带肩外旋 |
| 全身均衡 | 复合动作 50% + 核心 25% + 拉伸 25% |

### 决策顺序

LLM 设计训练菜单时**严格按以下步骤**：

1. 先看 `body_status.injury_history` —— 列出该用户**绝对禁动作清单**（严格于上表）
2. 再看 `body_status.chronic_issues` + `posture_concerns` —— 调整重点 + 替代动作
3. 然后按 `cycle-rules.md` 当前 phase 给出强度区间
4. 然后按 `training.preferred_modality` + `training.preferred_cardio` 选动作类型
5. 最后按 `training.home_equipment` + `gym_access`（详见 §选动作的器械逻辑）选 bodyweight 还是负重组
6. 按 `training.target_muscles` 给优先权重

每个最终选出的动作再回头对一次禁忌表，**禁忌优先于一切偏好**。

## 选动作的器械逻辑

输出训练菜单前，**先读 profile.training.home_equipment 与 training.gym_access**，按以下优先级挑动作：

1. **有月卡 / 健身房**：可推全表（含哑铃 / 壶铃组）
2. **家里有哑铃**：bodyweight 表 + 哑铃负重表
3. **家里有壶铃**：bodyweight 表 + 壶铃负重表（哑铃动作多数也能用壶铃替代，明确告知用户"用壶铃做即可"）
4. **只有瑜伽垫 / 弹力带 / 全徒手**：仅 bodyweight 表，**禁止推哑铃 / 壶铃动作**（用户做不了反而挫败）
5. **未填**：默认走 bodyweight 表 + 加一句"如果家里有哑铃可以告诉我，能给你升级版动作"

## 动作清单（仅文字，不带图）

### bodyweight 组（无器械可用）

| 中文名 | 类别 | 适配 phase | 备注 |
|---|---|---|---|
| 自重深蹲 | strength | follicular / ovulatory / luteal 早期 | 膝盖友好版：浅蹲至椅高 |
| 弓步走 | strength | follicular / ovulatory | 膝盖敏感者改原地弓步 |
| 标准俯卧撑 | strength | follicular / ovulatory | 肩腕敏感者改跪姿 |
| 跪姿俯卧撑 | strength | menstrual 后期 / follicular / ovulatory / luteal | 入门款 |
| 平板支撑 | strength | follicular / ovulatory / luteal 早期 | 腰伤改侧平板 |
| 单腿臀桥 | strength | 全周期友好 | 用瑜伽垫保护尾椎 |
| 双腿臀桥 | strength | 全周期友好 | 入门款 |
| 死虫子 | strength | 全周期友好 | 核心训练首选 |
| 鸟狗式 | strength | 全周期友好 | 体态修正友好 |
| 卷腹 | strength | follicular / ovulatory | luteal 慎做核心 |
| 侧平板（抬腿）| strength | follicular / ovulatory | 体态修正派：纠正 O 型腿 |
| 蚌式 | strength | 全周期友好 | 体态修正派：臀中肌激活 |
| 消防栓式 | strength | 全周期友好 | 体态修正派：臀外侧 |
| 站姿侧抬腿 | strength | 全周期友好 | 体态修正派：纠正假胯宽 |
| 跪姿后抬腿 | strength | 全周期友好 | 体态修正派：臀大肌 |
| 宽距深蹲（自重）| strength | follicular / ovulatory / luteal 早期 | 体态修正派：纠正假胯宽 |
| 鸽子式 | stretching | 全周期 | 髋部拉伸 |
| 猫牛式 | stretching | 全周期 | 脊柱灵活性 |
| 婴儿式 | stretching | 全周期 | 拉伸 + 放松 |

### 有氧 / 高强度 bodyweight

| 中文名 | 类别 | 适配 phase | 备注 |
|---|---|---|---|
| 开合跳（小幅）| cardio | follicular / ovulatory / luteal 早期 | 膝敏感者降幅 |
| 开合跳（全幅，类大字跳）| plyometrics | follicular / ovulatory | 高强度 |
| 波比跳 | plyometrics | follicular / ovulatory | 关节敏感慎选 |
| 登山者 | plyometrics | follicular / ovulatory | 腕腰敏感慎做 |
| 高抬腿 | plyometrics | follicular / ovulatory | |
| 提膝跳 | plyometrics | follicular / ovulatory | |
| 深蹲跳 | plyometrics | follicular / ovulatory | 膝伤禁 |
| 爆发式俯卧撑 | plyometrics | follicular / ovulatory | 进阶 |

### 负重组（需哑铃 / 壶铃）

| 中文名 | 器械 | 适配 phase | 备注 |
|---|---|---|---|
| 哑铃深蹲 | 一对哑铃 | follicular / ovulatory / luteal 早期 | |
| 哑铃相扑深蹲 | 一只哑铃 | follicular / ovulatory | 内收肌 + 臀外侧 |
| 高脚杯深蹲 | 壶铃或一只重哑铃 | follicular / ovulatory / luteal 早期 | |
| 哑铃弓步 | 一对哑铃 | follicular / ovulatory | |
| 哑铃俯身划船 | 一对哑铃 | follicular / ovulatory / luteal 早期 | 圆肩驼背用户重点推 |
| 哑铃肩推 | 一对哑铃 | follicular / ovulatory | 肩伤禁 |
| 哑铃二头弯举 | 一对哑铃 | 全周期（轻强度也适合 menstrual）| |
| 直腿哑铃硬拉 | 一对哑铃 | follicular / ovulatory | 腰伤禁 |
| 单臂壶铃摆动 | 壶铃 | follicular / ovulatory | 腰伤禁 |

### 不在表内的动作

LLM 给训练菜单时**只用本表里的动作**。如果一定要推表外动作（如瑜伽流 / 普拉提具体体式 / 跳绳）：

- 给文字描述（不需配图）
- 加一句"建议在 B 站 / 抖音 / Keep 上搜「动作名」看演示视频"
- 不要瞎编动作

## v0.1 扩充方向

待用户测试后再决定是否：
- 增加瑜伽 / 普拉提专项动作清单（目前只有少数）
- 增加跳绳 / 爬楼 / 游泳 等户外或器械类（目前缺）
- 是否再引入示范图（小红书 / B 站 / 自建素材）
