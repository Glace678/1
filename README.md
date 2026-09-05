# OpenMU Solo（仓库 1 · v2）

MU Online Season 6 私服一体化改造：**单机即开即玩、简体中文、高帧率、全手柄+震动、三平台桌面软件包**。

> 📌 完整状态、bug 修复记录、需求对照表见 **[`报告.md`](报告.md)**；总体方法论见 [`docs/改造计划书.md`](docs/改造计划书.md)；移动端路线见 [`docs/移动端触控与手势适配设计.md`](docs/移动端触控与手势适配设计.md)。

## 仓库结构

```
1/
├── MuMain/     客户端（C++/SDL3，fork 自 sven-n/MuMain + solo 定制：手柄/震动/帧率系统/简中/Rime 输入）
├── OpenMU/     服务端（C#/.NET，fork 自 MUnique/OpenMU + solo 定制：本地启动器/GM 桌面软件/数值预设/打包工具链）
├── .github/workflows/desktop-packages.yml   三平台构建流水线（Linux/macOS/Windows 软件包）
├── docs/       计划书 + 移动端设计
└── 报告.md     最新整合报告（先读这个）
```

**游戏素材不在本仓库**（447MB，版权归 Webzen，仅限个人研究）：CI 构建时自动从上游
`sven-n/MuMain` 的 `data-7fce146eb3fe34a0` 发布下载并校验 SHA256；本地手动构建请同样获取后
解压到 `MuMain/src/bin/Data/`。

## 拿现成软件（推荐）

Actions → 最新 `Desktop Packages` 运行 → Artifacts：
- **Windows**：`OpenMU-Local-*-win-x64.zip` → 解压双击 `OpenMU-Local.exe`（游戏+GM+数据库一体）
- **macOS (Apple Silicon)**：`OpenMU-Solo-osx-arm64-unsigned.tar.gz` → `OpenMU-Game.app` / `OpenMU-GM.app`（未签名：右键打开或 `xattr -dr com.apple.quarantine .`）
- **Linux**：`OpenMU-Solo-linux-x64-unsigned.tar.gz` → `./OpenMU-Game` / `./OpenMU-GM`

本机启动默认**自动登录**（记住凭据后跳过登录表单直达选角）；数值默认安装 Solo balance 预设，GM 面板可再调。

## 从源码构建

- 客户端：`MuMain/docs/build/`（Windows 预设 `cmake --preset windows-x64`；Linux/macOS 见对应指南）。需要 CMake≥3.25、.NET SDK 10、平台 C++ 工具链；Linux/macOS 另需 turbojpeg、glew、librime。
- 服务端/GM：`OpenMU/`（.NET 10；开发运行见 `docs-website/docs/getting-started/from-source.md`）。
- 打桌面包：`OpenMU/tools/local-package/README.md`（Windows 本地）与 `OpenMU/tools/cloud-build/Build-CloudDesktop.ps1`（Linux/macOS）。
- 部署给别人玩：`OpenMU/deploy/all-in-one`（Docker Compose 一键，VPS 场景）。

## 上游

- 客户端 https://github.com/sven-n/MuMain ｜ 服务端 https://github.com/MUnique/OpenMU（MIT）
- 本仓库为个人研究用途的整合分叉，请勿商用或公开再分发素材。
