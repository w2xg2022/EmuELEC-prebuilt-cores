# EmuELEC-prebuilt-cores

预编译 [w2xg2022/EmuELEC](https://github.com/w2xg2022/EmuELEC) 的重量级模拟器核心，供主建置**直接下载安装**，避免在主建置（尤其云端 CI）里重新编译这些核心，节省建置时间与磁盘。

> 本仓库前身为 `EmuELEC-MAME`（旧名只预编 MAME）。现已扩展到 15 平台的整套核心，故改名。旧 URL 由 GitHub 自动转址，不会失效。

## 为什么需要这个仓库

重核心（MAME、RetroArch、PPSSPP、Flycast、N64 等）编译耗时长、耗磁盘巨大，是云端 CI（GitHub Actions 免费 runner）建置慢/失败的主因之一。把它们的编译拆到本仓库单独处理、发布产物，主仓库对应的 `package.mk` 改成直接下载，从根本避开主建置的资源瓶颈。

- 主建置**完整编译**时间：约 7–8h（跨接力 session）→ 核心固化后可大幅缩短。
- 主建置**增量**：约 30–60min。

## 包含哪些核心

### libretro 核心（单文件 `<name>_libretro.so`）
`mame2003-plus` `mame2010` `nestopia` `snes9x` `genesis-plus-gx` `mgba` `pcsx_rearmed` `fbneo` `dosbox-pure` `applewin` `parallel-n64` `yabause` `beetle-pce`(mednafen_pce_fast) `ppsspp` `flycast`

### 独立模拟器 / 前端（打包整个 install 树成 `<pkg>.tar.zst`）
`flycastsa`（DC）`PPSSPPSDL`（PSP）`retroarch`
> 独立模拟器不是单一 `.so`，是「二进制+库+数据」整棵 install 树，故用 tarball。DC/PSP 让**其他型号**（能跑独立模拟器的好芯片）也复用。

## 产物 = 两种消费方式

| 类型 | 产物 | 主建置 package.mk 怎么用 |
|---|---|---|
| libretro | `xxx_libretro.so` | `curl` 下载到 `usr/lib/libretro/` |
| standalone | `xxx.tar.zst` | 下载后解压到 `${INSTALL}` |

## 编译方式（并行）

`.github/workflows/build-cores.yml`：**matrix 6 组并行**——`mame2003-plus`/`mame2010` 各独立（最重），其余按轻重分 `ra-psp` / `dc-psp-lr` / `mid` / `light` 四组。手动触发（`workflow_dispatch`，可指定 EmuELEC 的 ref）。每组各自编工具链（~1h）+ 本组核心，wall-clock ≈ 2h（顺序编要 4–5h）。

## Release 命名 & ⚠️ ABI 耦合

- 标签：`cores-<EmuELEC 主仓库 commit 短hash>`，对应该次编译的工具链 ABI。
- 主建置抓 **`releases/latest`**，所以 **EmuELEC 的工具链/glibc 一变，就要重跑本 workflow**，否则预编译 `.so` 与主建置 ABI 不匹配 → 核心加载失败。工具链很少变，可接受；变更后手动重跑即可。
