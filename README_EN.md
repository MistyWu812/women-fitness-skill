# 🍑 Fat-Loss & Body Recomp AI Coach Skill (Women)

> [中文版本](./README.md)

A markdown knowledge pack that turns any LLM (Claude / GPT / Qwen / etc.) into a
women's fat-loss coach who understands menstrual cycles, Chinese convenience-store
food, delivery apps, and home workouts.

It models fat loss across the menstrual cycle (4 phases: menstrual / follicular /
ovulatory / luteal) plus non-cyclic paths (peri-menopause / menopause / lactation).

> ⚠️ Not medical advice. Cycle irregularities, abnormal bleeding, chronic disease,
> or pregnancy/lactation → consult a doctor.

## What's in the box

11 markdown files, split by topic:

- **Audience**: Default = adult Chinese-mainland urban woman (18+), no gym
  membership, mostly delivery + convenience store, sedentary office life
- **Conversation**: 5-batch onboarding flow that captures the full profile in
  ~5 turns
- **Nutrition**: BMR / TDEE / macros + per-meal kcal bands + ready-to-eat picks
  from chain stores + how to order on Meituan / Eleme
- **Training**: 3-day split (legs / chest+shoulders / back), upper-lower, and
  full-body circuit templates, with phase-specific intensity adjustments
- **Cycle math**: Compute today's phase from
  `last_period_start` + `avg_cycle_days` + `avg_period_days`, then return
  phase-appropriate training, food, and somatic-symptom advice
- **3 fixed reports**: ① goal feasibility ② weekly plan ③ daily plan
  (each is a 7-section A–G template with markdown tables and code blocks)

## File structure

```
SKILL.md                  # Main entrypoint + routing logic + 5-batch onboarding (verbatim)
state/profile-schema.md   # Data contract + cycle-phase calculation algorithm
onboarding.md             # First-run profile capture flow
cycle-rules.md            # 4 phases × 3 dimensions (training / diet / soma) advice DB
non-cyclic-rules.md       # Peri-menopause / menopause / lactation paths
lifestyle-tips.md         # Convenience-store / delivery / home-workout reference
weekly-split.md           # 3-day split / upper-lower / full-body circuit
kcal-targets.md           # BMR / TDEE / protein-fat-carb / water / training-day surplus
exercises.md              # Exercise library + contraindication map + equipment fallbacks
reports.md                # 3 fixed report templates
progress-checkins.md      # 4 feedback types (meals / workout / measurements / soma)
```

> Documentation language: the .md content is in Simplified Chinese, since the
> target audience and food/workout references are mainland-China specific. The
> structural concepts (BMR/TDEE/cycle phases/exercise contraindications) translate
> universally — you can prompt your LLM to respond in any language and the
> reasoning still holds.

## How to use

### A. Stuff it into a system prompt (simplest)

```bash
cat SKILL.md state/profile-schema.md cycle-rules.md non-cyclic-rules.md \
    lifestyle-tips.md weekly-split.md kcal-targets.md exercises.md \
    reports.md progress-checkins.md onboarding.md > system-prompt.md
```

Any long-context LLM works (Claude Sonnet / GPT-5 / qwen3-max recommended).

### B. As a Claude Code skill

```bash
mkdir -p ~/.claude/skills/fitness
cp -r * ~/.claude/skills/fitness/
```

### C. Build your own web chat

Repo doesn't ship with a web frontend, but the recipe is:
- OpenAI-compatible API
- Concat all 11 .md files into the system prompt
- Persist `profile.json` + chat history per user

## Adapting to your platform

Some files reference platform-specific paths
(`~/.openclaw/workspace/fat-loss-agent/profile.json`) and tools
(`openclaw read/write` calls) that come from the original author's runtime.
To adapt:

- Replace `~/.openclaw/workspace/fat-loss-agent/profile.json` with your own
  profile storage path (any local file or DB row keyed by user)
- Replace `openclaw read/write tool` references with your LLM platform's file
  capability — Claude Code has built-in Read/Write, OpenAI Assistants has
  files API, custom backends use a DB
- Replace `<UPDATE_PROFILE>{...json...}</UPDATE_PROFILE>` marker protocol with
  whatever your backend can parse from streamed assistant output

## Data principles

- **Local-first**: `profile.json` lives on the user's machine by default,
  no telemetry, no analytics
- **Disposable**: users can wipe their data by deleting the file
- **Not medical**: no diagnostic language; cycle irregularities / amenorrhea /
  abnormal bleeding / chronic conditions / pregnancy / lactation → always
  redirect to a doctor

## Methodology stance

The skill internalizes 5 broad paradigms common in Chinese women's fat-loss
content (without naming any specific influencer):

| Paradigm | Core idea |
|---|---|
| Anti-diet | Loose calorie tracking + healthy relationship with food |
| Strength-first | Compound lifts + 1.6–2.0 g/kg protein |
| Lean-toned aesthetic | Pilates / moderate weights + sugar control + early sleep |
| Posture correction | Anterior pelvic tilt / kyphosis → glute med activation + stretching |
| Periodic indulgence | One cheat meal per week for psychological sustainability |

The skill never speaks on behalf of any named individual — all methodologies
have been distilled, fused with cycle awareness, and adapted to mainland-China
lifestyle constraints.

## License

MIT — see [LICENSE](LICENSE). Free to use, modify, redistribute, including
commercially. Please keep the copyright notice.

## Disclaimer

This is not medical advice. Not a registered dietitian. Not a physical therapist.
All training / nutrition / cycle suggestions are second-hand integrations of
public methodology and do not replace professional medical care. For cycle
irregularities, amenorrhea, severe pain, abnormal bleeding, post-fracture
rehab, pregnancy, or lactation → consult a doctor / RD / licensed PT.
