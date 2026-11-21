# 更新日志

所有重要变更记录在此文件，格式基于 [Keep a Changelog](https://keepachangelog.com/zh-CN/1.1.0/)。

## [1.1-beta] - 2026-11-21

### 新增

- USB 直通容器功能：设置面板一键扫描宿主机 USB 设备、勾选直通、保存后自动 bind-mount 到容器 `/dev/bus/usb`
- 支持 Linux（sysfs `/sys/bus/usb/devices` + `lsusb`）和 Windows（PowerShell `Get-PnpDevice -Class USB`）双平台扫描
- Windows 端扫描结果使用 `USB:VID:PID` 短格式路径，PowerShell 输出强制 UTF-8 编码并带 UTF-16 LE 自动降级转换

### 修复

- `parseLSUSB` 数组越界 panic：原字符串索引逻辑在某些 lsusb 输出上产生反向切片 `[58:18]`，改用正则 `ID\s+([0-9a-fA-F]{4}:[0-9a-fA-F]{4})\s+(.+)$` 安全提取
- `scanUSBSysfs` 过滤条件写反：`!strings.Contains(name, ":")` 实际跳过了所有真正的 USB 设备父节点，导致扫描结果为空。修复为跳过带冒号的 interface 节点和 `usb*` 根 hub
- 设置持久化丢失：`normalizeSettings()` 漏掉 `ContainerSettings.Devices` 字段，每次 Save 都重置为空数组，补上 `cleanStringList` 搬运
- 上下文自动压缩触发逻辑不一致：`preprocessTurns` 和 `CompressTurns` 内部用不同阈值判断，统一为 `budget` 计算
- `guessContextWindow` 无条件覆盖用户显式设置：改为 `else if`，用户设置优先
- 渗透 Agent 工具缺失自动安装：`ensureMissionTools` 仅在 `runPlan` 调用，普通 `run` 流程漏掉，修复为循环前也调用
- 上下文进度条三层硬截断：移除 `contextmanager.go` / `agentservice.go` / `ChatPanel.vue` 中 100% 硬截断，允许超额显示
- Windows 交叉编译 `.syso` 冲突：`wails3 build` 残留 `wails_windows_amd64.syso` 与 Makefile 生成的 `rsrc_windows_amd64.syso` 同时带 `.rsrc` section 导致 Go 链接报错，Makefile 在 `windows-resource` 开头清理 Wails 残留，`.gitignore` 增加 `*.syso`

### 改进

- 设置对话框默认 tab 从「终端」改为「常规」
- README 新增 USB 直通操作指南、三平台实现原理、独占警告与恢复方法
- Windows 资源版本号自动同步，Makefile / Taskfile / `build/config.yml` 版本统一为 `1.1-beta`

---

## [1.0-beta] - 2026-11-10

### 新增

- 跨平台桌面应用骨架（Linux / Windows / macOS），基于 Wails v3 + Vue 3 + Go
- AI Agent 对话与自主渗透双模式，流式展示 reasoning_content / thinking 字段
- Docker / Podman 容器双后端，Kali Rolling 工作容器自动构建
- 容器原生 PTY 终端（xterm.js），本地 shell 与容器 shell 双模式切换
- Skills 专家指令集管理面板：导入 / 导出 / 启停 / 排序 / 筛选
- 内置 2 个实战级 Skill：`sql_injection.md`（SQL 注入全流程）、`rustscan.md`（RustScan 从安装到实战）
- 安全边界：非 IT / 网络安全话题主动拒绝，命令输出和上下文按 token 截断
- 任务进度实时流式推送，暂停 / 接管终端 / 中断恢复
- 报告面板自动整理 findings、风险等级、修复建议，支持 Markdown / JSON 导出
- 模型预设（deepseek / qwen / glm / Ollama），一键切换本地与云端
- Wails bind 类型自动生成，前后端类型安全

### 已知限制

- 无 USB 直通功能（1.1-beta 补齐）
- Windows / macOS 容器网络能力受限（bridge / raw socket / 混杂模式）
- Agent 步数上限硬编码，后续可配置
