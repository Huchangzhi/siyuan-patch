# 思源笔记自定义构建指南

本指南说明如何使用 GitHub Actions 打包已去除 VIP 限制的思源笔记应用和 Docker 镜像。

## 前置准备

### 1. Fork 仓库
首先 Fork 这个包含去 VIP 限制修改的仓库到你自己的 GitHub 账号。

### 2. 配置 Docker Hub Secrets（仅需要构建 Docker 时）
如果你要构建 Docker 镜像并推送到 Docker Hub，需要在你的仓库中配置以下 Secrets：

进入你的仓库 → **Settings** → **Secrets and variables** → **Actions** → **New repository secret**：

| Secret 名称 | 值说明 |
|------------|--------|
| `DOCKER_HUB_USERNAME` | 你的 Docker Hub 用户名 |
| `DOCKER_HUB_PASSWORD` | 你的 Docker Hub 访问令牌（推荐使用 Access Token，不是登录密码） |

**获取 Docker Hub Access Token：**
1. 登录 https://hub.docker.com
2. 点击头像 → **Account Settings**
3. 左侧菜单选择 **Security**
4. 点击 **New Access Token**
5. 填写描述，选择读写权限，生成后复制保存

### 3. 配置 GitHub Release 权限（可选）
如果要自动上传到 GitHub Release，确保工作流有写入权限（已在 YAML 中配置）。

## 使用方法

### 方式一：手动触发构建（推荐）

1. 进入你的仓库 → **Actions** 标签页
2. 选择左侧的 **Build SiYuan Custom** 工作流
3. 点击 **Run workflow** 按钮
4. 填写构建选项：
   - **构建桌面客户端**: 选择是否构建 Windows/macOS/Linux 版本
   - **构建 Docker 镜像**: 选择是否构建并推送 Docker 镜像
   - **Docker 镜像标签**: 自定义标签（留空则使用 package.json 中的版本号）
   - **上传到 GitHub Release**: 如果是 tag 触发，可选择上传
5. 点击 **Run workflow** 开始构建

### 方式二：通过 Tag 自动触发

推送匹配以下模式的标签会自动触发桌面客户端构建：
- `*-dev*` (开发版本)
- `v*` (正式版本)

```bash
git tag v20240101-dev
git push origin v20240101-dev
```

## 构建产物

### 桌面客户端
构建完成后，产物会作为 Artifacts 保留 7 天：
- `siyuan-linux.AppImage` - Linux AppImage 包
- `siyuan-linux.tar.gz` - Linux 压缩包
- `siyuan-mac.dmg` - macOS Intel 版本
- `siyuan-mac-arm64.dmg` - macOS Apple Silicon 版本
- `siyuan-win.exe` - Windows 安装包

如果选择了"上传到 GitHub Release"且是 tag 触发，会自动上传到对应的 Release。

### Docker 镜像
构建成功后会推送到你的 Docker Hub：
- 镜像名：`YOUR_DOCKER_HUB_USERNAME/siyuan-custom:latest`
- 版本镜像：`YOUR_DOCKER_HUB_USERNAME/siyuan-custom:v2.x.x`

支持多平台：linux/amd64, linux/arm64, linux/arm/v7

## 注意事项

1. **构建时间**: 完整构建（所有平台+Docker）约需 30-50 分钟
2. **GitHub Actions 限额**: 免费账户每月有 2000 分钟构建时长限制
3. **Docker 镜像大小**: 完整镜像约 300-400MB
4. **代码修改**: 所有构建都会包含你仓库中的代码修改（如去 VIP 限制）

## 本地测试构建

### 桌面客户端
```bash
# 进入 app 目录
cd app

# 安装依赖
pnpm install

# 构建 UI
pnpm run build

# 构建内核（以 Linux 为例）
cd ../kernel
go build -tags fts5 -o ../app/kernel-linux/SiYuan-Kernel -ldflags "-s -w -X github.com/your-repo/kernel/util.Mode=prod"

# 打包 Electron 应用
cd ../app
pnpm run dist-linux
```

### Docker 镜像
```bash
docker build -t siyuan-custom:latest .
docker run -p 6806:6806 siyuan-custom:latest
```

## 故障排查

### 构建失败常见原因
1. **子模块未初始化**: 确保执行 `git submodule update --init --recursive`
2. **依赖版本不匹配**: 检查 Node.js 和 Go 版本是否符合要求
3. **Docker Hub 认证失败**: 检查 Secrets 配置是否正确
4. **磁盘空间不足**: Ubuntu runner 默认空间有限，工作流已包含清理步骤

### 查看构建日志
在 Actions 页面点击对应的构建任务，查看详细日志输出。
