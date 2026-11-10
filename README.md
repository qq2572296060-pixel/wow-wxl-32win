# wow-wxl-32win · WarcraftXL 32 位客户端注入版（core 1.1）

> 打包日期：2026-11-11　| 构建日期：2026-11-10 07:40–07:56
> 真机验证：**1999022 `pandarenserpentmount` 模型 / 贴图 / 粒子 / 动画全对**；`.morph 1999000~1999023` 整批通过
> 平台：**Win32 / PE32**（3.3.5a 客户端是 32 位，64 位构建不可用）

---

## 一、目录结构

```
wow-wxl-32win/
├── bin/                                   ← 直接可部署的成品（不需要编译）
│   ├── WarcraftXL.dll                     core，注入主体
│   ├── d3d9.dll                           D3D9 代理入口（可选载体）
│   ├── wxl-patcher.exe                    给 Wow.exe 加 .wxl 节 / 导入项（幂等，自动备份 .orig）
│   └── Extensions/
│       ├── wxl-db2/wxl-db2.dll            FileDataID → 路径解析 + 光照表 + 外观配方
│       └── wxl-modern-m2/wxl-modern-m2.dll MD21 原生读取 / 粒子 / combiner / anim 剥壳
├── src/
│   ├── core-1.1/                          上游 core 源码（CMake）—— **core 三件的来源**
│   │   └── extensions/                    ★ 编译时实际用的两个扩展（上游 zip 里没有）
│   ├── repos/                             两个扩展仓库工作树副本（已剔除 .git / store）
│   │   ├── wxl-modern-m2/                 = 构建树 + 1 处本地补丁
│   │   └── wxl-db2/                       与构建树零差异（22/22 md5 相同）
│   ├── patches/local-01-combiner-kblendone.patch   唯一一处本地修（diff 原文）
│   ├── ORIGIN.md                          **谁编出谁**：逐件对应 + commit + 比对证据
│   └── core_v11.zip                       上游 core 原始快照（未含 extensions）
└── wow-wxl-32win_core1.1_20261111.zip     整包副本（本目录的压缩）
```

## 二、成品清单（md5 于 2026-11-11 从本目录现取）

| 文件 | 字节 | md5 |
|---|---|---|
| `bin/WarcraftXL.dll` | 682,496 | `cefd3e19139d7fa4f3938d888e66e536` |
| `bin/d3d9.dll` | 142,848 | `bb0cff27cd8c575e515244a455333642` |
| `bin/wxl-patcher.exe` | 156,160 | `9ac95e75bf29b9c07a988dba13d8b601` |
| `bin/Extensions/wxl-db2/wxl-db2.dll` | 334,336 | `c05e4fdb5d3de49e404387659b2896ba` |
| `bin/Extensions/wxl-modern-m2/wxl-modern-m2.dll` | 210,432 | `832497e0636392b94dc48d5683d7f2f4` |

重算：`md5sum bin/*.dll bin/*.exe bin/Extensions/*/*.dll`

## 三、部署（客户端根 = `<CLIENT>`）

```bash
# 1) 先备份原版（只需一次）
cp <CLIENT>/Wow.exe <CLIENT>/Wow.exe.orig
cp <CLIENT>/DivxDecoder.dll <CLIENT>/DivxDecoder_orig.dll   # 若之前装过第四代 divxproxy

# 2) 放件
cp bin/WarcraftXL.dll bin/d3d9.dll            <CLIENT>/
cp -r bin/Extensions                          <CLIENT>/

# 3) 打补丁（幂等；已打过会跳过）
<CLIENT>/wxl-patcher.exe <CLIENT>/Wow.exe

# 4) 完全退出 Wow.exe 再启动（改了 DLL/DBC 都必须整进程重启）
```

启动成功判据（`<CLIENT>\Logs\wxl-core.log`）：
```
extensions: loaded 'wxl-db2' v1
extensions: loaded 'wxl-modern-m2' v1
extensions: 2 loaded
wxl-core ready
```

