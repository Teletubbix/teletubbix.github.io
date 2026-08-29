---
title: 系统配置详情
published: 2026-08-29
description: Arch Linux (WSL2) 开发机的系统配置、工具链与维护记录
image: "/covers/notes.png"
tags: ["系统", "配置", "学习笔记"]
category: 学习笔记
draft: false
---

# Arch Linux (WSL2) 系统配置详情

> 用途：让下一个对话 / AI / 未来的自己快速熟悉这台电脑的**运行情况与工作方式**。
> 生成时间：2026-08-28（全新重写，替代旧的 SYSTEM_HEALTH.md / AI_HANDOFF.md / system_info.png）。
> 覆盖：系统概览、软件包、开发工具链、交叉编译、Shell 与代理、git/GitHub、工作区与项目、服务、Windows 侧资源、维护记录。

---

## 一、系统概览

| 项 | 值 |
|----|----|
| 发行版 | **Arch Linux** (x86_64, WSL2)，主机名 `Aerial` |
| 内核 | 6.18.33.2-microsoft-standard-WSL2 |
| Shell | **zsh 5.9** + oh-my-zsh + powerlevel10k |
| Locale | `zh_CN.UTF-8`（~/.zshrc 导出；系统默认 C.UTF-8） |
| 时区 | Asia/Shanghai（CST, UTC+8） |
| CPU | 12th Gen Intel Core i7-12700H（20 线程） |
| 内存 | 15 GiB（可用约 13 GiB） |
| GPU | WSL2 内为虚拟显卡（Microsoft Basic Render Driver / WSLg）；物理 GPU 在 Windows 侧：**NVIDIA RTX 3060 Laptop (6GB)** + Intel 核显 |
| 磁盘 | `/` 1007G（已用 14G，2%）；Windows 侧：C: 201G（120G 已用）、D: 275G（87G 已用）、F: 932G（410G 已用） |
| systemd | **运行中**（WSL2 已启用） |
| 开机横幅 | shell 启动跑 `fastfetch` |
| 软件包 | 657 个（显式 65 个；AUR：`yay`、`wsl-open`） |
| 系统代理 | Clash：`http://127.0.0.1:7892`（zsh 导出 + git 全局）；Arch 镜像可直连 |

## 二、软件包

- **镜像**：`fastly.mirror.pkgbuild.com` / `geo.mirror.pkgbuild.com`（官方镜像，直连可用，无需代理）。
- **AUR 助手**：`yay` 13.0.1（`yay -S <包>` 装 AUR 包；AUR 走代理时注意 `no_proxy`，TLS 断连时可临时 `ALL_PROXY=` 直连）。
- **显式安装的类别**：系统基础（base、base-devel、sudo、openssh、zsh、nano、tmux、tree、bind、wsl-open、watchexec）、终端美化（fastfetch、eza、lsd、bat、fzf、git-delta、chafa、scrot、tealdeer）、字体（otf-comicshanns-nerd、noto-fonts 系列）、压缩网络（7zip、unzip、zip、wget）、包管理（pacman-contrib、yay、jq、python-yaml）、GUI（code、gtk4）、文档排版（pandoc-cli、poppler、texlive-*、dvisvgm、ghostscript）、文件磁盘（fd、btop、ncdu、duf、rsync）、编程（gcc、gdb、clang、lldb、cmake、meson、ninja、make、git、github-cli、mingw-w64-*）。
- **2026-08-28 移除的孤儿包**：`go`、`nodejs`（系统版，连同依赖 simdjson、ada，共 297M）——详见「维护记录」。

## 三、开发工具链（版本）

| 工具 | 版本/路径 | 说明 |
|------|-----------|------|
| gcc / g++ | GCC 16.2.1 | 本机编译（`/usr/sbin/gcc`） |
| clang / lldb | LLVM 22.1.8 | 编译/调试 |
| gdb / gcov | 随 gcc | 调试/覆盖率 |
| make / cmake / meson / ninja | 4.4.1 / 4.4.2 / 1.12.0 / 1.13.2 | 构建 |
| python3 | Python 3.14.7 | 脚本（gwtui 用它） |
| node / npm / pnpm | **nvm 管理：v26.7.0**（`~/.nvm/versions/node/v26.7.0`）/ pnpm 11.22.0 | 唯一 node 来源（系统版已移除） |
| git / gh | git 2.55.0 / gh CLI 2.98.0 | 版本控制 / GitHub 自动化 |
| jq / pkgconf / uv | 1.8.2 / 3.0.6 / 0.12.7 | 工具 |
| gtk4 | pacman 版 | Calculator GUI 的 Windows 包用 MSYS2 编 |

