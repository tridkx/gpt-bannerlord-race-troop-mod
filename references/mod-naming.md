# 种族 Mod 命名规范

新建种族 Mod 或种族 Mod 合集时，工程文件夹与 Mod 名称须包含 `race`，大小写不限，推荐采用 `Race`。命名时同时检查以下位置，便于在工作区、交付包及模组管理器中辨认：

| 位置 | 要求与例子 |
| --- | --- |
| 独立种族工程文件夹与模块 ID | 使用含 `Race` 的英文 ID，例如 `YueQingShuRace` |
| 种族系列／合集工程文件夹 | 名称含 `Race`，例如 `Xianjian7Race` 或 `Xianjian7RaceCollection`；仅作为工程容器时不额外创建游戏模块 |
| 可安装种族合集的模块 ID | 同样含 `Race`，例如 `Xianjian7RaceCollection` |
| `SubModule.xml` 的 Module/Name | 英文名称含 `Race`，例如 `Yue Qing Shu Race` 或 `Xianjian 7 Race Collection` |
| 交付包的模块目录与包名 | 沿用模块 ID，使种族包同样可辨认 |

工程 ID 使用英文字母、数字、下划线或连字符；游戏内角色、兵种和种族显示名可用中文，不要求添加英文 `race`。本规范用于种族资源 Mod 及其合集，不强制普通装备 Mod 或仅依赖种族资源的兵种配置 Mod 改名。

现有工程按本次任务授权范围改名，并同步工具与当前文档中的路径。仅改系列／合集容器时，保留各子模块 ID、资源名、GUID、版本、旧交付包和历史标签；不要为符合命名规范擅自改写已发布模块 ID、内部 race ID 或存档依赖。历史报告与历史变更记录保留原路径，当前路径在工作区历史路径对照中维护。