**回滚**：删 `WarcraftXL.dll` / `d3d9.dll` / `Extensions\`，用 `Wow.exe.orig` 覆盖 `Wow.exe`。

## 四、从源码重建

```bash
cmake -S src/core-1.1 -B build -G "Visual Studio 17 2022" -A Win32 -DCLIENT_PATH=<CLIENT>
cmake --build build --config Release --parallel
# 产物落在 build/Release/，并自动 deploy 到 CLIENT_PATH
```

- **必须 VS 生成器**（Ninja 不支持 `-A Win32`；32 位是硬要求）。
- 依赖 `deps/`（flatbuffers 等）已随源码；首次配置需要网络拉 CMake 依赖时按 `src/core-1.1/README.md`。
- ⚠ 扩展源码里有一处**本机本地修**：`wxl-modern-m2/src/render/CombinerPatch.cpp` 用 `kBlendOne = 1`
  代替 core v1.1 未提供的 `gx::blend::kOne`（改扩展侧，**不要改 core**）。

## 四·五、源码来源（与 `E:\src\WarcraftXL` 到底什么关系）

**有关联，但不是"整个目录都是它的源码"**。逐件比对结果（2026-11-11 现取，详见 `src/ORIGIN.md`）：

| 二进制 | 来源 | 关系 |
|---|---|---|
| `WarcraftXL.dll` · `d3d9.dll` · `wxl-patcher.exe` | `src/core-1.1/`（= `core_v11.zip`，上游 1.1 release） | 三件由该树直接编出 |
| `wxl-modern-m2.dll` | `src/repos/wxl-modern-m2/`（`dc0b7df`，2026-11-06） | **41 个文件相同 / 1 个不同**，差异 = `CombinerPatch.cpp` 的 `kBlendOne` 本地修 |
| `wxl-db2.dll` | `src/repos/wxl-db2/`（`30e4f2c`，2026-08-13） | **22/22 md5 全同，零差异** |
| —（**不是**来源） | `E:\src\WarcraftXL\wxl-core`（`aa4d014`，2026-08-03 浅克隆） | 与 `core-1.1/ src/` 有 **107 处差异**，且 `src/core`/`src/events`/`build_host` 只在仓库侧 ⇒ 结构已分叉 |

⇒ 所以**两个扩展仓库的源码已随包带上**（`src/repos/`，677 KB，仅源码不含 `.git`/`store`）；
`wxl-core` 仓库**没有**打进来——它不是本包二进制的源码，带了反而误导。

## 五、版本说明（别按体积/日期猜）

- `wxl-patcher.exe` 由本源码树（含 v1.1 的 `src/patcher/NamedPatchArchives.cpp` · `GlueUnlock.cpp`）编出；
  但**客户端现有 `Wow.exe` 是更早一次补丁的产物**，两者是否同源 **不可判** —— 需要时重跑一次 patcher（幂等）。
- 扩展 `wxl.json` 的 `abi` 必须是 **1.1**，与 core 版本不匹配时扩展会被拒载（日志有 `abi` 字样）。
- 与第四代 `divxproxy`（`DivxDecoder.dll`）**不要同时装**：两代抢同一条渲染链，切代前把另一代撤干净。

## 六、配套文档（技能内，随技能更新）

- `~/.workbuddy/skills/wow-高版本素材提取/references/18-wxl-core纯注入权威流程.md` —— 机制 / 编译 / 资产两件事 / 判据 / 诊断三态
- `~/.workbuddy/skills/wow-高版本素材提取/binaries/README.md` —— 二进制台账（本包的同源副本在技能 `binaries/wxl-core-1.1/`）
- 素材侧（`TextureFilePath.db2` 怎么造、零售表怎么补）：同技能 `scripts/gen_texturefilepath_db2.py` · `scripts/extract_retail_db2.py`
