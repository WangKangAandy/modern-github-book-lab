# TEST-LOG：公开仓库补测

时间：2026-11-19；范围 C02/C04/C09。完整叙述、URL 与限制见 [TEST-REPORT.md](TEST-REPORT.md)。

| 项目 | 前置 | 命令/UI 动作 | 实际结果 | 证据 | 失败原文/限制 | 等级/可信 |
|---|---|---|---|---|---|---|
| C02 配置 | public，ADMIN | PUT branches/main/protection；enforce_admins=true | 成功配置，仅 PR 可改 main | evidence/c02-protection-response.json | classic protection，未测 ruleset | A/是 |
| C02 拒绝恢复 | main 有本地 README commit | Git Bash commit/push → switch -c → branch -f main origin/main → PR | 直接推送 exit 1；PR #1 合并 ac08917 | evidence/c02-direct-push.txt、c02-recovery-shas.txt、c02-recovery-pr.json | GH006: Protected branch update failed for refs/heads/main. Changes must be made through a pull request. | A+B/是 |
| C04 CLI | gh token 无 project scope | gh auth refresh -h github.com -s project,read:project；浏览器设备授权 | 到 sudo 二次验证，无验证码；refresh 中止；scope 未获授 | evidence/c04-projects-error.txt | Enter the verification code；missing required scopes [read:project] | A+B/是（阻断） |
| C04 网页 | 浏览器已有登录 | Projects → New → Table → 创建 → 加 Issue #4 → Done → Close | 项目 #1 创建、条目 Done、Issue CLOSED、项目显示 Re-open this project | evidence/c04-project-done.png、c04-issue.json；https://github.com/users/WangKangAandy/projects/1 | 项目 private，访问需权限；CLI 未通过 | A/是 |
| C09 本地 | 书稿五个文件、原样 YAML | unittest / build | 9 tests OK；2 resources | evidence/c09-local-tests.txt | 无 | B/是 |
| C09 PR | PR #1 | Actions pull_request | build success；deploy skipped | run 44480366950，evidence/c09-pr-run.json | PR provenance 是合成 merge SHA | A/是 |
| C09 审批 | Pages workflow；release reviewer + main-only | GET pending_deployments → POST approved | waiting → approved → deploy success | run 44480532820，evidence/c09-approval-history.json、c09-main-run.json | 允许本人审批；无 secrets | A/是 |
| C09 artifact/页面 | deploy success | gh run download；解析 tar；HTTP GET 页面和 provenance | artifact 与在线 provenance 相同；HTTP 200；SHA ac08917 | evidence/c09-artifact-verification.json、c09-provenance-before.json | 非签名证明 | A+B/是 |
| C09 临时标题 | PR #2 build success | squash → approve deploy → HTTP | 200 临时标题；SHA 132d61c，run 44481043859 | evidence/c09-title-run.json、c09-provenance-temporary.json | 无 | A/是 |
| C09 revert | PR #3；git revert db22c96 | squash → approve deploy → HTTP | 200 原标题恢复；SHA 386ce82，run 44481298921 | evidence/c09-revert-run.json、c09-provenance-restored.json | 恢复提交可追溯，无重写远端历史 | A/是 |

Git Bash 安装：C:/Users/andy.wang/AppData/Local/Programs/Git/bin/bash.exe。直接将其作为执行器 shell 时命中了 WSL shim（无发行版）；改为 PowerShell 显式启动该安装路径后成功运行同一 bash 脚本。这是本机执行器路径差异，不是书稿 Git 命令失败。

原始命令与请求体、响应均保存在 evidence/；不保存授权设备码或 token。
