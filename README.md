# ci-runner-hub

ParaHuman 私有仓库的公共 GitHub Actions CI 中枢。

- 私有仓库保存源码；本仓库执行构建/测试/CI（GitHub-hosted ubuntu-latest，免费额度）。
- 触发方式：Actions → private-repo-ci → Run workflow → 选择白名单仓库 + 完整 40 位 Commit SHA。
- 工作流位于 `ci/v1` 分支（`.github/workflows/private-repo-ci.yml`）；首次验收运行在该分支上执行。
- 安全模型与残余风险见工作流文件头注与 `docs/RUNBOOK.md`。
- 不使用 self-hosted Runner；不产生付费 Runner 费用；私有源码不进入公开日志或 Artifact。

## 白名单

- murongruolan/parahuman-memory-core（Go 通道已接入）
- murongruolan/parahuman-aurabot（待接入）
- murongruolan/parahuman-research（待接入）
