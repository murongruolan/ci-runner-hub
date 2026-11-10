# CI-RUNNER-HUB-01-REPORT（2026-11-11 凌晨）

## 判定：首次真实验收 **PASS**——ParaHuman 私有仓库的云端 CI 能力已恢复

## 一、交付

| 项 | 值 |
|---|---|
| 公共仓库 | `murongruolan/ci-runner-hub`（public） |
| 当前分支 | `ci/v1`（空仓库首推即默认分支——如实说明：空库无法预置 main，首次推送必然成为默认分支） |
| 实现 Commit | `9c9ca11`（workflow + README + RUNBOOK） |
| 工作流 | `.github/workflows/private-repo-ci.yml` |
| 首次真实验收 Run | https://github.com/murongruolan/ci-runner-hub/actions/runs/42863658160 |

## 二、首次真实验收（WP-D）逐项结果

被测：`murongruolan/parahuman-memory-core` @ `4c7a08d407c5567027366545999a577859c78262`（QUALITY04 分支头——私有仓库 Linux CI 曾通过的对照基线）

| 步骤 | 结果 |
|---|---|
| 输入校验（白名单 + 40 位 SHA 正则） | ✅ |
| 私有仓库检出（PRIVATE_REPO_PAT，persist-credentials: false） | ✅ |
| **SHA 一致性核验**（`git rev-parse HEAD` == 请求 SHA，workflow 内强制） | ✅ |
| 持久化 Git 凭据清除 + PAT 环境变量残留检查 | ✅（env 无 PAT 残留） |
| Setup Go（go.mod 权威版本 1.26.0） | ✅ |
| go build / go vet / **go test 全量** / **go race 全量**（-timeout 25m） | ✅ 全部成功 |
| Job 摘要（Run URL + SHA + 结论写入 Step Summary） | ✅ |

Run 全长 **7 分 50 秒**（GitHub-hosted ubuntu-latest，免费额度，非 self-hosted，零付费）。

## 三、安全边界（WP-C，如实）

**已实现**：
1. 仅 `workflow_dispatch` 触发——无 push/PR 触发器，Fork PR 不携带 PAT
2. 调度权限 = 仓库 write 协作者（当前：主人 + OmniBridgeH）
3. 白名单 + 40 位 SHA 正则在 workflow 内强制校验
4. `actions/checkout` 固定 v4 tag SHA（`11d5960a…`）+ `persist-credentials: false`——PAT 不落 .git/config
5. 检出后显式清除持久化凭据 + PAT 环境变量残留断言（泄漏即 FAIL）
6. 无私有源码 Artifact 上传、无工作目录缓存
7. 公开日志泄漏检查：**零 Token 值**（`ghp_` 计数 0；唯一模式命中 = 工作流源码中的 Secret 名称字样，公开文件本就可见）

**如实声明的边界（未伪装解决）**：
- **手动调度 + SHA 校验不能阻止"测试代码读取运行环境"**——白名单 SHA 的测试代码运行于可联网 Runner，恶意测试理论上可外传私有源码。实际缓解 = 信任域一致：可调度的 SHA 必须存在于私有仓库，即可推送该 SHA 的人 = 私有仓库写入者 = 同一信任域
- PAT 在 checkout 步骤期间对 runner 进程可见（私有检出的必然）——`persist-credentials: false` + 步后清除已将暴露窗口缩到最短，但非零
- 公开日志含私有仓库的构建/测试路径与测试名（无源码正文、无密钥）

## 四、WP-E：后续扩展路径

| 仓库 | 接入方式 | 状态 |
|---|---|---|
| parahuman-memory-core | Go 通道（已验收） | ✅ |
| parahuman-aurabot | Node 通道：在 SCENARIOS/step 按其 package.json scripts 扩展 `if:` 分支 | 待主人指定验收 SHA |
| parahuman-research | 按实际结构定 | 待主人指定验收 SHA |

重复运行指定 SHA / 结果追溯：同 SHA 重跑 → concurrency 组排队；Run URL + inputs.target_sha = 审查证据关联。私有 PR Check 自动回写：本轮不做（PAT 只读，无私有仓库 Check 写权限——如实）。

## 五、故障排查

| 症状 | 处置 |
|---|---|
| checkout 失败 | PAT 失效或 SHA 不在私有仓库 → 核对 |
| race 超时 | -timeout 25m / job 45m 内未完 → 分包运行 |
| 白名单拒绝 | repo 拼写或未列入 workflow `options` → 编辑 workflow |
| 结果核对 | Run 页 inputs 区显示精确 SHA；Step Summary 有结论 |