> **惯例**：`main.c` + `Makefile`/`CMakeLists.txt`；本地 `make`/`cmake` 构建；`gwtui` 实时监控；`git push` 后 GitHub Actions（CI / release-please / release.yml）自动测试+发版；提交用 **GPG 签名**。
> **go 已移除**：无 Go 项目使用，如需要 `sudo pacman -S go` 即装回。

## 四、交叉编译（Windows 产物）

**Windows 包既要 Arch 侧 mingw 也要 Windows 侧 MSYS2：**

- **Arch 自带 mingw-w64**：`x86_64-w64-mingw32-gcc` / `-g++`（yay 装 `mingw-w64-gcc`）。
  ```bash
  x86_64-w64-mingw32-gcc main.c -o Detcalc.exe -lm    # 简单 C → Windows 可执行
  ```
- **Windows 侧 MSYS2 在 `/mnt/d/MSYS2/`**（`mingw64/bin` 内有 `x86_64-w64-mingw32-gcc.exe`、`gcc.exe`）。**GTK4 的 Windows 交叉编译必须用它**（Calculator GUI 依赖 GTK4；MSYS2 的 mingw64 有完整 GTK4 运行时，Arch 的没有）：
  ```bash
  /mnt/d/MSYS2/usr/bin/bash.exe -lc 'MSYSTEM=MINGW64; export PATH=/mingw64/bin:$PATH; cd ... && x86_64-w64-mingw32-gcc ...'
  ```
- 已跑通：`Detcalc`（CLI，Arch mingw64 可编）、`Calculator`（GTK4 GUI，用 MSYS2 编 Windows 包）。

## 五、Shell 配置（~/.zshrc 要点）

- **主题/插件**：powerlevel10k；插件 git、web-search、jsontools、z、zsh-autosuggestions、zsh-syntax-highlighting。
- **别名**：`ls`/`ll`/`la`/`tree`/`t`/`t3` → lsd 图标版。
- **`dsh` / `dsh-stop`**：`dsh` = `cd ~/deepseek-harness && pnpm dsh web`（启动 DeepSeek Harness Web GUI）；`dsh-stop` = `pkill -f "dsh web"`。
- **函数**：
  - `_code_repo <名>`：在 `$_croot=/home/Teletubbix/Code` 下 `find` 定位项目路径。
  - `gwtui <项目名>`：找到项目 → `python3 "$_croot/gwatch-tui.py" "$repo"`。
  - `explorer`：用 Windows 资源管理器打开当前目录（`explorer.exe` + `wslpath -w`）。
- **代理**：`export http_proxy/https_proxy/HTTP_PROXY/HTTPS_PROXY=http://127.0.0.1:7892`（Clash）。
- **其他**：`BROWSER=wsl-open`、nvm 加载、`LANG=zh_CN.UTF-8`、`PATH` 含 `~/.local/bin`。

## 六、git 与 GitHub

**全局配置（`git config --global --list`）：**

| 项 | 值 |
|----|----|
| user | `Teletubbix` / `3461530375@qq.com` |
| GPG 提交签名 | `commit.gpgsign=true`、`tag.gpgsign=true`、`user.signingkey=D1678C402A5DA75A`（rsa4096，2026-08-24 创建） |
| 代理 | `http.proxy` / `https.proxy` = `http://127.0.0.1:7892` |
| credential | `credential.helper=!gh auth git-credential`（HTTPS 自动借 gh 登录态，不再输密码） |
| 其他 | `core.pager=delta`（delta 分页器）、`url.https://github.com/.insteadof=git@github.com:` |

