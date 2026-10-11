# 种族与完整角色资源

## 种族、身体与装备的关系

种族 skin 决定基础外观和骨架；Monster 配置动作和相关身体/碰撞参数；兵种 NPCCharacter 指向种族并设置性别、身体参数、装备和升级目标。种族不是阵营、文化或等级，同一个文化可以有多个种族兵种。

同一模块可以装多个 TPAC、多个种族和兵种树。不同模型能否共用绑定骨架、尺寸与碰撞参数，要比较其比例和绑定姿态，不能因为都叫“少女模型”就共用全部参数。沿用原版动作也不要求模型等同原版成人比例，但骨骼语义、顺序和层级须与采用的资源路径兼容。

原版外观变化来自性别/年龄基础皮肤、脸部形变、发型胡须、身体参数与装备。一个种族不只一种外观；但相同性别和年龄阶段的两套完整固定身体，不会因为 skin 名不同就按兵种自动分配。不要把 gender、mesh_maturity_type 或 morph_key 当任意模型选择开关。

如果模型差异主要是衣服，可共用种族并改装备。完整角色已经合成身体时，要改成装备路径需设计身体遮盖、脸/发保留和穿戴槽；不能直接同时加载两套完整人形。需要同一内部种族且指定模型时，应明确做可验证的替换机制，而不是擅自改成两个 ID。

## 当前本机可用的注册组织

本机 WuduRaceTest 基线使用 Native v1.4.8.119303，未来版本需重新核对加载入口和结构。

| 文件/资产 | 职责 |
| --- | --- |
| `AssetPackages/*.tpac` | 当前模型自己的身体/隐藏占位 MetaMesh、材质贴图和绑定骨架 |
| `ModuleData/skins.xml` | race 与性别/年龄阶段 skin，body_meta_mesh、骨架、最小缩放及其他身体槽 |
| `ModuleData/monsters.xml` | 自定义 Monster，原生结构和骨骼映射、动作、尺寸等 |
| `ModuleData/action_sets.xml` | 引用原生动作集，但指向当前绑定骨架的自定义动作集 |
| `ModuleData/project.mbproj` | 该版本引擎的 skin/action_set/monster 文件入口 |
| `ModuleData/<troops>.xml` / `.xslt` | 兵种定义及实际替换规则 |
| `SubModule.xml` | 模块 ID、英文名、版本、依赖、当前版本适用的 XML 入口，以及可选运行时 DLL |

该版本参考的 project.mbproj：

```xml
<base type="solution">
  <file id="soln_skins" name="ModuleData/skins.xml" type="skin" />
  <file id="soln_action_sets" name="ModuleData/action_sets.xml" type="action_set" />
  <file id="soln_monsters" name="ModuleData/monsters.xml" type="monster" />
</base>
```

这是文件加载例证，不是完整模块模板。不要猜一个 `<XmlName id="skins">` 就能加载 skin；读取本机 Native 和可用工程确认当前规则。Monster/NPCCharacters 在参考模块中还使用 SubModule.xml 的相应入口。是否需要运行时 DLL 由实际加载和 UI 创建路径决定；纯数据注册可满足的种族不额外加 DLL，自定义骨架需要 UI 适配时按下述制作规范实现。

## 皮肤构建

从当前 Native 的适用 race/skin 结构复制后，修改自己的名称、骨架、身体与槽引用；保留性别/年龄键和当前引擎需要的容器。参考工程覆盖原版十种性别/年龄变体以避免 UI 或预览落到缺失皮肤；不要把“十种”固化为未来版本的常数。

固定完整角色的已验证方案：脸、头发、衣服、身体和双手均在一个身体 MetaMesh 中；其他自动生成槽用自己的微小内部占位网格，不用无效名称或空引用。在参考工程中 `uses_stitching=false`、`body_mesh_suffix` 为空、独立 `hands_mesh` 为隐藏占位。它适用于该完整身体方案，不是所有自定义种族都应关闭拼接。

完整身体包含自身衣服时，保留原兵防具会叠加装备或触发身体遮盖。按目标删去相关装备条目，保留用户要求的武器、盾牌和坐骑；检查每个战斗与平民 EquipmentRoster，以及装备集合引用，不能只删除第一个 roster。

