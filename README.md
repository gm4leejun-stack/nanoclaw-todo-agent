# 🤖 NanoClaw Todo Agent

> 智能待办 Agent skill for NanoClaw — 不是待办 App，是你的任务管家

## 功能特点

- **自然语言输入**：直接说话，无需记命令
- **AI 智能判断**：自动识别时间、判断紧急程度、主动询问提醒
- **主动提醒**：到点提醒、每日早报、逾期催促
- **任务过载检测**：任务堆积时主动帮你聚焦
- **群组支持**：每个群独立数据
- **低 token 消耗**：定时任务不消耗对话 token

## 使用方式

### 添加任务
直接发消息即可，Claude 自动判断是否需要提醒：
```
买牛奶
明天下午3点开会
记得交房租
给客户发报价单
```

### 查看任务
```
/t
```

### 完成/删除任务
```
完成1
删除2
```

### 其他
```
/t 已完成     查看已完成任务
/t 设置       查看设置项
```

## 安装

1. 克隆此仓库到 NanoClaw 项目目录
2. 使用 Claude Code 执行 `SKILL.md` 中的安装说明
3. 重启 NanoClaw

详见 [SKILL.md](./SKILL.md)

## 架构说明

```
用户消息（Telegram）
    ↓
Claude Agent（理解意图 + 智能判断）
    ↓
读写 /workspace/group/data/todos.json
    ↓
NanoClaw 定时任务（提醒，不消耗 token）
```

- 活跃任务：`todos.json`
- 已完成任务（3天后归档）：`todos_archive.json`
- 定时任务：每日早报（9:00）+ 逾期检查 + 单条任务提醒

## 兼容性

- NanoClaw core >= 1.2.0
- Telegram channel

## License

MIT
