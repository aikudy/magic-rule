# Repository Guidelines

## 仓库目标
本仓库用于维护一套面向 [FlClash](https://github.com/chen08209/FlClash) 的通用配置与规则文件，目标是作为多平台客户端的统一配置基础。贡献时应优先保证规则可读、可维护、可迁移，避免写成只适配单一设备或单一网络环境的临时配置。

核心分流原则如下：

- 国外站点：通过规则命中后走代理。
- 国内站点：直接请求，不走代理。
- 无法明确判定国内或国外的流量：默认走代理。

## 项目结构
- `clash/config.yaml`：主配置文件，包含 DNS、代理组、规则顺序和 `rule-providers` 定义。
- `rules/*.yaml`：自定义规则集，按域名、IP、Fake-IP 过滤等类别拆分维护。
- `README.md`：仓库说明。

新增规则时，优先放入 `rules/` 下已有分类文件；只有在语义明显不同、长期独立维护时才新增文件。

## 配置与维护约定
规则顺序必须服务于分流目标：

1. 先匹配局域网、私有地址、国内服务等直连规则。
2. 再匹配明确需要代理的国外站点规则。
3. 最后一条兜底规则默认走代理，而不是直连。

示例：

```yaml
- RULE-SET,cn_domain,直连
- RULE-SET,google_domain,代理
- MATCH,代理
```

如果修改 `rules/*.yaml`，应同步检查 `clash/config.yaml` 中是否已经正确引用对应的 `rule-providers` 与 `rules:` 条目。

## 编码风格与命名
- YAML 使用 2 空格缩进，不使用 Tab。
- 文件名使用小写加连字符风格，如 `private-direct-domain.yaml`。
- 域名规则保持 `payload:` 结构，每行一条，如 `- DOMAIN-SUFFIX,example.com`。
- IP 规则使用 CIDR 格式，如 `- 203.0.113.10/32`。
- 注释只保留必要说明，避免长期保留无效示例。

## 校验与测试
本仓库没有独立构建流程，提交前至少完成以下检查：

- 使用 `yamllint clash/config.yaml rules/*.yaml` 检查 YAML 格式。
- 使用 `mihomo -t -f clash/config.yaml` 或兼容工具验证主配置可加载。
- 人工确认一次关键路径：国内站点直连、明确国外站点走代理、未命中规则的站点最终走代理。

## 提交与合并请求
提交信息应简短明确，直接描述变更，例如：

- `update direct domain rules`
- `add proxy rules for ai services`
- `adjust fallback routing`

Pull Request 应说明变更范围、分流影响、验证方式；如果调整了主策略组或默认行为，应在描述中明确写出预期路由结果。

## 安全与敏感信息
不要提交订阅链接、认证凭据、密钥或个人私有配置。`*.secret`、`*.key`、`*.pem` 以及 `*-private.yaml` 已在忽略规则中排除，新增文件时也应遵守这一边界。
