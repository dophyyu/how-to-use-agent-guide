# How to Use Agent 项目规则

## 项目目标

本项目是面向中文用户的 Agent 教程中心，生产站点计划部署到 `https://learn.dophyyu.cn`。Codex Guide 与 WorkBuddy Guide 保持各自清晰的学习路径，共用同一套站点导航、搜索、视觉和发布流程。

## 修改前必读

按顺序读取：

1. `README.md`
2. `PRODUCT.md`
3. `DEPLOYMENT.md`
4. `.agents/skills/codexguide-development/SKILL.md`
5. 当前任务涉及的配置、内容或设计文档

## 目录职责

- `docs/`：VitePress 页面、教程内容与公开静态资源。
- `docs/.vitepress/`：站点配置、导航、主题、组件、样式与 SEO。
- `config/`：不含密钥的 Nginx 等部署配置模板。
- `.agents/skills/`：本项目可复用的开发规则。
- `design/`：设计稿和视觉参考，不作为生产页面主源。
- `docs/.vitepress/dist/`：构建产物，不纳入 Git，只用于发布。

不要创建空目录。新增文档先判断是公开教程、项目规则还是部署资料，再放入对应位置。

## 内容与品牌主源

- 站点名称统一为 `How to Use Agent`。
- 生产地址统一为 `https://learn.dophyyu.cn/`。
- 仓库地址统一为 `https://github.com/dophyyu/how-to-use-agent-guide`。
- 未核实的产品能力、价格、可用范围和安全策略不得写成当前事实。
- 移动、重命名或重排教程时，必须按项目 Skill 更新导航、侧栏、索引、SEO 和交叉链接。

## 部署约定

- 使用 VitePress 静态构建，生产环境由 Nginx 直接托管 `docs/.vitepress/dist/`。
- 服务器采用 `/opt/how-to-use-agent-guide/releases/<release-id>/` 与 `current` 软链接结构。
- 不新增 systemd 服务，不把教程源码迁入个人主站项目。
- `dophyyu.cn`、`www.dophyyu.cn`、`fund.dophyyu.cn` 和 `monitor.dophyyu.cn` 保持原有独立路由与服务。
- 部署卡、发布、验证和回滚步骤统一维护在 `DEPLOYMENT.md`。
- 公网页脚展示 `粤ICP备2026096174号` 与 `粤公网安备44030002014919号`，并使用各自官方查询链接。
- 现有 GitHub Pages workflow 不是当前生产发布主路径；未经确认不得启用、修改或依赖它发布。

## 编码与设计约束

- 使用简体中文文案；代码、变量和文件名优先使用英文。
- 保持 Codex 与 WorkBuddy 的区别清晰，同时维持统一的编辑型工具视觉。
- 保留现有导航、搜索、主题切换、教程路由和无障碍能力。
- 外部链接必须来自已核实来源，并使用安全打开方式。
- 保证键盘焦点可见、移动端可用，并支持 `prefers-reduced-motion`。
- 不引入数据库、鉴权、服务端运行时或无需求依赖。

## 验证要求

修改后至少运行：

```powershell
pnpm test
pnpm build
```

涉及界面、导航或部署路径时，还应检查：

- 1440px 桌面端与 390px 移动端渲染。
- 首页、教程入口、静态图片、搜索和 404 页面。
- 页面 canonical、sitemap、robots、备案页脚及外部链接。
- 构建产物体积和发布包中是否包含源码、密钥或运行时数据。

## 红线

以下动作必须获得用户单独确认：

- 删除或批量移动文件。
- commit、push、rebase、改写 Git 历史或配置远端。
- 修改 GitHub Actions、Vercel、DNS、Nginx、证书、生产环境或密钥。
- 将个人主站或其他子网站的业务代码迁入本项目。
