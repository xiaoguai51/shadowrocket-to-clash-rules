# Shadowrocket ADBlock Rules → Clash Premium Rule Sets

将 [Shadowrocket-ADBlock-Rules-Forever](https://github.com/johnshall/Shadowrocket-ADBlock-Rules-Forever) 项目的规则转换为 **Clash Premium** 内核兼容的规则集（RULE-SET），适用于所有基于 Clash Premium / mihomo 内核的 GUI 客户端。

使用 GitHub Actions **每周一北京时间 06:00** 自动拉取最新数据源并更新规则集。

---

## 规则集列表

| 规则集文件 | behavior | 条目数 | 说明 |
|-----------|----------|--------|------|
| `adblock-domain` | `domain` | ~53,800 | 广告拦截域名列表（DOMAIN-SUFFIX） |
| `adblock-ipcidr` | `ipcidr` | ~128 | 广告拦截 IP 段列表（IP-CIDR） |
| `proxy` | `classical` | ~30,700 | 代理域名列表（含 DOMAIN-KEYWORD） |
| `proxy-ipcidr` | `ipcidr` | ~25 | 代理 IP 段列表 |
| `direct-domain` | `domain` | ~378 | 直连域名列表 |
| `direct-ipcidr` | `ipcidr` | ~7 | 直连 IP 段列表 |

> 条目数随上游数据源更新而变化，以上为参考值。

每种规则集同时提供 `.yaml` 和 `.txt` 两种格式：

| 格式 | 适用场景 |
|------|---------|
| `.yaml` | Clash Premium / mihomo 内核原生格式 |
| `.txt` | 纯文本格式，部分客户端支持直接引用 |

---

## 使用方法

### 1. 配置 rule-providers

在 Clash 配置文件的 `rule-providers` 段添加需要使用的规则集。以下为完整示例：

```yaml
rule-providers:
  # 广告拦截
  adblock-domain:
    type: http
    behavior: domain
    url: https://raw.githubusercontent.com/xiaoguai51/shadowrocket-to-clash-rules/main/rules/adblock-domain.yaml
    path: ./ruleset/adblock-domain.yaml
    interval: 604800  # 7 天（秒）

  adblock-ipcidr:
    type: http
    behavior: ipcidr
    url: https://raw.githubusercontent.com/xiaoguai51/shadowrocket-to-clash-rules/main/rules/adblock-ipcidr.yaml
    path: ./ruleset/adblock-ipcidr.yaml
    interval: 604800

  # 代理规则
  proxy:
    type: http
    behavior: classical
    url: https://raw.githubusercontent.com/xiaoguai51/shadowrocket-to-clash-rules/main/rules/proxy.yaml
    path: ./ruleset/proxy.yaml
    interval: 604800

  proxy-ipcidr:
    type: http
    behavior: ipcidr
    url: https://raw.githubusercontent.com/xiaoguai51/shadowrocket-to-clash-rules/main/rules/proxy-ipcidr.yaml
    path: ./ruleset/proxy-ipcidr.yaml
    interval: 604800

  # 直连规则
  direct-domain:
    type: http
    behavior: domain
    url: https://raw.githubusercontent.com/xiaoguai51/shadowrocket-to-clash-rules/main/rules/direct-domain.yaml
    path: ./ruleset/direct-domain.yaml
    interval: 604800

  direct-ipcidr:
    type: http
    behavior: ipcidr
    url: https://raw.githubusercontent.com/xiaoguai51/shadowrocket-to-clash-rules/main/rules/direct-ipcidr.yaml
    path: ./ruleset/direct-ipcidr.yaml
    interval: 604800
```

> 如果无法访问 `raw.githubusercontent.com`，可将 URL 中的域名替换为 `cdn.jsdelivr.net/gh`，例如：
> `https://cdn.jsdelivr.net/gh/xiaoguai51/shadowrocket-to-clash-rules@main/rules/adblock-domain.yaml`
>
> 注意：jsDelivr 镜像内容更新会有约 12 小时延迟。

### 2. 配置 rules

根据使用场景选择模式：

#### 黑名单模式（推荐）

仅命中的流量走代理，其余直连。适合服务器质量一般、流量紧缺的用户。

```yaml
rules:
  # 广告拦截
  - RULE-SET,adblock-domain,REJECT
  - RULE-SET,adblock-ipcidr,REJECT,no-resolve

  # 代理
  - RULE-SET,proxy,PROXY
  - RULE-SET,proxy-ipcidr,PROXY,no-resolve

  # 直连
  - RULE-SET,direct-domain,DIRECT
  - RULE-SET,direct-ipcidr,DIRECT,no-resolve

  # 中国 IP 直连
  - GEOIP,CN,DIRECT,no-resolve

  # 兜底
  - MATCH,DIRECT
```

#### 白名单模式

仅命中的流量直连，其余走代理。适合服务器稳定、流量充足的用户。

```yaml
rules:
  # 广告拦截
  - RULE-SET,adblock-domain,REJECT
  - RULE-SET,adblock-ipcidr,REJECT,no-resolve

  # 直连
  - RULE-SET,direct-domain,DIRECT
  - RULE-SET,direct-ipcidr,DIRECT,no-resolve

  # 中国 IP 直连
  - GEOIP,CN,DIRECT,no-resolve

  # 代理
  - RULE-SET,proxy,PROXY
  - RULE-SET,proxy-ipcidr,PROXY,no-resolve

  # 兜底
  - MATCH,PROXY
```

#### 仅广告拦截

不改变现有分流策略，仅添加广告拦截能力。可与任何现有规则配合使用。

```yaml
rules:
  # 广告拦截（放在最前面）
  - RULE-SET,adblock-domain,REJECT
  - RULE-SET,adblock-ipcidr,REJECT,no-resolve

  # ... 你的其他规则 ...

  - MATCH,PROXY  # 或 MATCH,DIRECT
```

### 3. 关于 no-resolve

IP-CIDR 类型的规则集建议在 `rules` 段添加 `,no-resolve` 后缀，避免 Clash 对域名请求进行 DNS 解析后再匹配 IP，从而防止 DNS 泄漏并提升性能。

---

## 数据来源

本项目的规则集由以下上游项目的数据转换而来：

### 原始规则项目

- [Johnshall/Shadowrocket-ADBlock-Rules-Forever](https://github.com/johnshall/Shadowrocket-ADBlock-Rules-Forever) — 提供 Shadowrocket 格式的规则文件

### Shadowrocket 项目的数据源

| 类别 | 数据源 |
|------|--------|
| 分流基础 | GFWList、Greatfire Analyzer、Apple CDN 域名、全球 Top500 网站 |
| 广告拦截 | EasyList、EasyList China、Peter Lowe 隐私跟踪域名、乘风规则 |
| 懒人配置 | LOWERTOP/Shadowrocket、blackmatrix7/ios_rule_script |

### 转换说明

| Shadowrocket 文件 | 转换为 | 提取策略 |
|-------------------|--------|---------|
| `sr_ad_only.conf` | `adblock-domain` + `adblock-ipcidr` | Reject 规则 |
| `sr_top500_banlist.conf` | `proxy` + `proxy-ipcidr` | Proxy 规则 |
| `sr_top500_whitelist.conf` | `direct-domain` + `direct-ipcidr` | Direct 规则 |

转换过程中的处理：

1. **按策略拆分**：Shadowrocket 规则行包含策略（`Reject`/`Proxy`/`Direct`），Clash 规则集不包含策略。按策略拆分为独立规则集。
2. **按类型拆分**：`DOMAIN-SUFFIX`/`DOMAIN` → `domain` behavior；`IP-CIDR` → `ipcidr` behavior；`DOMAIN-KEYWORD` → `classical` behavior。
3. **去重排序**：所有规则集按字母排序并去重。
4. **过滤不兼容类型**：`URL-REGEX` 等 Clash 不支持的规则类型被过滤。
5. **忽略 FINAL 规则**：`FINAL,Proxy` 等兜底规则不转换，由用户在 `rules` 段用 `MATCH` 实现。

---

## 更新机制

- **更新频率**：每周一北京时间 06:00 自动更新
- **更新方式**：GitHub Actions 自动拉取 Shadowrocket 项目最新规则文件，运行转换脚本，提交到 `main` 分支
- **上游更新频率**：Shadowrocket 项目每日北京时间 08:00 更新，本项目每周同步一次

### 手动触发更新

在 GitHub 仓库页面进入 Actions → Update Rules → Run workflow 可手动触发更新。

---

## 致谢

- [Johnshall](https://github.com/johnshall) — Shadowrocket-ADBlock-Rules-Forever 项目维护者
- [Loyalsoldier](https://github.com/Loyalsoldier) — clash-rules 项目，本项目的文档和格式参考
- [h2y](https://github.com/h2y) — Shadowrocket-ADBlock-Rules 原作者
- 所有上游数据源项目的贡献者

---

## 兼容客户端

所有基于 Clash Premium / mihomo 内核的客户端：

- ClashX Pro (macOS)
- Clash Verge Rev (Windows / macOS / Linux)
- Clash for Windows
- Clash Meta for Android
- OpenClash (OpenWrt)
- mihomo (原 Clash Meta) 内核

---

## 许可证

[MIT License](LICENSE)

本项目仅用于学习和研究目的。规则数据版权归各自上游项目所有。
