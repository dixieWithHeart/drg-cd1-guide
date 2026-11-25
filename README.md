# CD1 自定义难度配置手册

> 把这份文档粘给网页 AI，让它生成或修改 CD1 JSON。只含生成所需的知识：生成规则、46 键、基线、字段语义、敌人、池、校验、配方。

## 目录

- [0. 生成规则（先读）](#0-生成规则先读)
- [1. 参数边界与需求映射](#1-参数边界与需求映射)
- [4. 配置结构](#4-配置结构)
- [5. 字段参考](#5-字段参考)
- [6. 敌人](#6-敌人)
- [7. 池操作](#7-池操作)
- [8. 校验与排错](#8-校验与排错)
- [9. 配方库](#9-配方库)
- [10. 常见坑](#10-常见坑)
- [12. 附录：Haz5 基线](#12-附录haz5-基线)

> 章节编号沿用完整参考；跳过的 2 / 3 / 11 是未收录进本手册的内容（机制原理、版本时间线、待核实清单），需要时查完整版。

---

## 0. 生成规则（先读）


1. 输出一个合法 JSON 对象，可直接粘贴保存。
2. 键名取自 §4.3 的 46 键；其余键由 mod 补 Haz5。
3. 7 个抗性/伤害数组与 `EnemyCountModifier` 各写 4 个数字，对应 1/2/3/4 人。
4. `HazardBonus` 取 0~1.33，`ResupplyCost` 取 0~80。
5. 改 `EnemyCountModifier` 时同步抬 `MaxActiveEnemies` 与 `MaxActiveSwarmers`。
6. 池用 `{"clear","add","remove"}` 三件套，自定义怪写进某个池的 `add`。
7. 用户点名的维度才改；联动抬上限时说明理由。
8. 用户只给一句话时，按“差异写法 + CD 版 Haz5 + 4 人数组”生成，最多问 2 个问题。优先级：① 怪的种类有歧义（王草/小王草、禁卫有 7 个变体、抓抓）必须问；② 人数；③ 是否要原版 Haz5 底座。不问就写时，明说“我按哪个解释写的”。
9. 黑话按 §6.4 的默认解释：娃娃机 = 深渊复仇者，抓抓需确认，暴君与禁卫是两种怪。



---

## 1. 参数边界与需求映射
## 1. 参数边界与需求映射

### 1.3 安全参数
先给安全值，再按需放开。

| 项 | 建议 |
| --- | --- |
| `MaxActiveEnemies` | 建议 80~240，按实际人数与机器性能取**一个固定值**（它不随人数缩放）；原版 60。四人常见 180，高倍率建议 180~240 |
| `MaxActiveSwarmers` | 90~180；原版 60 |
| `MaxActiveCritters` | 保持 40 左右；它管非敌对生物 |
| `SmallEnemyDamageResistance` | **保持 1.2**（保 M1000 等武器的伤害断点），除非用户明确要改 |
| `EnemyDamageResistance` | 1.2~2.5，越大怪越肉 |
| `EnemyDamageModifier` | 原版 Haz5 为 2.8~3.4；相对 Haz5 ±30% 已是明显体感变化（复刻低难度不受此限）。**写 0 想让“撞人不掉血”未实测**（C）：文档没有下限条文，且它是全局乘数，会连事件怪一起清零；只想某种怪无伤要走 `PawnStats` 的 `PST_MeleeDamage`（§6.2 标待实测） |
| `EnemyCountModifier` | 0.5~3.0；超过 3 要提醒性能与崩溃风险。>3 时同屏上限建议 180~240 封顶、按机器性能下调；倍率再高也受上限约束（无可靠换算公式，C） |
| `HazardBonus` | 0~1.33，**不能超过 1.33**（mod 保持 Approved 状态的限制） |
| `ResupplyCost` | 0~80，**不能超过 80**；原版 80 |
| `StartingNitra` | 0~100；给太多会让前期失去资源压力 |
| `VeteranNormal` / `VeteranLarge` | 权重区间，不是裸数字；要 100% 老兵写 `[{"weight":1,"range":{"min":1,"max":1}}]`，会明显变难 |

**用户没给这些时的默认口径**（B，项目内实测与社区口径）：

- **人数**：按 4 人写全数组；单人实际只吃第 1 项。用户说“单人”时再只调第 1 项。
- **底座**：不写其他键 = CD1 缺省模板（CD 版 Haz5），与原版 Haz5 差 3 处（§4.5）。用户说“原版 Haz5”时显式写那 3 键。
- **完整度**：先给差异写法；用户要“完整 / 分享 / 复刻”再给 46 键。
- **只给一句话的需求**：先按“差异写法 + CD 版 Haz5 + 4 人数组”起草，最多问 2 个关键问题，按此优先级挑：① 怪的种类有歧义（王草/小王草、禁卫 7 个变体、抓抓）——必须问；② 人数；③ 原版 Haz5 底座。不阻塞时按默认解释写，并在回答里点明用的是哪个解释。

### 1.7 需求翻译
| 用户说 | 你该做什么 | 注意 |
| --- | --- | --- |
| “做一个 N 级难度” | 用第 4.4 节的 HazN 值覆盖 7 个数组 + 标量 + 区间；不要只改 `Name` | CD1 缺省是 Haz5，不是当前选中的难度 |
| “6 级 / 7 级” | 用 §4.4.5 的社区 H6/H7 值 | 优先现成值，不用 Haz5 外推 |
| “6.5 / 7.5” | 相邻档逐键取平均（6.5 = (H6+H7)/2；7.5 = H7+(H7−H6)/2） | 无社区值，按曲线插值 |
| “8 级” | H7+(H7−H6) | 仓库无 H8 |
| “6 级 2 倍虫” | H6 值 + `EnemyCountModifier` ×2，上限抬到 180/180 | 社区有 6x2 现成文件 |
| “深潜一 / 二 / 三阶段” | 用第 4.4.4 节的 3.5 / 4.5 / 5.5 列 | 社区口径（B）；不要拿 Haz5 模板顶替 5.5 |
| “数量翻 X 倍” | 只改 `EnemyCountModifier` = HazN 的值 × X，并抬 `MaxActiveEnemies` / `MaxActiveSwarmers`；**不要同时改 `*Difficulty` 点数** | 不抬上限会触发超限删除（旧怪消失）；倍率与点数叠乘会失控；X>3 必须提醒风险 |
| “不要 XX 怪” | 在它所属的池里 `remove`（映射见第 7.2 节） | 先确认是哪一只（普通/王虫/幽灵）；不确定就四池都 remove |
| “怪更肉 / 更脆” | 改 `EnemyDamageResistance`（较大怪）与 `SmallEnemyDamageResistance`（其余） | 不要默认动 `SmallEnemyDamageResistance`（保武器断点） |
| “怪打人更疼 / 更轻” | 改 `EnemyDamageModifier` | 这是全体攻击的倍率 |
| “波次更密 / 更稀” | 改 `EnemyWaveInterval`（播报潮）与 `EnemyNormalWaveInterval`（自然潮） | 单位是秒，越小越密 |
| “一波里种类更多” | 改 `EnemyDiversity` | 老兵与基础怪占同一槽 |
| “每波最多刷 N 只” | 改点数预算：`EnemyNormalWaveDifficulty`（自然潮）/ `EncounterDifficulty`（房间遭遇），用 N × 单怪点数反推 | 没有绝对只数键；播报潮读波次控制器常量（`EWC_*`），不受这套影响 |
| “同屏同时最多 N 只” | 改 `MaxActiveEnemies` / `MaxActiveSwarmers`（两类分别计数，相加不等于 N） | 超限即删；Boss 与大自爆白名单不吃；机制见 1.8.1 闸门② |
| “补给更贵 / 更便宜” | 改 `ResupplyCost` | 上限 80 |
| “开局给点硝石” | 改 `StartingNitra` | — |
| “关掉某类机制”（固定怪/干扰怪） | 固定敌人：`clear: true` 清空 `StationaryEnemies` + `StationaryEnemyDiversity` 归零；干扰敌人：`DisruptiveEnemyPoolCount: {"min":0,"max":0}`（没有 `*Diversity` 键） | 清了不 `add` 会让该类怪完全消失 |
| “加一种新怪” | `EnemyDescriptors` 定义 + 某池 `add` | 只定义不加池 = 不刷 |

用户用黑话时，先查 6.4 节别名表，按默认解释处理；抓抓这类有两个含义的词再确认。

遇到口径不明时，优先问这三个：**人数**、**数量还是上限**、**具体哪只怪**。

### 1.8 参数联动与修改边界
这一节回答三个高频追问：数量倍率作用在哪层、会不会和点数重复放大、AI 能不能顺手改别的。

#### 1.8.1 三个“数量”参数的分工

| 参数 | 作用层 | 语义 | 证据 |
| --- | --- | --- | --- |
| `EnemyCountModifier` | 全局倍率 | 乘在各生成器的**并发上限**上（`Ceil(MaxConcurrentEnemies × 倍率)`）；不乘总量上限，也不是“每波点数” | A（`ENE_EnemySpawner.CanSpawn` 逻辑核对：`Ceil(MaxConcurrentEnemies × 倍率)`）/ B（波次侧社区实测） |
| `MaxActiveEnemies` / `MaxActiveSwarmers` / `MaxActiveCritters` | 全局硬上限 | 同屏最多存在多少只；固定值，不随难度、不按人数 | A |
| `EnemyNormalWaveDifficulty` / `EncounterDifficulty` / `StationaryDifficulty` | 每波预算 | 一次刷怪事件能花多少**难度点数**；点数除以单怪 `DifficultyRating` 得到只数 | A（`EWC_*` 预算常量）/ B（点数池） |

要点：

- **倍率与点数是两套独立旋钮**。倍率调“生成器愿意放多少”，点数调“这一波有多少预算”。
- 播报潮的预算是波次控制器里的常量（如 `EWC_Generic` 的 270/220/120/90），不读难度表；`*Difficulty` 主要影响普通波、遭遇与固定敌人的预算抽取。
- 全局上限是最后一道闸：生成器再积极，同屏也只能到上限。

**三层闸门（A，`ENE_EnemySpawner.CanSpawn` 逻辑核对）**

| 层 | 判据 | CD 能改什么 |
| --- | --- | --- |
| ① 生成器并发 | `AliveCount < Ceil(MaxConcurrentEnemies × EnemyCountModifier)` | 只改倍率 |
| ① 生成器总量 | `SpawnCount < MaxTotalSpawnCount`（≤0 = 无限制） | **改不到** |
| ② 同屏硬上限 | `Manager.CanSpawn` → `MaxActiveEnemies` / `MaxActiveSwarmers` / `MaxActiveCritters` | 三个上限键 |
| ③ 单次数量 | 描述符 `MinSpawnCount` / `MaxSpawnCount` / `IdealSpawnSize` / `SpawnAmountModifier` | `EnemyDescriptors` 覆盖 |

- `EnemyCountModifier` 只乘并发上限，不乘总量上限。
- `MaxConcurrentEnemies` / `MaxTotalSpawnCount` 是关卡刷怪器实例的资产值，46 键无映射。要“某个刷怪器只出固定只数”，CD1 做不到，只能用点数预算间接压制。

#### 1.8.2 联动规则

1. **改倍率必须同时抬上限**。否则超出全局上限的同类敌人会被立刻删除（Boss、大自爆等白名单除外），体感就是“一波刷出来、下一波把旧的挤没了”。这不是 bug，是上限删除。
2. **不要同时乘倍率与点数**。两者都放大数量，叠乘会失控（10 × 10 = 100 倍预算）。要“更多怪”，优先只动 `EnemyCountModifier`。
3. **只改点数也能加怪，但更慢**。点数变、倍率不变时，单波强度上升、频率不变；同样受上限约束。
4. **上限与倍率要匹配**。10 倍倍率配 60 上限 = 必然删怪；建议上限 180~240、倍率 3~4 更稳，5 以上必须抬上限并提示风险。经验起点（B，无公式，仅作初值）：倍率 1.5 → 上限 90；2 → 120；3 → 180；5 → 240，再按机器性能下调。这是**自配时的保守起步值**；社区 6x2 现成文件实际写 180/180（见 4.4.5），两者不冲突——有现成值就照抄，没有才用起步值试。
5. **低难度 + 高倍率是合法组合**。Haz3 的低点数不会阻止倍率生效，只是单波预算基数低，绝对量小于“Haz5 × 同倍率”。这正是“三级但怪多”的预期效果。

#### 1.8.3 修改边界（强制）

| 允许 | 需说明 | 禁止 |
| --- | --- | --- |
| 改用户点名的维度 | 联动抬上限（因为不改就实现不了） | 擅自改伤害、抗性、老兵、速度等难度维度来“平衡”数量 |
| 补 HazN 的五档值（做 N 级时） | 说明“这是复刻 N 级所需” | 把三级伤害偷偷改成五级 |
| 加 `EnemyDescriptors` 与池操作 | 说明该怪会占池槽位 | 未经用户要求时清空池子；用户明确要“只刷某类”时，清空是合法的（见 1.7、9.7） |

如果用户拒绝抬上限，明确告诉他：“实际增量会被上限吃掉，表现为刷出即删。”

#### 1.8.4 参考写法：Haz3 ×10

只展示数量相关的三个键；其余维度按 4.4 节的 Haz3 值填。

```json
{
  "Name": "Haz3 x10",
  "EnemyCountModifier": [5, 5, 8, 11],
  "MaxActiveEnemies": 240,
  "MaxActiveSwarmers": 240
}
```

**不要**再写 `EnemyNormalWaveDifficulty: [ ... ×10 ... ]`，否则就是双重放大。

---


---


---

## 4. 配置结构



### 4.1 两种写法

**差异写法（推荐）**：只写要改的键，其余自动补 Haz5。

```json
{
  "Name": "Haz 4.5",
  "Description": "Haz4 的伤害，Haz5 的数量",
  "EnemyDamageModifier": [2.4, 2.55, 2.7, 2.9],
  "ResupplyCost": 60
}
```

**完整写法**：46 键全写，适合精确复刻或分享。基线见第 12 节。

### 4.2 类型语法

| 型 | 写法 | 约束 |
| --- | --- | --- |
| 字符串 | `"text"` | `Name`、`Description` |
| 数字 | `1`、`0.85` | 整数浮点等价 |
| 4 元素数组 | `[a,b,c,d]` | 7 个抗性/伤害数组 + `EnemyCountModifier`，对应 1/2/3/4 人。**校验器硬编码长度 4**（A，`ValidateDifficultyAttribtue` 校验器规则：`Array_Length == 4`，否则报 `must be a number or array of numbers with size of 4`）。**但社区仍在写 8 项**（如 `S6失乐园`，按 1–8 人递增）：要么老版本接受、要么另有路径，未实测——见到 8 项别直接判非法，标“待验” |
| 权重区间 | `[{"weight":w,"range":{"min":a,"max":b}}]` | 每项必须含 `range` 与 `weight` |
| 整数区间 | `{"min":a,"max":b}` | 只有 `DisruptiveEnemyPoolCount` |
| 池补丁 | `{"clear":false,"add":[],"remove":[]}` | 三件套 |
| 描述符映射 | `{"名字":{...}}` | 值必须是对象，`Base` 必须存在 |
| 字符串数组 | `["SE_..."]` | 只有 `SeasonalEvents` |
| 组件对象 | `{...}` | 只有 `EscortMule` |

### 4.3 46 键总表

按官方模板顺序（也是枚举 `CD_EDifficultyAttribute` 的顺序）。类型与语义详见第 5–7 节。

| # | 键 | 分组 | 类型 | 约束 | Haz5 默认 | 一句话作用 |
| --- | --- | --- | --- | --- | --- | --- |
| 1 | `ExtraLargeEnemyDamageResistance` | 抗性 | number[4] | 恰好 4 项 | `[0.75,0.8,1.2,1.7]` | **extra large 档**：三种无畏、BET-C、勘探无人机、看守者（含进气口） |
| 2 | `ExtraLargeEnemyDamageResistanceB` | 抗性 | number[4] | 恰好 4 项 | `[0.7,0.7,1,1.3]` | **extra large 2 档**：科洛克王草 |
| 3 | `ExtraLargeEnemyDamageResistanceC` | 抗性 | number[4] | 恰好 4 项 | `[0.6,0.7,1,1.6]` | **extra large 3 档**：OMEN 除虫器、核心石 |
| 4 | `ExtraLargeEnemyDamageResistanceD` | 抗性 | number[4] | 恰好 4 项 | `[1.3,1.3,1.5,1.65]` | **elite 档**：精英变体 |
| 5 | `EnemyDamageResistance` | 抗性 | number[4] | 恰好 4 项 | `[1.2,1.2,1.4,1.5]` | 较大敌人（禁卫/暴君/速射酸虫/黏液轰炸蝇/大自爆等）承伤 |
| 6 | `SmallEnemyDamageResistance` | 抗性 | number[4] | 恰好 4 项 | `[1.2,1.2,1.2,1.2]` | 其余敌人（战士、异虫蝇、蜂群等）承伤 |
| 7 | `EnemyDamageModifier` | 伤害 | number[4] | 恰好 4 项 | `[2.8,3,3.2,3.4]` | 敌人对玩家的全部伤害 |
| 8 | `EnemyCountModifier` | 数量 | number[4] | 恰好 4 项 | `[0.85,0.85,1.25,1.5]` | 刷怪数量 |
| 9 | `EncounterDifficulty` | 节奏 | weighted | `weight+range` | 见第 5.4 节 | 房间遭遇的难度点数 |
| 10 | `StationaryDifficulty` | 节奏 | weighted | 同上 | 见第 5.4 节 | 固定位置敌人的难度点数 |
| 11 | `EnemyWaveInterval` | 节奏 | weighted | 同上 | 见第 5.4 节 | 播报潮间隔（秒） |
| 12 | `EnemyNormalWaveInterval` | 节奏 | weighted | 同上 | 见第 5.4 节 | 自然潮间隔（秒） |
| 13 | `EnemyNormalWaveDifficulty` | 节奏 | weighted | 同上 | 见第 5.4 节 | 自然潮的难度点数 |
| 14 | `EnemyDiversity` | 多样性 | weighted | 同上 | 见第 5.4 节 | 每次刷怪的敌人种类数 |
| 15 | `PointExtractionScalar` | 标量 | number | 通常 ≤1；Haz5 为 1.1，>1 效果待测（C） | `1.1` | 采点/常压波间隔（公式见第 5.6 节） |
| 16 | `StationaryEnemyDiversity` | 多样性 | weighted | 同上 | 见第 5.4 节 | 固定敌人种类数 |
| 17 | `DisruptiveEnemyPoolCount` | 池 | `{min,max}` | 两个整数 | `{min:3,max:3}` | 干扰敌人槽位数 |
| 18 | `VeteranNormal` | 老兵 | weighted | 同上 | 见第 5.4 节 | 战士/异虫蝇变老兵的概率 |
| 19 | `VeteranLarge` | 老兵 | weighted | 同上 | 见第 5.4 节 | 禁卫变大型老兵的概率 |
| 20 | `MinPoolSize` | 池 | number | 整数 | `8` | 刷怪池容量（反直觉，见第 5.5 节） |
| 21 | `MaxActiveElites` | 标量 | number | 整数 | `0` | 精英同屏上限 |
| 22 | `EnvironmentalDamageModifier` | 标量 | number | 无 | `1.1` | 环境伤害倍率（社区实测存疑） |
| 23 | `FriendlyFireModifier` | 标量 | number | 无 | `0.7` | 玩家之间友伤 |
| 24 | `WaveStartDelayScale` | 标量 | number | 无 | `0` | 开局第一波延迟 |
| 25 | `SpeedModifier` | 标量 | number | 无 | `1.15` | 地面敌人移速 |
| 26 | `AttackCooldownModifier` | 标量 | number | 无 | `1.5` | 攻击速度/频率（越大越快） |
| 27 | `ProjectileSpeedModifier` | 标量 | number | 无 | `1.7` | 敌人投射物速度 |
| 28 | `HealthRegenerationMax` | 标量 | number | 比例 | `0.1` | 玩家回血上限 |
| 29 | `ReviveHealthRatio` | 标量 | number | 比例 | `0.1` | 复活后血量 |
| 30 | `EliteCooldown` | 标量 | number | 秒 | `0` | 精英生成间隔 |
| 31 | `Name` | 元信息 | string | — | `"Hazard 5"` | 显示名 |
| 32 | `ResupplyCost` | 资源 | number | ≤80 | `80` | 补给舱硝石成本 |
| 33 | `MaxActiveCritters` | 上限 | number | 整数 | `40` | 非敌对生物上限 |
| 34 | `MaxActiveSwarmers` | 上限 | number | 整数 | `60` | 蜂群级敌人上限 |
| 35 | `MaxActiveEnemies` | 上限 | number | 整数 | `60` | 其余敌人上限 |
| 36 | `StartingNitra` | 资源 | number | 0~100 整数 | `0` | 开局硝石 |
| 37 | `EnemyPool` | 池 | pool | 三件套 | 空 | 本局遇敌池（核选项） |
| 38 | `CommonEnemies` | 池 | pool | 三件套 | 空 | 常见敌人源池 |
| 39 | `SpecialEnemies` | 池 | pool | 三件套 | 空 | 特殊敌人源池 |
| 40 | `DisruptiveEnemies` | 池 | pool | 三件套 | 空 | 干扰敌人源池 |
| 41 | `StationaryEnemies` | 池 | pool | 三件套 | 空 | 固定敌人源池 |
| 42 | `SeasonalEvents` | 事件 | string[] | 数组 | 3 个 `SE_*` | 赛季事件（最新版疑失效） |
| 43 | `EnemyDescriptors` | 描述符 | map | 对象 | `{}` | 自定义敌人变体 |
| 44 | `EscortMule` | 骡 | object | 4 数字 | 见第 5.8 节 | 护送骡承伤 |
| 45 | `Description` | 元信息 | string | 可空 | `""` | 描述 |
| 46 | `HazardBonus` | 标量 | number | 0~1.33 | `1.33` | 结算奖励倍率 |

### 4.4 五档基线（做“N 级难度”用）

> **关键**：CD1 的缺省基线是 Haz5。用户说“做一个三级难度”时，不能只改名字——必须把下列键改成 Haz3 的值。

做 N 级难度时，**7 个数组以本表为准**（包括 `SmallEnemyDamageResistance`）；第 1.3 节里“保持 1.2”只适用于 Haz5 附近的自定义难度，不适用于复刻低难度。

下表是原版 `DFC_Haz_01..05` 的数值（A）。未列出的键五档相同：`MaxActiveEnemies` 60、`MaxActiveSwarmers` 60、`MaxActiveCritters` 40、`ResupplyCost` 80、`DisruptiveEnemyPoolCount` 3/3、`MaxActiveElites` 与 `EliteCooldown` 0。

> Haz5 列是**原版**值，与 CD1 的缺省模板差 3 处（见 4.5 节）。做 Haz1~4 不受影响。

#### 4.4.1 抗性与伤害（1/2/3/4 人）

> 表中 `...ResistanceB/C/D` 是 `ExtraLargeEnemyDamageResistanceB/C/D` 的缩写；写入 JSON 时必须写全名。

| 键 | 档 | 1 人 | 2 人 | 3 人 | 4 人 |
| --- | --- | --- | --- | --- | --- |
| `EnemyDamageModifier` | H1 | 0.5 | 0.5 | 0.5 | 0.5 |
| | H2 | 0.7 | 0.8 | 0.9 | 1.0 |
| | H3 | 1.2 | 1.3 | 1.4 | 1.5 |
| | H4 | 2.0 | 2.15 | 2.3 | 2.5 |
| | H5 | 2.8 | 3.0 | 3.2 | 3.4 |
| `EnemyCountModifier` | H1 | 0.15 | 0.25 | 0.45 | 0.65 |
| | H2 | 0.25 | 0.35 | 0.65 | 0.8 |
| | H3 | 0.5 | 0.5 | 0.8 | 1.1 |
| | H4 | 0.75 | 0.75 | 1.15 | 1.35 |
| | H5 | 0.85 | 0.85 | 1.25 | 1.5 |
| `EnemyDamageResistance` | H1 | 0.45 | 0.55 | 0.7 | 0.85 |
| | H2 | 0.65 | 0.75 | 0.9 | 1.0 |
| | H3 | 0.8 | 0.9 | 1.0 | 1.1 |
| | H4 | 1.0 | 1.0 | 1.2 | 1.3 |
| | H5 | 1.2 | 1.2 | 1.4 | 1.5 |
| `SmallEnemyDamageResistance` | H1 | 0.7 | 0.7 | 0.7 | 0.7 |
| | H2 | 0 | 0 | 0 | 0 |
| | H3 | 1.1 | 1.1 | 1.1 | 1.1 |
| | H4 | 1.2 | 1.2 | 1.2 | 1.2 |
| | H5 | 1.2 | 1.2 | 1.2 | 1.2 |
| `ExtraLargeEnemyDamageResistance` | H1 | 0.2 | 0.4 | 0.6 | 0.8 |
| | H2 | 0.3 | 0.4 | 0.7 | 1.0 |
| | H3 | 0.5 | 0.55 | 0.9 | 1.2 |
| | H4 | 0.7 | 0.75 | 1.0 | 1.6 |
| | H5 | 0.75 | 0.8 | 1.2 | 1.7 |
| `...ResistanceB` | H1 | 0.2 | 0.4 | 0.6 | 0.8 |
| | H2 | 0.3 | 0.4 | 0.7 | 1.0 |
| | H3 | 0.5 | 0.5 | 0.9 | 1.1 |
| | H4 | 0.7 | 0.7 | 1.0 | 1.3 |
| | H5 | 0.7 | 0.7 | 1.0 | 1.3 |
| `...ResistanceC` | H1 | 0.2 | 0.4 | 0.6 | 0.8 |
| | H2 | 0.3 | 0.4 | 0.7 | 0.9 |
| | H3 | 0.4 | 0.5 | 0.9 | 1.2 |
| | H4 | 0.6 | 0.7 | 0.9 | 1.6 |
| | H5 | 0.7 | 0.8 | 1.1 | 1.7 |
| `...ResistanceD` | H1 | 0.2 | 0.3 | 0.5 | 0.8 |
| | H2 | 0.3 | 0.4 | 0.65 | 0.9 |
| | H3 | 0.5 | 0.55 | 0.8 | 1.0 |
| | H4 | 0.7 | 0.75 | 0.9 | 1.2 |
| | H5 | 0.75 | 0.8 | 1.0 | 1.3 |

> `SmallEnemyDamageResistance` 的 Haz2 在原版资产里未序列化，取构造器默认 0；直接写 0 可能让小怪完全无减伤，做 Haz2 时建议沿用 1.1~1.2（C）。

#### 4.4.2 标量

| 键 | H1 | H2 | H3 | H4 | H5 |
| --- | --- | --- | --- | --- | --- |
| `HazardBonus` | 0.25 | 0.5 | 0.75 | 1.0 | 1.33 |
| `FriendlyFireModifier` | 0.1 | 0.2 | 0.3 | 0.4 | 0.7 |
| `EnvironmentalDamageModifier` | 0.75 | 1.0 | 1.0 | 1.33 | 2.0 |
| `WaveStartDelayScale` | 1.0 | 1.0 | 0.75 | 0.5 | 0 |
| `SpeedModifier` | 0.8 | 0.9 | 1.0 | 1.0 | 1.15 |
| `AttackCooldownModifier` | 0.8 | 0.9 | 1.0 | 1.25 | 1.5 |
| `ProjectileSpeedModifier` | 1.0 | 1.0 | 1.0 | 1.3 | 1.7 |
| `HealthRegenerationMax` | 0.15 | 0.15 | 0.15 | 0.15 | 0.1 |
| `ReviveHealthRatio` | 0.6 | 0.5 | 0.4 | 0.2 | 0.1 |
| `PointExtractionScalar` | 0.7 | 0.8 | 0.9 | 1.0 | 1.1 |
| `MinPoolSize` | 5 | 5 | 6 | 7 | 8 |

#### 4.4.3 波次与多样性

格式：`重量/最小-最大`；多项用顿号分隔。

| 键 | H1 | H2 | H3 | H4 | H5 |
| --- | --- | --- | --- | --- | --- |
| `EncounterDifficulty` | 1/50-150、6/250-350、1/400-500 | 同 H1 | 同 H1 | 1/100-200、6/300-400、1/400-500 | 同 H4 |
| `StationaryDifficulty` | 15/0-0、1/150-200、1/250-350 | 8/0-0、1/150-200、1/250-350 | 5/0-0、2/150-200、1/250-350 | 2/0-0、2/150-200、1/250-350 | 1/0-0、1/150-200、1/250-350 |
| `EnemyWaveInterval` | 8.5/350-500 | 8/300-450 | 1.5/180-200、7.5/250-350 | 2/180-200、7.5/250-300 | 2/160-180、7.5/230-280、0.5/299-299 |
| `EnemyNormalWaveInterval` | 2/150-250、4/300-450 | 1/150-250、6/300-350 | 2/180-200、6/250-350 | 3/180-200、6/250-300 | 4/120-150、2/180-250 |
| `EnemyNormalWaveDifficulty` | 1/100-200 | 1/150-250 | 1/200-300 | 1/250-350 | 1/350-450 |
| `EnemyDiversity` | 1/1-2、8/3-4 | 1/1-2、8/3-3、1/4-4 | 2/1-2、8/3-4、0.5/5-5 | 2/1-2、8/3-4、1/5-6 | 2/1-2、8/3-4、1/5-6 |
| `StationaryEnemyDiversity` | 1/2-2 | 1/2-3 | 1/2-3 | 1/2-4 | 1/2-4 |
| `VeteranNormal` | 1/0-0 | 2/0-0、1/0.05-0.15 | 6/0-0、3/0.15-0.3 | 4/0-0、6/0.15-0.3、1/0.4-0.6、0.5/0.75-0.75 | 3/0-0、9/0.15-0.3、2/0.4-0.6、0.5/1-1 |
| `VeteranLarge` | 未配置 | 1/0.1-0.1 | 1/0.125-0.125 | 1/0.15-0.15 | 1/0.175-0.175 |

#### 4.4.4 深潜三档（3.5 / 4.5 / 5.5）

社区说的“深潜一 / 二 / 三阶段”对应原版深潜的三条独立曲线：3.5 / 4.5 / 5.5（B，社区口径）。深潜不套用普通 Haz 档。

| 键 | 3.5 | 4.5 | 5.5 |
| --- | --- | --- | --- |
| `EnemyDamageModifier` | 1.5/1.6/1.7/1.8 | 2.3/2.45/2.6/2.8 | 3.0/3.2/3.4/3.6 |
| `EnemyCountModifier` | 0.6/0.6/0.9/1.2 | 0.8/0.8/1.2/1.4 | 0.85/0.85/1.25/1.5 |
| `EnemyDamageResistance` | 0.85/0.95/1.1/1.2 | 1.1/1.1/1.3/1.4 | 1.3/1.3/1.5/1.6 |
| `ExtraLargeEnemyDamageResistance` | 0.6/0.65/0.95/1.4 | 0.7/0.75/1.1/1.6 | 0.75/0.8/1.2/1.7 |
| `...ResistanceB` | 0.6/0.6/1/1.2 | 0.7/0.7/1.1/1.3 | 0.7/0.7/1.1/1.3 |
| `...ResistanceC` | 0.4/0.5/0.9/1.2 | 0.6/0.7/0.9/1.6 | 0.6/0.7/1/1.6 |
| `...ResistanceD` | 0.6/0.65/0.85/1.1 | 0.7/0.75/0.9/1.2 | 0.75/0.8/1/1.3 |
| `SmallEnemyDamageResistance` | 1.1/1.1/1.1/1.1 | 1.2/1.2/1.2/1.2 | 1.2/1.2/1.2/1.2 |
| `EnvironmentalDamageModifier` | 1.15 | 1.66 | 2.2 |
| `HazardBonus` | 0.75 | 1.0 | 1.33 |
| `FriendlyFireModifier` | 0.3 | 0.55 | 0.7 |
| `WaveStartDelayScale` | 0.75 | 0.5 | 0 |
| `SpeedModifier` | 默认 1.0 | 1.1 | 1.2 |
| `AttackCooldownModifier` | 1.15 | 1.35 | 1.5 |
| `ProjectileSpeedModifier` | 1.15 | 1.5 | 1.7 |
| `HealthRegenerationMax` | 0.15 | 0.15 | 0.1 |
| `ReviveHealthRatio` | 默认 0.4 | 0.2 | 0.1 |
| `PointExtractionScalar` | 0.95 | 1.05 | 1.15 |
| `MinPoolSize` | 6 | 7 | 8 |
| `MaxActiveElites` / `EliteCooldown` | 0 / 0 | 0 / 0 | 0 / 0 |

数组项按 1/2/3/4 人排列，与普通难度一致。

波次池（A，`scripts\parse_uexp_tree.py` 逐项实测，2026-11-25）：深潜 5.5 的 `EncounterDifficulty` 只有 2 项——3/100-200 与 6/250-350，**不是** Haz5 的 6/300-400（原“按 Haz5 对齐”的推断已实测推翻）。三档实测为：3.5 = 1/75-175、6/275-375、1/400-500；4.5 = 1/100-200、6/300-400、1/400-500；5.5 = 3/100-200、6/250-350。

5.5 与 Haz5 的其余池只有一处不同：`EnemyWaveInterval` 第三项是 0.5/350-450（Haz5 是 0.5/299-299）。以下与 Haz5 逐项一致：`StationaryDifficulty` 1/0-0、1/150-200、1/250-350；`EnemyNormalWaveInterval` 4/120-150、2/180-250；`EnemyNormalWaveDifficulty` 1/350-450；`EnemyDiversity` 2/1-2、8/3-4、1/5-6；`StationaryEnemyDiversity` 1/2-4；`VeteranNormal` 3/0-0、9/0.15-0.3、2/0.4-0.6、0.5/1-1；`VeteranLarge` 1/0.175-0.175；`DisruptiveEnemyPoolCount` 3/3。3.5 / 4.5 的其余池未逐项展开（C）。

> 做“深潜三阶段”就把 5.5 列写进 JSON，其余键补 Haz5；一、二阶段换 3.5 / 4.5 列。不要拿 CD1 的 Haz5 模板当 5.5——`ExtraLargeEnemyDamageResistanceD` 与 `EnvironmentalDamageModifier` 差得很明显。

**做“N 级难度”的最短路径**：把上表 N 档的 7 个抗性/伤害数组 + `EnemyCountModifier` + 11 个标量 + 9 个区间/概率写进 JSON，其余键补 Haz5 无妨（未列的键五档相同）。

> 数量倍率与同屏上限没有可靠的换算公式；把倍率乘上去之后，用上限兜底即可，不要试图用倍率直接推上限。

#### 4.4.5 高难档位与曲线（5++ / 6 / 7 / 8）

**官方五档的增量**（单人，A）：`EnemyDamageModifier` 0.5→0.7→1.2→2.0→2.8（H4→H5 +0.8）；`EnemyDamageResistance` 0.45→0.65→0.8→1.0→1.2（+0.2）；`EnemyCountModifier` 0.15→0.25→0.5→0.75→0.85（+0.1）；`SpeedModifier` 0.8→0.9→1.0→1.0→1.15（+0.15）；`AttackCooldownModifier` 0.8→0.9→1.0→1.25→1.5（+0.25）；`ProjectileSpeedModifier` 1.0→1.0→1.0→1.3→1.7（+0.4）。

**官方 5++**（A，样例）：伤害 5.6、抗性 2.1、小怪抗性 1.596、数量 1.275、速度 1.518、冷却 3、弹速 2.04、上限 90/90。5++ 是特化档，不是 H6。

**社区 6 / 7 / 6x2**（B，Ike / trumank 仓库，2026-11-25 取数）：

| 键 | H6 | H7 | H6x2 |
| --- | --- | --- | --- |
| `EnemyDamageModifier` | 3.3 / 3.65 / 3.9 / 4.1 | 3.8 / 4.1 / 4.4 / 4.6 | 同 H6 |
| `EnemyDamageResistance` | 1.4 / 1.5 / 1.6 / 1.75 | 1.6 / 1.8 / 1.9 / 2.0 | 同 H6 |
| `EnemyCountModifier` | 0.95 / 1.05 / 1.35 / 1.65 | 1.15 / 1.25 / 1.45 / 1.75 | 1.9 / 2.1 / 2.7 / 3.3 |
| `ExtraLargeEnemyDamageResistance` | 1 / 1.3 / 1.7 / 2.1 | 1.4 / 1.7 / 2 / 2.5 | 同 H6 |
| `...ResistanceB` | 1 / 1.3 / 1.5 / 1.7 | 1.4 / 1.7 / 2 / 2.5 | 同 H6 |
| `...ResistanceC` | 1 / 1.3 / 1.7 / 2.1 | 1.4 / 1.7 / 2 / 2.5 | 同 H6 |
| `...ResistanceD` | 1.5 / 1.55 / 1.75 / 1.9 | 2 / 2.1 / 2.25 / 2.45 | 同 H6 |
| `SpeedModifier` | 1.25 | 1.35 | 1.25 |
| `AttackCooldownModifier` | 1.75 | 2 | 1.75 |
| `ProjectileSpeedModifier` | 1.9 | 1.9 | 1.9 |
| `EnvironmentalDamageModifier` | 1.4 | 1.6 | 1.4 |
| `ResupplyCost` | 60 | 50 | 40 |
| `MinPoolSize` | 12 | 12 | 12 |
| 上限（`MaxActiveEnemies` / `MaxActiveSwarmers`） | 90 / 150 / 180 / 180 | 90 / 150 / 180 / 180 | 90 / 150 / 180 / 180 |

**推算规则**：

- 6 / 7：直接用上表社区值。
- 6.5 / 7.5：相邻档逐键取平均（6.5 = (H6+H7)/2；7.5 = H7+(H7−H6)/2）。
- 8：H7+(H7−H6)，仓库无 H8。
- 2 倍虫：`EnemyCountModifier` ×2，上限同步抬到 180/180（社区 6x2 写法）。
- 上限在社区文件里写 4 项数组；当前 CD 是否接受待验，保守写标量。

**偏差**：Ike 的 H6 与“H5+(H5−H4)”对比，数量、抗性、冷却吻合；伤害低 0.3、速度低 0.05、弹速低 0.2。插值以社区值为锚，纯公式只用于中间档。

### 4.5 CD1 模板与原版 Haz5 的差异

| 键 | CD1 默认 | 原版 Haz5 | 影响 |
| --- | --- | --- | --- |
| `ExtraLargeEnemyDamageResistanceC` | `[0.6,0.7,1,1.6]` | `[0.7,0.8,1.1,1.7]` | OMEN 承伤偏高 |
| `ExtraLargeEnemyDamageResistanceD` | `[1.3,1.3,1.5,1.65]` | `[0.75,0.8,1,1.3]` | 精英敌人明显更硬 |
| `EnvironmentalDamageModifier` | `1.1` | `2.0` | 环境伤害减半 |

只写差异时，得到的是“CD 版 Haz5”。想复刻原版 Haz5，把这 3 个键显式写成原版值。

---


---



---

## 5. 字段参考



本节解释每个字段的语义、边界与常见误用。数值速查见第 4.3 节。

### 5.1 元信息

| 键 | 说明 |
| --- | --- |
| `Name` | 下拉框显示名，建议短且带版本 |
| `Description` | 任务界面里的说明；多行用 `\n`，不要把真实换行写进 JSON 字符串（会解析失败） |

### 5.2 上限与资源

| 键 | 说明 |
| --- | --- |
| `MaxActiveCritters` | 非敌对生物同屏上限；具体包含哪些生物待测。原版 40 |
| `MaxActiveSwarmers` | 蜂群级敌人（蜂拥、小水母、粉碎者等）同屏上限。原版 60 |
| `MaxActiveEnemies` | 其余敌人同屏上限（“一镐子敲不死的多半在这类”）。原版 60 |
| `ResupplyCost` | 补给舱硝石成本，最多 80 |
| `StartingNitra` | 开局赠送硝石 |

> 三个上限键是**固定值**，不随难度档变化、也不按人数缩放（A）。难度只影响刷怪倍率；倍率再高，同屏也受上限约束。想“数量翻很多倍”，必须同时抬上限，否则倍率会被吃掉。
>
> 超出上限的同类敌人会被**立刻删除**（Boss、大自爆等白名单除外），所以“刷出来又消失”通常是触发了上限，不是 bug。

### 5.3 抗性

- 4 项 = 1/2/3/4 人，不是四档难度；单人只吃第 1 项。**当前校验器只收 4 项**（A，校验器规则），但社区有多份按 8 项写（1–8 人递增）的配置——见到 8 项先问用户游戏版本，别直接断言“存不进去”。
- 数值越大越耐打，负值表示额外易伤。
- 对应关系（**B**：官方 wiki `Difficulty_Scaling#Resistance_Scaling` 与作者网络教程第 66–87 行；两者对 extra large 名单不一致，以 wiki 为准。源码只证四个字段与四个 getter 存在，看不到分档；蓝图侧零调用）：

| 键 | 作用对象 |
| --- | --- |
| `ExtraLargeEnemyDamageResistance` | **extra large 档**：三种无畏、BET-C、勘探无人机、看守者（含进气口） |
| `...ResistanceB` | **extra large 2 档**：科洛克王草 |
| `...ResistanceC` | **extra large 3 档**：OMEN 除虫器、核心石 |
| `...ResistanceD` | **elite 档**：精英变体 |
| `EnemyDamageResistance` | 较大敌人：禁卫、暴君、速射酸虫、黏液轰炸蝇、大自爆等 |
| `SmallEnemyDamageResistance` | 其余：战士、异虫蝇、蜂群等 |
| `EnemyDamageModifier` | 敌人对玩家的全部伤害 |

> **四键的边界（2026-11-25 按官方 wiki 校正）**：
> - 四项按**抗性分档**切（large / extra large / extra large 2 / extra large 3 / elite），不是按 `CreatureSize` 枚举。别拿描述符的 `CreatureSize` 推档：`ED_PatrolBot_Caretaker` 是 Medium，而官方把 Rival Patrol Bot 归在 large 抗性档。
> - **看守者整体吃无后缀项（A，2026-11-25 实测）**：把无后缀项设成 `0.001`，打进气口“不费吹灰之力”，说明进气口也按 extra large 结算。官方 wiki 把 `intakes` 单列为 large 档，与实测冲突——以实测为准。`EnemyDamageResistance`（large）对看守者是否还有叠加作用，未测。
> - `ExtraLargeEnemyDamageResistanceD` 里的 “Elite” 指精英变体（与 `MaxActiveElites` 同一套机制），不是“大体型怪”——大体型走 `EnemyDamageResistance`。
> - 无后缀项的完整名单是“三种无畏 + BET-C + 勘探无人机 + 看守者（眼睛）”；早期作者教程只写 “three Dreadnought variants”，漏了后三者。
>
> 经验：`SmallEnemyDamageResistance` 保持 1.2 能保住 M1000 等武器的伤害断点。改它会让武器手感整体漂移（B）。
>
> 注意：本节数组与 6.2 的 `PST_*` 语义不同——这里 1.2 = 更耐打；分类 `PST_*Resistance` 里 1 = 免疫、0 = 正常。例外是 `PST_DamageResistance`：它是伤害乘数（1 = 正常、0 = 免疫、>1 更脆、负值无敌）。不要互相抄。

### 5.4 刷怪节奏与概率

```json
{
  "EnemyWaveInterval": [
    { "weight": 7.5, "range": { "min": 230, "max": 280 } }
  ]
}
```

- 每次事件按 `weight` 加权抽一个区间，再在 `min~max` 内随机取值。
- `weight: 0` = 不选该区间（保留位置便于调参）。

| 键 | Haz5 默认（weight / min-max） | 单位 | 何时掷骰 |
| --- | --- | --- | --- |
| `EncounterDifficulty` | 1/100-200、6/300-400、1/400-500 | 难度点数 | 每个房间/每次遭遇 |
| `StationaryDifficulty` | 1/0-0、1/150-200、1/250-350 | 难度点数 | 每个房间/每次遭遇 |
| `EnemyWaveInterval` | 2/160-180、7.5/230-280、0.5/299-299 | 秒 | 每次播报潮 |
| `EnemyNormalWaveInterval` | 4/120-150、2/180-250 | 秒 | 每次自然潮 |
| `EnemyNormalWaveDifficulty` | 1/350-450 | 难度点数 | 每次自然潮 |
| `EnemyDiversity` | 2/1-2、8/3-4、1/5-6 | 种类数 | 每次刷怪事件 |
| `StationaryEnemyDiversity` | 1/2-4 | 种类数 | 任务开始 |
| `VeteranNormal` | 3/0-0、9/0.15-0.3、2/0.4-0.6、0.5/1-1 | 概率 | 每次事件或每只怪 |
| `VeteranLarge` | 1/0.175-0.175 | 概率 | 每次事件或每只怪 |

语义边界（B，来自社区实测）：

| 键 | 边界 |
| --- | --- |
| `EnemyWaveInterval` | 管**播报潮**（Mission Control 先喊话再刷）；不影响一波持续多久 |
| `EnemyNormalWaveInterval` | 管**自然潮**（无播报）；采点任务与精炼厂维修段的常压潮不算自然潮 |
| `EncounterDifficulty` | 玩家靠近未探索房间时触发的遭遇；不含固定敌人与特殊事件；刷出的怪未激怒 |
| `StationaryDifficulty` | 固定位置敌人（吐珠、母巢、大水母、洞穴水蛭）；破坏任务的炮塔可能也算；刷出时未激怒 |
| `EnemyDiversity` | 一次刷怪里的**不同敌人种类数**；老兵与基础怪占同一槽（Diversity=1 也可能同时出战士和刀锋）；影响自然潮、播报潮内的小波、防守目标、撤离、虫蛋潮、mini MULE 潮。**破坏性敌人也算一种**；刀锋、护卫、三眼、暴君这类变体不单独占槽（B，社区教程） |

#### 5.4.1 常用怪点数（DifficultyRating）

点数预算除以单怪点数，就是这一波最多能刷几只。

| 敌人 | 点数 | 敌人 | 点数 |
| --- | --- | --- | --- |
| 蜂拥异虫 | 6 | 蛭尾异虫 | 75 |
| 战士异虫 | 10 | 典狱长 | 90 |
| 自爆异虫 / 吐丝异虫 | 25 | 禁卫异虫 | 90 |
| 吐刺异虫蝇 | 45 | 速射酸虫 / 潜影 / 捕手 / 黏液轰炸蝇 | 95 |
| 三颚蝇 | 50 | 大自爆虫 | 120 |
| 吐酸异虫 / 脓毒异虫 | 55 | 地虱 | 145 |

来源：B（资产 `DifficultyRating` + wiki 对照，详见《刷怪上限与点数池机制》§2.1）。未列出的敌人可在沙盒或资产里查 `DifficultyRating`。

换算示例：自然潮预算 1850 点，全刷战士约 185 只，全刷禁卫约 20 只；但假设你把同屏上限抬到 160，会先截住 185 只。**点数够 ≠ 都能同时在场**。

### 5.5 老兵与多样性

**“老兵”（Veteran）不是独立敌人，游戏里也不会显示这个词**：它是生成时把基础怪**替换**成更强变体的机制，替换后显示的是变体名。对应关系：

| 概率键 | 原形 → 老兵形态 |
| --- | --- |
| `VeteranNormal` | 战士异虫 → 刀锋异虫 / 护卫异虫；吐刺异虫蝇 → 坚甲异虫蝇 / 三颚蝇 |
| `VeteranLarge` | 禁卫异虫 → 暴君异虫 |

| 键 | 说明 |
| --- | --- |
| `VeteranNormal` | 战士/异虫蝇变老兵的概率。写 1 时典狱长仍可能召普通战士（B） |
| `VeteranLarge` | 禁卫变大型老兵的概率 |
| `EnemyDiversity` | 见第 5.4 节；写成超过池内可用种类数时，行为未定义（C），实战别超过池内种类数 |
| `StationaryEnemyDiversity` | 固定敌人的种类数，任务开始时确定（B） |
| `DisruptiveEnemyPoolCount` | 干扰敌人（捕手、典狱长、滚滚、速射酸虫、黏液轰炸蝇、大自爆）的槽位数；按任务掷骰，任务中不变（B）。**写 0 不能保证完全关掉**：破坏性单位仍可能按池数量被抽（B，社区教程） |
| `MinPoolSize` | **反直觉**：社区实测倾向于“刷怪池里敌人总数的上限”，不是“最小池大小”。源码字段名就是 `MinPoolSize`，两种解释并存；调整时小步试（B/C）。社区另有一条口径：`EnemyDiversity` 的优先级高于 `MinPoolSize`，总池应控制在 Diversity 之下，否则会出现“想要的不刷、不想要的刷”（B）。**取值**：池里还剩 ≥3 种写 4–6；只剩 1 种写 1（见 §7.1，后者 C 级未实测）。 |

### 5.6 标量

| 键 | 说明 |
| --- | --- |
| `EnvironmentalDamageModifier` | 名义上是玩家受到的环境伤害倍率；**社区有限测试未观察到效果**（C）。自定义难度保守取 1.0~1.1；复刻原版 Haz5 用 2.0（见 4.5） |
| `PointExtractionScalar` | 采点任务潮间隔 = `165 / 该值` 秒；常压计时 = `40 / 该值` 秒；**通常 ≤1 最有效**；第一波约 4 分钟后才生效（B）。例：0.7 → 潮间隔约 236 秒。原版 Haz5 写 1.1，>1 的实际效果待测（C） |
| `HazardBonus` | 结算奖励倍率，0~1.33；上限是为了保住 mod 的 Approved 状态 |
| `FriendlyFireModifier` | 玩家之间的友伤倍率 |
| `WaveStartDelayScale` | 采矿任务第一波的出现时机；0 或负值 = 落地即播报（B） |
| `SpeedModifier` | 地面敌人移速；异虫蝇与小水母不受影响（B） |
| `AttackCooldownModifier` | 攻击速度与频率，**数值越大攻击越快**（会加快攻击动画）（B） |
| `ProjectileSpeedModifier` | 敌人投射物速度；不影响玩家武器（B） |
| `HealthRegenerationMax` | 玩家可回血的上限比例；不影响回血速率（B） |
| `ReviveHealthRatio` | 被队友复活后的血量比例；不影响 Bosco 复活（B） |
| `MaxActiveElites` | 精英同时存在/生成的上限（C）。**没有取值依据**：原版五档全 0，文档无社区值，别拿 `MaxActiveEnemies` 的 60 类比。默认 0 未必表示“可放开”，抬高上限也不等于造出精英来源。要给建议时用阶梯逼近：8 → 20 → 40，每档开一局看是否真出精英、有无掉帧 |
| `EliteCooldown` | 精英生成间隔，单位秒（B）。默认 0，再往下没有更小值——“更频繁”在这条路上已到底；想稳就别动它 |

### 5.7 赛季事件

```json
{
  "SeasonalEvents": ["SE_ProspectorDrone", "SE_ProspectorDataDeposit", "SE_RivalBomb"]
}
```

- 原意是白名单：删掉条目就不再出现。
- **最新版社区反馈该字段已失效**：Rival 事件与 Lithophage 陨石都不再被标记为赛季事件（B）。保守做法：保留默认，不要指望靠它关事件。

### 5.8 护送骡

```json
{
  "EscortMule": {
    "FriendlyFireModifier": 0.1,
    "BigHitDamageModifier": 0.5,
    "BigHitDamageReductionThreshold": 6,
    "NeutralDamageModifier": 0.1
  }
}
```

| 键 | 说明 |
| --- | --- |
| `FriendlyFireModifier` | 玩家对 Doretta 的伤害 |
| `BigHitDamageModifier` | “重击”对 Doretta 的伤害；**越大减伤越多**；正好 1 = 不减伤（B） |
| `BigHitDamageReductionThreshold` | 多大的单次伤害算“重击” |
| `NeutralDamageModifier` | 疑为环境伤害，作用不明（C） |

只在护送任务生效，不打护送可整段省略。

---


---



---

## 6. 敌人



### 6.1 自定义敌人变体（EnemyDescriptors）

```json
{
  "EnemyDescriptors": {
    "我的精英刀锋": {
      "Base": "ED_Spider_Grunt_Attacker",
      "Elite": true,
      "DifficultyRating": 25,
      "IdealSpawnSize": 2,
      "MinSpawnCount": 2,
      "MaxSpawnCount": 4,
      "Rarity": 10,
      "Scale": 1.1,
      "TimeDilation": 1,
      "PawnStats": {
        "PST_PhysicalResistance": 100,
        "PST_FireResistance": -100
      }
    }
  }
}
```

> 示例里 `PST_PhysicalResistance: 100` = 物理完全免疫；`PST_FireResistance: -100` = 火焰双倍伤害（弱点）。**这两个值是百分比，不是 0–1 倍率**：`100` 免疫、`50` 减半、`0` 无修正、`-100` 双倍。按 0–1 量级写（`0.5`、`-1`）只有零点几的百分比效果，等于白写。数值语义详见 §6.2。

#### 6.1.1 继承机制

| 项 | 说明 |
| --- | --- |
| `Base` | 必填，指向已存在的描述符（原版 `ED_*` 或本文件里已定义的名字）；先继承其全部属性，再用本条目覆盖。它也是“单属性抗性”的唯一来源：指向寒霜禁卫、辐射禁卫等变体，可继承其游戏原生抗性（B） |
| 自定义名 | 可含字母、数字、空格、部分特殊字符；建议用中文或英文短名，别用引号与斜杠 |

**继承的三个边界（2026-11-25 由 31 份社区配置归纳）**

| 写法 | 现象 | 处置 |
| --- | --- | --- |
| `Base` 互指成环（A→B、B→A） | 社区最高频写法之一（约 8 份配置有），手册与现有资料都没有环检测条文 | 判“行为未定义”；别主动写，读到环按“链不终止、可能整条不生成”处理 |
| 原版键自引用（`"ED_X": {"Base": "ED_X"}`） | 与自定义名自引用不是一回事：原版名在 `EnemyDescriptorReferences` 表里，理论上能克隆 | 判“待实测”，不要像自定义名那样直接判死 |
| `Base` 指向废案（`ED_Spider_Shooter_Ground`、`ED_Spider_ShooterQueen`） | 社区用它派生可用怪 | 判“能过校验、能否生成待实测”，别当主力 |

#### 6.1.2 可覆盖属性

| 属性 | 类型 | 说明 |
| --- | --- | --- |
| `EnemyID` | string | 敌人 ID 资产名 |
| `EnemyClass` | string | 敌人 Pawn 类路径 |
| `VeteranScaling` | enum | 老兵缩放档 |
| `VeteranClasses` | string[] | 可替换成的老兵描述符 |
| `BiomeEnemyClassOverrides` | map | 按生物群覆盖 Pawn 类 |
| `EliteEnemyClass` | string | 精英形态类 |
| `BannedMissionTypes` | string[] | 禁止出现的任务类型 |
| `EnemySignificance` | enum | AI 显著性（性能分级） |
| `SpawnOffset` | `{X,Y,Z}` | 生成位置偏移 |
| `UsesSpawnEffects` | bool | 生成特效 |
| `CreatureSize` | enum | 体型档 |
| `Positioning` | string | 生成点定位器 |
| `PlacementCategories` | — | 枚举里有、源码未见（C） |
| `CaveInfluencer` | string | 洞穴影响器 |
| `Debris` | 结构数组 | 死亡碎块 |
| `InfluencerRange` | number | 影响器范围 |
| `PlacementCarver` | string | 放置雕刻器 |
| `PathfinderType` | enum | 寻路类型（地面/飞行/天花板） |
| `PlacementRoomQuery` | tag query | 房间放置过滤 |
| `SpawnSpread` | number | 生成散布半径 |
| `IdealSpawnSize` | int | 理想单次生成数量 |
| `CanBeUsedForConstantPressure` | bool | 可用于常压刷怪（持续生成） |
| `CanBeUsedInEncounters` | bool | 可用于进新洞穴/初始潮 |
| `DifficultyRating` | number | 难度点数成本 |
| `MinSpawnCount` / `MaxSpawnCount` | int | 单次生成数量下限/上限 |
| `Rarity` | number | 稀有度；**越高越稀有**，越不容易被优先选中（B）。**过高会完全不刷**：社区样例写到 1800 时该怪一次都没出现，期望刷 1 只时别用极端值（B） |
| `SpawnAmountModifier` | number | 数量修正 |
| `RequiredMainCampaignProgress` | int | 主线进度门槛 |
| `UsesSpawnRarityModifiers` | bool | 启用稀有度修正 |
| `SpawnRarityModifiers` | 结构数组 | 稀有度修正表 |
| `CheatClass` | enum | 作弊菜单分类 |
| `Base` | string | 继承源 |
| `Elite` | bool | 精英标记。CD 在生成回调里读它：为 true 就给生成的 `FSDPawn` 加 `BP_MOD_MakeElite_C` 组件（A，CD_UI 校验逻辑），精英特效来自组件，不限敌人种类。**只在自然刷时触发，召唤不触发**（B，实测）。**要挂在原版键覆盖上**：回调按描述符名查 `EnemyDescriptorMeta`，自定义名的条目匹配不到，`Elite` 等于没写（B，实测：自定义名掠夺虫无特效，原版键覆盖才有）。条目必须带 `Base`（缺了报 `EnemyDescriptors must be a map of enemy descriptors`）。**自引用要分两种**（B/C）：**自定义名自引用**（键名与 `Base` 都是自定义名）只能过校验、实测会“不生成”——`Base` 名若不在 CD 的 `EnemyDescriptorReferences` 表里就跳过克隆，字段留默认、`EnemyClass` 空；**原版键同名自引用**（`"ED_X": {"Base": "ED_X"}`）另当别论，原版名本来就在表里，理论上能克隆，实测未做（见 6.1.1）。可靠写法 = **自定义名 + `Base` 指原版名 + 把自定义名加进池**。另有类替换式精英（`EliteEnemyClass`），只有吐丝异虫、速射酸虫、异虫蝇、三颚蝇有独立 `_Elite` 类 |

**数量四字段（单次出几只）**

| 字段 | 默认 | 作用 |
| --- | --- | --- |
| `IdealSpawnSize` | 10 | 凑一批的参考值，不是上限 |
| `MinSpawnCount` / `MaxSpawnCount` | 1 / 9999 | 单次只数下限 / 上限 |
| `SpawnAmountModifier` | 1.0 | 最后乘的数量修正 |

推导（C，源码实现静态拿不到）：`单次只数 ≈ clamp(预算 ÷ DifficultyRating, Min, Max) × SpawnAmountModifier`。默认 `MaxSpawnCount = 9999` 等于不限，原版多数怪靠预算定数量；`0 / 0` 是禁用占位。

`Rarity` 控制“被抽中的机会”，不是“一次出几只”：想让它少出现调 `Rarity`，想压单次数量用 `MaxSpawnCount` / `SpawnAmountModifier`。两者体感相似、机制不同。

**覆盖原版键的四条边界**

| # | 边界 |
| --- | --- |
| 1 | 只作用于该描述符本身；氪化/寒霜/放射性/矿化/岩痘禁卫、重甲禁卫、哨卫都是独立条目，不连带（A） |
| 2 | 老兵形态不跟覆盖走（`VeteranClasses` 换的是另一个描述符）；老兵吃不吃新抗性未验证（C） |
| 3 | **不写** `DifficultyRating` 时预算按原值算（A）；**写了是否生效未实测**——社区多份配置在改它（大自爆 120→50 让便宜、→1500 压稀有），与“改了不生效”冲突（C） |
| 4 | 原版键覆盖不用再进 `add`（池里本来有它，覆盖会替换池中成员）；自定义名必须进池，否则不刷（B） |

#### 6.1.3 三个扩展键

| 键 | 类型 | 作用 |
| --- | --- | --- |
| `Scale` | number | 体型缩放；1 = 原样 |
| `TimeDilation` | number | 时间流速；1 = 正常，<1 变慢，>1 变快 |
| `PawnStats` | map | 对该敌人生成后的 Pawn 属性写入修正（抗性、移速等）。分类抗性有效但数值是百分比（`100` = 100%），血量无效；详见 §6.2（B） |

### 6.2 PawnStats 键表

`PawnStats` 的键必须是游戏里的 `PST_*` 资产名。常用：

> **CD 写入实测（B，2026-11-25）**：
> - 有效：分类抗性（`PST_PhysicalResistance` / `Fire` / `Kinetic` …）——**数值是百分比**：`100` = 完全免疫、`50` = 减伤 50%、`-100` = 双倍伤害。实测火焰抗性 100 完全免疫火焰、-100 约双倍；`PST_KineticResistance: 100` 完全免疫枪械，`-0.75` 几乎无差异（只有 +0.75%）。
> - 有效：`PST_MovementSpeed`——写 5 后怪明显变快。
> - 有效：`PST_DamageResistance`——**伤害乘数**（`实际伤害 = 原伤害 × v`）：`0.001` 怪只受极少伤害、`1.59` / `5` 明显更脆、`-1` 完全无敌。
> - 无效：`PST_MaxHealth`（写 10 倍不变厚）。
> - **偶发崩溃（B，2026-11-25）**：`PST_KineticResistance: 100` 首次生成时崩过一次（`EXCEPTION_ACCESS_VIOLATION reading 0x34`），复测 `100` 与 `-0.75` 均正常；按偶发记录，暂不归因于键或值。
>
> **伤害类型错位（关键）**：`PST_PhysicalResistance` 只管**近战/物理武器**（镐子、钻机、斧、捕兽夹），**不管枪械**；枪械子弹走 `PST_KineticResistance`。用枪打禁卫测六档（`0` / `0.5` / `1` / `-1` / `2` / `-80`）完全无差异——不是抗性无效，是打错了类型。
>
> **武器 → 抗性键**（A，资产 `DMG_*` 的 `ResistanceStat`）：
>
> | 伤害来源 | DamageClass | 抗性键 |
> | --- | --- | --- |
> | 枪械子弹（步枪、M1000、机枪、手枪、SMG 等） | `DMG_Kinetic` | `PST_KineticResistance` |
> | 镐子、钻机、斧、捕兽夹 | `DMG_Physical` | `PST_PhysicalResistance` |
> | 火焰喷射器、燃烧 DoT | `DMG_Fire` | `PST_FireResistance` |
> | 爆炸（榴弹、C4 等） | `DMG_Explosive` | `PST_ExplosionResistance` |
> | 弩箭、针刺 | `DMG_Piercing` | `PST_PiercingResistance` |
> | 冰、电、腐蚀、毒、辐射 | `DMG_Cold` / `Electric` / `Corrosive` / `Poison` / `Radiation` | 对应 `PST_*` |
>
> **三套“抗性”分工**：
>
> | 层 | 载体 | 键 | CD 能否改 |
> | --- | --- | --- | --- |
> | 按体型 | `DifficultySetting` 的 4 元素数组 | `EnemyDamageResistance`、`SmallEnemyDamageResistance`、`ExtraLarge...` | 能（A，源码） |
> | 按伤害类型 | `HealthComponent.Resistances`（`TMap<UPawnStat*, float>`） | `PST_PhysicalResistance`、`Fire`、`Kinetic`… | 能（B，实测；数值是百分比） |
> | 全局乘数 | `PawnStatsComponent` | `PST_DamageResistance` | 能（B，实测；数值是乘数） |
>
> 两类数值不要混：分类抗性 `PawnStatType = Additive`（百分比，`100` = 100%）；`PST_DamageResistance` 是 `Multiplicative`，`StartingValue = 1`（乘数，`1` = 正常）。
>
> 想“免疫所有伤害只留火焰”：把所有分类抗性写 `100`，火焰写 `0`（正常）或 `-100`（双倍）；不要用 `PST_DamageResistance: -1`，它是全局的，会把火焰也免掉。

**数值语义（重要）**

分类抗性键（`PST_FireResistance`、`PST_ColdResistance` 等）是**百分比**（`PawnStatType = Additive`，A）：

```text
实际伤害 = 原伤害 × (1 − v / 100)
```

| 值 `v` | 效果 |
| --- | --- |
| `100` | 完全免疫（伤害 ×0） |
| `50` | 减伤 50% |
| `0` | 无修正 |
| `-100` | 双倍伤害（弱点） |

来源：A（`HealthComponent.Resistances` + PST 资产 `Additive`）+ B（2026-11-25 实测：火焰 100 完全免疫、-100 约双倍）。

所以：

- 想做“免疫某属性” → 把对应分类抗性写成 **`100`**，不是 `1`。
- 想做“双倍弱点” → 写 `-100`。
- 想做“减伤一半” → 写 `50`。
- **别写社区常见的 `-0.75` 这种 0–1 量级**：按百分比只有 +0.75% 伤害，几乎无效果；要“-75% 抵抗”就写 `-75`。
- 别拿 `PST_PhysicalResistance` 测枪械：它只管近战；枪械用 `PST_KineticResistance`。
- `PST_DamageResistance` 是**乘数**，不是百分比：`1` = 正常、`0.001` ≈ 免疫、`>1` 更脆、负值无敌（B，实测）。
- 非抗性键（`PST_MovementSpeed`、`PST_DamageBonus` 等）按倍率写入：`1` = 原值，`>1` 增强，`<1` 削弱（C，待实测）；`PST_MaxHealth` 实测无效。

| 键 | 含义 |
| --- | --- |
| `PST_DamageResistance` | **伤害乘数**：`实际伤害 = 原伤害 × v`（B，实测）。`0.001` ≈ 免疫、`1` = 正常、`>1` 更脆、负值 = 无敌；社区写 `0.3` 是很硬、`5` 是很脆、`300` 是一碰就碎 |
| `PST_FireResistance` | 火焰抗性 |
| `PST_ColdResistance` | 冰冻抗性 |
| `PST_ElectricResistance` | 电抗性 |
| `PST_ExplosionResistance` | 爆炸抗性 |
| `PST_CorrosiveResistance` | 腐蚀抗性 |
| `PST_PhysicalResistance` | 近战/物理抗性（镐、钻机、斧、捕兽夹）；**不含枪械** |
| `PST_KineticResistance` | 枪械子弹抗性（步枪、M1000、机枪、手枪等） |
| `PST_PoisonResistance` | 毒抗性 |
| `PST_RadiationResistance` | 辐射抗性 |
| `PST_InternalDamageResistance` | 内部伤害抗性 |
| `PST_PiercingResistance` | 穿刺抗性（弩箭、针刺）；在 1.40 转储里（旧 50 项清单漏了），写错不报错 |
| `PST_InternalDamageResistance` | 内部伤害抗性；无对应 `DMG_Internal` 资产，映射待核 |
| `PST_MovementSpeed` | 移动速度 |
| `PST_MaxHealth` | 最大生命 |
| `PST_MaxShield` | 最大护盾 |
| `PST_DamageBonus` | 伤害加成 |
| `PST_MeleeDamage` | 近战伤害 |
| `PST_CritChance` | 暴击率 |
| `PST_CritBonusDamage` | 暴击额外伤害 |
| `PST_FriendlyFire` | 友伤 |

完整 54 项资产清单（A，1.40 转储）：

```text
PST_AirControl, PST_BarrelKicking, PST_CarriableThrowing, PST_CarryingCapacity,
PST_CarryingSpeedModifier, PST_CaveLeechSense, PST_ColdResistance, PST_CorrosiveResistance,
PST_CritBonusDamage, PST_CritChance, PST_DamageBonus, PST_DamageFromPlayers,
PST_DamageResistance, PST_DepositSpeed, PST_DirtMiningStrength, PST_ElectricResistance,
PST_EnemyAttractiveness, PST_EventExplosionResistance, PST_ExplodeOnDeath,
PST_ExplosionResistance, PST_FallingResistance, PST_FireResistance, PST_FlareThrowStrength,
PST_FriendlyFire, PST_GoldMining, PST_HoverBootsDuration, PST_InternalDamageResistance,
PST_KineticResistance, PST_LithofoamResistance, PST_MaxHealth, PST_MaxShield, PST_MeleeDamage,
PST_MorkiteMining, PST_MovementSpeed, PST_MovementSpeedEnvironmentalPenalty,
PST_MovementSpeedEnvironmentalPenaltyReduction, PST_MovementSpeedPenalty,
PST_MovementSpeedPenaltyReduction, PST_PhysicalResistance, PST_PiercingResistance,
PST_PoisonResistance, PST_PowerAttackCooldownRate, PST_RadiationResistance, PST_RedSugarHeal,
PST_ResourceMiningStrength, PST_ResupplyHealing, PST_ResupplySpeed, PST_ReviveSpeed,
PST_RockMiningStrength, PST_ShieldRegeneratoinRate, PST_SlipperyFloor, PST_SprintSpeed,
PST_Ziplline_DownBoost, PST_ZipllineSpeed
```

> 比旧清单多 `BarrelKicking` / `EventExplosionResistance` / `LithofoamResistance` / `PiercingResistance`。社区三份代码从未启用 `PST_FallingResistance` 与 `PST_LithofoamResistance`，`PST_EventExplosionResistance` 只出现 2 次——用到这些键先用小值实测。

### 6.3 敌人代码表

社区整理的游戏内描述符名与中文名对应（适用到第 5 赛季）。带“废案”的敌人可能无 AI 或行为异常，别拿来当主力。

#### 6.3.1 异虫科

| 描述符 | 中文名 |
| --- | --- |
| `ED_Spider_Swarmer` | 蜂拥异虫 |
| `ED_EggSpider` | 虫蛋爆出的蜂拥异虫 |
| `ED_Spider_Swarmer_Pheromone_NOFX` | 无畏吐出的红蜂拥 |
| `ED_Spider_Spawn` | 异虫幼虫（母巢小绿虫） |
| `ED_Spider_Swarmer_Ice` | 冰封岩层蜂拥 |
| `ED_Spider_Swarmer_Mutated` | 放射性禁区蜂拥 |
| `ED_TunnelSwarmer` | 隧道蜂拥 |
| `ED_Spider_Grunt` | 战士异虫 |
| `ED_Spider_Grunt_Ice` | 冰封岩层战士 |
| `ED_Spider_Grunt_Mutated` | 放射性禁区战士 |
| `ED_Spider_GruntTutorial` | 典狱长召唤的战士 |
| `ED_Spider_Grunt_Rock` | 矿化战士 |
| `ED_Spider_Grunt_Guard` | 护卫异虫 |
| `ED_Spider_Grunt_Attacker` | 刀锋异虫 |
| `ED_Spider_Stalker` | 潜影异虫（隐身刀锋） |
| `ED_Spider_Buffer` | 典狱长 |
| `ED_Spider_Stinger` | 蛭尾异虫 |
| `ED_Spider_Tank` | 禁卫异虫 |
| `ED_Spider_Tank_Amber` | 氪化禁卫 |
| `ED_Spider_Tank_Ice` | 寒霜禁卫 |
| `ED_Spider_Tank_Mutated` | 放射性禁卫 |
| `ED_Spider_Tank_Rock` | 矿化禁卫 |
| `ED_Spider_Charger` | 重甲禁卫 |
| `ED_Spider_Tank_HeavySpawn` | 哨卫异虫 |
| `ED_Spider_ShieldTank` | 暴君异虫 |
| `ED_Spider_Tank_Boss` | 无畏异虫 |
| `ED_Spider_Boss_Heavy` | 巢主无畏 |
| `ED_Spider_Boss_TwinA` | 强弩无畏 |
| `ED_Spider_Boss_TwinB` | 重斧无畏 |
| `ED_Spider_Shooter` | 吐酸异虫 |
| `ED_Spider_Amber_Shooter` | 氪化吐酸 |
| `ED_Spider_Spitter` | 吐丝异虫 |
| `ED_Spider_Lobber` | 脓毒异虫 |
| `ED_Spider_RapidShooter` | 速射酸虫 |
| `ED_Spider_Exploder` | 自爆异虫 |
| `ED_Spider_Exploder_Warning` | 自爆群袭自爆虫 |
| `ED_Spider_ExploderTank` | 大自爆虫 |
| `ED_Spider_ExploderTank_King` | 自爆王虫 |
| `ED_Spider_ExploderTankGhost` | 幽灵大自爆 |
| `ED_SpiderSpawner` | 异虫母巢 |

废案或异常：`ED_Spider_Drone`（黄色双马尾纸片虫，无 AI）、`ED_Spider_Shooter_Ground`（只吐抛物线）、`ED_Spider_ShooterQueen`（酸液虫后）。

#### 6.3.2 异虫蝇科

| 描述符 | 中文名 |
| --- | --- |
| `ED_Mactera_Shooter_Normal` | 吐刺蝇 |
| `ED_Mactera_Shooter_Amber` | 氪化吐刺蝇 |
| `ED_Mactera_Shooter_HeavyVeteran` | 坚甲蝇 |
| `ED_Mactera_TripleShooter` | 三颚蝇 |
| `ED_Bomber` | 黏液轰炸蝇 |
| `ED_Bomber_Ice` | 冰霜轰炸蝇 |
| `ED_Bomber_Explosive` | 爆炸轰炸蝇（废案） |
| `ED_Grabber` | 捕手蝇 |

#### 6.3.3 纳多赛特科

| 描述符 | 中文名 |
| --- | --- |
| `ED_Jelly_Swarmer` | 电胞水母（蓝小水母） |
| `ED_JellyBreeder` | 饲育水母 |
| `ED_Jelly_Spawn` | 饲育水母幼体（粉小水母） |

#### 6.3.4 丘罗那科 / 纳亚卡科 / 雷鸣科 / 科洛克科

| 描述符 | 中文名 |
| --- | --- |
| `ED_Woodlouse` | 地虱（滚滚） |
| `ED_Woodlouse_Youngling` | 地虱幼体 |
| `ED_Shark` | 纳亚卡鲨虫 |
| `ED_PlagueShark` | 岩痘纳亚卡鲨虫 |
| `ED_InsectSwarm_Spawner` | 雷鸣蜂巢 |
| `ED_HydraWeed` | 科洛克王草 |

#### 6.3.5 自动机械 / 强敌科技

| 描述符 | 中文名 |
| --- | --- |
| `ED_InfectedMule` | 战斗矿骡 BET-C |
| `ED_PatrolBot` | 巡逻机器人 |
| `ED_PatrolBot_Caretaker` | 看守者召唤的巡逻机器人 |
| `ED_Shredder` | 粉碎者 |
| `ED_FacilityTurret_Barrier` | 推斥炮塔 |
| `ED_FacilityTurret_Burst` | 连射炮塔 |
| `ED_FacilityTurret_Sniper` | 狙击炮塔 |
| `ED_Prospector` | 勘探无人机 |
| `ED_Terminator` | 深渊复仇者 |

#### 6.3.6 其他 / 噬岩体 / 节假日

| 描述符 | 中文名 |
| --- | --- |
| `ED_InfestationLarva` | 肉食幼虫 |
| `ED_Spider_Hoarder` | 嗜矿异虫 |
| `ED_ShootingPlant` | 瘟疫吐珠 |
| `ED_BarrageInfector` | 瘟疫霰射吐珠 |
| `ED_CaveLeech` | 洞穴水蛭 |
| `ED_Flea` | 脓蚤 |
| `ED_PumpkinLootBug` | 掠夺虫（猫猫虫） |
| `ED_FlyingSmartRock` | 飞石 |
| `ED_TentaclePlant` | 瓦托克鳞甲荆丛 |
| `ED_PlagueLarva` | 岩痘幼虫 |
| `ED_Spider_Grunt_RockpoxPlague` | 岩痘战士 |
| `ED_Spider_Tank_RockpoxPlague` | 岩痘禁卫 |
| `ED_Spider_Shooter_Rockpox_Plague` | 岩痘吐酸 |
| `ED_Spider_Exploder_Rockpox_Plague` | 岩痘自爆 |
| `ED_Bomber_Rockpox_Plague` | 岩痘黏液轰炸蝇 |
| `ED_JellyBreeder_RockpoxPlague` | 岩痘饲育水母 |
| `ED_WalkingPlagueheart` | 岩痘腐化者 |
| `ED_Crawler` | 匍行核岩孽生兽 |
| `ED_GreatEggHunt_SpringBunny` | 春季兔兔 |
| `ED_Halloween_Skull` | 万圣骷髅头 |
| `ED_Nisse` | 圣诞小精灵 |

> 表里没列到的敌人，可以先用 `ED_Spider_Grunt` 之类的占位测试，或在游戏内沙盒 mod 的敌人列表里找英文名。

### 6.4 别名与黑话

社区玩家常用黑话。**AI 遇到黑话时先查本表**，再决定用哪个描述符；不要直接把黑话写进 JSON（`Base` 与池 `add`/`remove` 只认描述符名或本文件里定义的自定义名）。

| 玩家可能说 | 对应描述符 | 正式名 | 备注 |
| --- | --- | --- | --- |
| 滚滚 | `ED_Woodlouse` | 丘罗纳地虱 | 当玩家提到滚滚，默认是大滚滚小滚滚是`ED_Woodlouse_Youngling` |
| 大自爆、大字报 | `ED_Spider_ExploderTank` | 大自爆虫 | 黄金变体是 Crassus，特殊事件，不受池控制 |
| 自爆王虫 | `ED_Spider_ExploderTank_King` | 自爆王虫 | 与大自爆同族但独立描述符 |
| 幽灵大自爆 | `ED_Spider_ExploderTankGhost` | 幽灵大自爆 | 无法被杀死 |
| 水蛭、抓抓、Grabby Patty | `ED_CaveLeech` | 洞穴水蛭 | “娃娃机”偶尔也有人用来指它，默认指复仇者 |
| 王草、大王草 | `ED_HydraWeed` | 科洛克王草 | 需打掉附属物才能打本体。**歧义词：用户只说“王草”时先问是大王草还是小王草** |
| 小王草 | `ED_TentaclePlant` | 瓦托克鳞甲荆丛 | 机制类似王草，体量小。**别默认取这条** |
| 吐珠、土著、土猪 | `ED_ShootingPlant` | 瘟疫吐珠 | 外形像豌豆射手 |
| 速射 | `ED_Spider_RapidShooter` | 速射酸虫 |  |
| 大吐珠、霰射吐珠、喷子、马桶、肥珠、肥猪 | `ED_BarrageInfector` | 瘟疫霰射吐珠 | 吐珠的连射变体 |
| 蝎子、蝎子莱莱、抓抓 | `ED_Spider_Stinger` | 蛭尾异虫 | 尾巴抓人；与洞穴水蛭的“抓抓”区分 |
| 小水母、Jellies | `ED_Jelly_Swarmer` | 电胞水母 | 与蜂拥异虫是不同敌人 |
| 蜂拥、粉红仔、Pinkies | `ED_Spider_Swarmer` | 蜂拥异虫 | 蜂群级敌人 |
| 禁卫 | `ED_Spider_Tank` | 禁卫异虫 | — |
| 暴君 | `ED_Spider_ShieldTank` | 暴君异虫 | **与禁卫不同**；社区常混用，AI 要确认 |
| 巢主 | `ED_Spider_Boss_Heavy` | 巢主无畏异虫 | — |
| 强弩 | `ED_Spider_Boss_TwinA` | 强弩无畏异虫 | 双子之一 |
| 重斧 | `ED_Spider_Boss_TwinB` | 重斧无畏异虫 | 双子之一 |
| 沙鲨、鲨鱼 | `ED_Shark` | 纳亚卡鲨虫 | 沙地游动 |
| 猫猫虫、猫猫虫、豆泥猫 | `ED_PumpkinLootBug` | 掠夺虫 | 打死掉矿，可作为资源补给 |
| 嗜矿异虫、嗜矿 | `ED_Spider_Hoarder` | 嗜矿异虫 | 会逃跑，身上带矿 |
| 苍蝇、蚊子 | `ED_Mactera_Shooter_Normal` 等 | 异虫蝇 | 变体见 6.3.2 |
| 娃娃机 | `ED_Terminator` | 深渊复仇者 | — |
| BETC | `ED_InfectedMule` | 战斗矿骡 BET-C | — |

**使用规则**：

1. 先查本表，再查 6.3 代码表；两者都没有时，让用户描述外形或行为。
2. 抓抓这类有两种含义的词，先确认一句再写。
3. 翻译成描述符后再写进 JSON；黑话只用于对话。
4. 用户说“不要某只怪”时，翻译后按第 7.2 节找它所属的池。

### 6.5 阅读复杂社区代码

社区难度常把“描述符名”和“真实血统”拆开写。按名字读会错，按下面的顺序读。

#### 6.5.1 三步读法

1. **先看池**：只读五个池的 `add`。不在任何 `add` 里的**自定义名**不会刷，直接跳过。
   - `clear: false` 时池里**还有原版内容**；`add` 为空只表示“没加自定义怪”，不表示“该类怪不刷”。
   - **例外**：原版键（`ED_*`）被覆盖时，即便不写进任何 `add`，池里那份原版成员也会跟着变——“定义但未进池”的原版键不是死条目。
2. **追 `Base`**：对池里的每个名字，沿 `Base` 追到“不是自定义名”的那一个（通常是 `ED_*` 原版），那才是真正刷出来的怪。
   - 自引用分两种：**自定义名自引用**实测不生成（`EnemyClass` 空）；**原版键同名自引用**未实测，别直接判死（见 6.1.1）。
   - **链可能不终止**：社区有 `Base` 互指成环的写法，读到环就标“行为未定义”。
3. **沿链收集覆盖项**：`Elite`、`Scale`、`TimeDilation`、`PawnStats`、`MinSpawnCount` / `MaxSpawnCount` 可能写在不同层，后层覆盖前层。

#### 6.5.1a 进池只是入选，还要过四道筛选

`add` 里有名字不等于会刷出来。对每个进池名再核四项，任一不过就标“进池但不落地”：

| 筛选 | 检查 | 反例 |
| --- | --- | --- |
| 配额 | `MinSpawnCount` / `MaxSpawnCount` 是否都 > 0 | 0/0 是禁用占位 |
| 开关 | `CanBeUsedForConstantPressure` 与 `CanBeUsedInEncounters` 是否至少一个 true | 全 false 只能靠事件/脚本 |
| 容量 | `DifficultyRating` 是否在池预算内；`DisruptiveEnemyPoolCount` / `EnemyDiversity` / `MaxActive*` 是否容得下 | Boss 进干扰槽可能崩 |
| 池位 | 怪的类型是否匹配第 7.2 节的池映射 | 固定敌人塞进干扰池 |

#### 6.5.2 六种常见混淆手法

| 手法 | 表现 | 读法 |
| --- | --- | --- |
| 键名克隆原版名 | 键叫 `ED_PumpkinLootBug`、`ED_Shredder`，实际是自定义条目 | 不要按名字认怪，看 `Base` |
| 键名与 `Base` 错位 | 键 `ED_PumpkinLootBug` 的 `Base` 是刀锋异虫 | 以 `Base` 为准 |
| 多层继承 | `赛博刺客` → `ED_Shredder` → `ED_PumpkinLootBug` → 刀锋 | 追到链底再下结论 |
| 池里只放中间层 | 池里只有 `赛博刺客`，`ED_Shredder` 与猫猫虫都没进池 | 从池里的名字开始追，不要从定义表开始 |
| 定义但未进池 | 整份代码 43% 的条目不在任何池里 | 先筛池，再看其余 |
| 句子作键名 | `巢主建议亮黄甲C4炸下一轮核弹炸死`、`小孩A` | 键名只是标签，不影响血统 |
| 池位错放 | 固定敌人塞进 `DisruptiveEnemies`，或 Boss 进干扰槽 | 用第 7.2 节映射表核对 |

#### 6.5.3 一个真实例子

某社区难度里的“猫猫虫”：

```text
CommonEnemies.add "赛博刺客"
  → 赛博刺客.Base = "ED_Shredder"
    → ED_Shredder.Base = "ED_PumpkinLootBug"（Elite: true）
      → ED_PumpkinLootBug.Base = "ED_Spider_Grunt_Attacker"
```

正确结论：刷出来的是 **12~15 只带精英特效的刀锋异虫**（`ED_PumpkinLootBug` 的 `Base` 换成刀锋 + `Elite: true` 挂 `BP_MOD_MakeElite` 组件），不是掠夺虫，也不是粉碎者。召唤路径不换类（会出普通掠夺虫），自然刷才生效（2026-11-25 实测）。**想保留掠夺虫外形就别改 `Base`**——`Base` 指刀锋等于换类；只写 `Elite: true` + 开池生成才能得到精英掠夺虫。只读键名或只读 `Base` 都会得到错误答案。

#### 6.5.4 复杂度对比（四份含描述符的样例）

| 样例 | 描述符条目 | 最大 Base 链深 | 名不符实 | 定义但未进池 | 池 add |
| --- | --- | --- | --- | --- | --- |
| 官方 Haz5 / 5.5 / 5++ | 0 | 0 | 0 | 0 | 0 |
| 社区自制（日文） | 57 | 2 | 12 | 15 | 110 |
| 个人作品 | 7 | 1 | 0 | 1 | 6 |
| 材料 5x3 多样性 | 60 | 3 | 18 | 26 | 34 |

> “池 add 多”不等于“结构复杂”（日文那份是同一只怪重复进多个池）；真正拉高理解成本的是继承链深度、名不符实与未进池比例。另有两份官方基线（Haz5 / 5.5 / 5++）不含描述符，未列入。
>
> 表中“名不符实”是结构计数（键名以 `ED_` 开头且 `Base` 不等于键名），包含正常的变种命名；语义上的真错位约为它的一半左右。引用这个数字时说明口径。

---


---



---

## 7. 池操作



### 7.1 五个池

```json
{
  "CommonEnemies": {
    "clear": false,
    "add": ["ED_Spider_Grunt", "我的自定义怪"],
    "remove": ["ED_Spider_Swarmer"]
  }
}
```

| 池 | 包含什么（B） | 何时改 |
| --- | --- | --- |
| `EnemyPool` | 本局已构建的遇敌池 | **核选项**：任务中也能立即加，但加的怪可能没有正确生成逻辑（不刷或刷爆），慎用 |
| `CommonEnemies` | 战士、禁卫及其同族变体（刀锋、护卫、暴君） | 任务前 |
| `SpecialEnemies` | 吐丝/吐酸、自爆、异虫蝇及变体、蜂群 | 任务前 |
| `DisruptiveEnemies` | 捕手、典狱长、滚滚、速射酸虫、黏液轰炸蝇、大自爆 | 任务前 |
| `StationaryEnemies` | 吐珠、霰射吐珠、母巢、饲育水母、洞穴水蛭、荆丛 | 任务前 |

规则：

- `add` 可写**原版描述符名**（`ED_*`，见第 6.3 节）或 **`EnemyDescriptors` 里的自定义名**。
- 写错名字会被校验器抓：`Could not find EnemyDescriptor=<名字>`。
- 自定义怪只在 `EnemyDescriptors` 里定义、没进任何池 → 不会刷出。
- 加进 `CommonEnemies` 等源池会占用池槽位，且必须任务前改。
- 清空某类池后（`clear: true` 且 `add` 空），按**剩余种类数**定 `MinPoolSize` 与 `EnemyDiversity`：
  - 还剩 ≥3 种：`MinPoolSize` 4–6、`EnemyDiversity` 1–2（B，社区口径）。
  - 只剩 1 种：`MinPoolSize` 写 **1**、`EnemyDiversity` 1–1（C，未实测；4–6 与「总池应低于 Diversity」冲突）。
  - 不压会出“想要的不刷”。
- 只 `remove` 一两种、没清空池时，`MinPoolSize` 与 `EnemyDiversity` 保持原值即可；删到剩余少于 3 种再按上面规则重算（C，未实测）。
- **`clear: true` + `add` 非空**是合法写法，等于“该类只留这几个”：先清空再把 `add` 的名字放回（社区常用，语义与 `clear: false` 的增量不同）。
- **四源池写同一份名单**（甚至把 `EnemyPool` 也写成同一份）是社区常态，作用是抹平池分工、让所有类都从同一批怪里抽；代价是 `MinPoolSize` / `EnemyDiversity` 的算法前提被打破，容易“想要的不刷”。
- **在难度文件里写 `EnemyPool.clear: true`** 也是社区写法（3 份配置），但建池阶段是否有内容可清、会不会把基础池清空导致不刷，未实测；保守用 `clear: false` + `remove`。

### 7.2 怪物 → 池映射

“不要某只怪”时，先查它在哪个池：

| 怪物 | 池 |
| --- | --- |
| 战士、刀锋、护卫、禁卫、暴君、重甲禁卫、哨卫 | `CommonEnemies` |
| 吐丝、吐酸、自爆、异虫蝇、三颚蝇、蜂群、粉碎者 | `SpecialEnemies` |
| 捕手、典狱长、滚滚、速射酸虫、黏液轰炸蝇、大自爆 | `DisruptiveEnemies` |
| 吐珠、霰射吐珠、母巢、饲育水母、洞穴水蛭、荆丛 | `StationaryEnemies` |
| 无畏、巢主、双子、王草、OMEN | 任务/事件生成，通常不在池里（B） |
| 勘探无人机、嗜矿异虫、Crassus 大自爆、掠夺虫 | 特殊事件，不受池控制（B） |

注意：

- 同一个怪可能同时出现在多个池；**不确定时四个源池（`CommonEnemies` / `SpecialEnemies` / `DisruptiveEnemies` / `StationaryEnemies`，不含 `EnemyPool`）都写 `remove`**，比漏删安全。
- `remove` 只影响本难度；不会改游戏原文件。
- 若任务里仍见到该怪，再在 `EnemyPool.remove` 里加一次（源池的 `remove` 在任务前重建池时生效，`EnemyPool` 是已构建池；B）。
- **事件生成的怪写 `remove` 基本空转**：王草、无畏、OMEN、掠夺虫不走源池，CD 也没有“禁事件”键。不报错，但别承诺生效（B）。
- **社区改用“停用式覆盖”**（B，8 份配置的共通写法，未实测）：把原版键覆盖成低配额 + 双开关关 + 稀有度拉满：

  ```json
  "ED_InfectedMule": {
    "Base": "ED_Spider_Grunt",
    "CanBeUsedForConstantPressure": false,
    "CanBeUsedInEncounters": false,
    "MinSpawnCount": 0,
    "MaxSpawnCount": 0,
    "Rarity": 99,
    "PawnStats": {}
  }
  ```

  它同时压四道闸：配额 0、两条刷怪路径都关、稀有度拉满。社区用它停 BET-C / 王草 / 勘探无人机 / 娃娃机 / 黄金大自爆，但**没有实测能证明事件生成也被拦住**——先按“可能有效”交付，并说明这是待验写法。
- **只剩 1 种怪会不会刷未验证**（C）：池过滤能把池压到一种，但单怪池的波次表现没有实测记录，先让用户开一局确认“至少会刷”。

---


---



---

## 8. 校验与排错



| 报错 | 原因 | 修法 |
| --- | --- | --- |
| `Invalid JSON` | 语法错（尾逗号/注释/字符串里直接换行） | 用 JSON 校验器过一遍，多行描述用 `\n` |
| `Root value must be an object` | 顶层是数组或字符串 | 改成 `{...}` |
| `Unknown difficulty attribute: X` | 键名拼错或不在 46 键里 | 对照第 4.3 节 |
| `X must be a string` | `Name` 或 `Description` 写成了数字 | 加引号 |
| `X must be a number or array of numbers with size of 4` | 抗性数组不是 4 项或含非数字 | 恰好 4 个数字 |
| `X must be a number` | 标量位置写了字符串或对象 | 改数字 |
| `X must be <= 80` | `ResupplyCost` 超上限 | 降到 80 以内 |
| `X must be between 0 and 1.33` | `HazardBonus` 越界 | 收进 0~1.33 |
| `X must be a weighted array of ranges` | 区间缺 `range` 或 `weight` | 按 `{weight, range:{min,max}}` 写 |
| `X must be a map of enemy descriptors` | `EnemyDescriptors` 不是对象 | 改成 `{名字:{...}}` |
| `X must be an enemy pool config` | 池缺 `clear`/`add`/`remove` | 三件套写全 |
| `X must be an array of strings` | `SeasonalEvents` 混入数字 | 全字符串 |
| `X must be a FriendlyHealthComponent` | `EscortMule` 缺键或类型错 | 四个数字键 |
| `Could not find EnemyDescriptor=X` | `Base` 或池引用了不存在的名字 | 用第 6.3 节代码表或已定义的名字 |
| `Base object not valid` | `Base` 指向的不是描述符 | 指向 `ED_*`，不是 `ENE_*` |

---


---



---

## 9. 配方库



> 所有配方都是差异写法，其余键自动补 Haz5。

### 9.1 整体难一点（伤害与数量各 +20%）

```json
{
  "Name": "Haz5 +20%",
  "EnemyDamageModifier": [3.36, 3.6, 3.84, 4.08],
  "EnemyCountModifier": [1.02, 1.02, 1.5, 1.8]
}
```

### 9.2 更多怪，但单只不更肉

```json
{
  "Name": "Horde Lite",
  "EnemyCountModifier": [1.2, 1.2, 1.8, 2.2],
  "MaxActiveEnemies": 120,
  "MaxActiveSwarmers": 120,
  "EnemyDamageResistance": [1.2, 1.2, 1.4, 1.5]
}
```

### 9.3 怪更肉，但数量不变

```json
{
  "Name": "Tanky",
  "EnemyDamageResistance": [1.8, 1.9, 2.1, 2.3],
  "ExtraLargeEnemyDamageResistance": [1.6, 1.7, 1.9, 2.1],
  "EnemyDamageModifier": [2.8, 3, 3.2, 3.4]
}
```

### 9.4 快节奏（波次更密、开局即战）

```json
{
  "Name": "Fast Pace",
  "EnemyWaveInterval": [{ "weight": 1, "range": { "min": 120, "max": 150 } }],
  "EnemyNormalWaveInterval": [{ "weight": 1, "range": { "min": 80, "max": 110 } }],
  "WaveStartDelayScale": 0
}
```

### 9.5 资源紧张

```json
{
  "Name": "Scarce",
  "ResupplyCost": 80,
  "StartingNitra": 0,
  "PointExtractionScalar": 0.7
}
```

> 补给成本已是原版上限 80，**无法更贵**；`StartingNitra: 0` 也是默认值。本配方真正生效的是 `PointExtractionScalar: 0.7`——按 5.6 节的 `165 / 该值`，采点潮间隔从 Haz5 的约 150 秒拉到约 236 秒，**潮更稀、单局出怪更少**（省心，不是加压）。

### 9.6 单刷友好

```json
{
  "Name": "Solo Friendly",
  "EnemyDamageModifier": [2.2, 3.0, 3.2, 3.4],
  "EnemyCountModifier": [0.6, 0.85, 1.25, 1.5],
  "ReviveHealthRatio": 0.4,
  "MaxActiveEnemies": 90
}
```

> 只有第 1 项（单人）降伤害；后 3 项保持 Haz5 原值（3.0 / 3.2 / 3.4），避免多人时失衡。

### 9.7 关掉固定敌人

```json
{
  "Name": "No Stationary",
  "StationaryEnemies": { "clear": true, "add": [], "remove": [] },
  "StationaryEnemyDiversity": [{ "weight": 1, "range": { "min": 0, "max": 0 } }]
}
```

### 9.8 加一只自定义精英怪

```json
{
  "Name": "Custom Elite",
  "EnemyDescriptors": {
    "ED_PumpkinLootBug": {
      "Base": "ED_Spider_Grunt_Attacker",
      "Elite": true,
      "MinSpawnCount": 1,
      "MaxSpawnCount": 2,
      "DifficultyRating": 30,
      "Rarity": 15,
      "Scale": 1.25,
      "PawnStats": {
        "PST_KineticResistance": 50,
        "PST_MovementSpeed": 1.2
      }
    }
  },
  "CommonEnemies": { "clear": false, "add": ["ED_PumpkinLootBug"], "remove": [] }
}
```

> 精英写法（B，2026-11-25 实测）：`Elite: true` 要挂在**原版键覆盖**上，并靠**自然刷**触发——实测覆盖 `ED_PumpkinLootBug` + `Base` 刀锋 + `Elite: true`，自然刷出大量精英刀锋；召唤同一配置只出普通掠夺虫。自定义名 + 中间层套娃的写法在当前版本未复现（自定义名召唤无特效、自引用不生成）。原因已定位：`Base` 名不在 CD 的 `EnemyDescriptorReferences` 表里就跳过克隆，字段留默认、`EnemyClass` 空（A，`Set DifficultyAttribute` 逻辑核对）。`Base` 决定生成类：指刀锋就换类；想保留原类目前没有可靠写法。分类抗性是百分比（`100` 免疫、`50` 减半、`-100` 双倍），`Physical` 只管近战、枪械用 `Kinetic`；`PST_MaxHealth` 无效；整体硬度用 `PST_DamageResistance`（乘数）。

### 9.9 复刻原版 Haz5

```json
{
  "Name": "Vanilla Haz5",
  "ExtraLargeEnemyDamageResistanceC": [0.7, 0.8, 1.1, 1.7],
  "ExtraLargeEnemyDamageResistanceD": [0.75, 0.8, 1, 1.3],
  "EnvironmentalDamageModifier": 2.0
}
```

### 9.10 全老兵（高风险）

```json
{
  "Name": "All Veterans",
  "VeteranNormal": [{ "weight": 1, "range": { "min": 1, "max": 1 } }],
  "VeteranLarge": [{ "weight": 1, "range": { "min": 1, "max": 1 } }]
}
```

> 老兵与精英是两套机制：老兵是生成时替换形态，精英是另一种标记；不要用 `MaxActiveElites` 去“加强老兵”。

---


---



---

## 10. 常见坑



1. **不是把 json 文件丢进文件夹**：走界面粘贴 + 存档。
2. **4 元素数组 = 1/2/3/4 人**，不是四档难度。
3. **缺省 = Haz5**，不是“保持原版当前难度”。
4. **CD1 的 Haz5 ≠ 原版 Haz5**（环境伤害与 OMEN/精英抗性有偏差），见第 4.5 节。
5. **`HazardBonus` 不能超 1.33**，`ResupplyCost` 不能超 80。
6. **`Description` 里直接换行会让 JSON 失效**，用 `\n`。
7. **自定义怪必须进池才会刷**；进了源池会占槽位，且必须任务前改。
8. **`EnemyPool` 是核选项**：任务中也能生效，但可能不刷或刷爆；加怪前先小范围试。
9. **把某些怪塞进某些池可能崩溃**，尤其 Boss、事件怪、炮塔；试之前做好存档。
10. **`SeasonalEvents` 最新版可能已失效**，别指望靠它关事件。
11. **`EnvironmentalDamageModifier` 社区未观察到效果**，别把它当核心调节项。
12. **`MinPoolSize` 语义反直觉**，小步调，别一次拉到几百。
13. **`Rarity` 越高越稀有**，不是“越高越常见”。
14. **`PawnStats` 写错键不报错**，只是静默无效。
15. **同名难度**：建议 `Name` 带版本，避免下拉框里分不清。
16. **倍率与点数叠乘**：同时乘 `EnemyCountModifier` 和 `*Difficulty` 会双重放大，容易失控。
17. **低难度 + 高倍率**：必须同时抬全局上限，否则会“刷出即删”，体感是旧怪凭空消失。

---


---



---

## 12. 附录：Haz5 基线



46 键完整写法。适合精确复刻、分享或作为模板改。

```json
{
  "Name": "Hazard 5",
  "Description": "",
  "MaxActiveCritters": 40,
  "MaxActiveSwarmers": 60,
  "MaxActiveEnemies": 60,
  "ResupplyCost": 80,
  "StartingNitra": 0,
  "ExtraLargeEnemyDamageResistance": [0.75, 0.8, 1.2, 1.7],
  "ExtraLargeEnemyDamageResistanceB": [0.7, 0.7, 1, 1.3],
  "ExtraLargeEnemyDamageResistanceC": [0.6, 0.7, 1, 1.6],
  "ExtraLargeEnemyDamageResistanceD": [1.3, 1.3, 1.5, 1.65],
  "EnemyDamageResistance": [1.2, 1.2, 1.4, 1.5],
  "SmallEnemyDamageResistance": [1.2, 1.2, 1.2, 1.2],
  "EnemyDamageModifier": [2.8, 3, 3.2, 3.4],
  "EnemyCountModifier": [0.85, 0.85, 1.25, 1.5],
  "EncounterDifficulty": [
    { "weight": 1, "range": { "min": 100, "max": 200 } },
    { "weight": 6, "range": { "min": 300, "max": 400 } },
    { "weight": 1, "range": { "min": 400, "max": 500 } }
  ],
  "StationaryDifficulty": [
    { "weight": 1, "range": { "min": 0, "max": 0 } },
    { "weight": 1, "range": { "min": 150, "max": 200 } },
    { "weight": 1, "range": { "min": 250, "max": 350 } }
  ],
  "EnemyWaveInterval": [
    { "weight": 2, "range": { "min": 160, "max": 180 } },
    { "weight": 7.5, "range": { "min": 230, "max": 280 } },
    { "weight": 0.5, "range": { "min": 299, "max": 299 } }
  ],
  "EnemyNormalWaveInterval": [
    { "weight": 4, "range": { "min": 120, "max": 150 } },
    { "weight": 2, "range": { "min": 180, "max": 250 } }
  ],
  "EnemyNormalWaveDifficulty": [
    { "weight": 1, "range": { "min": 350, "max": 450 } }
  ],
  "EnemyDiversity": [
    { "weight": 2, "range": { "min": 1, "max": 2 } },
    { "weight": 8, "range": { "min": 3, "max": 4 } },
    { "weight": 1, "range": { "min": 5, "max": 6 } }
  ],
  "StationaryEnemyDiversity": [
    { "weight": 1, "range": { "min": 2, "max": 4 } }
  ],
  "VeteranNormal": [
    { "weight": 3, "range": { "min": 0, "max": 0 } },
    { "weight": 9, "range": { "min": 0.15, "max": 0.3 } },
    { "weight": 2, "range": { "min": 0.4, "max": 0.6 } },
    { "weight": 0.5, "range": { "min": 1, "max": 1 } }
  ],
  "VeteranLarge": [
    { "weight": 1, "range": { "min": 0.175, "max": 0.175 } }
  ],
  "DisruptiveEnemyPoolCount": { "min": 3, "max": 3 },
  "MinPoolSize": 8,
  "MaxActiveElites": 0,
  "EnvironmentalDamageModifier": 1.1,
  "PointExtractionScalar": 1.1,
  "HazardBonus": 1.33,
  "FriendlyFireModifier": 0.7,
  "WaveStartDelayScale": 0,
  "SpeedModifier": 1.15,
  "AttackCooldownModifier": 1.5,
  "ProjectileSpeedModifier": 1.7,
  "HealthRegenerationMax": 0.1,
  "ReviveHealthRatio": 0.1,
  "EliteCooldown": 0,
  "EnemyDescriptors": {},
  "EnemyPool": { "clear": false, "add": [], "remove": [] },
  "CommonEnemies": { "clear": false, "add": [], "remove": [] },
  "SpecialEnemies": { "clear": false, "add": [], "remove": [] },
  "DisruptiveEnemies": { "clear": false, "add": [], "remove": [] },
  "StationaryEnemies": { "clear": false, "add": [], "remove": [] },
  "SeasonalEvents": ["SE_ProspectorDrone", "SE_ProspectorDataDeposit", "SE_RivalBomb"],
  "EscortMule": {
    "FriendlyFireModifier": 0.1,
    "BigHitDamageModifier": 0.5,
    "BigHitDamageReductionThreshold": 6,
    "NeutralDamageModifier": 0.1
  }
}
```
