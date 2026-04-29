# labeled_cases

人工标注的真值案例库，是 [Agent 准确性机制 #2](../../sim-parse/docs/design.md#11-agent-临场推断的支撑机制)（校准检验）的真值依据。

## 用途

每个 intent（如 `is_extinguished` / `is_converged`）需要 **5-10 个 labeled cases**：
- 包含 positive 真值（如熄火案例）和 negative 真值（如正常燃烧案例）
- 用于自动校准判据阈值
- 用于审核新临场判据是否准确

## 目录约定

```
labeled_cases/
├── _schema.yaml                       ← ground_truth.yaml 字段 schema
├── <intent_or_domain>/                ← 按物理域 / intent 分组
│   ├── README.md
│   └── <case_id>/
│       └── ground_truth.yaml          ← 必有
└── ...
```

例：

```
combustion_extinction/
├── DLR_A_cavity_normal/
│   └── ground_truth.yaml             # is_extinguished: false
├── CH4_jet_blowout/
│   └── ground_truth.yaml             # is_extinguished: true
└── ...
```

## ground_truth.yaml 必有字段

见 [_schema.yaml](_schema.yaml)。简要：

- `case_id` —— 唯一标识
- `case_path` —— 案例文件夹绝对路径（或相对 sim-knowledge 的）
- `labels` —— 真值（一个或多个 intent → 期望值）
- `context` —— 物理上下文（fuel, pressure, mechanism, ...）
- `labeled_by` / `labeled_at` —— 谁标的什么时候标的
- `notes` —— 自由文本说明

## 标注成本

每个 case 5-15 分钟（看 ParaView / 残差曲线 / log 末尾后判定）。

5 个 intent × 8 个 case × 10 分钟 = **每个物理域约 5-8 小时人工**。

## 当前

- ✅ `combustion_extinction/DLR_A_cavity_normal/` —— sim-parse Phase 1 验收
- 🚧 其他全部待补
