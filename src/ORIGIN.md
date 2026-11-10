# 源码来源对照（谁编出了 bin/ 里那五件）

> 结论先给：**有关联，但不是"整个 `E:\src\WarcraftXL` 都是它的源码"**。
> 五个二进制 = **上游 core 1.1 release（`core_v11.zip`）** + **两个扩展仓库（`wxl-modern-m2` / `wxl-db2`）+ 1 处本地补丁**。
> 仓库 `wxl-core` **不是**本包的 core 来源（证据见下表第 4 行）。

## 一、逐件对应关系（2026-11-11 现取比对）

| 二进制 | 源码 | 对应证据 |
|---|---|---|
| `bin/WarcraftXL.dll`<br>`bin/d3d9.dll`<br>`bin/wxl-patcher.exe` | `src/core-1.1/`（= `src/core_v11.zip`，上游 1.1 release 原样） | 这三件由该树 `cmake --build` 产出；构建目录 `build/Release/` 内同名文件 md5 与 `bin/` 逐件相同 |
| `bin/Extensions/wxl-modern-m2/wxl-modern-m2.dll` | `src/core-1.1/extensions/wxl-modern-m2/`<br>≈ `src/repos/wxl-modern-m2/`（+1 补丁） | 与该仓库逐文件比对：**41 个相同 / 1 个不同**，唯一差异 = `src/render/CombinerPatch.cpp`；补丁原文 `src/patches/local-01-combiner-kblendone.patch` |
| `bin/Extensions/wxl-db2/wxl-db2.dll` | `src/core-1.1/extensions/wxl-db2/`<br>= `src/repos/wxl-db2/` | 逐文件比对：**22/22 全部 md5 相同，零差异** |
| —（不是来源） | `../wxl-core`（`E:\src\WarcraftXL\wxl-core`） | 与 `core-1.1` 的 `src/` 有 **107 处差异**，且 `src/core` / `src/events` / `build_host` 只存在于仓库侧 ⇒ **结构已分叉，不是同一份代码**；仓库 `main` 只有 2026-08-03 一个提交 `aa4d014`（浅克隆） |

## 二、仓库版本（git 现取）

| 仓库 | commit | 日期 | 说明 |
|---|---|---|---|
| `wxl-modern-m2` | `dc0b7df` | 2026-11-06 | Merge PR #8（manatactical/main） |
| `wxl-db2` | `30e4f2c` | 2026-08-13 | feat(ModelStore): AppearanceStore |
| `wxl-core` | `aa4d014` | 2026-08-03 | **仅供参考，不是本包来源** |

> ⚠ `src/repos/` 是**工作树副本，已剔除 `.git`（`wxl-modern-m2` 69 MB / `wxl-db2` 657 KB）与 `store/`（69 MB / 516 KB）**，只保留源码。
> 需要提交历史请回 `E:\src\WarcraftXL\wxl-modern-m2` / `wxl-db2` 原仓库。

## 三、那 1 处本地补丁改了什么

`wxl-modern-m2/src/render/CombinerPatch.cpp`：core v1.1 的 `gx::blend` 只声明 `kSrcAlpha(5)` / `kInvSrcAlpha(6)`，
扩展却引用 `kOne` ⇒ 编译不过。**改扩展侧、不动 core**：本地写 `constexpr unsigned kBlendOne = 1 /* D3DBLEND_ONE */`。
完整 diff → `src/patches/local-01-combiner-kblendone.patch`。

## 四、复现（从本目录重编，产物应逐字节等于 `bin/`）

```bash
# 1) 把扩展放进 core 的 extensions/（本包 src/core-1.1 已是此形态）
#    若用 src/repos 的干净副本，需先打补丁：
patch -p0 < src/patches/local-01-combiner-kblendone.patch   # 或用已修好的 src/core-1.1/extensions

# 2) 配置 + 编译（必须 32 位）
cmake -S src/core-1.1 -B build -G "Visual Studio 17 2022" -A Win32 -DCLIENT_PATH=<CLIENT>
cmake --build build --config Release --parallel

# 3) 校验：产物 md5 应等于 README 第二节的表
md5sum build/Release/WarcraftXL.dll build/Release/d3d9.dll build/Release/wxl-*.dll build/Release/wxl-patcher.exe
```

⚠ 「逐字节复现」受编译器版本 / 优化开关影响；同版本 MSVC 下实测该树可产出与 `bin/` **相同 md5** 的成品
（本包 `bin/` 就是从这套源码 + `build/Release` 原样拷出的）。换编译器版本后只需**功能等价**，别拿 md5 当绝对判据。
