---
name: git-squash
description: 将当前分支上属于自己的 commit 按改动相近原则智能合并（squash）整理，生成语义清晰的中文 commit。当用户想要整理/压缩/合并 commit、清理提交历史时触发。
metadata:
  short-description: 智能合并当前分支的 commit 历史
---

# Git Squash

针对当前分支，找出属于自己的 commit，按改动内容相近的原则智能合并为若干个语义清晰的 commit。

> ⚠️ **所有 commit message 必须使用中文书写。**

## 工作流程

### 第一步：识别"我的 commit"

```bash
git log --oneline                          # 查看当前分支所有 commit
git log --format="%H %ae %s"              # 获取 hash、作者邮箱、摘要
git config user.email                      # 获取当前用户邮箱
```

筛选规则：
- 邮箱与当前用户一致 → **我的 commit**，参与合并
- 邮箱不一致 → **他人的 commit**，原封不动保留，**绝不参与合并**

退出条件：
- "我的 commit" 数量为 0 → 提示用户，退出
- "我的 commit" 只有 1 个 → 提示无需合并，退出

### 第二步：分析并规划合并方案

逐一分析每个 commit 的 diff：

```bash
git show <commit-hash> --stat
git show <commit-hash>
```

**分组原则**（满足越多越适合合并为一组）：

1. 改动文件高度重叠
2. 变更类型一致（同为功能/修复/重构等）
3. 业务语义相近，服务同一功能点
4. 规模相当，避免差异悬殊的强行合并

不满足上述条件的 commit 保持独立，**不要强行压缩为一个**。

分析完成后，向用户展示方案：

```
📋 合并方案预览：

分组 1（共 3 个 commit → 合并为 1 个）
  - abc1234  feat: 添加登录接口
  - def5678  feat: 添加登录页面 UI
  - ghi9012  feat: 添加登录状态管理
  → 合并后 commit 信息：✨ feat: 实现用户登录功能

分组 2（共 2 个 commit → 合并为 1 个）
  - jkl3456  fix: 修复金额精度问题
  - mno7890  fix: 修复金额展示格式
  → 合并后 commit 信息：🐛 fix: 修复金额计算与展示问题

保留不变（他人 commit）
  - pqr1234  chore: 升级基础依赖（张三）
```

等待用户确认。用户可回复"确认"/"ok"执行，或提出调整意见后重新展示，直到确认为止。

### 第三步：执行合并

**推荐方式（逐组处理，更安全）**：

```bash
# 以分组 1（3 个 commit）为例
git reset --soft HEAD~3
git commit -m "<新的 commit 信息>"
```

若分组中夹杂他人 commit，改用 `git rebase -i` 精确控制：他人 commit 保持 `pick`，我的 commit 使用 `squash`。

**commit 信息规范**：格式 `<emoji> <type>: <中文描述>`，描述不超过 50 字。

| emoji | type | 场景 |
|-------|------|------|
| ✨ | feat | 新功能 |
| 🐛 | fix | Bug 修复 |
| 🔒️ | fix | 安全修复 |
| 📝 | docs | 文档 |
| ♻️ | refactor | 重构 |
| ⚡️ | perf | 性能优化 |
| ✅ | test | 测试 |
| 🔧 | chore | 工具/配置 |
| 💄 | style | 样式/格式 |
| 🚀 | ci | CI/CD |

### 第四步：完成确认

```bash
git log --oneline    # 展示整理后的 commit 列表
```

告知用户完成，并提示推送需使用：

```bash
git push --force-with-lease    # 禁止使用 --force
```

---

## 重要约束

- 他人 commit **绝对不参与合并**，顺序不变
- 目标是"相近的合并，不相近的保留"，**不强制压缩为一个**
- 涉及历史重写，操作前确认当前分支**不是 master/main 等受保护分支**
- 任何步骤失败，立即停止并告知用户，不跳过继续执行