**GitHub 认证**：`gh auth status` ✓（账号 Teletubbix，https 协议，token 权限含 repo/workflow/admin:gpg_key 等）。**任何 HTTPS 操作 GitHub 都靠 gh 的登录态。**

**自动发版机制**（以 `C/Detcalc`、`C/Calculator` 为例，同构）：
- `.github/workflows/ci.yml`：push/PR 触发，构建+测试。
- `.github/workflows/release-please.yml`：`google-github-actions/release-please-action@v4`，`token: secrets.BOT_TOKEN || secrets.GITHUB_TOKEN` + config-file/manifest-file → 自动打 tag/生成 Release。
- `.github/workflows/release.yml`：tag 后构建并把 Windows 产物上传 Release。
- **`BOT_TOKEN`**：来自 `~/.dsh-token`（常驻 fine-grained PAT）。新仓库只需建好后在 Settings→Secrets 配 `BOT_TOKEN` 即可自动发版。

## 七、工作区与监控

- **唯一工作区**：`/home/Teletubbix/Code/{C,Cpp,JavaScript,Python}`（其他目录一般不放工作文件）。
- **工作区**：`/home/Teletubbix/Code/{C,Cpp,JavaScript,Python}`（唯一工作区）。
- **博客/知识库**：`JavaScript/blog`（Quartz v5）→ [teletubbix.com](https://teletubbix.com)（已挂自定义域名），内容在 `content/{notes,papers,bili,projects}`；**托管 = Cloudflare Pages**（绑 GitHub `teletubbix.github.io` 仓库，push 自动构建部署，`NODE_VERSION=22`，构建命令 `npm ci && npx quartz plugin install && npx quartz build`），域名 DNS 在 Cloudflare（NS: magnolia/memphis.ns.cloudflare.com）。GitHub Actions 的 deploy workflow 已停用（避免重复构建）。
- **项目**：
  | 项目 | 路径 | 版本 |
  |------|------|------|
  | Calculator | `C/Calculator`（GTK4 GUI，AGPL） | 6.0.7 |
  | Detcalc | `C/Detcalc`（C CLI，AGPL） | 0.2.0 |
- **监控工具 `gwtui`**：`/home/Teletubbix/Code/gwatch-tui.py`（本目录非 git 仓库）。两栏 TUI：左栏=文件/Git 状态（增绿/删红/改橙/新品红 + 文件图标），右栏=代码 diff（-U8，去元数据），GitHub 状态同流；文件变更 ≈1s 真监听，GitHub 每 15s 保底重绘；`q`/`Esc` 退出。用法：`gwtui <项目名>`。
- **`~/.dsh-token`**：常驻 PAT（BOT_TOKEN 来源，跨仓库通用）。

## 八、systemd 服务（运行中）

dbus-broker、systemd-homed、systemd-journald、systemd-logind、systemd-networkd、systemd-nsresourced、systemd-resolved、systemd-timedated、systemd-udevd、systemd-userdbd、user@1000。

## 九、Windows 侧资源

- 挂载：`/mnt/c`（C:）、`/mnt/d`（D:）、`/mnt/f`（F:）。
- **MSYS2**：`/mnt/d/MSYS2/`（Windows 交叉编译用，见第四节）。
- VS Code：WSL 远程开发（`~/.vscode-server`），Windows 侧 VS Code 连接。
- 终端：Windows Terminal；GUI 程序走 WSLg（Wayland）。

## 十、维护记录（2026-08-28）

**本次清理（旧文档重写前）：**
- ❌ 删除旧文档 `SYSTEM_HEALTH.md`、`AI_HANDOFF.md`、`system_info.png`（由本文件替代）。
- ❌ 删除 `~/msys2-installer.exe`（82M，MSYS2 已装于 /mnt/d/MSYS2）。
- ❌ 删除 `~/.wine`（776M，空 Wine 前缀，无程序）。
- ❌ 移除孤儿包 `go`(226M)、`nodejs`(62M) + 依赖 simdjson、ada，共 **297M**；当前孤儿包为 0。
- ❌ 删除 `Code/__pycache__`、`C/csapp-openbook-exp/src/__pycache__`、`C/Calculator/build`（可再生成）。
- ❌ `pnpm store prune`：释放 **326M**（45 个未引用包）。
- ❌ `journalctl --vacuum-size=50M`：释放 **136M**。
- ❌ `paccache -rk1`：缓存保持每包 1 个版本（此前体检已清理过，本次仅 12M）。

**系统升级：** `pacman -Syu` 升级 6 个包：aom 3.15.0、ca-certificates-mozilla 3.128、harfbuzz 14.4.0、nss 3.128、pkgconf 3.0.6、uv 0.12.7（全部成功，证书库已重建）。AUR 无待更新。

**关于 nodejs 26.8.1 的说明**：Arch 官方仓库的 `nodejs` 包跟踪 Node.js **Current 主线**，26.8.1 是 26.x 系列的补丁版本，版本号正常（并非异常）。nvm 里是 26.7.0（安装时的最新版，nvm 不会自动升级）。系统版已移除，**今后 node 版本统一走 nvm**，如需同步可 `nvm install 26.8.1`。

**待办/建议**：
- `nvm install 26.8.1` 同步 nvm 到最新（**注意**：nodejs.org 直连和代理均不通，需先在 Clash 规则中让 nodejs.org 走代理）。
- 定期：`paccache -rk1`、`journalctl --vacuum-size=50M`、`pnpm store prune`。
- 下次体检可做：`pacman -Qtdq` 查孤儿、`yay -Qua` 查 AUR 更新。

## 十一、大检查记录（2026-08-28 第二轮）

**结果：** 磁盘（/ 用 14G/2%）、服务（无失败 unit）、端口（仅 DNS + 127.0.0.1:3080 Harness）、包完整性（MD5 抽查通过）、代理（Clash 7892 正常）、工具引用（zshrc 全部存在）均正常。

**本次变更：**
- ❌ **删除 CSAPP 项目**：`C/CSAPP-cheatsheet`、`C/csapp-openbook-exp`（用户确认，两个目录均无 git 版本控制）。
- ⬆️ **MSYS2 升级**（Windows 侧 `/mnt/d/MSYS2`）：7 个包更新。

**已知噪音（无需处理）：**
- `WSL CheckConnection: getaddrinfo() failed: -5` 刷 journal/dmesg（约每 20s 两条）：Mirrored 网络模式下 WSL 周期性连通性探测失败的已知噪音，实际 DNS/网络正常。
- `pacman -Qkk` 报部分包"修改时间不匹配"：系统镜像导入时文件 mtime 被归一化为 2026-08-01 08:00:00，MD5 内容校验通过，属良性；升级过的包已恢复正常 mtime。
- nodejs.org 经代理也不通（exit 35）：Clash 规则疑似将其设为直连，影响 nvm 联网安装新版本。

**建议（未执行）：**
- `.wslconfig` 仅配了 `networkingMode=Mirrored`，未限内存；如需可加 `memory=`/`swap=` 限制。
- `~/.vscode-server/data/agent-host/sdk-cache`（306M，Claude SDK 缓存）与 `~/.local/share/fonts`（671M，Sarasa 字体全集）可按需精简；均为功能资源，保留。
- Windows C: 盘已用 120G/201G（60%），留意空间。

## 十二、Windows C 盘清理记录（2026-08-28 第三轮）

**背景**：C 盘（201G，曾用 120G/60%）。用户确认后执行清理，全程只动确认项。

**已删除（合计约 47G+）：**
- ❌ `C:\Users\34615\CrossDevice\Xiaomi 14`（**41.9G**，小米手机互联镜像：Galgame 32G、Download 4.7G、Aokana 3.3G、照片等）——已卸载微软手机连接（Microsoft.YourPhone）+ 跨设备应用（MicrosoftWindows.CrossDevice），停止 CrossDeviceResume/CrossDeviceService 进程后删除。
- ❌ `AppData\Local\Temp\Highlights`（1.99G，战争雷霆死亡回放录像）
- ❌ `Temp\XiaomiClipboard`（0.69G）、`Temp\vscode-stable-user-x64`（0.22G）
- ❌ `AppData\Local\Microsoft\Office\SolutionPackages`（652M，Office 功能包缓存）
- ❌ Edge 缓存（Cache/Code Cache/GPUCache ≈ 530M，浏览器本体保留）
- ❌ `AppData\Local\npm-cache`（0.29G，npm 包缓存，删了自动重建）
- ❌ `AppData\Local\QuarkCloudDrive`（0.43G，夸克缓存；夸克本体在 F:\QuarkCloudDrive 不受影响，需重新登录）
- ❌ `C:\OneDriveTemp`、`C:\Users\34615\OneDrive` 空壳、`ansel`/`.PowerTree` 空目录、`-1.18-windows.xml` 散文件

**已卸载（管理员脚本，2026-08-28）：**
- 微软 OneDrive（客户端早已不在，仅清残余）
- 微软手机连接 `Microsoft.YourPhone` + `MicrosoftWindows.CrossDevice`（Appx 卸载）
- **休眠已关闭**（`powercfg /h off`，释放 hiberfil.sys 13.7G）
- **pagefile 已移至 D 盘**（注册表 `PagingFiles = D:\pagefile.sys 0 0`，**重启后生效**，C 盘再释放 ~17.2G）
- **DISM 组件清理**（WinSxS 旧版组件，预计回收 2-5G）
- **卸载 Tobii 眼动仪**（160M）与 **X-Rite 校色助手**（145M）
- 保留：MuMu 模拟器（本体 F 盘）、联想电脑管家（3.26G）、MATLAB、夸克网盘（F 盘）

**小米互联存储迁移**：小米互联服务保留，存储位置改为 `D:\存储\下载\小米互联服务\`（已整理：删重复 zip）。C 盘 CrossDevice 不再同步。

**⚠️ 待办**：重启 Windows 后确认 pagefile 生效（`C:\pagefile.sys` 消失、`D:\pagefile.sys` 出现），并复查 C 盘空间。

## 十三、第二轮核查 + D 盘清理记录（2026-08-28）

**核查结论：**
1. **CrossDevice 42G 来源**：8月11日（装系统当天）小米互联连接手机时镜像的手机存储（文件时间戳 2024-2026 为证），与"最近同步"无关；断开连接后文件留存本地。
2. **删除量 vs 释放量不符**：声称删除 ~60G（CrossDevice 42G + 其他 5G + 休眠 13.7G），磁盘实际释放 ~19G，**~40G 差额未确认**——最大嫌疑为系统还原/卷影副本（SVI）在删除大目录时保留快照，需管理员 vssadmin 确认（待办）。
3. **小工具盘点**：Watt Toolkit（Steam++，改 GitHub DNS 加速）= `D:\Watt Toolkit` 336M；Geek Uninstaller（卸载 Office 365 用）= `D:\Geek Uninstaller` 7M；Win11 激活工具未找到（系统已激活 Licensed，推测用完已删）；Visio 2024 部署包 = `D:\Visio 2024`。
4. **`-1.18-windows.xml`**：MuMu 模拟器 VirtualBox 配置，每次启动重建，属 MuMu 运行文件，不可删。
5. **WSL 虚拟磁盘位置**：`D:\WSL\Arch`（34.3G，内部只用 14G，~20G 空洞待收缩）。

**D 盘清理（用户确认，释放约 21G，D 盘现 66G/24%）：**
- ❌ D 盘回收站 1.7G
- ❌ `D:\msys2-installer.exe`（84.9M）+ `D:\rustup-init.exe`（12.8M）
- ❌ `D:\下载\驱动`（2.27G，20 个联想原厂驱动包，驱动已装好）
- ❌ 4 个已解压的字体 zip（~490M；CascadiaCode/Sarasa 等无目录 zip 保留）
- ❌ `D:\Visio 2024` 的 ISO（2.87G，Visio 已装好；部署脚本/配置保留）
- ❌ `D:\Code` 旧测试项目（Hello-World、gtk4-hello、build）

**待办：** ① 用户确认 UAC 后跑 vssadmin 查卷影，确认 C 盘 40G 差额；② WSL 磁盘收缩（`wsl --shutdown` + Optimize-VHD，重启后执行，预计回收 ~20G）。

---

*本文件由 DeepSeek Harness 于 2026-08-28 重写生成。*
