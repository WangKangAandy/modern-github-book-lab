# 公开仓库补测报告

执行日期：2026-11-19，Asia/Shanghai。账号 WangKangAandy；仓库 [WangKangAandy/modern-github-book-lab](https://github.com/WangKangAandy/modern-github-book-lab)，public、ADMIN。
书稿基线 `22827850b579c4c1151e4589ec0763915af8045b`，只读。此次范围仅 C02 分支保护、C04 Projects、C09 Pages 及回滚；不代表全书全部实验通过。

## 执行摘要

| 项目 | 补测结论 | 证据等级与可信范围 |
|---|---|---|
| C02 | 通过：main 保护创建成功，管理员直接 push 被拒绝，提交保留到分支、恢复 main、经 PR 合并 | A+B：API 回执、Git Bash 输出、PR #1；只测 classic branch protection，未另测 ruleset |
| C04 | 网页最小实验通过；CLI scope 刷新仍因二次验证阻断 | A：项目 #1、Issue #4、截图；网页创建/状态修改/关闭，未声称 CLI Projects 成功 |
| C09 | 通过本次范围：原样书稿 workflow 的 PR build、main deploy、release 审批、artifact、页面 provenance、临时标题及 revert 回滚 | A+B：六个 PR/push runs、三次 HTTP 页面、SHA、审批和 artifact 下载核对；没有签名证明或 Release 发布声明 |

公开仓库消除了上次私有仓库分支保护 403 和 Pages 422 的计划限制。Projects 的 CLI scope 与仓库可见性是两回事；浏览器既有登录可操作，并不说明 CLI 已获新权限。

## C02：真实保护拒绝与恢复

- 前置：公开仓库管理员；main 原无保护（GET 404 `Branch not protected`）。
- PUT `repos/WangKangAandy/modern-github-book-lab/branches/main/protection` 成功。设置 `enforce_admins=true`，要求 PR，审批数 0；禁止 force push 和删除。审批数 0 是单账号实验选择，未测试第二人批准。
- 用 Git Bash（通过 PowerShell 启动其实际安装路径）提交 README 后，`git push origin main` 返回 exit 1：
  `GH006: Protected branch update failed for refs/heads/main.`
  `Changes must be made through a pull request.`
- 保留提交 `4d8734d` 到 `retest/public-pages`，`git branch -f main origin/main` 恢复本地 main，未强推远端。
- [PR #1](https://github.com/WangKangAandy/modern-github-book-lab/pull/1) squash 合并；合并 SHA `ac0891747fa13232565072a42bb8a05a638563b6`。
- 原始证据：[保护回执](evidence/c02-protection-response.json)、[拒绝输出](evidence/c02-direct-push.txt)、[恢复 SHA](evidence/c02-recovery-shas.txt)、[PR 回执](evidence/c02-recovery-pr.json)。

## C04：网页成功与 CLI 阻断分别记录

执行 `gh auth refresh -h github.com -s project,read:project`，在浏览器确认账号并提交本机 device code 后，GitHub 要求 `Confirm access` / `Enter the verification code`（二次验证）。没有验证码，不读取密码或绕过验证；停止等待中的 refresh 进程。

`gh project list --owner WangKangAandy --format json` 原始错误：
`error: your authentication token is missing required scopes [read:project]`
`To request it, run: gh auth refresh -s read:project`
见 [错误文件](evidence/c04-projects-error.txt)。设备码未保存进仓库。

通过已有浏览器登录完成：
1. 打开用户 Projects → New project → Table。
2. 命名 `C04 modern-github-book-lab retest`，取消批量导入其他仓库条目。
3. 创建 [项目 #1](https://github.com/users/WangKangAandy/projects/1)。
4. 新建并加入 [Issue #4](https://github.com/WangKangAandy/modern-github-book-lab/issues/4)，修改 Status 为 Done。
5. 页面随后显示 Issue closed；API 确认 CLOSED，时间 `2026-11-19T07:43:38Z`。未手动用 CLI 关闭，观察与项目自动工作流相符。
6. Settings → Close this project；页面显示 `Re-open this project`，确认关闭并保留审计内容。

项目默认 private，未更改可见性，因此其链接需要账号权限；[截图](evidence/c04-project-done.png)与 [Issue 回执](evidence/c04-issue.json)可复核。未测试 Projects CLI 写入、全部视图、sub-issue/dependency。

## C09：Pages、审批与来源

从书稿 C09 的 YAML 代码块原样提取 `.github/workflows/pages.yml`，复制 C07 五个资产到根目录：`.gitignore`、`build.py`、`catalog.py`、`resources.csv`、`test_catalog.py`。
本地执行书稿命令 `python -m unittest discover -s . -p 'test_*.py' -v`：9 项通过；`python build.py` 生成 2 条资源。[本地输出](evidence/c09-local-tests.txt)。

配置：
- Pages Source = workflow，HTTPS 启用：[API 回执](evidence/c09-pages-settings.json)。
- Environment `release`：WangKangAandy required reviewer、允许本人审批、管理员不能 bypass、Selected branches 仅 main，无 secrets：[环境](evidence/c09-environment-response.json)、[分支](evidence/c09-branch-policy.json)。
- main run 实际进入 waiting；GET pending_deployments 返回 `current_user_can_approve=true`；POST approval 后 deploy 执行。[等待](evidence/c09-pending-deployment.json)、[审批历史](evidence/c09-approval-history.json)。

| 阶段 | PR / run | 提交与结果 |
|---|---|---|
| 初始 PR build | [PR #1](https://github.com/WangKangAandy/modern-github-book-lab/pull/1) / [44480366950](https://github.com/WangKangAandy/modern-github-book-lab/actions/runs/44480366950) | Build success，Deploy skipped；artifact 已下载 |
| 初始 main deploy | [44480532820](https://github.com/WangKangAandy/modern-github-book-lab/actions/runs/44480532820) | success；`ac0891747fa13232565072a42bb8a05a638563b6` |
| 临时标题 PR build | [PR #2](https://github.com/WangKangAandy/modern-github-book-lab/pull/2) / [44480871992](https://github.com/WangKangAandy/modern-github-book-lab/actions/runs/44480871992) | Build success，Deploy skipped |
| 临时标题 main deploy | [44481043859](https://github.com/WangKangAandy/modern-github-book-lab/actions/runs/44481043859) | success；`132d61c418e668333e87c2c2991885921df8e4da` |
| revert PR build | [PR #3](https://github.com/WangKangAandy/modern-github-book-lab/pull/3) / [44481138172](https://github.com/WangKangAandy/modern-github-book-lab/actions/runs/44481138172) | Build success，Deploy skipped；revert commit `db22c96` |
| 恢复 main deploy | [44481298921](https://github.com/WangKangAandy/modern-github-book-lab/actions/runs/44481298921) | success；`386ce826353ee837b52a64cfe91fa39f739f70ac` |

三次通过 HTTP GET 验证 [Pages](https://wangkangaandy.github.io/modern-github-book-lab/)：
- 初次：200，h1 学习资源导航；provenance SHA=ac08917…，run=44480532820。
- 临时：200，h1 临时标题；SHA=132d61c…，run=44481043859。
- 回滚：200，h1 学习资源导航；SHA=386ce82…，run=44481298921。
- [线上 provenance](https://wangkangaandy.github.io/modern-github-book-lab/provenance.json) 与对应运行 GITHUB_SHA 相符。保存的三份 HTML 与 provenance 在 evidence/。
- `gh run download 44480532820 --name github-pages` 真正下载 tar，比较其中 provenance 与线上 JSON 完全相同：[验证](evidence/c09-artifact-verification.json)、[artifact 元数据](evidence/c09-main-artifact.json)。
- PR 的 provenance 使用合成 merge SHA（`43af6a9…`），不同于 head SHA 是 pull_request 事件语义，不是部署来源错误；main 部署按合并提交核对。
- 这是书稿自定义 provenance 文件，不是签名 attestation。Release 发布和非 main 部署拒绝不在本次执行声明中。

## 清理与保留

- 临时 Project 已关闭，Issue #4 已关闭；三个实验 PR 合并并删除远端分支。
- main 保护、release Environment、Pages 保留供审计和后续实验；不涉及真实 secrets 或付费资源。
- 证据报告用独立文档 PR 合并；报告提交使用 `[skip ci]`，避免仅记录证据又触发额外部署。线上源码 SHA 因而停留在已验证的回滚提交，未把报告 commit 当作部署 commit。
- 没有修改书稿工作树；本地下载产物位于本会话 work/，不提交嵌套仓库、缓存或 scratch。

## 对上次报告的更正

1. C07 初测漏复制 build.py 是执行错误；这次复制书稿完整五个资产后 9 项 unittest 和 build 均通过，不能据此声称书稿缺少 build.py。
2. `gh run --log` 的 `log not found` 单独不能证明私有仓库日志资格限制；上次没有取得 annotations 根因，结论应仅写“日志未取得，原因未确认”。
3. 本次 Projects 网页成功修正了“Projects 整体阻断”的表述，剩余仅 CLI 新 scope 授权未完成。
4. 上次未实测的 C01 网页、C03 全部恢复命令、C06 表单加载、Agent 资格等不因本次三项补测而变为通过。

