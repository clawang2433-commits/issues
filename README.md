# Issues — 多项目 Bug 追踪

本仓库为 **公开仓库**，仅用于管理各项目的 bug 和改进建议，**不存放任何代码**。

## ⭐ SSOT 规则文件 (所有 Agent 必读)

**路径**: [`bug-routing-rules.yml`](./bug-routing-rules.yml)

本仓库根目录的 `bug-routing-rules.yml` 是 bug 路由规则的**唯一权威来源 (SSOT)**。所有 Agent (Trae/ZCode/CW/AG/DS) 和 GitHub Actions workflow 都必须遵循该规则。

规则内容:
- **仓库映射**: 哪个 repo 存代码, 哪个 repo 存 issue
- **标签体系**: project / type / priority / version / process 五类标签
- **触发关键词**: 哪些 commit message 触发自动建 issue / 关闭 issue
- **创建规则**: commit 含 bug 关键词 → 自动建 issue; bazi repo 误建 issue → 自动转移
- **关闭规则**: commit 含 `fixes #N` → 自动关闭 issues repo 的 #N
- **例外情况**: 纯文档/测试代码/merge commit 不触发
- **Agent 行为约束**: R1-R5 五条规则
- **验证规则**: CI 闸门检查

修改规则请通过 PR, 不要直接 push main。

## 项目索引
| 项目 | 代码仓库 (Private) | Issue 仓库 (Public) | 目录 | 说明 |
|:--|:--|:--|:--|:--|
| Bazi | clawang2433-commits/bazi | clawang2433-commits/issues | `bazi/` | 八字排盘系统 |

## 使用方式

### Agent 创建 bug (必走 issues repo)
```bash
gh issue create --repo clawang2433-commits/issues \
  --title "[Bug N] 简述 [待修复]" \
  --label "bazi,bug,P1,v10.8" \
  --body "..."
```

### Agent 修复后关闭
```bash
# 方式 1: commit message 自动关闭
git commit -m "fix: 修复 XX bug

RootCause: ...
fixes #N  # N 为 issues repo 的 issue 编号
cd955"

# 方式 2: 手动关闭
gh issue close --repo clawang2433-commits/issues <N> --reason completed
```

### 禁止行为
- ❌ 在 bazi 代码仓库创建任何 issue (会被 workflow 自动转移)
- ❌ 在 issues 仓库提交任何源代码 (.py/.ts/.js 等)
- ❌ 直接 push main 修改 bug-routing-rules.yml (必须走 PR)
- ❌ Agent 自行决定 issue 创建位置 (一律 issues repo)

## GitHub Actions

### 本仓库 (issues repo) workflows
- [`issue-triage.yml`](./.github/workflows/issue-triage.yml) — 自动打标签 (监听 issue 事件 + 每日巡检)
- [`post-deploy-verify.yml`](./.github/workflows/post-deploy-verify.yml) — 部署后 smoke test

### bazi 仓库 workflows (跨 repo 操作)
- `bug-routing.yml` — 监听 bazi repo 的 push (含 bug 关键词 → 建 issue) + issue opened (转移)
- 该 workflow 需要 `ISSUES_REPO_TOKEN` secret (PAT, 有 issues repo 的 write 权限)

## 目录结构
```
issues/
├── README.md                    # 本文件
├── bug-routing-rules.yml        # ⭐ SSOT 规则文件
├── .github/workflows/
│   ├── issue-triage.yml         # 自动打标签
│   └── post-deploy-verify.yml   # 部署验证
└── bazi/                        # Bazi 项目 bug 文档
    └── README.md
```
