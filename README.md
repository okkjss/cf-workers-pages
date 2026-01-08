# CF_Workers_Pages

Cloudflare Workers 项目，可在 Workers 或 Pages（Pages Functions）上部署与运行。

- 入口：`src/index.ts`
- 主逻辑：`src/worker.ts`
- Pages Functions 入口：`functions/_worker.ts`

## 部署

### 一键部署（Cloudflare Workers）

[![Deploy to Cloudflare Workers](https://deploy.workers.cloudflare.com/button)](https://deploy.workers.cloudflare.com/?url=https://github.com/yoogg/CF_Workers_Pages)

Workers 一键部署时会读取 `wrangler.toml` 的 `[vars]` 作为默认值，部署向导/控制台中可覆盖。

### 部署到 Cloudflare Pages

Pages 通过 Pages Functions 复用同一份 Worker 逻辑（入口：`functions/_worker.ts`）。

推荐流程：先 Fork 本仓库，再在 Cloudflare Pages 导入该仓库创建项目。

1. 在 GitHub Fork：`https://github.com/yoogg/CF_Workers_Pages`
2. 打开 Cloudflare Dashboard → Workers & Pages
3. 选择 Create application → Pages → Connect to Git
4. 选择你 Fork 的仓库与分支（通常为 `main`），完成创建

创建 Pages 项目建议（可按需调整）：

- Framework preset：None
- Build command：留空（或 `npm run typecheck` 仅做校验）
- Build output directory：`.`

Pages 的环境变量与 KV 绑定需要在 Pages 项目中单独配置。默认值可参考 `pages-env.example`。

## 配置

配置优先级：KV 配置 > 环境变量 > 默认值。

### 基础配置

| 变量        | 含义               | 说明                                                             |
| ----------- | ------------------ | ---------------------------------------------------------------- |
| `u` / `U`   | UUID               | 必需；用于订阅与管理入口校验                                     |
| `d` / `D`   | 自定义路径         | 可选；支持多级路径，如 `/path/to/sub`（不以 `/` 开头会自动补全） |
| `p` / `P`   | proxyip / fallback | 可选；回落地址或自定义 ProxyIP                                   |
| `s` / `S`   | SOCKS5             | 可选；格式 `user:pass@host:port` 或 `host:port`                  |
| `wk` / `WK` | 地区代码           | 可选；手动指定 Worker 地区（如 `SG`、`US`、`JP`）                |

### 协议配置

| 变量 | 值       | 说明                         |
| ---- | -------- | ---------------------------- |
| `ev` | `yes/no` | 启用 VLESS（默认启用）       |
| `et` | `yes/no` | 启用 Trojan（默认禁用）      |
| `ex` | `yes/no` | 启用 xhttp（默认禁用）       |
| `tp` | 文本     | Trojan 密码；留空则使用 UUID |

### 高级控制

| 变量        | 值       | 说明                                                                                |
| ----------- | -------- | ----------------------------------------------------------------------------------- |
| `scu`       | URL      | 订阅转换地址（默认 `https://url.v1.mk/sub`）                                        |
| `rcu`       | URL      | 远程配置 URL（订阅转换的 `config` 参数）                                            |
| `hd` / `HD` | 域名     | 伪装域名（用于节点链接的 Host/SNI）；留空则使用当前 Worker 域名                     |
| `yx` / `YX` | 文本     | 自定义优选 IP/域名，逗号分隔，支持命名：`1.1.1.1:443#香港节点,example.com:443#节点` |
| `yxURL`     | URL      | 优选来源 URL（可选）                                                                |
| `epd`       | `yes/no` | 启用优选域名                                                                        |
| `epi`       | `yes/no` | 启用优选 IP                                                                         |
| `egi`       | `yes/no` | 启用 GitHub 默认优选                                                                |
| `yxby`      | `yes`    | 关闭所有优选功能，仅使用原生地址                                                    |
| `dkby`      | `yes`    | 仅生成 TLS 节点（不生成非 TLS）                                                     |
| `rm` / `RM` | `no`     | 关闭地区智能匹配                                                                    |
| `qj`        | `no`     | 启用降级模式（直连失败 → SOCKS5 → fallback）                                        |
| `ae`        | `yes`    | 开启 API 管理（默认关闭）                                                           |

## KV 配置（可选）

用于图形化管理与持久化配置。

1. 在 Cloudflare Dashboard 创建 KV Namespace
2. 绑定到 Worker：绑定名必须为 `C`
3. 重新部署

KV 约定：

- 绑定名：`C`
- 键：`c`
- 值：JSON（例如 `{"p":"example.com","scu":"https://url.v1.mk/sub"}`）

## API

前提：需要开启 `ae=yes`。API 路径会校验 UUID 或自定义路径（`d`）。

- 查询优选：`GET /{UUID或路径}/api/preferred-ips`
- 添加（单个/批量）：`POST /{UUID或路径}/api/preferred-ips`
- 删除：`DELETE /{UUID或路径}/api/preferred-ips`

示例（添加单个）：

```bash
curl -X POST "https://your-worker.workers.dev/{UUID或路径}/api/preferred-ips" \
  -H "Content-Type: application/json" \
  -d '{"ip": "1.2.3.4", "port": 443, "name": "香港节点"}'
```

示例（批量添加）：

```bash
curl -X POST "https://your-worker.workers.dev/{UUID或路径}/api/preferred-ips" \
  -H "Content-Type: application/json" \
  -d '[
    {"ip": "1.2.3.4", "port": 443, "name": "节点1"},
    {"ip": "5.6.7.8", "port": 8443, "name": "节点2"}
  ]'
```

## 本地开发

- 安装依赖：`npm install`
- 本地运行：`npm run dev`
- 部署：`npm run deploy`
- 类型检查：`npm run typecheck`
- 格式化：`npm run format`
