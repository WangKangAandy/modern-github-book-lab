# 公开仓库补测计划

2026-11-19，WangKangAandy/modern-github-book-lab；书稿仅只读。

| 项目 | 执行动作 | 验收 |
|---|---|---|
| C02 | 配置 main 保护（管理员也受限），直接 push 预期拒绝；保留提交到工作分支，恢复 main，通过 PR 合并 | 保护 API、拒绝原文、恢复前后 SHA、PR |
| C04 | gh auth refresh -s project,read:project；授权可用才创建/关闭临时 Project | 项目 URL 或实际认证阻断 |
| C09 | 使用书稿原样 YAML 和资源；PR build；main deploy；release reviewer、main-only；下载 artifact；核对网页及 provenance；临时标题和 revert | run/jobs/审批回执/页面/来源 SHA/回滚 |

证据 A：GitHub API/run/commit/HTTP 页面；B：本地命令；未执行步骤不宣称通过。证据文件仅含实验仓库元信息，不保存凭据。
