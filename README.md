# SiteProxy 自托管发布快照

基于 [netptop/siteproxy](https://github.com/netptop/siteproxy)，保留原 MIT License 和 `Copyright (c) 2020 netptop`。这里发布的是退役服务使用的运行代码快照与脱敏部署示例，不是新的公共免费代理服务。

## 发布边界与退役说明

原公共托管服务 `proxy.wolfeovo.top` 进入退役流程；本仓库只用于自行部署，不承诺任何原域名的可用性。准备源码不等于完成线上停机。上游说明见 `README.upstream.md`（历史文档，描述的安全性、模型版本及外部服务可用性未经本快照验证；其中部分历史相对链接未随本白名单快照发布）。

上游 2.x 提供混淆后的 JavaScript 产物。本快照保留 `bundle.mjs`、`build/worker.js`、`build/cf_page/_worker.js`，**不声称包含完整可读源码、反混淆源码或可重建这些产物的构建链**。Node bundle 和 Cloudflare worker 是不同运行平台的现存产物，本次没有重新构建或公开网络测试。

未复制生产配置、路径密码、API token、历史备份、缓存、debug 文件、编辑器配置或不明源码归档。示例 `/default/` 是公开的路径前缀，**不是密码**。

## 本地 Node.js 运行

需要 Node.js **22 或以上**。Node 运行产物不需要安装 npm 运行依赖；Wrangler 仅为可选的 Cloudflare 开发工具。

1. 打开根目录 `config.json`，按需要修改配置。当前示例为：
   - `proxy_url`: `http://localhost:5006`
   - `token_prefix`: `/default/`
   - `local_listen_port`: `5006`
2. 在仓库根目录运行：

   ```sh
   node bundle.mjs
   # 或 npm start（等价命令）
   ```

3. 本地示例入口为 `http://localhost:5006/default/`，按 Ctrl+C 退出。直接 Node 启动可能监听所有网卡，不应假定它只绑定回环；请使用防火墙/网络隔离。推荐下面仅映射回环的容器部署。
4. 自托管外网部署时，把 `proxy_url` 改为自己的 HTTPS 地址，再配置反向代理；不要使用原退役域名或文档占位域名。

## Docker / Compose

```sh
docker compose -f docker-node/docker-compose.yml config --quiet
docker compose -f docker-node/docker-compose.yml up -d
```

Compose 使用 `node:22`，实际执行 `node /home/node/siteproxy/bundle.mjs`，仅发布 `127.0.0.1:5006:5006`。已移除生产容器的固定 `container_name`，避免与原服务重名；如端口已被占用，请修改端口映射及相应公开地址。不要在仍运行原服务的机器上直接执行此示例。仓库同时保留可用 `docker build` 构建的 `Dockerfile`。

## Cloudflare 配置示例

- `wrangler.jsonc`：Pages，产物目录 `build/cf_page`。
- `wrangler.worker.jsonc`：Workers，入口 `build/worker.js`。
- 名称 `siteproxy-example`、`https://example.invalid` 均为占位值，必须换成自己控制的项目/地址。示例不包含 Cloudflare 账户 ID 或 token。
- Wrangler 的安装、登录和实际部署由部署者自行完成；不要把凭据写入 Git。本次未登录 Cloudflare，未部署也未访问任何他人域名资源。

## 已移除的本机字体补丁

Node bundle 原有一段指向原托管域名 `/fleet-fonts/shs-sc-vf.css` 的 CSS `@import`。本发布仅删除这一段 host-specific `<style>`，保留原有 `siteproxy-response-injected.js?v=4` 注入脚本；不提供或编造字体资产。删除前后 bundle SHA256 记录在发布目录上级的 `provenance.json` 和审计报告中，worker 产物未修改。

## 安全限制

- 路径前缀不是健全的身份认证，会暴露在 URL、浏览器历史、访问日志及可能的 Referer 中。公开 `/default/` 不能提供访问控制。
- 这是访问目标网站的代理，存在开放代理滥用及 SSRF 风险；不要允许访问回环、内网、链路本地地址或云实例元数据服务。应在网络出口层强制限制，不能假定混淆代码已经可靠防护。
- 应置于 VPN、可信网络或有独立身份认证的反向代理之后，配置目标白名单、速率限制、日志脱敏和出口限制；在完成独立安全审查前，不应向公众开放。
- 上游含外部主页、响应注入及第三方网站适配逻辑；混淆使审计更困难，勿据此承诺安全或隐私保护。不要通过不可信部署输入账号密码或其他敏感数据。
- 使用者须遵守目标服务条款与法律。本快照不提供公共代理、可用性保障或第三方登录兼容性保证。

## 许可证

MIT，见 `LICENSE`。保留上游版权与许可声明。
