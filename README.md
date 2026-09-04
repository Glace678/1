# MU-Freedom（仓库名：1）

本仓库是 **OpenMU（服务端）+ MuMain（开源客户端）** 的改造版代码快照，目标是把 MU Online（Season 6 Episode 3）改造成：帧率/分辨率自由切换、简体中文、Windows/macOS/Linux 全平台、方便部署、（规划中）全手柄支持、单机向数值体验。

> 完整背景、架构与后续任务见 `docs/改造计划书.md`（详细到文件级，可直接交给执行 AI）。
> 本次已完成的改动明细见 `修改清单.md`。

## 上游与基线

| 组件 | 上游 | 基线 |
|---|---|---|
| 服务端 | `https://github.com/MUnique/OpenMU` | master @ 2026-09-04（MIT） |
| 客户端 | `https://github.com/sven-n/MuMain` | main @ 2026-09-04 |
| 客户端资源 | MuMain Releases | `data-7fce146eb3fe34a0`（约 447MB，**不在本仓库内**） |

**本仓库不含游戏素材**（`Data/` 资源体）。运行时素材请从上游 MuMain Releases 获取对应的 `data-<id>` 包，按官方"Assemble a release"流程与本仓库客户端产物组装。

## 本次已完成（v1）

### 1. 帧率自由切换（M1，客户端）
- 选项界面新增"帧率"下拉：**30 / 60 / 90 / 120 / 144 / 180 / 显示器上限(实时显示 Hz) / 无限制**，切换即时生效并持久化到 `config.ini [Render] FpsPreset`。
- 新增 `MuSetFpsPreset` / `MuReapplyFpsPreset`（延迟到帧首应用，与既有 VSync 机制同构）；切换分辨率/显示器后自动按新显示器刷新率重挂。
- VSync 开关逻辑已改为尊重帧率预设（固定档位优先于 VSync 默认目标）。
- 游戏逻辑速度与帧率无关是上游既有保证（`docs/adr/0001-preserve-25-fps-reference-timing.md`），本次未改。
- 注意：选项窗口为容纳新行，"窗口模式"行与关闭按钮整体下移 39px（边框 slat 数 30→34），如上游布局变动需重新核对坐标。

### 2. 简体中文（M2，部分完成）
客户端（MuMain）：
- 新增 `src/Localization/Game.zh-CN.resx`（3227 条，基于 zh-TW 简繁转换 + 大陆游戏术语校正）
- 新增 `Metadata.zh-CN.resx`、`Editor.zh-CN.resx`
- 新增 UI 键：`Frame rate` / `Monitor maximum` / `Unlimited`（en、zh-TW、zh-CN 均已补）
- `tools/ResxGen/CppEmitter.cs` 注册 `zh-CN` 显示名；选项界面语言下拉新增"简体中文"
- **中文字体已内置**：`src/bin/fonts/NotoSansSC-Regular.otf` / `NotoSansSC-Bold.otf`（思源黑体，OFL 许可）。发行配置建议 `[UI] Font=Noto Sans SC`（或留空走平台默认字体管线）
服务端（OpenMU，.NET 卫星资源，缺键自动回退英文）：
- `PlayerMessage.zh-CN.resx`（196/196，全部游戏内系统消息）
- `Web/AdminPanel` `Resources.zh-CN.resx`（289/289，GM 面板界面）
- `Web/Shared`（65）、`Web/Map`（21）、`DataModel`（19）

**M2 未完成部分（后续任务，按优先级）：**
1. `Dialog.zh-CN.resx`（233 条 NPC 对话，当前回退英文）——直接翻译 `Dialog.en.resx` 即可，机制相同
2. `Data/Local/<语言>/` 数据文本：物品名、套装说明、地图名图片等 bmd 文件（需用客户端内置编辑器构建 `-DENABLE_EDITOR=ON`，F12 打开，或仓库内 bmd 读写工具；新增语言目录后在 `config.ini [LOGIN] Language=` 配置映射，入口代码 `Winmain.cpp` `g_strSelectedML`）
3. 服务端 `PlugInResources.resx` 中文（插件名/描述，约 1163 条，面板内可见）
4. 中文文案质量人工校对（本次为程序化转换 + 术语表校正）

### 3. 部署与 GM（M3）
服务端本身即 .NET + Docker 全平台：`server/deploy/all-in-one` 一键 compose（OpenMU + PostgreSQL + nginx）。GM 系统为浏览器管理面板（任何操作系统），游戏内 GM 命令齐备。详见 `docs/改造计划书.md` 第 5 章与下方"快速部署"。

### 4. 数值改造（M5）
无需改代码：全部数值在服务端数据库，通过管理面板 Configuration 页修改。`docs/改造计划书.md` 第 7 章给出"单人舒适向"推荐档位（经验 ×10–30、怪物削弱、掉率提升、商店补齐、传送降费、混沌机器成功率上调等）与操作步骤。

## 尚未开始（后续执行）
- **M4 全手柄支持 + 震动**（计划书第 6 章：四层架构、29 个界面适配清单、17 项震动事件表、分期方案）
- M1 验收测试（高刷显示器逐档验证）
- M2 剩余项（见上）

## 快速上手

### 运行服务端
```bash
cd server/deploy/all-in-one
docker compose up -d --no-build
# 打开 http://localhost/ → 立即创建管理员账号 → 再暴露公网（务必先建账号+HTTPS）
```

### 构建客户端（三平台指南见上游 `docs/build/`）
需要：CMake ≥3.25、.NET SDK 10、C++ 工具链（Windows: VS2022；Linux: gcc/clang + Vulkan 驱动；macOS: Xcode，arm64）。
```bash
cd client
cmake --preset <平台preset>      # 如 windows-x64 / linux-x64-release 等，见 CMakePresets.json
cmake --build --preset <对应release preset>
# 产物在构建目录，需与 MUnique.Client.Library 网络库同目录（构建自动产出）
# 游戏数据：从上游 Releases 下载 data-<id> 解包到可执行文件旁
```
连接服务器：`config.ini [CONNECTION SETTINGS] ServerIP/ServerPort`（MuMain 用 **44406** 端口）。

### 客户端配置（config.ini）
```ini
[Render]
VSync=1
FpsPreset=0      ; 0=显示器上限, -1=无限制, 30/60/90/120/144/180=固定
[UI]
Locale=zh-CN     ; 界面语言
Font=Noto Sans SC
[LOGIN]
Language=Eng     ; 游戏数据文本目录（中文数据目录完成后改为对应值）
```

## 已知注意事项
1. **未编译验证**：本环境无客户端构建链（无 CMake/.NET SDK），C++ 改动为逐行对照源码模式编写，执行 AI 拿到后第一件事是全平台编译并修掉可能的编译问题（预计风险点：选项窗口布局常量、I18N 新键在生成代码中的标识符 `I18N::Game::FrameRate` / `MonitorMaximum` / `Unlimited`）。
2. 上游迭代很快，合并上游新提交时优先保留本仓库新增文件（冲突概率低），选项窗口/Winmain 的挂点需人工核对。
3. 素材版权属 Webzen，仅限个人学习研究，勿公开再分发素材包。
4. 服务端分布式部署（distributed）上游标注损坏，勿用；all-in-one 足够。

## 目录结构
```
1/
├── README.md            本文件
├── 修改清单.md           本次全部改动的文件级明细
├── docs/改造计划书.md    总体执行计划书（含后续任务）
├── client/              MuMain 客户端改造版源码（不含 Data 素材）
└── server/              OpenMU 服务端改造版源码
```
