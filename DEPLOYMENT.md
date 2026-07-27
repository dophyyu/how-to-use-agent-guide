# How to Use Agent 部署与回滚

> 状态：本地验证通过，待生产变更确认
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

## 当前只读预检

2026-07-27 已核实：

- `learn.dophyyu.cn` 尚无 DNS A 记录。
- 服务器为 Ubuntu 22.04，Nginx 配置检查通过。
- 服务器根分区可用空间约 33 GB。
- 现有证书不包含 `learn.dophyyu.cn`，需在 DNS 生效后独立签发。

正式发布前必须重新执行这些检查，不能把本节作为实时状态。

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
