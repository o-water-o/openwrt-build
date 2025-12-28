# 本项目主要用于自定义编译软路由镜像，openwrt ImmortalWrt 等
- 可能有 openwrt ImmortalWrt 的不同版本

## 如何生成编译配置文件 (.config)

由于 `make menuconfig` 需要在 Linux 环境下运行，Windows 用户推荐使用 **WSL (Windows Subsystem for Linux)**。

### 1. 准备环境 (WSL Ubuntu)
如果你还没有安装 WSL，请在 PowerShell 中管理员模式运行 `wsl --install` 并重启电脑。
进入 Ubuntu 终端后，安装编译基础依赖：

```bash
sudo apt update
sudo apt install -y build-essential git python3
```

### 2. 获取源码并生成配置
在 WSL 终端中执行以下命令（确保与 GitHub Action 使用的分支一致）：

```bash
# 1. 克隆源码 (与 Workflow 分支一致)
git clone -b openwrt-23.05 https://github.com/immortalwrt/immortalwrt
cd immortalwrt

# 2. 手动添加 Feeds (包含 Passwall)
# 这一步很重要，必须添加和 Workflow 中一样的源，否则生成的配置文件会缺少插件
echo "src-git passwall https://github.com/xiaorouji/openwrt-passwall2.git;main" >> feeds.conf.default
echo "src-git passwall_packages https://github.com/xiaorouji/openwrt-passwall-packages.git;main" >> feeds.conf.default

# 3. 更新并安装 Feeds
./scripts/feeds update -a
./scripts/feeds install -a

# 4. 打开配置菜单
make menuconfig
```

### 3. 选择配置
在该界面中：
1.  **Target System**: 选择 `x86` -> `x86_64` (软路由通常是这个)。
2.  **LuCI -> Applications**: 找到并选中需要的插件，例如 `luci-app-passwall`。
3.  完成后选择 `<Save>`，保存文件名为 `.config`。
4.  选择 `<Exit>` 退出。

### 4. 导出并上传
将生成的 `.config` 文件从 WSL 复制到你 Windows 的项目目录中，**并重命名为 `.config.immortalwrt`**：

```bash
# 假设你的 Windows 项目在 D:\Projects\RouterLab\openwrt-build
# /mnt/d/ 代表 D 盘
cp .config /mnt/d/Projects/RouterLab/openwrt-build/.config.immortalwrt
```

最后，将配置文件提交并推送到 GitHub 仓库：
```bash
git add .config.immortalwrt
git commit -m "Update immortalwrt build config"
git push
```
这将自动触发 GitHub Action 开始编译。