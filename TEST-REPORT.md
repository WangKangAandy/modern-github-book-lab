# 公开仓库补测报告

执行日期：2026-11-19，Asia/Shanghai。账号 WangKangAandy；仓库 [WangKangAandy/modern-github-book-lab](https://github.com/WangKangAandy/modern-github-book-lab)，public、ADMIN。
书稿基线 `22827850b579c4c1151e4589ec0763915af8045b`，只读。范围包括 C02 分支保护、C04 Projects、C08 Resource CI 补测证据、C09 Pages 及回滚、C12 本地 gh-aw 验证与编译记录；不代表全书全部实验通过。

## 执行摘要

| 项目 | 补测结论 | 证据等级与可信范围 |
|---|---|---|
| C02 | 通过：main 保护创建成功，管理员直接 push 被拒绝，提交保留到分支、恢复 main、经 PR 合并 | A+B：API 回执、Git Bash 输出、PR #1；只测 classic branch protection，未另测 ruleset |
| C04 | 网页最小实验通过；CLI scope 刷新仍因二次验证阻断 | A：项目 #1、Issue #4、截图；网页创建/状态修改/关闭，未声称 CLI Projects 成功 |
| C08 | Resource CI 成功路径、重复 URL 拒绝及 required check 门禁均已验证通过；同一失败 PR 修复转绿未验证 | A：PR #6 两 job 成功；PR #7 故意失败；PR #9 在 Check resources 必须通过且 strict=true 时失败，API 显示 blocked、merged=false |
| C09 | 通过本次范围：原样书稿 workflow 的 PR build、main deploy、release 审批、artifact、页面 provenance、临时标题及 revert 回滚 | A+B：六个 PR/push runs、三次 HTTP 页面、SHA、审批和 artifact 下载核对；没有签名证明或 Release 发布声明 |
| C12 | 本地 validate/compile 通过（执行窗口交接）；云端模型/凭据仍受账号资格阻断 | B：gh-aw v1.2.2，1 workflow、0 warnings、143.1 KB；本窗口直接复核版本与清理，未重新编译；没有云端 run 或事件链成功证据 |

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

## C08：干净复测与预期失败（新增）

本节由另一测试窗口执行后提供 run/PR 编号，本窗口通过 GitHub API 和失败日志重新核实，未冒称在本窗口重新触发运行。

| 场景 | PR / run | 实际结果与 SHA |
|---|---|---|
| 正常资源与 artifact | [PR #6](https://github.com/WangKangAandy/modern-github-book-lab/pull/6) / [Resource CI 44490990698](https://github.com/WangKangAandy/modern-github-book-lab/actions/runs/44490990698) | success；Check resources、Verify artifact 两 job 均 success；head SHA `b856427780addf7520f12d9f8eaeace58405a947` |
| 重复 docs.github.com URL | [PR #7](https://github.com/WangKangAandy/modern-github-book-lab/pull/7) / [Resource CI 44491808019](https://github.com/WangKangAandy/modern-github-book-lab/actions/runs/44491808019) | failure；Validate catalog 失败，后续 build/upload 和 Verify artifact 跳过；head SHA `091b03f48bdf0d627e7d23a5b7b31c02e0271d9c` |
| 同一无效清单触发 Pages 检查 | [Pages 44491807943](https://github.com/WangKangAandy/modern-github-book-lab/actions/runs/44491807943) | failure；Validate catalog 失败，Deploy site 跳过；同一 head SHA |

成功路径包含 unittest、清单校验、构建、上传，以及另一 job 下载 artifact 并执行 Verify artifact contents。PR #6 已合并，merge SHA `878ff1788cad5d83214705a5d7cbbc945b60bf02`。

失败原文：`Catalog error: Line 5: duplicate URL`；`Process completed with exit code 1.`。这是故意无效数据被正确拒绝，不是书稿缺陷。PR #7 CLOSED、未合并；远端 `test/c08-failure` 查询 HTTP 404，符合已删除状态。没有把“关闭失败 PR”记成“修复后转绿”。PR #7 归档时 required_status_checks 尚未配置；后续 PR #9 已完成下述 required check 验证，替代此前“门禁未验证”的结论。

### Required check 补测：已通过验证

main protection 已实际设置 `required_status_checks.contexts=["Check resources"]`、`strict=true`，绑定 GitHub Actions app_id=15368，管理员也受保护规则约束。本窗口重新读取 [配置回执](evidence/c08-required-protection.json) 确认设置。

- [PR #9](https://github.com/WangKangAandy/modern-github-book-lab/pull/9) 的 `test/c08-required-check` 分支加入重复 docs.github.com URL；head SHA `bdfbb99af3d0829839ae1c71085566877143625c`。
- [Resource CI run 44493493607](https://github.com/WangKangAandy/modern-github-book-lab/actions/runs/44493493607) failure，Check resources 在 Validate catalog 失败；Verify artifact skipped。[jobs 证据](evidence/c08-required-ci-failure.json)、[失败原文](evidence/c08-required-failure-log.txt)。
- [Pages run 44493493780](https://github.com/WangKangAandy/modern-github-book-lab/actions/runs/44493493780) failure。[jobs 证据](evidence/c08-required-pages-failure.json)。
- 实验窗口在失败后观察 PR API `mergeable_state=blocked`、`merged=false`；本窗口在 PR 关闭后重新读取 API，仍返回上述值及 `state=closed`。[PR 回执](evidence/c08-required-pr9.json)。区分历史阻断观察与当前关闭状态，不冒称再次尝试合并。
- PR #9 已关闭、未合并；远端分支 GET 返回 404，[清理证据](evidence/c08-required-branch-deleted.txt)。

结论：C08 classic branch protection 的 required check 与 Ruleset 的 required check 均已验证在失败时阻止合并。没有强推、管理员 bypass 或关闭规则来合并失败变更；同一失败 PR 修复转绿仍未执行。

### Ruleset 补测：独立门禁证据

公开 lab 另创建并保留 Ruleset ID `27624499`，target=branch、source=本仓库、enforcement=active，条件只匹配 `refs/heads/main`，包含：

- `pull_request` rule，允许 squash/rebase/merge，审批数 0；
- `required_status_checks`，context `Check resources`，`strict_required_status_checks_policy=true`；
- `non_fast_forward`；
- `current_user_can_bypass=never`。

这与 classic protection 分开记录，避免把两类设置混为一谈。移除重叠 classic protection 后，PR #12 的故意重复 URL 变更仍被 Ruleset 门禁阻断：

- [PR #12](https://github.com/WangKangAandy/modern-github-book-lab/pull/12)：head SHA `816834decf2bf252d9a79a7f9f916d2a62f6252f`，API 快照 `mergeable_state=blocked`、`merged=false`、`state=closed`；
- [Resource CI 44496259367](https://github.com/WangKangAandy/modern-github-book-lab/actions/runs/44496259367)：Validate catalog failure，后续 Verify artifact skipped；
- PR #12 已关闭，`test/c08-ruleset-block` 远端分支 GET 404；Ruleset 27624499 保留。

失败日志下载本次为空，不能补造未取得的原始 validator 文本；jobs/PR/Ruleset API 证据足以证明门禁阻断，但不把“日志可见”另行宣称通过。完整响应见 [Ruleset](evidence/c08-ruleset-27624499.json)、[PR](evidence/c08-ruleset-pr12.json)、[run](evidence/c08-ruleset-run.json)、[分支删除](evidence/c08-ruleset-branch-deleted.txt)。

证据：[成功 jobs](evidence/c08-success-run.json)、[CI 失败 jobs](evidence/c08-expected-failure-run.json)、[失败日志](evidence/c08-expected-failure-log.txt)、[Pages 失败 jobs](evidence/c08-pages-expected-failure-run.json)、[PR 状态](evidence/c08-prs.json)、[分支删除回执](evidence/c08-deleted-branch.txt)。

归因修正：此前 xallor 私有 lab 的 C08 实验使用了自行改写的脚本，还出现将字面量反引号 n 写入 YAML 的执行错误；该轮失败属于实验脚本问题，不归因书稿。对于后来仅返回 `log not found` 的具体 run，不补造未取得的运行时根因；以本次干净复测作为书稿流程的有效验证依据。

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

## C12：本地校验与编译，云端仍阻断

执行窗口在 WangKangAandy lab 安装 gh-aw v1.2.2，使用书稿 C12 最小 staged workflow，实际执行：

```bash
gh aw validate resource-review
gh aw compile resource-review
```

交接结果：两条命令均成功；编译汇总为 **1 workflow、0 warnings、143.1 KB**。临时 workflow 和 lock 已删除。

本窗口实测 `gh aw version` 得到 `gh aw version v1.2.2`，扩展列表确认 `github/gh-aw v1.2.2`；更新报告前工作树干净，workflows 目录仅 ci.yml/pages.yml。编译命令的成功来自执行窗口交接，本窗口未重跑、也未取得原始 stdout，完整来源说明见 [本地实验交接记录](evidence/c12-local-handoff.md)。

C12 结论由“未安装扩展”更新为“本地工具链验证/编译通过，云端模型/凭据仍为账号资格阻断”。**本地编译不证明模型调用、staged 审批或事件链成功**。本次没有云端 run URL，也未收到资格阻断原始报错文本；不编造错误原文或把本地结果升级为云端通过。

## 清理与保留

- 临时 Project 已关闭，Issue #4 已关闭；三个实验 PR 合并并删除远端分支。
- main 保护、release Environment、Pages 保留供审计和后续实验；不涉及真实 secrets 或付费资源。
- 证据报告用独立文档 PR 合并；此前无 required check 时使用 `[skip ci]`。配置 required check 后，本次报告 PR 正常运行 CI 并在 Check resources 成功后合并，不跳过必需检查。C09 表格记录的是当时实际验证的部署快照，后续提交不改变历史证据。
- C08 PR #6 已合并；故意失败的 PR #7 已关闭、分支已删除。临时 resources.csv.bak 与 resources.csv 的 SHA256 完全相同，整理日志时清除此重复备份，不提交 scratch。
- C08 required check 实验 PR #9 已关闭、分支已删除；main 的 Check resources strict 保护保留。
- C12 临时 resource-review workflow/lock 已清理；gh-aw v1.2.2 保留安装，没有新增模型凭据或 secrets。
- 没有修改书稿工作树；本地下载产物位于本会话 work/，不提交嵌套仓库、缓存或 scratch。

## 对上次报告的更正

1. C07 初测漏复制 build.py 是执行错误；这次复制书稿完整五个资产后 9 项 unittest 和 build 均通过，不能据此声称书稿缺少 build.py。
2. `gh run --log` 的 `log not found` 单独不能证明私有仓库日志资格限制；上次没有取得 annotations 根因，结论应仅写“日志未取得，原因未确认”。
3. 本次 Projects 网页成功修正了“Projects 整体阻断”的表述，剩余仅 CLI 新 scope 授权未完成。
4. 上次未实测的 C01 网页、C03 全部恢复命令、C06 表单加载、Agent 资格等不因本次三项补测而变为通过。
5. C08 私有 lab 初测的实验脚本错误不归因书稿；公开仓库干净复测确认正常 CI/artifact 路径成功、重复 URL 被拒绝。未完成项仍保留，不将 C08 全章一概标成通过。
6. C12 “gh-aw 未安装”的旧状态已过时；现为 v1.2.2 本地 validate/compile 成功，云端仍阻断。C08 required check 已通过验证，保留 PR #9 的 blocked/merged=false 与配置证据。
