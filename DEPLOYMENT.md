# How to Use Agent 部署与回滚

> 状态：生产 HTTPS 已发布并通过验收
> 最后核验：2026-07-27

## 部署卡

```text
project_name: How to Use Agent
local_path: D:\AI Project\1.个人项目\how-to-use-agent-guide
runtime: Nginx 静态文件
build_command: pnpm build
build_output: docs/.vitepress/dist
health_check: GET / 返回 200，标题包含 How to Use Agent
service_port: none，复用 Nginx 80/443
public_domains: learn.dophyyu.cn
server_ip: 1.12.247.55
server_user: ubuntu
ssh_key_path: C:\Users\dophyyu\.ssh\dophyyu_fund_tencent_ed25519
remote_base_dir: /opt/how-to-use-agent-guide
systemd_service: none
nginx_site: how-to-use-agent-guide
icp_number: 粤ICP备2026096174号
public_security_number: 粤公网安备44030002014919号
https_domains: learn.dophyyu.cn
rollback_method: 将 current 恢复到上一 release；首次发布则停用教程站 Nginx 配置
```

## 边界

- 本站只接管 `learn.dophyyu.cn`。
- `dophyyu.cn`、`www.dophyyu.cn`、`fund.dophyyu.cn` 和 `monitor.dophyyu.cn` 不在教程站发布范围。
- 生产环境不需要 Node.js、pnpm、数据库、`.env` 或 systemd 服务。
- 发布包只包含 `docs/.vitepress/dist/` 内的静态产物，不包含源码、`.git`、日志、缓存和 `node_modules`。
- 当前 GitHub Pages workflow 保留但不作为本次生产发布路径；启用或修改 CI/CD 需单独确认。

## 当前生产状态

2026-07-27 已核实：

- `learn.dophyyu.cn` 的 DNS A 记录已指向 `1.12.247.55`。
- HTTP 自动跳转 HTTPS，HTTPS 返回 200，证书域名匹配。
- Let's Encrypt 证书有效期至 2026-10-25，`certbot.timer` 为 active。
- 服务器为 Ubuntu 22.04，Nginx 配置检查通过且服务为 active。
- 服务器根分区可用空间约 33 GB。

## 当前本地验证

2026-07-27 基于上游提交 `87457a8` 和当前未提交的部署适配改动完成：

- `pnpm install --frozen-lockfile` 通过，锁文件供应链策略检查通过。
- `pnpm test` 通过：2 个测试文件、16 个测试。
- `pnpm build` 通过，生成 1237 个文件，总体积约 324.2 MB。
- 首页 canonical、sitemap 和 robots 均指向 `https://learn.dophyyu.cn/`。
- 首页包含 ICP、公安备案号和可加载的备案图标。
- 桌面端 1440px 与移动端 390px 无横向溢出，键盘焦点可见。
- `/codex/`、`/workbuddy/` 返回 200，未知路径返回 404，本地搜索可打开。
- 构建存在一个非阻塞提示：部分压缩后 chunk 超过 500 kB；本次不做无需求拆分。

视觉核验文件：

- `learn-desktop.png`
- `learn-mobile-learning-viewport.png`

以上截图保存在本次 Codex 可视化工作区，不纳入项目仓库。

## 当前发布

```text
release_id: 20260727155640-6cc9cf2-learn
current: /opt/how-to-use-agent-guide/releases/20260727155640-6cc9cf2-learn
archive_sha256: 52adbba634a4f9f9cd1d0ed6bfe5fb352f0be6f6f7e56f893aa1a046a4a09d75
certificate: /etc/letsencrypt/live/learn.dophyyu.cn/fullchain.pem
certificate_expiry: 2026-10-25
nginx_http_backup: /etc/nginx/sites-available/how-to-use-agent-guide.bak-20260727164304-before-https
published_at: 2026-07-27
```

