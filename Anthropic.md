# 长时间运行 Agent 工作规范

> 原文: https://www.anthropic.com/engineering/effective-harnesses-for-long-running-agents

---

## 核心原则

**AI 代理无持久记忆，必须通过文件系统传递上下文。**

---

## 一、必需文件（必须创建）

| 文件名 | 用途 | 创建时机 |
|--------|------|----------|
| `init.sh` | 环境初始化脚本 | 项目初始化 |
| `progress.txt` | 进度记录 | 项目初始化 |
| `features.json` | 功能列表管理 | 项目初始化 |

### features.json 格式

```json
{
  "project": "MyProject",
  "version": "1.0.0",
  "features": [
    {
      "id": "feat-001",
      "name": "功能名称",
      "description": "功能详细描述",
      "priority": "high",
      "steps": ["step1", "step2"],
      "passes": false
    }
  ]
}
```

### progress.txt 格式

```
# Project Progress Log
## Session: YYYY-MM-DD HH:MM
- [x] 完成项1
- [x] 完成项2
- 下一步: 待办项
```

### init.sh 格式

```bash
#!/bin/bash
npm install
npm run db:migrate
npm run dev
```

---

## 二、双代理架构

| 代理类型 | 职责 |
|---------|------|
| Initializer Agent | 首个会话：创建 init.sh、progress.txt、features.json，提交初始 Git |
| Coding Agent | 后续会话：增量开发，保持环境干净，确保下一个代理可直接继续 |

---

## 三、会话工作流程

### 会话启动流程（每个会话必须执行）

1. `pwd` 确认工作目录
2. 检查 init.sh、progress.txt、features.json 是否存在
3. 读取 progress.txt 获取历史工作记录
4. `git log --oneline -10` 查看最近提交
5. 读取 features.json 找到 passes=false 的最高优先级功能
6. 执行 init.sh 启动环境
7. 运行冒烟测试验证系统可用

### 功能开发流程（每个功能必须执行）

1. 从 features.json 领取一个 passes=false 的功能
2. 按 priority 字段选择最高优先级
3. 实现功能代码
4. 运行端到端测试验证
5. 测试通过后更新 features.json 的 passes=true
6. 更新 progress.txt 记录进度
7. `git commit -m "..."` 提交代码

---

## 四、操作规范

### Git 提交规范

**提交时机**：
- 每个功能完成后**必须**提交
- 会话结束前**必须**提交

**提交格式**：

```bash
git commit -m "feat: 实现功能名称

- 完成项1
- 完成项2
- 更新 progress.txt：完成 feat-001
- 更新 features.json：passes=true"
```

### 测试要求

**测试优先级**：
1. 端到端测试（必须）
2. 冒烟测试（必须）
3. 单元测试（可选）

**验证流程**：
1. 代码实现完成
2. 运行端到端测试
3. 验证用户流程正常
4. 如有问题则修复后重新测试
5. 测试通过才更新 passes=true

### 状态规则

- features.json **只允许**修改 passes 字段
- **禁止**删除或重写功能项
- passes=false 表示未完成，passes=true 表示已验证通过

---

## 五、检查清单

### 会话开始

- [ ] pwd 确认目录
- [ ] 检查必要文件存在
- [ ] 读取 progress.txt
- [ ] git log 查看最近提交
- [ ] features.json 领取任务
- [ ] init.sh 启动环境
- [ ] 冒烟测试验证

### 功能开发

- [ ] 实现功能代码
- [ ] 端到端测试验证
- [ ] 更新 features.json passes=true
- [ ] 更新 progress.txt
- [ ] git commit 提交

### 会话结束

- [ ] 代码无重大 bug
- [ ] 代码有序有注释
- [ ] 所有文件已提交
- [ ] 环境处于干净状态

---

## 六、失败处理

| 失败场景 | 处理方式 |
|---------|---------|
| 测试失败 | 修复代码 → 重新测试 → 通过后才提交 |
| 功能复杂 | 拆分功能 → 每次只完成一个子功能 |
| 环境问题 | 检查 init.sh → 修复 → 重新执行 |
| 上下文丢失 | 读取 progress.txt + git log + features.json 恢复 |

---

## 七、核心禁令

- **禁止** one-shot 完成多个功能
- **禁止** 未经测试就标记 passes=true
- **禁止** 跳过 git commit
- **禁止** 不更新 progress.txt
- **禁止** 删除或重写 features.json 中的功能项
- **禁止** 代码没有中文注释
- **禁止** 会话不用中文回复
