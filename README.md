# CDNBestIP

一个基于 [**CloudflareSpeedTest**](https://github.com/XIU2/CloudflareSpeedTest) 获取最佳 CDN IP 的工具，用于自动测速并更新最佳的 CDN IP 到 Cloudflare DNS 记录。

## 功能特点

- 🚀 **自动化测速**：基于 CloudflareSpeedTest 进行 CDN IP 速度测试
- 🌐 **DNS 管理**：自动更新 CloudFlare DNS 记录到最佳 IP
- 📊 **多源支持**：支持 CloudFlare、GCore、CloudFront、AWS 等 IP 数据源
- 🎯 **智能配置**：根据 IP 源自动配置相应的测试端点
- 🔧 **灵活参数**：完整的命令行界面与环境变量支持
- 🐳 **容器化**：Docker 支持，便于部署和定时任务
- 📝 **详细日志**：多级别日志记录，便于调试和监控
- 🔒 **安全认证**：支持 API 令牌和 API 密钥两种认证方式
- ⚡ **高性能**：支持并发测试和结果缓存
- 🌍 **跨平台**：支持 Windows、Linux、macOS 等多平台

## 快速开始

### 安装

**Python 版本要求：** Python 3.13+

**PyPI 包地址：** [https://pypi.org/project/cdnbestip/](https://pypi.org/project/cdnbestip/)

```bash
# 使用 pip 安装
pip install cdnbestip
uv tool install cdnbestip

# 使用 pip + git 安装
pip install git+https://github.com/idev-sig/cdnbestip.git
# 或使用 uv 安装
uv tool install git+https://github.com/idev-sig/cdnbestip.git

# 指定版本
uv tool install git+https://github.com/idev-sig/cdnbestip.git@v0.1.0
```

### 基本用法

```bash
# 运行速度测试
cdnbestip -d example.com -p cf -s 2

# 测试并更新 DNS 记录
cdnbestip -t YOUR_API_TOKEN -d example.com -p cf -s 2 -n
```

详细使用说明请参阅 [使用指南](USAGE.md)。

## Docker 支持

### 镜像获取

#### 本地构建

```bash
docker buildx build \
  --platform linux/amd64,linux/arm64,linux/arm/v7,linux/arm/v6,linux/386 \
  -f docker/Dockerfile \
  -t idevsig/cdnbestip:local \
  --push .
```

镜像支持以下常用 Linux 架构，Docker 会自动选择当前主机对应的镜像：

`linux/amd64`、`linux/arm64`、`linux/arm/v7`、`linux/arm/v6`、`linux/386`

GitHub Actions 会在推送 `main`、`dev*` 分支或版本 Tag 时自动构建多架构 manifest，也支持手动运行和定时运行。GHCR 使用 GitHub Actions 自动提供的 `GITHUB_TOKEN`，不需要 Docker Hub、阿里云或腾讯云账号密钥。

#### 使用预构建镜像

> **版本标签：** `latest`, `main`, `<TAG>`

| Registry | Image |
| --- | --- |
| [**GitHub Container Registry**](https://ghcr.io/) | `ghcr.io/<GitHub用户名>/cdnbestip` |

```bash
# 拉取镜像；将 OWNER 替换为 GitHub 仓库所有者
docker pull ghcr.io/OWNER/cdnbestip:latest
```

## 使用

### Python 使用方式

**先决条件：**
- Python 3.13 以上
- [CloudflareSpeedTest](https://github.com/XIU2/CloudflareSpeedTest) v2.3.4 以上

1. 安装（见[**上一节**](#安装)）

2. 使用
```bash
# 基本用法（使用邮箱）
cdnbestip -a user@example.com -k api_key -d example.com -p cf -s 5 -n -o

# 使用 GCore IP 源（自动使用 GCore 测试端点）
cdnbestip -a user@example.com -k api_key -d example.com -p gc -s 5 -n -o -i gc

# 使用 GCore IP 源 + 自定义测试 URL
cdnbestip -a user@example.com -k api_key -d example.com -p gc -s 5 -n -o -i gc -u https://hk2-speedtest.tools.gcore.com/speedtest-backend/garbage.php?ckSize=100
```

### Docker 使用方式

#### 单次运行

```bash
# 基本用法（使用邮箱）
docker run --rm ghcr.io/OWNER/cdnbestip:latest cdnbestip -a user@example.com -k api_key -d example.com -p cf -s 5 -n -o

# 使用 GCore IP 源（自动使用 GCore 测试端点）
docker run --rm ghcr.io/OWNER/cdnbestip:latest cdnbestip -a user@example.com -k api_key -d example.com -p gc -s 5 -n -o -i gc

# 使用 GCore IP 源 + 自定义测试 URL
docker run --rm ghcr.io/OWNER/cdnbestip:latest cdnbestip -a user@example.com -k api_key -d example.com -p gc -s 5 -n -o -i gc -u https://hk2-speedtest.tools.gcore.com/speedtest-backend/garbage.php?ckSize=100
```

#### 使用 Docker Compose

1. 创建 `docker-compose.yml` 文件：

```yaml
services:
  cdnbestip:
    image: ghcr.io/OWNER/cdnbestip:latest
    container_name: cdnbestip
    restart: unless-stopped
    environment:
      - CLOUDFLARE_EMAIL=user@example.com
      - CLOUDFLARE_API_KEY=api_key
      - TZ=Asia/Shanghai
    command: ["daemon"]
```

2. 启动服务：

```bash
docker compose up -d
```

3. 配置定时任务：

```bash
# 添加定时计划 (每天凌晨4:15运行)
docker exec cdnbestip sh -c "echo '15 4 * * * cd /app; cdnbestip -d example.com -p cf -r -n -q 5' | crontab -"

# 启用定时服务
docker exec -d cdnbestip crond -b -l 8

# 管理定时服务
# 停止
docker exec cdnbestip pkill crond
# 重启
docker exec cdnbestip pkill -HUP crond
```

## 文档

- [使用指南](USAGE.md) - 完整的命令行参数和使用示例
- [部署指南](DEPLOYMENT.md) - 部署和分发信息

## 帮助

```
usage: cdnbestip [-h] [-a EMAIL] [-k API_KEY] [-t API_TOKEN] [-d DOMAIN] [-p PREFIX] [--type TYPE] [-s THRESHOLD] [-P PORT] [-u URL]
                 [-T SECONDS] [-q COUNT] [-S MINUTES] [-i SOURCE] [-r] [-n] [-o] [-c URL] [-e STRING] [--debug] [-v]
                 [--log-level {DEBUG,INFO,WARNING,ERROR,CRITICAL}] [--no-console-log] [--no-file-log] [--version]

CloudFlare DNS speed testing and management tool

options:
  -h, --help            show this help message and exit
  --version             show program's version number and exit

CloudFlare Credentials:
  -a, --account EMAIL              CloudFlare account email
  -k, --key API_KEY                CloudFlare API key
  -t, --token API_TOKEN            CloudFlare API token (alternative to key+email)

DNS Settings:
  -d, --domain DOMAIN   Domain name (required for DNS operations)
  -p, --prefix PREFIX   DNS record prefix (required for DNS operations)
  -y, --type TYPE       DNS record type (default: A)

Speed Test Settings:
  -s, --speed THRESHOLD
                        Download speed threshold in MB/s (default: 0.0, 0 means no speed filtering)
  -P, --port PORT       Speed test port (0-65535)
  -u, --url URL         Speed test URL
  -T, --timeout SECONDS
                        Speed test timeout in seconds (default: 600)
  -q, --quantity COUNT  Number of DNS records to create (default: 0 = unlimited; excess prefixed records are removed)
  -S, --schedule MINUTES
                        Repeat the complete workflow every N minutes (first run starts immediately)

IP Data Source:
  -i, --ip-url SOURCE    IP data source: cf, as13335, as209242, gc, ct, aws, all, or custom URL

Operations:
  -r, --refresh         Force refresh result.csv file
  -n, --dns             Update DNS records after speed test
  -o, --only            Only update one DNS record (fastest IP)

Advanced Options:
  -c, --cdn URL         CDN URL for file acceleration
  -e, --extend STRING   Extended parameters for CloudflareSpeedTest (use -e="-param" or -e "\\-param")
  -x, --proxy URL       Proxy URL for Cloudflare API and IP list downloads

Logging and Debugging:
  -D, --debug           Enable debug mode with detailed logging
  -v, --verbose         Enable verbose output
  -L, --log-level {DEBUG,INFO,WARNING,ERROR,CRITICAL}
                        Set logging level (default: INFO)
  -C, --no-console-log  Disable console logging
  -F, --no-file-log     Disable file logging

Examples:
  cdnbestip -a user@example.com -k api_key -d example.com -p cf -s 2 -n -o
  
  export CLOUDFLARE_API_KEY="api_key"
  export CLOUDFLARE_EMAIL="user@example.com"
  cdnbestip -d example.com -p cf -s 2 -n -o

IP Data Sources:
  cf   - CloudFlare IPs
  as13335 - Cloudflare AS13335 IPv4 prefixes
  as209242 - Cloudflare AS209242 IPv4 prefixes
  gc   - GCore IPs  
  ct   - CloudFront IPs
  aws  - Amazon AWS IPs
  all  - Merge all predefined IPv4 sources, remove duplicates, and auto-test sources with built-in endpoints
  <url> - Custom IP data URL

Zone Types:
  A, AAAA, CNAME, MX, TXT, SRV, NS, PTR
```

### 参数说明

**CloudFlare 认证：**
> `-a` / `--account`:          CloudFlare 账号邮箱
> `-k` / `--key`:              CloudFlare API 密钥   
> `-t` / `--token`:            CloudFlare API 令牌（推荐，可替代 key+email）   

**DNS 设置：**
> `-d` / `--domain`:           域名（DNS 操作必需）   
> `-p` / `--prefix`:           DNS 记录前缀（DNS 操作必需）   
> `-y` / `--type`:             DNS 记录类型（默认：A）   

**速度测试设置：**
> `-s` / `--speed`:            下载速度阈值，单位 MB/s（默认：0.0，0表示不进行速度过滤，仅使用延迟过滤）     
> `-P` / `--port`:             速度测试端口（0-65535）   
> `-u` / `--url`:              速度测试 URL   
> `-T` / `--timeout`:          速度测试超时时间，单位秒（默认：600）   
> `-q` / `--quantity`:         创建的 DNS 记录数量（默认：0 = 无限制）   
> `-S` / `--schedule`:         内置定时执行间隔，单位分钟（首次立即执行）   

> `-s` 只设置下载速度阈值，不会自动附加延迟限制。需要限制延迟或覆盖下载测试数量时，可使用 `-e "-tl 400 -dn 5"`。

**IP 数据源：**
> `-i` / `--ip-url`:            IP 数据源：cf, as13335, as209242, gc, ct, aws, all 或自定义 URL

**操作选项：**
> `-r` / `--refresh`:          强制刷新 result.csv 文件    
> `-n` / `--dns`:              测试后更新 DNS 记录   
> `-o` / `--only`:             仅更新一条 DNS 记录（最快的 IP）   

**高级选项：**
> `-c` / `--cdn`:              文件加速的 CDN URL     
> `-e` / `--extend`:           CloudflareSpeedTest 的扩展参数 (使用 -e="-参数" 或 -e "\\-参数")   
> `-x` / `--proxy`:            代理服务器 URL，用于 Cloudflare API 和 IP 列表下载   

**日志和调试：**
> `-D` / `--debug`:            启用调试模式和详细日志   
> `-v` / `--verbose`:          启用详细输出   
> `-L` / `--log-level`:        设置日志级别   
> `-C` / `--no-console-log`:   禁用控制台日志   
> `-F` / `--no-file-log`:      禁用文件日志   

### 认证方式

- **方式一（推荐）**：使用 API 令牌 `-t` 或设置环境变量 `CLOUDFLARE_API_TOKEN`
- **方式二**：使用 API 密钥 + 邮箱 `-k` + `-a` 或设置环境变量 `CLOUDFLARE_API_KEY` + `CLOUDFLARE_EMAIL`

**账号参数支持格式：**
- 邮箱格式：`user@example.com`
- 账号ID格式：`b9b779dc8c2e097c2a467261a8fa0000`（32位十六进制字符串）

获取 API 令牌：[CloudFlare Dashboard](https://dash.cloudflare.com/profile/api-tokens) -> `API Tokens` -> `Create Token`   
获取 API 密钥：[CloudFlare Dashboard](https://dash.cloudflare.com/profile/api-tokens) -> `API Keys` -> `Global API Key`   

> 使用 API 令牌更新 DNS 时，需要对目标域名授予 `Zone Read` 和 `DNS Write` 权限。程序会按 `-d/--domain` 查询该域名，不再调用需要额外用户权限的 `/user` 接口。

### 代理配置

工具支持通过代理服务器进行 Cloudflare API 调用和 IP 列表下载。支持的代理类型：

- **HTTP 代理**：`http://proxy.example.com:8080`
- **HTTPS 代理**：`https://proxy.example.com:8080`

**使用方式：**

```bash
# 命令行参数（长参数）
cdnbestip --proxy http://proxy.example.com:8080 -d example.com -p cf -s 2 -n

# 命令行参数（短参数）
cdnbestip -x http://proxy.example.com:8080 -d example.com -p cf -s 2 -n

# 环境变量
export CDNBESTIP_PROXY="http://proxy.example.com:8080"
cdnbestip -d example.com -p cf -s 2 -n
```

**注意：** 代理仅用于 Cloudflare API 调用和 IP 列表下载，不影响 CloudflareSpeedTest 工具的测速过程。

## IP 数据源和测试端点

### 自动配置（推荐）

工具会根据选择的 IP 数据源自动配置相应的测试端点：

| IP 源 | 提供商 | 自动测试端点 | 需要 `-u` 参数？ |
|-------|--------|-------------|-----------------|
| `cf` | CloudFlare | `https://cf.xiu2.xyz/url` | 否 |
| `as13335` | Cloudflare AS13335 IPv4 宣告网段 | `https://cf.xiu2.xyz/url` | 否 |
| `as209242` | Cloudflare AS209242 IPv4 宣告网段 | `https://cf.xiu2.xyz/url` | 否 |
| `gc` | GCore | `https://hk2-speedtest.tools.gcore.com/speedtest-backend/garbage.php?ckSize=100` | 否 |
| `ct` | CloudFront | `https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip` | 否 |
| `aws` | Amazon AWS | 无 | **是** |
| `all` | 全部预定义 IPv4 源（自动去重） | 按数据源自动选择 | 否（AWS 全量源无安全默认地址） |
| 自定义 URL | 自定义 | 无 | **是** |

### 使用示例

```bash
# 无 IP 源 - 使用 CloudflareSpeedTest 默认设置
cdnbestip -d example.com -p cf -s 2 -n

# CloudFlare IP 源 - 自动使用 CF 测试端点
cdnbestip -i cf -d example.com -p cf -s 2 -n

# AS13335 IPv4 宣告网段
cdnbestip -i as13335 -d example.com -p cf -s 2 -n

# AS209242 IPv4 宣告网段
cdnbestip -i as209242 -d example.com -p cf -s 2 -n

# 合并全部 IPv4 数据源并去重（自动分组测速，无需 -u）
cdnbestip -i all -d example.com -p cf -s 2 -n

# 使用统一 URL 测试全部已合并 IP（可选；CloudFront/AWS 需要适配它们的 URL）
cdnbestip -i all -u https://example.com/test -d example.com -p cf -s 2 -n

# GCore IP 源 - 自动使用 GCore 测试端点
cdnbestip -i gc -d example.com -p gc -s 2 -n

# CloudFront IP 源 - 自动使用 AWS 官方 CloudFront 下载对象
cdnbestip -i ct -d example.com -p ct -s 2 -n

# 自定义测试 URL（覆盖默认设置）
cdnbestip -i gc -u https://custom-test.example.com/test -d example.com -p gc -s 2 -n

# 使用扩展参数传递给 CloudflareSpeedTest
cdnbestip -d example.com -p cf -e="-cfcolo HKG" -s 2 -n
cdnbestip -d example.com -p cf -e "\-cfcolo HKG -a 1" -s 2 -n

# 使用代理服务器
cdnbestip -d example.com -p cf --proxy http://proxy.example.com:8080 -s 2 -n
```

### 内置定时执行

使用 `-S` / `--schedule` 设置间隔分钟数。程序会立即执行一次，完成后按间隔重复执行；按 `Ctrl+C` 停止。
定时模式下，如果某次测速、IP 源下载或二进制检查临时失败，程序会记录错误并保留调度器，下一周期自动重试。

```bash
# 每 6 小时自动测速并更新 DNS
cdnbestip -t YOUR_API_TOKEN -d example.com -p cf -s 2 -n -o --schedule 360

# 也可以使用环境变量
export CDNBESTIP_SCHEDULE=360
cdnbestip -t YOUR_API_TOKEN -d example.com -p cf -s 2 -n -o
```

### 手动指定测试 URL（`-u` 或 `--url` 参数）

#### [CloudFlare](https://www.cloudflare.com/)

```bash
# CloudflareSpeedTest 兼容的 CloudFlare 测试端点
https://cf.xiu2.xyz/url
```

> `speed.cloudflare.com/__down` 是 Cloudflare 官方测速接口，但通过候选 IP 直接请求时可能返回 HTTP 403，不建议作为 CloudflareSpeedTest 的 `-u` 地址。

#### [GCore](https://gcore.com/)

**香港：**
```bash
https://hk2-speedtest.tools.gcore.com/speedtest-backend/garbage.php?ckSize=100
```

**日本：**
```bash
https://cc1-speedtest.tools.gcore.com/speedtest-backend/garbage.php?ckSize=100
```

**新加坡：**
```bash
https://sg1-speedtest.tools.gcore.com/speedtest-backend/garbage.php?ckSize=100
```

> `ckSize` 为文件大小，单位 `MB`。可自行修改，最大为 100MB。

#### [CacheFly](https://www.cachefly.com/)

```bash
https://cachefly.cachefly.net/100mb.test
```

#### [AWS CloudFront](https://aws.amazon.com/cloudfront/)

```bash
# AWS 官方大文件下载对象，项目对 CloudFront 源默认使用此地址
https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip
```

> 该地址只适合测试 CloudFront IP 源。`aws` 数据源包含多个 AWS 服务的网段，不能用单个 CloudFront 对象代表全部 AWS IP，因此仍需通过 `-u` 指定与你的目标服务匹配的地址。

## 许可证

本项目采用 Apache License 2.0 许可证。

## 仓库镜像

* [https://git.jetsung.com/idev/cdnbestip](https://git.jetsung.com/idev/cdnbestip)
* [https://framagit.org/idev/cdnbestip](https://framagit.org/idev/cdnbestip)
* [https://gitcode.com/idev/cdnbestip](https://gitcode.com/idev/cdnbestip)
* [https://github.com/idev-sig/cdnbestip](https://github.com/idev-sig/cdnbestip)
