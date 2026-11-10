# NOTICE / 来源与声明

## 一、本仓库是什么

`WarcraftXL` **core 1.1（Win32 构建）** 的成品与源码打包：让 **World of Warcraft 3.3.5a (build 12340)** 客户端
**直接读取 12.1 的模型资源**（M2 `MD21` / `.skin` / `.blp`）——零售文件原样进 MPQ，DLL 在内存里翻译，**不做离线降级、不改模型文件**。

- 平台：**Win32 / PE32**（3.3.5a 客户端是 32 位，64 位构建不可用）
- **验证范围：仅坐骑。** 12.1 商城坐骑**已全部验证通过**（模型 / 贴图 / 粒子 / 动画四项全对）；
  武器、护甲、建筑、WMO 等其它类别**未验证**，不代表可用
- 本仓库**不含任何暴雪游戏资产**（无 `.m2` / `.blp` / `.dbc` / MPQ）

## 二、上游来源与版本

| 组件 | 上游 | 版本 | 许可 |
|---|---|---|---|
| `WarcraftXL.dll` · `d3d9.dll` · `wxl-patcher.exe` | WarcraftXL core（`core_v11.zip`，见 `src/`） | core **1.1** | GPL-3.0 |
| `wxl-modern-m2.dll` | `wxl-modern-m2` | `dc0b7df`（2026-11-06）**+1 处本地补丁** | GPL-3.0 |
| `wxl-db2.dll` | `wxl-db2` | `30e4f2c`（2026-08-13），零差异 | GPL-3.0 |

- 逐件对应关系、比对证据、commit、复现步骤 → **`src/ORIGIN.md`**
- 唯一一处本地修改 → `src/patches/local-01-combiner-kblendone.patch`
  （core 1.1 的 `gx::blend` 未提供 `kOne`，扩展侧本地写 `constexpr unsigned kBlendOne = 1 /* D3DBLEND_ONE */`，**不改 core**）

## 三、许可证

本仓库整体按 **GNU GPL v3** 发布（上游全部为 GPL-3.0，见 `LICENSE`）。
源码随二进制一并提供（`src/`），符合 GPL-3.0 的源码可获取要求。

`src/core-1.1/deps/` 内含第三方库（如 flatbuffers），各自保留原许可，见其目录内声明。

## 四、免责声明

- 本项目**与 Blizzard Entertainment 无关**，未获其授权或认可。
- 仅供**私服研究 / 单机学习**使用；使用需自备合法取得的游戏客户端。
- 不提供、不分发任何游戏本体文件或美术资源。
- 修改客户端可执行文件（`wxl-patcher`）可能违反游戏服务条款；由此产生的一切后果由使用者自负。