- [x] 静态 release 已上传并切换 `current`。
- [x] 教程站 HTTP Nginx 配置已启用，`sudo nginx -t` 通过。
- [x] 使用指定 Host 验证首页、Codex、WorkBuddy 返回 200，未知路径返回 404。
- [x] DNSPod 已新增 `learn` A 记录并指向 `1.12.247.55`。
- [x] 已为 `learn.dophyyu.cn` 签发证书并切换 HTTPS 配置。
- [x] 公网桌面端与 390px 移动端渲染、canonical 和备案页脚验证通过。
- [x] 根域、`www`、基金站和 Agent Monitor 回归验证通过。

## 本地预检与构建

```powershell
git status --short
node -v
pnpm -v
pnpm install --frozen-lockfile
pnpm test
pnpm build
```

构建后核验：

- `docs/.vitepress/dist/index.html` 存在。
- 首页标题、canonical 和 sitemap 指向 `https://learn.dophyyu.cn/`。
- 首页包含 `粤ICP备2026096174号`、`粤公网安备44030002014919号` 和公安备案图标。
- Codex、WorkBuddy、搜索、静态资源及 404 页面可访问。
- 记录构建产物体积，确认服务器空间足够。

## GitHub 内容更新后的发布流程

生产服务器只保存构建后的静态文件，不直接拉取 GitHub，也不会自动发布。日常更新统一从本地仓库发起：

1. 确认当前工作区没有需要保留但尚未提交的改动。
2. 切换到生产分支并快进同步 GitHub：

```powershell
git switch main
git pull --ff-only origin main
```

3. 当 `package.json` 或 `pnpm-lock.yaml` 变化时执行 `pnpm install --frozen-lockfile`。
4. 运行 `pnpm test` 和 `pnpm build`；任一步失败都停止发布。
5. 只打包 `docs/.vitepress/dist/`，创建新的 release 目录并上传到服务器。
6. 在服务器核验新 release 后，将 `/opt/how-to-use-agent-guide/current` 切换到它。
7. 验证首页、教程入口、搜索、静态资源、404、备案页脚和原有子站。

部署适配已通过 GitHub Pull Request #1 合并到 `main`，后续统一按上述流程维护。

## 发布结构

```text
/opt/how-to-use-agent-guide/
├── releases/<release-id>/
└── current -> releases/<active-release>/
```

静态 release 不需要共享运行时数据。发布归档上传到 `/tmp/`，在新的 release 目录中解压并检查后再切换 `current`。

## 生产发布顺序

以下步骤涉及生产写入，执行前需要用户单独确认：

1. DNSPod 新增 A 记录：`learn` 指向 `1.12.247.55`。
2. 上传并解压新的静态 release。
3. 先启用 `config/nginx-learn-dophyyu-http.conf`，执行 `sudo nginx -t` 后 reload。
4. 验证 `http://learn.dophyyu.cn/` 可访问且内容正确。
5. 为 `learn.dophyyu.cn` 独立签发证书。
6. 切换为 `config/nginx-learn-dophyyu-https.conf`，再次执行 `sudo nginx -t` 后 reload。
7. 完成公网、备案、路由和原有子站回归验证。

## 公网验收

- `http://learn.dophyyu.cn/` 跳转 HTTPS。
- `https://learn.dophyyu.cn/` 返回 200，证书域名匹配。
- 首页、Codex Guide、WorkBuddy Guide、搜索、图片和 404 页面正常。
- ICP 与公安备案号、图标和查询链接可见。
- `dophyyu.cn`、`fund.dophyyu.cn` 和 `monitor.dophyyu.cn` 行为不变。

## 回滚

已有 release 时：

```bash
ln -sfn /opt/how-to-use-agent-guide/releases/<previous-release> /opt/how-to-use-agent-guide/current
sudo nginx -t
sudo systemctl reload nginx
```

首次发布失败时，停用教程站的 Nginx 配置并 reload；不删除 release，不修改个人主站、基金站或 Agent Monitor 配置。

## GitHub Pages 说明

仓库仍保留 `.github/workflows/deploy-pages.yml`。它属于旧的 GitHub Pages 发布路径，与 `learn.dophyyu.cn` 的腾讯云 Nginx 发布方案不同。除非重新评估发布架构并获得确认，否则不要启用或修改该 workflow。
