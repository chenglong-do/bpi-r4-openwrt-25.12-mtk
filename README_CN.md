# Banana Pi BPI-R4 OpenWrt 固件构建（内核 6.12）

本仓库包含用于构建 Banana Pi BPI-R4（MT7988，Wi-Fi 7）OpenWrt 25.12 固件的脚本和 GitHub Actions 工作流，使用 MediaTek SDK。

目标：无需本地构建环境，你可以在 GitHub 上触发构建并下载可直接使用的固件镜像。高级用户可以根据需要调整配置。

正常构建请使用 "latest" 工作流，仅在需要特定 commit/branch/tag 时使用 "commit" 工作流。

---

## 本分支的改动：
- CPU 超频至 2.2 GHz（添加了 2.0 GHz @ 950mV 和 2.2 GHz @ 1000mV 频率档位）
- 预装 LuCI 应用：Argon 主题、Lucky、Nikki、OpenClash、cpufreq、mwan3、sqm、statistics、ttyd 等（完整列表见 [my_final_defconfig](my_files/my_final_defconfig)）

[English README](README.md)

---

## Fork 的重要说明

如果你创建了本仓库的 **Fork**，GitHub Actions 不会自动运行。

要使构建工作流在你的 Fork 中正常工作，你必须：

1. 进入你 Fork 的仓库的 **Actions** 标签页，点击按钮为此仓库**启用工作流**。
2. 在你的 Fork 中，打开 **Settings → Actions → General** 并检查：
   - **Actions 权限**：允许 GitHub Actions 在此仓库中运行。
   - **工作流权限**：将 `GITHUB_TOKEN` 设置为 **读写权限**，否则工作流无法创建发布、标签或上传构建产物。

---

## 1. 快速开始 - 现成镜像（"latest" 工作流）

如果你只需要 BPI-R4 的预构建固件，不关心特定的 commit，请使用 "latest" 工作流。

它总是从仓库的当前状态（main 分支的最新 commit）构建。

步骤：

1. 打开本仓库的 "Actions" 标签页。
2. 选择工作流 "Build OpenWrt (BPI-R4, kernel 6.12, latest)"。
3. 点击 "Run workflow" 并确认。你无需更改任何内容。

工作流完成后：

1. 打开 "Releases" 标签页。
2. 下载最新发布（标签 bpi-r4-latest）。

发布中的重要文件：

- `openwrt-mediatek-filogic-bananapi_bpi-r4-squashfs-sysupgrade.itb` – 用于从现有 OpenWrt 升级。
- `openwrt-mediatek-filogic-bananapi_bpi-r4-sdcard.img.gz` – SD 卡镜像。
- `sha256sums` – 用于验证的校验和。

发布总是在同一个标签 bpi-r4-latest 下更新，所以 GitHub 上的 "Created" 日期不会改变。检查文件列表和 "Updated" 时间以查看最新构建。

此用例下你无需更改仓库中的任何内容。

---

## 2. 从特定 commit/branch 构建（"commit" 工作流）

如果你想从特定的 commit、分支或标签构建固件，请使用 "commit" 工作流。

这在需要复现旧版本构建或测试功能分支时很有用。

步骤：

1. 打开本仓库的 "Actions" 标签页。
2. 选择工作流 "Build OpenWrt (BPI-R4, 6.12, commit)"。
3. 点击 "Run workflow"。

在对话框中：

1. 在 "Git reference" 字段中输入你想构建的内容：
   - commit SHA（例如 1234abcd…），或
   - 分支名称（例如 main, feature-x），或
   - 标签名称（例如 v1.0）。

2. 确认以启动工作流。

工作流完成后：

1. 打开 "Releases" 标签页。
2. 下载最新发布（标签 bpi-r4-latest），与第 1 节相同。发布内容反映了你启动工作流时选择的 commit/branch/tag。

注意：如果你不确定该用什么，请坚持使用第 1 节的 "latest" 工作流。

---

## 3. 添加/删除软件包（仅 GitHub）

你可以通过在 GitHub 上直接编辑最终配置文件来稍微自定义固件。

构建仍然在 GitHub 的服务器上运行，而不是在你的机器上。

第 1 步 – 打开配置文件

