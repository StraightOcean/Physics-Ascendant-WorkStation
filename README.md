<h1 align="center">Physics Ascendant — 创意工坊</h1>
<h3 align="center">Community MOD Repository</h3>

<p align="center">
  <b>为《物理法则之下：科技死斗》创建和分享自定义卡牌扩展包</b>
</p>

---

## 快速开始

MOD 就是一个 **JSON 文件**。写一个 `your-mod.json`，放到游戏的 `mods/` 目录，启动游戏即可自动加载。

### 最小示例

```json
{
  "id": "my-first-mod",
  "name": "我的第一个MOD",
  "version": "1.0.0",
  "engineVersion": "0.2.0",
  "author": "你的名字",
  "description": "新增一张自定义卡牌",
  "cards": [
    {
      "id": "quantum_boost",
      "name": "量子强化",
      "type": "资源",
      "cost": 1,
      "level": 0,
      "isUpgradeReward": false,
      "description": "能量+3，抽1张牌。",
      "quantity": 4,
      "effects": [
        { "type": "gain_energy", "amount": 3 },
        { "type": "draw_cards", "source": "main", "count": 1 }
      ],
      "addToMainDeck": true
    }
  ]
}
```

### 目录结构

```
physics-ascendant/
└── mods/                          # 游戏读取此目录
    ├── my-mod.json                # 你的 MOD 文件
    └── another-mod.json           # 可以放多个 MOD
```

游戏启动时会自动扫描 `mods/` 目录并加载所有 `.json` MOD 文件。

---

## MOD 包结构（完整参考）

```typescript
interface ModPackage {
  id: string;                    // 唯一标识，如 "neutron-star-nexus"
  name: string;                  // 显示名称
  version: string;               // 语义版本号 "1.0.0"
  engineVersion: string;         // 目标引擎版本 "0.2.0"
  author?: string;               // 作者署名
  description?: string;          // 简介

  cards: ModCardDef[];           // 新增卡牌（必需）
  effectTypes?: ModEffectTypeDef[];    // 自定义效果类型（高级）
  cosmicEvents?: ModCosmicEventDef[];  // 自定义宇宙骰子事件
  upgrades?: ModUpgradeDef[];          // 自定义升级等级
}
```

---

## 卡牌定义

每张卡牌需要定义 ID、名称、费用、等级、效果和数量。

### 卡牌字段

| 字段 | 类型 | 必需 | 说明 |
|:---|:---|:---:|:---|
| `id` | string | ✅ | 唯一标识，全小写下划线分隔 |
| `name` | string | ✅ | 显示名称 |
| `type` | string | ✅ | 分类：`"资源"` `"部署"` `"位移"` `"攻击"` `"防御"` `"科技"` |
| `cost` | number | ✅ | 打出费用（0-10） |
| `level` | number | ✅ | 科技等级（0=基础，1-6=对应科技等级） |
| `description` | string | ✅ | 卡牌描述文本 |
| `quantity` | number | ✅ | 洗入牌库的张数 |
| `isUpgradeReward` | boolean | ✅ | 是否为升级即得卡（升级时自动获得，不放副牌库） |
| `effects` | array | ✅ | 效果列表（见下方） |
| `addToMainDeck` | boolean | ❌ | 是否加入基础牌库（仅 level=0 有效） |
| `addToTechDeckLevel` | number | ❌ | 加入第几级科技副牌库（仅 level>0 有效） |

### 卡牌分类推荐

| type | 说明 | 推荐 level |
|:---|:---|---|
| `"资源"` | 获得/消耗能量、降低熵值 | 0 |
| `"部署"` | 生成、转化粒子 | 0-2 |
| `"位移"` | 移动、交换、推动粒子 | 0-2 |
| `"攻击"` | 增加对手熵值、移除对手粒子 | 0-4 |
| `"防御"` | 护盾、免熵、抽牌 | 1-4 |
| `"科技"` | 综合/高级效果 | 3-6 |

---

## 效果系统

游戏内置 **26 种原子效果**，组合使用即可创建无穷变化的卡牌。每个效果在 `effects` 数组中顺序执行。

### 资源类

| 效果类型 | 参数 | 示例 |
|:---|:---|:---|
| `gain_energy` | `amount`, `canExceedCap?` | `{ "type": "gain_energy", "amount": 3 }` |
| `gain_entropy` | `target`, `amount` | `{ "type": "gain_entropy", "target": "opponent", "amount": 2 }` |
| `reduce_entropy` | `amount` | `{ "type": "reduce_entropy", "amount": 1 }` |
| `transfer_entropy` | `from`, `to`, `amount`, `compensateEnergy?` | `{ "type": "transfer_entropy", "from": "self", "to": "opponent", "amount": 1 }` |
| `gain_shield` | `amount` | `{ "type": "gain_shield", "amount": 3 }` |
| `increase_max_entropy` | `amount` | `{ "type": "increase_max_entropy", "amount": 5 }` |