固定脸也要保留原版所需的发型/胡须容器和零号占位。不是把所有 skin 子节点清空就能禁用脸部生成；启动与招募失败条件见故障参考。

## 变换、缩放和动作

- 源网格与骨架统一转换单位和朝向，测人体本身与含头饰总高，记录定义；不将观察“到成人胸口”硬编码为通用米数。
- 模型顶点尺寸、skin 的 min_scale、运行时 AgentScale、Monster 镜头与胶囊是不同因素。用同地面场景、骨骼头脚位置及全局变换判定实际高度；实体包围盒可能包含武器和布料。
- 改 wrist 支点时核对前臂末端及扭转骨的连接关系，调整附近混合权重，保持非目标部位的世界绑定姿态。不要借手指任务重绑整个人体。
- 沿用原生骨架语义与动作，先检查站立、跑步、持剑、持弓、骑乘等相关动作；手部形变、布料求解、IK 和碰撞另行验证。28 骨是当前 human 样本，不是所有生物通用骨数。
- 布料参数应按当前模型拟合；最终包需有匹配的预处理数据。读取装备技能的布料参考，不直接把 Wudu 或天香的胶囊、顶点编号和活动距离复制到新模型。

## 捏脸与人物预览制作规范

使用自定义绑定骨架时，把站立人物预览纳入首次制作范围；捏脸初次创建与重建仅在用户明确要求捏脸界面支持时纳入范围。默认不制作捏脸形变或面部表情动画。下述捏脸专用入口和适配步骤仅适用于已纳入范围的任务。每个纳入任务范围的 UI 创建或重建入口都必须使用与当前种族、性别和绑定骨架匹配的动作集；预览重建后仍保持正确的局部绑定坐标、位置、缩放和动作进度。战场显示正常不能替代这个要求，也不能通过把源模型四肢拉伸到 human 骨架来满足预览。

### 配置动作集并覆盖创建与重建

从当前游戏 DLL/XML 确定界面实际使用的入口和后缀，为任务骨架注册所需男女动作集；保留所选性别，不为固定外观角色擅自强制女性 SkeletonType。检查动作集有效且其 skeleton 引用指向任务骨架。沿用原生动作轨道时，也要让 UI 通过自己的动作集使用自己的绑定骨架。

本机 Native v1.4.8 的入口与动作如下；未来版本需重新核对，不将本表当作固定接口：

| 入口 | 动作集与动作 |
| --- | --- |
| `BodyGeneratorView` 捏脸初次创建 | 通过 `MBGlobals.GetActionSetWithSuffix(monster, isFemale, "_facegen")` 选集；通道 1 使用 `act_inventory_idle_start` → `act_inventory_idle`。原生女性 facegen 集的动画片段为 `anim_female_custom_start` → `anim_female_custom`，对应骨骼动画 `anim_female_idle_long` |
| `BodyGeneratorView` 捏脸重建 | 该版本重建的 `AgentVisualsData` 未传 `.ActionSet(...)`，默认动作集索引 0 会落到 `as_human_warrior`。必须在此路径提供或恢复当前种族动作集，不能只修初次创建 |
| `CharacterTableau` 站立人物预览 | 默认选 `_warrior` 集，动作同为 `act_inventory_idle_start` → `act_inventory_idle`，片段 `inventory_idle_start` → `inventory_idle` 使用 `anim_inventory_idle`；显式 `IdleAction` 或其他指定动画可覆盖默认动作 |
| `BasicCharacterTableau` 旧预览入口 | 使用 `GetHardcodedAnimationSystemDataForHumanSkeleton`；若目标界面实际走此入口，要单独适配，不能假定所有预览都走 `CharacterTableau` |

该版本动作集名由 `ActionSetCode.GenerateActionSetNameWithSuffix` 生成：`as_` +（非空 `Monster.BaseMonster`，否则 `Monster.StringId`）+ 女性时的 `_female` + 后缀。核对最终生成名称与注册项，不只看 Monster 的 `action_set` 字段。捏脸切换性别、年龄、身体参数、衣着或其他引起人物重建的操作后，仍须使用匹配的集；衣着关闭时 Equipment 可为空。

### 实现完整的 UI 骨架重建

