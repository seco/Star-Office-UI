---
name: star-office-ui
description: 多 Agent 像素办公室看板：可视化状态、远端加入、昨日小记展示。用于部署、联调、接入与开源发布。
---

# Star Office UI Skill

## 目标

把 OpenClaw / AI 助手的协作状态可视化为“像素办公室”中的动态角色，支持：

1. 主 Agent 状态展示（idle / writing / researching / executing / syncing / error）
2. 多 Agent 远端加入与实时同步
3. 昨日小记展示（从 `memory/*.md` 提取）

---

## 核心能力

### A. 状态可视化
- 状态归一化：`working -> writing`，`sync -> syncing` 等
- 区域映射：
  - `idle -> breakroom`
  - `writing/researching/executing/syncing -> writing`
  - `error -> error`
- UI 动画：主角色 + 访客角色 + 状态气泡

### B. 多 Agent 协作
- `POST /join-agent`：加入办公室（基于 join key）
- `POST /agent-push`：持续推送状态
- `GET /agents`：前端拉取并渲染
- `POST /leave-agent`：离开与回收

### C. 昨日小记
- `GET /yesterday-memo` 从 `memory/` 中找昨日/最近日记
- 对展示文本做基础隐私清理（路径、ID、邮箱、IP 等）

---

## 目录与关键文件

- 后端：`backend/app.py`
- 前端：`frontend/index.html`、`frontend/layout.js`
- 主状态：`state.json`（运行时）
- 多 Agent 状态：`agents-state.json`（运行时）
- join key：`join-keys.json`
- 主状态切换：`set_state.py`
- 远端推送：`office-agent-push.py`

---

## 快速联调流程（10 分钟）

1. 启动服务：
   - `python3 -m pip install -r backend/requirements.txt`
   - `cd backend && python3 app.py`
2. 浏览器打开 `/`，确认 UI 可见
3. 本地切状态：`python3 set_state.py writing "联调中"`
4. 远端执行 join + push，确认访客进入工作区
5. 访问 `/yesterday-memo`，确认能返回摘要

---

## 常见问题

### 1) 访客一直在休息区
- 远端推送是否持续是 `idle`
- 远端是否读取错了状态源
- `/agent-push` 是否返回成功

### 2) join 失败（403 / 429）
- 403：join key 无效或不匹配
- 429：同 key 并发达到上限（默认 3）

### 3) 之前在线的 Agent 突然掉线
- 超过 5 分钟无推送会被标记 `offline`
- 恢复推送可自动回到 `approved`

---

## 开源发布注意事项

- 不提交运行时文件：`state.json`、`agents-state.json`、`*.log`、`*.out`、`*.pid`
- 不提交本地环境与缓存：`.venv/`、`__pycache__/`
- README 需写清楚素材版权与非商用限制
- 对外默认使用示例配置（`state.sample.json`）
