# Surge 双端配置模板（iOS / macOS）

一套同源维护的 [Surge](https://nssurge.com/) 配置，iOS 与 macOS 各一份，开箱即用：

- **双端拆分**：平台专属配置各归其位，共享的分流规则逐行一致，不会漂移
- **DNS 防泄漏**：国内域名走阿里 DoH 直连，其余走 Cloudflare DoH 经代理；劫持 53 端口、拦截第三方应用自带 DoH/DoT、拒绝 STUN 探测
- **国内 CDN 调度**：Apple / 微软 / 游戏与软件下载域名用国内 DNS 解析，App Store、Windows 更新、Steam 下载直连拉满速度
- **QUIC 智能放行**：直连流量自由使用 HTTP/3，代理侧由 Surge 按节点 UDP 能力自动决定（`block-quic=auto`）
- **MITM 模板**：定向 hostname 清单（GitHub 429 修复、知乎优化等），证书由你本地生成，仓库不含任何私钥

## 仓库结构

```
├── surge-ios.conf        # iOS 版配置
├── surge-macos.conf      # macOS 版配置
└── rulesets/
    ├── cdn-cn.list       # 跨国厂商国内 CDN 调度域名集（自维护）
    └── block-doh.list    # 第三方加密 DNS 拦截清单（自维护）
```

## 版本要求

Surge iOS ≥ 5.14.3，Surge Mac ≥ 5.10.0（`[Host]` 的 DOMAIN-SET 语法所需）。

## 快速开始

### 1. 导入配置

- **iOS**：Surge → 首页左上角配置名 → 「从 URL 下载配置」，填入：
  `https://raw.githubusercontent.com/lihuaye/surge/main/surge-ios.conf`
- **macOS**：Surge → Profiles → 「Install Profile from URL」，填入：
  `https://raw.githubusercontent.com/lihuaye/surge/main/surge-macos.conf`

配置首行带 `#!MANAGED-CONFIG` 托管头，导入后每 24 小时自动同步本仓库更新。

### 2. 接入你自己的节点（必做）

模板中 `AllServer` 组的订阅链接（`policy-path`）指向作者本人的 Sub-Store，**你必须换成自己的**：

- 用 [Sub-Store](https://github.com/sub-store-org/Sub-Store)：替换为你的集合链接（target=Surge）
- 或直接用机场订阅：把 `policy-path` 换成机场提供的 Surge 订阅地址

另注意：`Self Server` 组按节点名含「自建」二字筛选（`policy-regex-filter=自建`），`AI` 组默认首选它。如果你的节点没有这个命名习惯，请改掉该筛选词，或把 `AI` 组指向其他地区组，否则 AI 分流会落空。

### 3. macOS 开启增强模式（必做）

Surge Mac → 开启「增强模式」（Enhanced Mode）。不开的话，DNS 劫持与接管只对走系统代理的应用生效，防泄漏体系不完整。iOS 无需操作（VPN 模式天然全接管）。

### 4. 生成 MITM 证书（可选）

想让 GitHub 429 修复、知乎重写等 https 规则生效：

- Surge → MitM → 生成新的 CA 证书 → 安装
- **iOS**：设置 → 通用 → VPN 与设备管理 → 安装描述文件，再到 通用 → 关于本机 → 证书信任设置 → 开启完全信任
- **macOS**：按提示存入钥匙串并设为「始终信任」

⚠️ 证书（`ca-p12`）与口令只会写入你设备的本地配置。**如果你 fork 了本仓库，绝不要把真实证书提交上去**——那等于把你设备的 TLS 安全公开送人。

### 5. 验证

- [dnsleaktest.com](https://dnsleaktest.com)（Extended）：应只出现 Cloudflare 节点，出现本地运营商 DNS 即为异常
- [browserleaks.com/webrtc](https://browserleaks.com/webrtc)：不应显示你的公网真实 IP
- App Store 下一个大体积应用，确认走了国内 CDN 的速度

## Fork 指引

fork 后需要改 4 处 `lihuaye/surge` 引用，全部换成你的 `用户名/仓库名`：

| 位置 | 文件 |
|---|---|
| 首行 `#!MANAGED-CONFIG` 托管头 | 两份 .conf 各 1 处 |
| `[Rule]` 中 `block-doh.list` 的 RULE-SET 引用 | 两份 .conf 各 1 处（同一行内容） |
| `[Host]` 中 `cdn-cn.list` 的 DOMAIN-SET 引用 | 两份 .conf 各 1 处（同一行内容） |

之后按需调整：`FINAL` 兜底默认指向 `AI` 组（作者有意为之，未匹配流量交自建节点承接），常规做法是改成 `Proxy`；两份配置自 `[Proxy Group]` 起需保持逐行一致，改分流规则请两边同步。

自维护域名集的扩展方式：国内站点 CDN 调度不佳 → 域名追加进 `rulesets/cdn-cn.list`（`.` 前缀，如 `.example.cn`）；发现新的公共 DoH 端点 → 追加进 `rulesets/block-doh.list`。

## 免责声明

- 本仓库仅为 Surge 软件的**配置文件模板**，供学习与网络技术研究使用，不提供任何代理服务器、节点或订阅服务
- 请在遵守你所在国家/地区法律法规的前提下使用本配置；因使用本配置产生的任何直接或间接后果，由使用者自行承担，作者不承担任何责任
- 配置中引用的第三方规则集（blackmatrix7、Sukka 等）版权归各自作者所有，其内容变动不受本仓库控制
- MITM 功能涉及 TLS 解密，仅应用于你本人拥有的设备与账号；启用与否及由此产生的风险由使用者自行决定与承担
- 本配置按「现状」提供，不对可用性、准确性或适用性作任何保证
