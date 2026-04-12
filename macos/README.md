# sing-box-config-templates

[![Configuration](https://img.shields.io/badge/config-decoupled-blue)](https://github.com/Mice-Tailor-Infra/sing-box-config-templates)

本项目是个人 **Sing-box / Mihomo** 配置文件的上游模板仓库。

采用 **IaC (Infrastructure as Code)** 思想管理网络配置，通过分支策略适配不同硬件环境（Android/Linux/Windows），并实现了配置逻辑与敏感凭证的完全解耦。

## ✨ 核心架构亮点

- **全端脱敏架构 (Decoupled Secrets)**

  - 仓库内仅包含逻辑模板 (`config.template.json`)，不包含任何私人订阅链接或密钥。
  - 运行时通过 `envsubst` 动态注入环境变量生成最终配置，确保 Git 历史绝对清白。

- **多分支环境隔离**

  - `main`: 通用 Linux 生产环境配置，追求极致的路由稳定性。
  - `macos`: macOS 桌面端适配版。默认使用 `127.0.0.1:7890` 的 localhost mixed 入站并自动写回系统代理，不启用 TUN。
  - `mobile`: Android (ColorOS) 深度适配版。包含 **VoLTE/VoNR 物理隔离**（通过 `exclude_package`）、FCM 唤醒优化及针对移动端基带的功耗控制。
  - `win11`: Windows 桌面端适配版，针对虚拟网卡特性关闭了不必要的重定向参数。

- **路由逻辑工程化**
  - **单节点手选**：底层只暴露 `LA s1 / LA s2 / LA s3 / JP s4 Fast / EU s5 / Bulk s801` 六个原子节点，不做国家分组，也不做 `urltest` 自动切换。
  - **服务组分层**：上层只保留 `Apple / Telegram / Google / AI / 18comic / dlsite / Steam` 七个服务组，默认继承 `🔰 节点选择`，需要时手工切换到指定节点。
  - **规则源单一真源**：`sing-box` 使用 `MetaCubeX/meta-rules-dat` 的 `sing` 分支；`mihomo` 使用同仓库的 `meta` 分支，不混用别的规则仓库。

## 当前模板

- `config.template.json`
  - `sing-box` 主模板。
  - macOS 分支默认关闭 TUN，改用 `127.0.0.1:7890` mixed 入站 + `set_system_proxy`，不再暴露 `7891` 辅助端口。
  - 保留 Steam CDN 直连、Google Play 区域修复等现有知识。
- `mihomo.template.yaml`
  - `mihomo/clash.meta` 模板。
  - 节点命名、服务组语义与 `sing-box` 完全对齐。
  - 以 iOS 使用场景为主，不迁移 `fcm.hosts` 方案，FCM 直接走直连规则。
  - 不设计应用级排除；iOS 侧统一采用服务组分流。

## 🛠️ 使用指南

本仓库通常作为 **KernelSU/Magisk 模块** 或 **CI/CD 流水线** 的上游子模块使用。

### 手动生成配置（Linux/MacOS）

依赖工具：`gettext` (提供 `envsubst` 命令)

1. 复制环境变量模板：

   ```bash
   cp .env.example .env
   # 编辑 .env 填入您的真实订阅链接和密钥
   # 注意：此文件会被 shell source，带空格/emoji 的值必须保留引号
   vim .env
   ```

2. 渲染配置文件：

   ```bash
   # 加载环境变量并渲染
   set -a && source .env && set +a
   envsubst < config.template.json > config.json
   envsubst < mihomo.template.yaml > mihomo.yaml
   ```

3. 启动 Sing-box：
   ```bash
   sing-box run -c config.json
   ```

4. 或启动 Mihomo：
   ```bash
   mihomo -f mihomo.yaml
   ```

## 订阅要求

- 统一只保留一条上游订阅 `SUB_URL_1`。
- `SUB_URL_1` 必须返回纯节点列表，不能夹带上游自带的 `proxy-groups` 和 `rules`。
- 如果你继续使用订阅转换器，请把 `list=false` 改成 `list=true`。

## ⚠️ 免责声明

本项目仅供技术研究与系统工程实验使用。使用者应遵守当地法律法规。
