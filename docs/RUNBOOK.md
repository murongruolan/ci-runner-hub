# CI 中枢运行手册

## 手动触发

1. GitHub → ci-runner-hub → Actions → private-repo-ci → Run workflow
2. 填 target_repo（白名单三选一）+ target_sha（完整 40 位 commit SHA）
3. Run。结果与日志公开；被测 SHA 精确可溯。

## 命令行触发（gh）

gh workflow run private-repo-ci.yml --ref ci/v1 -f target_repo=murongruolan/parahuman-memory-core -f target_sha=<40位SHA>
gh run watch  # 或 gh run list --workflow=private-repo-ci.yml

## 结果关联

Run URL + inputs.target_sha = 私有仓库审查证据关联（Run 页面 inputs 区显示精确 SHA）。

## 故障排查

- checkout 失败：PAT 失效或 SHA 不存在 → 核对私有仓库该 SHA 是否存在
- race 超时：-timeout 25m 内未完成 → 分包运行
- 未知字段类失败：Core API 变更后 workflow 同步
