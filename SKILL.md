# SKILL: Todo Agent

**版本**：1.0.0
**兼容**：NanoClaw core >= 1.2.0
**渠道**：Telegram
**说明**：智能待办 Agent，让 Claude 成为用户的任务管家

---

## 安装说明

> 以下步骤由 Claude Code 在 NanoClaw 项目目录执行

### 第一步：创建数据目录结构

为每个群组创建数据目录（以主群为例，其他群组同理）：

```bash
mkdir -p /workspace/group/data
```

将以下模板文件复制到数据目录：
- `templates/todos.json` → `/workspace/group/data/todos.json`
- `templates/todos_archive.json` → `/workspace/group/data/todos_archive.json`

### 第二步：添加 skill 加载机制

修改 `container/agent-runner/src/index.ts`，在构建 system prompt 的位置加入 skill 文件加载逻辑：

在文件顶部引入 fs 模块（如未引入）：
```typescript
import fs from 'fs';
import path from 'path';
```

在 system prompt 构建处，添加 skill prompt 加载：
```typescript
// Load skill prompts from /workspace/group/.skills/
const skillsDir = '/workspace/group/.skills';
let skillPrompts = '';
if (fs.existsSync(skillsDir)) {
  const skillFiles = fs.readdirSync(skillsDir).filter(f => f.endsWith('.md'));
  for (const file of skillFiles) {
    const content = fs.readFileSync(path.join(skillsDir, file), 'utf-8');
    skillPrompts += '\n\n' + content;
  }
}
```

将 `skillPrompts` 追加到 system prompt 末尾。

### 第三步：部署 skill prompt 文件

为需要启用 Todo Agent 的群组，创建 skill 目录并放入 prompt 文件：

```bash
mkdir -p /workspace/group/.skills
cp todo-agent-prompt.md /workspace/group/.skills/todo-agent.md
```

如需为特定群组安装，替换路径为对应群组的 workspace 路径。

### 第四步：重新编译并重启 NanoClaw

```bash
cd /path/to/nanoclaw
npm run build
# 重启 NanoClaw 进程
```

### 第五步：初始化定时任务

重启后，在已安装 Todo Agent 的群组中发送以下消息触发初始化：

```
/todo init
```

或者直接由 Claude Code 通过 NanoClaw API 创建以下定时任务：

**每日早报**（每天 09:00，中国时区）：
```json
{
  "schedule_type": "cron",
  "schedule_value": "0 9 * * *",
  "prompt": "执行每日待办早报：读取 /workspace/group/data/todos.json，运行归档检查（将完成超过3天的任务移入 todos_archive.json，将超过3天未处理的 active 任务改为 sleeping 状态），然后按 todo-agent-prompt.md 中的「每日早报格式」发送今日待办摘要。"
}
```

**每周复盘**（每周日 20:00）：
```json
{
  "schedule_type": "cron",
  "schedule_value": "0 20 * * 0",
  "prompt": "执行每周待办复盘：读取 /workspace/group/data/todos.json 和 todos_archive.json，统计本周完成的任务数量，按 todo-agent-prompt.md 中的「每周复盘格式」发送本周小结。"
}
```

---

## 卸载

1. 删除 `/workspace/group/.skills/todo-agent.md`
2. 取消相关定时任务
3. （可选）保留或删除 `/workspace/group/data/todos*.json`

---

## 文件清单

```
nanoclaw-todo-agent/
├── SKILL.md                    # 本文件（安装说明）
├── README.md                   # 项目说明
├── todo-agent-prompt.md        # Agent 行为指令（核心）
└── templates/
    ├── todos.json              # 空任务列表模板
    └── todos_archive.json      # 空归档模板
```

---

## 更新日志

### v1.0.0
- 初始版本
- 自然语言任务识别
- 智能提醒判断（自动设置 / 主动询问）
- 每日早报 + 每周复盘
- 任务过载检测
- 3天自动归档已完成任务
