# TEST-LOG：公开仓库补测

时间：2026-11-19；范围 C02/C04/C08/C09/C12。C08 实验由另一窗口执行，本窗口重新查询 run/PR/日志核实并归档。C12 本地编译按执行窗口交接记录归档，本窗口直接复核工具版本和清理状态。完整叙述、URL 与限制见 [TEST-REPORT.md](TEST-REPORT.md)。

| 项目 | 前置 | 命令/UI 动作 | 实际结果 | 证据 | 失败原文/限制 | 等级/可信 |
|---|---|---|---|---|---|---|
| C02 配置 | public，ADMIN | PUT branches/main/protection；enforce_admins=true | 成功配置，仅 PR 可改 main | evidence/c02-protection-response.json | classic protection，未测 ruleset | A/是 |
| C02 拒绝恢复 | main 有本地 README commit | Git Bash commit/push → switch -c → branch -f main origin/main → PR | 直接推送 exit 1；PR #1 合并 ac08917 | evidence/c02-direct-push.txt、c02-recovery-shas.txt、c02-recovery-pr.json | GH006: Protected branch update failed for refs/heads/main. Changes must be made through a pull request. | A+B/是 |
| C04 CLI | gh token 无 project scope | gh auth refresh -h github.com -s project,read:project；浏览器设备授权 | 到 sudo 二次验证，无验证码；refresh 中止；scope 未获授 | evidence/c04-projects-error.txt | Enter the verification code；missing required scopes [read:project] | A+B/是（阻断） |
| C04 网页 | 浏览器已有登录 | Projects → New → Table → 创建 → 加 Issue #4 → Done → Close | 项目 #1 创建、条目 Done、Issue CLOSED、项目显示 Re-open this project | evidence/c04-project-done.png、c04-issue.json；https://github.com/users/WangKangAandy/projects/1 | 项目 private，访问需权限；CLI 未通过 | A/是 |
| C08 正常 CI | PR #6、有效清单、Resource CI workflow | gh run view 44490990698 --json jobs,conclusion,headSha；核实 PR | Check resources 与 Verify artifact 均 success；PR 合并 878ff1788cad5d83214705a5d7cbbc945b60bf02 | [run](https://github.com/WangKangAandy/modern-github-book-lab/actions/runs/44490990698)、evidence/c08-success-run.json | 此阶段未测门禁，后续 PR #9 已验证 | A/是 |
| C08 故意失败 | PR #7、重复 docs.github.com URL | gh run view 44491808019 --log-failed；查询两 workflow | Resource CI failure；Pages run 44491807943 failure；校验之后步骤跳过 | [CI](https://github.com/WangKangAandy/modern-github-book-lab/actions/runs/44491808019)、[Pages](https://github.com/WangKangAandy/modern-github-book-lab/actions/runs/44491807943)、evidence/c08-expected-failure-log.txt | Catalog error: Line 5: duplicate URL；Process completed with exit code 1. | A/是 |
| C08 清理/限制 | PR #7 失败实验结束 | gh pr view 7；GET git/ref/heads/test/c08-failure；查询 protection | PR CLOSED、未合并，远端分支 404；当时 required_status_checks 无配置值 | evidence/c08-prs.json、c08-deleted-branch.txt | 关闭 PR 不等于修复转绿；后续门禁见下一行 | A/是 |
| C08 required check | main context=Check resources，strict=true，管理员受限；PR #9 重复 URL | 另一窗口配置并观察阻断；本窗口 GET protection/pulls/9、gh run view、下载失败日志 | Resource CI 44493493607 failure；Pages 44493493780 failure；PR API blocked、merged=false；PR 已关闭，分支 404 | [PR #9](https://github.com/WangKangAandy/modern-github-book-lab/pull/9)、[CI](https://github.com/WangKangAandy/modern-github-book-lab/actions/runs/44493493607)、[Pages](https://github.com/WangKangAandy/modern-github-book-lab/actions/runs/44493493780)、evidence/c08-required-*.json/txt | required check 已通过验证；当前 PR API 为关闭后快照，未重新尝试合并 | A/是 |
| C08 Ruleset | Ruleset 27624499 active，target main，context Check resources，strict=true，pull_request + non_fast_forward；PR #12 重复 URL | 另一窗口移除重叠 classic protection 后配置并观察；本窗口 GET Ruleset/PR/run、分支引用 | Resource CI 44496259367 failure；PR API blocked、merged=false；PR 已关闭，分支 404；Ruleset 保留 | [Ruleset](evidence/c08-ruleset-27624499.json)、[PR #12](https://github.com/WangKangAandy/modern-github-book-lab/pull/12)、[run](https://github.com/WangKangAandy/modern-github-book-lab/actions/runs/44496259367)、evidence/c08-ruleset-*.json/txt | Ruleset 门禁已验证；失败日志文件为空，未伪造 validator 原文 | A/是（API/jobs）；日志文本不宣称 |
| C09 本地 | 书稿五个文件、原样 YAML | unittest / build | 9 tests OK；2 resources | evidence/c09-local-tests.txt | 无 | B/是 |
| C09 PR | PR #1 | Actions pull_request | build success；deploy skipped | run 44480366950，evidence/c09-pr-run.json | PR provenance 是合成 merge SHA | A/是 |
| C09 审批 | Pages workflow；release reviewer + main-only | GET pending_deployments → POST approved | waiting → approved → deploy success | run 44480532820，evidence/c09-approval-history.json、c09-main-run.json | 允许本人审批；无 secrets | A/是 |
| C09 artifact/页面 | deploy success | gh run download；解析 tar；HTTP GET 页面和 provenance | artifact 与在线 provenance 相同；HTTP 200；SHA ac08917 | evidence/c09-artifact-verification.json、c09-provenance-before.json | 非签名证明 | A+B/是 |
| C09 临时标题 | PR #2 build success | squash → approve deploy → HTTP | 200 临时标题；SHA 132d61c，run 44481043859 | evidence/c09-title-run.json、c09-provenance-temporary.json | 无 | A/是 |
| C09 revert | PR #3；git revert db22c96 | squash → approve deploy → HTTP | 200 原标题恢复；SHA 386ce82，run 44481298921 | evidence/c09-revert-run.json、c09-provenance-restored.json | 恢复提交可追溯，无重写远端历史 | A/是 |