#### target 取值

| 值 | 含义 |
|:---|:---|
| `"self"` | 自己 |
| `"opponent"` | 选定的对手 |
| `"all_others"` | 所有其他玩家 |
| `"all"` | 所有玩家（含自己） |

### 粒子操作类

| 效果类型 | 参数 | 说明 |
|:---|:---|:---|
| `spawn_particle` | `player`, `pos`, `particle` | 在空格生成粒子 |
| `move_particle` | `player`, `from`, `direction`, `steps` | 直线移动 |
| `diagonal_move` | `player`, `from`, `direction` | 斜向移动1格 |
| `push_particle` | `player`, `origin`, `direction`, `dealDamageOnCollision?` | 推出碰撞 |
| `pull_particle` | `player`, `target`, `direction` | 拉近1格 |
| `remove_particle` | `player`, `pos`, `entropyPenalty?`, `energyReward?` | 移除粒子 |
| `swap_particles` | `player`, `p1`, `p2` | 交换两个粒子位置 |
| `transform_particle` | `player`, `pos`, `from`, `to` | 粒子类型转变 |
| `merge_particles` | `player`, `from`, `to`, `requireAdjacent?` | 合并两个同种粒子 |

#### particle / from / to 取值

| 值 | 含义 |
|:---|:---|
| `"Q"`, `"E"`, `"P"` | 正物质（夸克/电子/质子） |
| `"Ā"`, `"Ē"`, `"P̄"` | 反物质 |
| `"choose_q_or_e"` | 由玩家选择 Q 或 E（需 UI 交互） |
| `"any_regular"` | 任意正物质粒子（Q/E/P） |
| `"its_antimatter"` | 自动转换为对应反粒子 |
| `"empty_slot"` | 自动选一个空格 |

#### player 取值

| 值 | 含义 |
|:---|:---|
| `"self"` | 自己的实验场 |
| `"opponent"` | 选定对手的实验场 |
| `"any"` | 任意玩家（需 UI 选位） |

#### direction 取值

| 值 | 效果 |
|:---|:---|
| `"up"`, `"down"`, `"left"`, `"right"` | 四方向 |
| `"up-left"`, `"up-right"`, `"down-left"`, `"down-right"` | 四对角线（仅 diagonal_move） |

### 手牌/牌库类

| 效果类型 | 参数 |
|:---|:---|
| `draw_cards` | `source`: `"main"` / `"tech"`, `count` |
| `discard_cards` | `player`, `count`, `highestTechOnly?` |
| `steal_card` | `from`: `"opponent"`, `count`, `random?` |
| `shuffle_hand` | `keepTech?`, `drawCount` |

### 特殊类

| 效果类型 | 参数 | 说明 |
|:---|:---|:---|
| `lock_particle` | `player`, `pos` | 锁定粒子一回合 |
| `random_move_particle` | `player` | 随机移动一个粒子 |
| `board_permute` | — | 棋盘变换（2人颠倒/4人旋转） |
| `shuffle_particles` | `player` | 打乱所有粒子 |
| `rearrange_lab` | — | 重新排列实验场 |
| `extra_turn` | — | 获得额外回合 |
| `skip_turn` | `target` | 跳过目标回合 |
| `cosmic_expansion` | — | 实验室扩展为 5×5 |
| `quantum_fluctuation` | — | 量子涨落（粒子随机转化） |
| `direct_upgrade` | `level` | 直接提升研究所等级 |
| `conditional` | `condition`, `then`, `else?` | 条件判断（如猜粒子） |
| `placeholder` | `cardId` | 暂未实现的效果占位符 |

---

## 自定义效果类型（高级）

如果你的卡牌需要全新逻辑（内置效果无法表达），可以注册自定义效果处理器：

### 1. 在 MOD JSON 中声明

```json
"effectTypes": [
  {
    "type": "neutron_decay",
    "paramsSchema": {
      "targetPos": "目标位置 GridPos",
      "particleType": "粒子类型 ParticleType"
    },
    "description": "移除粒子获取能量并对敌人造成熵增"
  }
]
```

### 2. 在代码中注册处理器（高级用户）

