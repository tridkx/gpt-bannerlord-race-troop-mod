# RaceTroopBuilder 替换器与固有护甲接入

适用：RaceTroopBuilder v0.2.0、Bannerlord v1.4.8 原版单人战役。用于把已注册的外部种族分配给现有原版兵种；制作新种族资源或修改升级边仍走种族工程流程。

## 工具定位与主要文档

在当前工作区寻找 `mods/RaceTroopBuilder/`；本机工程为 `D:/gpt-mb2-mod2/mods/RaceTroopBuilder/`。发布包自带 Windows x64 桌面运行时，不要求用户安装 .NET SDK。不要把工具或此技能复制进种族模块。

- [项目入口](D:/gpt-mb2-mod2/mods/RaceTroopBuilder/README.md)：GUI、目录、依赖与限制。
- [CLI 与工程格式](D:/gpt-mb2-mod2/mods/RaceTroopBuilder/docs/interface.md)、[工程 Schema](D:/gpt-mb2-mod2/mods/RaceTroopBuilder/schemas/project.schema.json)：命令和字段的主要规范。
- [固有护甲接口](D:/gpt-mb2-mod2/mods/RaceTroopBuilder/docs/armor.md)：种族默认数据、覆盖数据和运行入口的主要规范；接入前必须读取。
- [验证与验收](D:/gpt-mb2-mod2/mods/RaceTroopBuilder/docs/validation.md)：离线检查和游戏验收范围。

工作区移动时按实际工程根定位这些文档；交付包也保留 `README.md`、`docs/` 和 `schemas/`。字段或版本有变化时核对当前代码、Schema 和游戏 DLL，不把此适用版本推广到其他版本。

## 人类 GUI 工作流

运行工具工程或发布包的 `Launch-GUI.cmd`。扫描游戏目录和种族来源；默认读取当前工作区 `mods/`，可选择单个模块或其他来源目录，支持系列嵌套。一个模块 ID 只选一个实际来源，避免工程与已安装目录重复。

选择兵种、依赖种族、性别与脸参数，再选单节点、全部升级分支或文化范围。文化范围包括民兵、守卫等普通士兵，不等于普通招募线；不要扩大用户指定范围。完整身体通常使用固定脸参数并清除防具，具体性别和年龄必须符合依赖种族的成年 skin。

按需选固有护甲继承或覆盖，编辑战斗装备候选套，再点击“应用配置”。未应用的草稿不参与保存或导出。顶部语言菜单切换兵种和物品名称；搜索同时支持中文、英文与 ID，操作标签仍为中文。GUI 与 AI 共用 JSON 工程，GUI 更新时保留 AI 已配置的自定义名和平民装备。

保存工程，先检查和查看修改对比，再导出；用户需要安装时使用最近构建的检查报告校验安装。只更新输出模块，不代替用户复制依赖资源，不启动游戏。启动器从上到下为原版依赖 → 种族依赖 → 生成的兵种模块。

## AI 拼装工作流

发布包根目录的 `rtb.cmd` 或 `app/cli/RaceTroopBuilder.Cli.exe` 提供 JSON 输出；源码入口见主要接口文档。不要用显示名代替稳定 ID。

```powershell
.\rtb.cmd scan
.\rtb.cmd catalog --kind races
.\rtb.cmd catalog --kind troops --filter 帝国 --lang zh-CN
.\rtb.cmd catalog --kind items --filter sword --lang en
.\rtb.cmd troop --id imperial_recruit
.\rtb.cmd new --out my-troops.json --id MyRaceTroops --name "My Race Troops"
.\rtb.cmd configure --project my-troops.json --rule my-rule.json
.\rtb.cmd validate --project my-troops.json
.\rtb.cmd preview --project my-troops.json
.\rtb.cmd build --project my-troops.json --out output/build-001
.\rtb.cmd package --report output/build-001/MyRaceTroops.validation.json --out output/MyRaceTroops.zip
```

规则先按 Schema 写入 `my-rule.json`；`configure` 只在全工程检查通过后写回。`--game` 指定游戏目录，重复 `--mods` 指定种族来源。构建使用新的输出目录以保留旧构建；安装命令在已授权的安装范围内执行，报告路径来自本次成功构建。

退出码 0 为成功、2 为配置检查失败、1 为操作失败；按 JSON `issues[].code` 处理错误，不匹配中文提示文本。工程相对路径相对于 JSON 所在目录。名称字段包括 `nameEn/nameZh/nameKey`，`catalog` 根据 `--lang` 返回 `displayName`；配置始终使用 ID，输出 XML 保留原版本地化键。

每个兵种只能指定一个种族。工具保留原版 ID、文化、等级、技能和升级关系；不支持新增招募入口或改升级边。同槽候选按原版机制随机，不保证整套配对：Item0 的两把单手候选不会同时占两槽，但可以与 Item1 的另一套双手候选组合。给弓弩等固定弹药槽，检查随机组合提示。

## 种族默认护甲接入约定

制作严格无装备但有护甲的种族时，使用运行组件和协议，不虚构 `skins.xml` 护甲属性，不自行加隐藏防具。接入按主要护甲文档完成：

- 默认数据属于种族依赖的 `ModuleData/rtb_race_armor.xml`，根为 `RaceArmor`、`schemaVersion="1"`；`Race` 节点用已注册的种族 `id`，明确填写 `head/body/arms/legs` 四项 0～500 整数。数值来自用户要求或已确认的种族设计，不从五毒等案例套常数。协议文件由运行组件读取，不注册为 Items、NPCCharacters 或 skins。
- 默认值作用于该种族的人形角色，包括英雄和未被替换器选中的兵种。兵种规则省略 `armor` 或 `mode=inherit` 时沿用；`mode=override` 明确提供四项并仅替换固有部分，零值有效。实际装备护甲独立相加，原版特长随后计算；清除防具使用单独的 `stripArmor`。
- 生成模块只保存覆盖，不复制种族默认文件或模型资源；有默认协议或覆盖配置时自动附带运行入口。种族独立使用时，选择发布包的 `runtime-provider/RaceTroopArmor` 作为依赖，或按主要文档将 DLL 和 SubModule 入口集成到该种族。不要只放默认 XML 就宣称固有护甲已生效。
- 未声明协议的种族默认值未知；继承不干预已有机制。外部 DLL 硬编码护甲需接入协议或专门适配，不能承诺仅靠加载顺序可靠覆盖。多个默认声明或重复兵种覆盖应修正冲突，不用重复叠加解决。

## 验证与边界

默认文件、注册种族、性别、模块依赖和目标范围检查通过后，对实际 NPCCharacters 应用 XSLT，检查目标与无关节点。安装还核对种族默认文件和其他依赖的版本、哈希；源工程与已安装依赖不一致时停止安装并修正来源。

护甲组件针对原版 Sandbox 战役属性更新，使用 net472 和随包 Harmony；其离线真实方法补丁编译不代表已在游戏中验收。用户测试空防具默认值、兵种覆盖和零值、升级与读档、骑乘／徒步及特长变化。替换整体计算模型或另改最终护甲的模组需要兼容回归。固有护甲不自动增加装备负重、价值或自动战斗战力；自动战斗保持原版规则，除非用户另有要求。
