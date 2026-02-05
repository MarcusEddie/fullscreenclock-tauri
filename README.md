# Fullscreen Clock - Tauri Application

一个基于 Tauri 2 构建的全屏时钟应用。

## 环境要求

- Ubuntu (已在 Ubuntu 22.04/24.04 上测试)
- Node.js (可选，用于前端开发)

## 快速开始

### 1. 克隆代码

```bash
git clone <your-repository-url>
cd fullscreenclock-tauri
```

### 2. 安装系统依赖

```bash
sudo apt update
sudo apt install libwebkit2gtk-4.1-dev build-essential curl wget file libssl-dev libayatana-appindicator3-dev librsvg2-dev imagemagick
```

### 3. 安装 Rust

```bash
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
```

安装过程中选择默认选项（输入 `1`）。安装完成后加载环境：

```bash
source $HOME/.cargo/env
```

验证安装：

```bash
rustc --version
cargo --version
```

### 4. 安装 Tauri CLI

```bash
cargo install tauri-cli
```

首次安装会编译，需要几分钟。

## 图标生成

Tauri 打包时需要 RGBA 格式的 PNG 图标。如果遇到以下错误：

```
help: message: icon icons/32x32.png is not RGBA
```

请使用以下命令从原始图片生成符合要求的图标：

### 从原始图片生成各尺寸图标

假设你的原始图片为 `original-icon.png`：

```bash
cd src-tauri/icons

# 生成 32x32 RGBA 格式图标
convert original-icon.png -resize 32x32 -background transparent -alpha set PNG32:32x32.png

# 生成 128x128 RGBA 格式图标
convert original-icon.png -resize 128x128 -background transparent -alpha set PNG32:128x128.png

# 生成 128x128@2x RGBA 格式图标 (256x256)
convert original-icon.png -resize 256x256 -background transparent -alpha set PNG32:128x128@2x.png

# 生成默认图标 (256x256)
convert original-icon.png -resize 256x256 -background transparent -alpha set PNG32:icon.png
```

### 从零创建占位图标

如果没有原始图片，可以生成纯色占位图标：

```bash
cd src-tauri/icons

convert -size 32x32 xc:transparent -fill black -draw "rectangle 0,0 32,32" PNG32:32x32.png
convert -size 128x128 xc:transparent -fill black -draw "rectangle 0,0 128,128" PNG32:128x128.png
convert -size 256x256 xc:transparent -fill black -draw "rectangle 0,0 256,256" PNG32:128x128@2x.png
convert -size 256x256 xc:transparent -fill black -draw "rectangle 0,0 256,256" PNG32:icon.png
```

关键点：`PNG32:` 前缀强制输出为 32 位 RGBA 格式，解决 "is not RGBA" 错误。

## 本地开发测试

### 开发模式运行

```bash
cargo tauri dev
```

开发模式特点：

- 支持热重载，修改 HTML/CSS/JS 后刷新即可看到效果
- 编译速度快，适合调试
- 生成的是未优化的调试版本

### 构建发布版本

```bash
cargo tauri build
```

生成的文件位于 `src-tauri/target/release/bundle/` 目录下：

- `deb/` - Debian/Ubuntu 安装包
- `appimage/` - 便携版 AppImage 文件

### 只生成特定格式

```bash
# 只生成 AppImage（便携版）
cargo tauri build --bundles appimage

# 只生成 deb 安装包
cargo tauri build --bundles deb

# 只生成可执行文件，跳过打包
cargo tauri build --bundles none
```

## 项目结构

```
fullscreenclock-tauri/
├── src/
│   └── index.html          # 前端页面
├── src-tauri/
│   ├── icons/              # 应用图标
│   │   ├── 32x32.png
│   │   ├── 128x128.png
│   │   ├── 128x128@2x.png
│   │   └── icon.png
│   ├── src/
│   │   └── main.rs         # Rust 入口文件
│   ├── build.rs            # 构建脚本
│   ├── Cargo.toml          # Rust 依赖配置
│   └── tauri.conf.json     # Tauri 配置文件
└── README.md
```

## 常见问题

### Q: VMware 虚拟机中出现 3D 加速警告

```
VMware: No 3D enabled (0, Success).
libEGL warning: egl: failed to create dri2 screen
```

这些是虚拟机环境下的正常警告，不影响应用运行。

### Q: 应用窗口显示空白

检查 `index.html` 是否位于 `src/` 目录下（不是 `src-tauri/src/`），并确认 `tauri.conf.json` 中的 `frontendDist` 路径配置正确。

### Q: AppImage 文件太大 (70+ MB)

这是因为 AppImage 包含了完整的 WebKit 依赖。可选方案：

- 使用 `.deb` 格式，依赖系统库，体积更小（2-5 MB）
- 直接使用 `target/release/fullscreen-clock` 可执行文件
- 在 macOS/Windows 上构建，体积会小很多（5-15 MB）

## 跨平台构建

Tauri 不支持交叉编译。要构建其他平台的应用，需要在对应平台上编译：

- Windows `.exe` - 在 Windows 机器上运行 `cargo tauri build`
- macOS `.app` - 在 macOS 机器上运行 `cargo tauri build`
- Linux `.AppImage` / `.deb` - 在 Linux 机器上运行 `cargo tauri build`

建议使用 GitHub Actions 实现 CI/CD 自动化多平台构建。