```typescript
import { registerEffectHandler } from '@physics-ascendant/engine';

registerEffectHandler('neutron_decay', (state, effect, params) => {
  // 你的自定义逻辑
  // 返回出局玩家 ID 数组
  return [];
});
```

> 纯 JSON MOD 无法注册自定义处理器。如需新效果类型，请提交 PR 到引擎仓库，或使用内置 26 种效果组合实现。

---

## 宇宙骰子事件

自定义骰子事件会追加到内置 6 种事件之后：

```json
"cosmicEvents": [
  {
    "id": 7,
    "name": "中子星碰撞",
    "description": "所有玩家熵值+1，各抽取2张牌作为补偿"
  }
]
```

| 字段 | 说明 |
|:---|:---|
| `id` | 事件编号（7-99，避免与内置 1-6 冲突） |
| `name` | 事件名称 |
| `description` | 事件描述 |
| `effects` | 可选，事件触发时执行的效果列表 |

---

## 完整示例

### 示例 1：简单资源卡

```json
{
  "id": "particle_beam",
  "name": "粒子射线",
  "type": "攻击",
  "cost": 3,
  "level": 2,
  "isUpgradeReward": false,
  "description": "移除对手场上1个粒子，对手熵值+2。",
  "quantity": 2,
  "effects": [
    { "type": "remove_particle", "player": "opponent", "pos": { "row": 0, "col": 0 } },
    { "type": "gain_entropy", "target": "opponent", "amount": 2 }
  ],
  "addToTechDeckLevel": 2
}
```

### 示例 2：多效果终极卡

```json
{
  "id": "magnetar_burst",
  "name": "磁星爆发",
  "type": "科技",
  "cost": 8,
  "level": 6,
  "isUpgradeReward": false,
  "description": "所有对手熵值+2，你获得3点护盾。",
  "quantity": 2,
  "effects": [
    { "type": "gain_entropy", "target": "all_others", "amount": 2 },
    { "type": "gain_shield", "amount": 3 }
  ],
  "addToTechDeckLevel": 6
}
```

### 示例 3：升级即得卡

```json
{
  "id": "quark_gluon_plasma",
  "name": "夸克-胶子等离子体",
  "type": "科技",
  "cost": 10,
  "level": 6,
  "isUpgradeReward": true,
  "description": "【升级即得】生成2个夸克(Q)并获得4点能量。",
  "quantity": 1,
  "effects": [
    { "type": "spawn_particle", "player": "self", "pos": "empty_slot", "particle": "Q" },
    { "type": "spawn_particle", "player": "self", "pos": "empty_slot", "particle": "Q" },
    { "type": "gain_energy", "amount": 4, "canExceedCap": true }
  ],
  "addToTechDeckLevel": 6
}
```

---

## 发布你的 MOD

### 提交到本仓库

1. Fork 本仓库
2. 在 `mods/` 目录下创建你的 `<mod-id>.json` 文件
3. 在 `mods/README.md` 表格中添加一行介绍
4. 提交 PR

### 命名规范

- `id` 使用全小写下划线：`neutron-star-nexus`
- 文件名与 `id` 一致：`neutron-star-nexus.json`
- 版本号遵循语义版本：`major.minor.patch`

### PR 检查清单

- [ ] MOD 文件为合法 JSON（可用 [JSONLint](https://jsonlint.com/) 验证）
- [ ] `id` 不与已有 MOD 冲突
- [ ] 卡牌 `id` 不与官方卡牌冲突（查看 [Cards.md](../Cards.md)）
- [ ] `engineVersion` 与当前引擎版本匹配
- [ ] 所有效果使用的类型在本文档的效果系统中存在
- [ ] 卡牌设计平衡（费用与效果对等）

---

## 效果速查表

| 类别 | 可用类型 | 数量 |
|:---|---:|---:|
| 资源 | gain_energy, gain_entropy, reduce_entropy, transfer_entropy, gain_shield, increase_max_entropy | 6 |
| 粒子 | spawn_particle, move_particle, diagonal_move, push_particle, pull_particle, remove_particle, swap_particles, transform_particle, merge_particles | 9 |
| 手牌 | draw_cards, discard_cards, steal_card, shuffle_hand | 4 |
| 特殊 | lock_particle, random_move_particle, board_permute, shuffle_particles, rearrange_lab, extra_turn, skip_turn, cosmic_expansion, quantum_fluctuation, direct_upgrade, conditional, placeholder | 12 |

---

## 许可证

本仓库中的 MOD 作品遵循其作者指定的许可证。官方示例 MOD 使用 MIT License。

提交 MOD 即表示你同意将其以开源方式共享给社区。