1. 在本仓库中，打开 "my_files" 目录。
2. 点击文件 "my_final_defconfig"。

第 2 步 – 在 GitHub 上编辑文件

1. 点击铅笔图标（"Edit this file"）。
2. 在文件中你会看到类似这样的行：

   `CONFIG_PACKAGE_iperf3=y`  
   `CONFIG_PACKAGE_htop is not set`

   含义：

   - `CONFIG_PACKAGE_xyz=y` → 启用软件包。
   - `CONFIG_PACKAGE_xyz is not set` → 禁用软件包。

3. 要启用软件包，将：

   `CONFIG_PACKAGE_htop is not set`

   改为：

   `CONFIG_PACKAGE_htop=y`

4. 要禁用软件包，将：

   `CONFIG_PACKAGE_iperf3=y`

   改为：

   `CONFIG_PACKAGE_iperf3 is not set`

5. 重要：只更改以 "CONFIG_PACKAGE_" 开头的行。
   除非你确实了解，否则不要碰内核 / target / MTK SDK 选项。

6. 在页面底部点击 "Commit changes" 保存文件。

第 3 步 – 触发新构建

1. 进入 "Actions" 标签页。
2. 运行任一工作流：

   - "Build OpenWrt (BPI-R4, 6.12, latest)" – 从最新 commit 构建。
   - "Build OpenWrt (BPI-R4, 6.12, commit)" – 从特定 commit/branch/tag 构建。

关于依赖

构建脚本将 "my_final_defconfig" 复制到 ".config"，然后运行：

   `make defconfig`

OpenWrt 构建系统会自动填充缺失的选项和依赖。
如果你只切换几个 CONFIG_PACKAGE_… 选项，构建应该保持一致。

---

## 4. 本地构建（可选，高级）

如果你更喜欢在 Linux 上本地构建：

要求（示例：Ubuntu 22.04）

- 大约 120 GB 可用磁盘空间。
- 基本构建工具和库：
   ```
   sudo apt-get update  
   sudo apt-get install -y \
     build-essential clang flex bison g++ gawk gcc-multilib g++-multilib \
     gettext libncurses-dev libssl-dev python3-distutils python3-setuptools \
     rsync swig unzip zlib1g-dev file wget libelf-dev ccache git
   ```

克隆并构建
   ```
   git clone https://github.com/woziwrt/bpi-r4-6.12.git  
   cd bpi-r4-6.12  
   chmod +x ./bpi-r4-openwrt-builder.sh  
   ./bpi-r4-openwrt-builder.sh  
   ```

脚本完成后，镜像位于：

   `openwrt/bin/targets/mediatek/filogic/`

---

## 5. 仓库内容

- `bpi-r4-openwrt-builder.sh` – 主构建脚本（克隆 OpenWrt + MTK，准备，构建，应用配置）。
- `my_files/` – 自定义配置（my_final_defconfig）、补丁、自定义软件包和 LuCI 应用。
- `.github/workflows/` – GitHub Actions 工作流：
  - 释放磁盘空间，
  - 运行构建脚本，
  - 清理构建输出，
  - 上传构建产物，
  - 创建仅包含相关 BPI-R4 文件的发布，
  - 仅保留最新发布。

---

## 6. 说明

- 此构建仅适用于 Banana Pi BPI-R4。
- 对于简单的自定义构建，只更改 my_final_defconfig 中的 CONFIG_PACKAGE_… 行。
- OpenWrt 和 MTK SDK 的 commit 是固定的；更新它们需要手动操作。

### 关于 GitHub 运行器和镜像的说明

此工作流在 GitHub 托管的运行器上运行，可用磁盘空间不保证。
有时运行器只提供足够构建最小镜像的空间，但不足以构建包含许多额外软件的完整发布。

如果构建失败并显示 "no space left on device" 或类似的磁盘相关错误，通常意味着分配的运行器没有足够的可用磁盘空间。
在这种情况下，只需稍后重新运行工作流；当运行器恰好有大约 100 GB 的可用空间时，包含许多软件包的完整构建更有可能成功。

此外，构建系统从外部镜像下载源码和软件包源。
如果某些镜像暂时不可用或很慢，工作流也可能失败或需要更长时间完成。
稍后重新运行工作流通常可以解决此类暂时的镜像问题。