优先在可控制的创建／重建入口明确传入正确的 ActionSet。原生入口不能直接配置时，将 UI 适配作为正式运行模块实现并随模组交付，和只读诊断模式分开。处理范围限定为当前任务种族的 UI `AgentVisuals`，跳过 MissionScreen、战场 Agent、其他种族以及已使用正确骨架的对象。

重建前保存原人物数据、实体位置和朝向、缩放、通道 0/1 的动作与进度。用原数据副本只改必要字段，保留 race、Monster、SkeletonType、装备、身体参数及 MorphAnims 等原设置。动作集无效或骨架引用不匹配时停止本次适配并记录原因，不退回 human 集继续运行。

Native v1.4.8 的 `AgentVisuals.Refresh` 在旧装备非空且武器未变时可能跳过骨架创建；仅换 ActionSet 或调用普通刷新不能保证换掉旧骨架。已验证的完整重建调用顺序是：

1. 在写入前确认私有 `_data` 字段与 `Refresh(bool, bool, Equipment, bool, bool)` 方法存在。版本不匹配时不修改对象。
2. 通过 `GetCopyAgentVisualsData()` 保留原数据。保存 `GetScale()` 与实体 `GetFrame()`；Frame 的 rotation 先按 `1 / scale` 去除已有缩放，再设置 `.ActionSet(ownSet).Frame(unscaledFrame).Scale(scale)`，避免二次缩放。
3. 写入 `_data`，调用 `Refresh(false, true, null, false, false)`；`removeSkeleton=true` 且 `oldEquipment=null` 用于完整重建，不表示清空人物原数据中的装备。
4. 在新骨架上恢复两条通道中非 `act_none` 的动作及原进度，再调用 `TickAnimationsAndForceUpdate` 更新姿态。保留原实体变换，不使人物跳位或动画重置。

私有成员方案仅适用于已核对的版本；升级时重新检查接口和分支语义。运行检查应限制扫描范围与日志数量，重复处理保持幂等，日志失败不影响游戏。此处规定结果与关键数据保留，不要求所有工程复制同一种扫描架构或艾达的 ID、骨架尺寸与检查间隔。

### 首次交付的 UI 验收

离线阶段核对动作集解析名称、任务范围内的创建／重建入口及最终绑定骨架；用户游戏阶段覆盖队伍／百科／物品栏站立预览，以及穿脱装备后恢复待机；用户要求捏脸界面支持时，另覆盖捏脸初次显示和重建。记录实际 race、Monster、SkeletonType、ActionSet 名称及 skeleton 引用、实体骨架名称、局部 rest frame、两条动画通道和进度；不要只记录骨数或 XML 声明。观察肩臂、袖口与手腕连接，同时检查缩放、位置和动画连续性。固定脸种族仅在用户要求捏脸界面支持时验证创建页面的显示与重建；此适配不等同于制作捏脸形变或表情。

把离线检查、运行时记录和用户实机反馈分别写入工程报告；未测试界面不记为通过。纯 UI 适配应核对 TPAC、源模型、绑定数据和权重保持不变，并回归战场行为。通过本流程避免为 UI 的错误骨架重绑完整身体。

本机实现参照：`D:/gpt-mb2-mod2/mods/ResidentEvil/REAdaWongRace/tools/UIProbe/Probe.cs`。REAdaWongRace v0.1.3 在 Native v1.4.8 中采用上述完整重建方式；2026-10-11 用户确认捏脸与待机预览修复完成。该例证支持 UI 骨架适配方法，不代表所有性别／年龄分支、手势形变、战场 IK 或性能均已验证。

## 最终检查重点

确认每个任务兵种选择的 race/gender/age 能找到皮肤与 Monster；每个身体/动作/骨架引用确实在当前加载集合中。检查最终 TPAC 的身体组成，确认双手或头部没有在拆分时从所有实际显示路径移除。使用自定义骨架时，按[捏脸与人物预览制作规范](#捏脸与人物预览制作规范)完成任务范围内的 UI 创建、重建及站立预览检查；捏脸和表情不作为默认交付要求，记录各界面验证范围。

记录当前游戏版本、目标兵种、原始输入哈希、最终包哈希、尺寸定义、实际生效的模块文件及用户已验收范围。使用新 ID 的种族或兵种要保留测试存档的依赖说明，不擅自重命名已发布 ID。
