# combustion_extinction labeled cases

判据 [is_extinguished](../../../sim-parse/docs/criteria_reference.md#531-is_extinguished核心本项目重点示例) 的真值集。

## 标注规则

| 真值 | 判定标准（人工，看 ParaView / 时序） |
|---|---|
| `is_extinguished: true` | Qdot 域内峰值 < ~1e6 W/m³（或基准的 5%）<br> AND T_max - T_inlet < ~100K<br> AND OH 几乎不存在（< 1e-6） |
| `is_extinguished: false` | 上述任一不满足（即火焰存在并维持） |

## 目标 case 数（min）

- 4× `is_extinguished: true`（不同燃料/工况）
- 4× `is_extinguished: false`（标准燃烧）

## 现有

| Case | is_extinguished | 备注 |
|---|---|---|
| `DLR_A_cavity_normal/` | false | DLR-A 标准 CH4-H2 jet flame，已收敛、火焰稳定 |

## 待补

| 候选 case | 期望真值 | 来源建议 |
|---|---|---|
| CH4_jet_blowout | true | OpenFOAM tutorial 中 reactingFoam case 故意把 inlet velocity 调到吹熄 |
| H2O2_extinguished | true | 高拉伸率工况 |
| H2_O2_normal | false | 标准 H2 火焰 |
| kerosene_jet_normal | false | 高压煤油算例 |
| ...（自行补） | | |