| C12 本地 | gh-aw v1.2.2、书稿最小 staged workflow | 执行窗口：gh aw validate resource-review；gh aw compile resource-review；本窗口：gh aw version / extension list | validate/compile 成功；1 workflow、0 warnings、143.1 KB；版本已直接复核 | evidence/c12-local-handoff.md | 原始编译 stdout 未提供；未在本窗口重跑 | B/按交接来源可信，仅本地 |
| C12 清理与云端边界 | 本地编译结束 | 删除临时 workflow/lock；本窗口 git status 与 workflows 目录复核 | 更新报告前工作树干净，仅 ci.yml/pages.yml；云端模型/凭据仍受账号资格阻断 | evidence/c12-local-handoff.md | 没有云端 run 或事件链成功证据；资格阻断原始报错未提供 | B/版本清理直接核实；云端阻断按交接记录 |

Git Bash 安装：C:/Users/andy.wang/AppData/Local/Programs/Git/bin/bash.exe。直接将其作为执行器 shell 时命中了 WSL shim（无发行版）；改为 PowerShell 显式启动该安装路径后成功运行同一 bash 脚本。这是本机执行器路径差异，不是书稿 Git 命令失败。

原始命令与请求体、响应均保存在 evidence/；不保存授权设备码或 token。

C08 归因更正：此前 xallor 私有 lab 的失败是实验脚本问题（含字面量反引号 n 导致的 YAML 错误），不归因书稿；日志未取得的后续 run 不推断具体根因。本次正常与异常路径分别按云端实际结果记录。
