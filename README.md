# sim-knowledge

> **状态**：v0.2 — 物理判据 + 格式知识 首批完整化
> **架构**：见 [sim-parse/docs/design.md §2 三轴架构](../sim-parse/docs/design.md)

CAE **物理判据 + 格式探索**的结构化知识库，是 sim-* 生态中"**判**"和"**读**"两个动作的载体。

## 三轴中的位置

| 动作 | 仓库 | 内容 |
|---|---|---|
| 跑 (run) | `sim-skills/` | 求解器操作（markdown） |
| 读 (extract) | **`sim-knowledge/formats/`** ✅ | 格式识别 / quirks / playbooks |
| 判 (judge) | **`sim-knowledge/physics/`** ✅ | 物理判据 / 变量字典 / 阈值表 |
|  + | **`sim-knowledge/labeled_cases/`** ⚠️ 1/50 | 标注真值（机制 #2 校准检验依据） |

## 当前内容（v0.2）

```
sim-knowledge/
├── README.md                                              ← 本文档
├── physics/                                               ← "判"
│   ├── domain_inference.yaml      ← meta-routing: case 信号 → 候选 domain（sim-parse Tier 7 candidate_domains 消费）
│   ├── combustion/
│   │   ├── criteria.yaml          (7 intents)
│   │   ├── variables.yaml         (10 canonical vars + cross-solver aliases)
│   │   └── thresholds.yaml        (5 threshold tables, per-fuel)
│   ├── numerics/
│   │   ├── criteria.yaml          (4 intents)
│   │   └── variables.yaml         (4 canonical vars)
│   ├── fluid_dynamics/
│   │   ├── criteria.yaml          (6 intents)
│   │   └── variables.yaml         (8 canonical vars + QOI mappings)
│   ├── heat_transfer/
│   │   ├── criteria.yaml          (4 intents)
│   │   └── variables.yaml         (4 vars + wall_heat_flux QOI)
│   └── structural/
│       ├── criteria.yaml          (4 intents)
│       └── variables.yaml         (5 vars: stress/displacement/strain/...)
├── formats/                                               ← "读"
│   ├── openfoam/
│   │   ├── structure.yaml         (识别签名 + solver 列表 + 目录约定)
│   │   ├── quirks.yaml            (14 实战 quirks)
│   │   └── extraction_map.yaml    (字段 → 来源)
│   ├── hdf5/structure.yaml        (HDF5 + 子格式 disambiguation)
│   ├── cgns/structure.yaml
│   ├── fluent/structure.yaml      (legacy + CFF 双格式)
│   ├── lsdyna/structure.yaml
│   ├── conventions/
│   │   └── log_residual_patterns.yaml  (跨求解器 log 残差正则)
│   └── playbooks/
│       └── unknown_hdf5_inspection.yaml  (Discovery Agent 探索剧本)
└── labeled_cases/                                          ← 真值锚点
    ├── README.md
    ├── _schema.yaml                                        ← ground_truth.yaml 字段 schema
    ├── TEMPLATE_ground_truth.yaml                          ← 新 case 标注模板
    └── combustion_extinction/
        ├── README.md
        └── DLR_A_cavity_normal/
            └── ground_truth.yaml                           ← 1 个真值（待补 4-9 个）
```

**统计**：

- **physics/**：5 个域、25 个 intent、40+ canonical 变量定义、阈值表 5 张
- **formats/**：5 种格式（openfoam/hdf5/cgns/fluent/lsdyna）+ 跨格式 conventions + 1 个 playbook
- **labeled_cases/**：1 个真值标注 + 1 套 schema + 模板

## sim-parse 自动消费

sim-parse 从 v0.7+ 起，会**自动检测**到这个 sibling 仓库并合并 physics/criteria.yaml 进 Tier 5 规则集。

实测（DLR_A_cavity）：
- 仅 sim-parse 自带规则：14 条 QOI
- 加 sim-knowledge：**27 条 QOI**（多 13 条物理判据）

### 用法（从 sim-parse 角度）

```python
from sim_parse import parse_case

# 1. 默认：sim-parse 自带 + sim-knowledge（如果存在）合并
r = parse_case(case, target_tier=5)
# tier_5_qoi 自动包含合并后的所有 27+ 条规则

# 2. 完全替换：用自己的 YAML
r = parse_case(case, target_tier=5, qoi_rules_path="my_rules.yaml")
```

## 内容来源（按 ROI 三档）

按 [design.md §11.3](../sim-parse/docs/design.md) 的来源分档：

| 档 | 占比 | 当前状态 |
|---|---|---|
| **Gold**（人写 + 人审） | ~20% | ✅ physics/combustion 的 is_extinguished 由我们手写 + 文献引用 |
| **Silver**（LLM 起草 + 人审） | ~50% | ⚠️ 当前几乎都是手写；后续可靠 LLM 加速 |
| **Bronze**（用户查询自动提议） | ~30% | ❌ 需 sim-query Agent 跑起来后才有 |

## 未来工作

| 任务 | 优先级 | 估时 |
|---|---|---|
| labeled_cases 第二批（每 intent +5 个 case） | ⭐⭐⭐⭐⭐ | 1-2 周（人工） |
| 加更多格式（Tecplot / EnSight / Plot3D / Abaqus） | ⭐⭐⭐ | 1 周 |
| Phase 2 表达式判据（如 `T_max - T_inlet`） | ⭐⭐⭐⭐ | 几天 |
| LLM-assisted Silver 起草（从教科书抽 YAML） | ⭐⭐⭐⭐ | 几天 |
| CI 同步检查（sim-parse builtin ↔ sim-knowledge） | ⭐⭐⭐ | 半天 |
| Discovery Agent 真实实现（消费 formats/ + playbooks/） | ⭐⭐ | 1-2 周 |
| Tier 7 LLM 集成（消费 physics/ + few-shot） | ⭐⭐ | 1-2 周 |

## 文档配套

- [sim-parse/docs/design.md](../sim-parse/docs/design.md) — 总架构（三轴 / Tier / Path）
- [sim-parse/docs/extraction_reference.md](../sim-parse/docs/extraction_reference.md) — "读"轴细节（11 种格式）
- [sim-parse/docs/criteria_reference.md](../sim-parse/docs/criteria_reference.md) — "判"轴细节（5 个物理域）

本仓库的 YAML 是上面两份 markdown 的**结构化执行版**。markdown 给人读，YAML 给 LLM Agent 检索。
