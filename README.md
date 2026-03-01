# Star Office UI

一个可视化 **AI 助手（OpenClaw）协作看板**：把龙虾们的实时状态渲染成像素办公室中的角色行为，支持多 Agent 加入、移动端查看、以及“昨日小记”展示。

## 项目实现了什么

Star Office UI 解决的是“多智能体协作过程看不见”的问题：

- 把抽象状态（`idle / writing / researching / executing / syncing / error`）转成可见位置和动画
- 让多个远端 OpenClaw 通过 join key 加入同一个办公室
- 用轻量后端管理状态流、授权、并发、离线回收
- 在 UI 中展示“昨日小记”（从 `memory/*.md` 提取可展示内容）

## 核心功能

### 1) 可视化龙虾工作状态

支持以下状态，并映射到办公室区域：

- `idle`：待命 / 休息区
- `writing`：工作区
- `researching`：工作区
- `executing`：工作区
- `syncing`：同步区（同属工作流）
- `error`：Bug 区

此外支持：
- 多 Agent 同屏渲染
- 状态气泡与随机思考文案
- 动画精灵角色与移动端浏览

### 2) 昨日小记（Yesterday Memo）

UI 会读取 `memory/` 下的日记文件，优先展示“昨天”的记录；若昨天缺失，会回退到最近可用日期。

后端提供接口：
- `GET /yesterday-memo`

用于在看板中展示昨日工作摘要（已做基础隐私清理）。

## 项目截图（可在此补充）

> 建议你把截图放到 `docs/screenshots/`，然后在这里引用。

示例：

```md
![主界面](docs/screenshots/office-main.png)
![多Agent状态](docs/screenshots/multi-agent.png)
![昨日小记](docs/screenshots/yesterday-memo.png)
```

## 架构概览

```text
star-office-ui/
  backend/
    app.py                # Flask API + 页面服务
    requirements.txt
    run.sh
  frontend/
    index.html            # 主 UI（Phaser）
    join.html             # 加入说明页面
    invite.html           # 邀请说明页面
    layout.js             # 场景布局
    ...assets
  docs/
    FEATURES_NEW_2026-03-01.md
    PROJECT_SUMMARY_2026-03-01.md
    STAR_OFFICE_UI_OVERVIEW.md
  office-agent-push.py    # 远端 agent 状态推送脚本
  set_state.py            # 本地主 agent 状态切换脚本
  state.sample.json       # 示例状态文件
  join-keys.json          # join key 配置（可复用）
  LICENSE
  SKILL.md
  README.md
```

## 快速开始

### 1) 安装依赖

```bash
cd star-office-ui
python3 -m pip install -r backend/requirements.txt
```

### 2) 准备状态文件（首次）

```bash
cp state.sample.json state.json
```

### 3) 启动后端

```bash
cd backend
python3 app.py
```

打开：
- `http://127.0.0.1:18791`

### 4) 切换主 Agent 状态

在项目根目录执行：

```bash
python3 set_state.py writing "正在整理文档"
python3 set_state.py syncing "同步数据中"
python3 set_state.py error "发现异常，排查中"
python3 set_state.py idle "待命中"
```

## 多 Agent 加入（简要）

- 远端 Agent 先调用 `/join-agent` 获取 `agentId`
- 然后周期调用 `/agent-push` 推送状态
- UI 通过 `/agents` 拉取并渲染

详细接入可参考：
- `frontend/join-office-skill.md`
- `office-agent-push.py`

## API（常用）

- `GET /health`：健康检查
- `GET /status`：主 agent 状态
- `POST /set_state`：设置主 agent 状态
- `GET /agents`：获取多 agent 列表
- `POST /join-agent`：加入办公室
- `POST /agent-push`：推送 agent 状态
- `POST /leave-agent`：离开办公室
- `GET /yesterday-memo`：读取昨日小记

## 开源与资产说明

- 代码遵循仓库 LICENSE（MIT）
- **美术素材版权归原作者/工作室所有**
- 本仓库素材仅用于学习与演示，**未经授权禁止商用**

## 安全建议

- 不要在 `detail` 中写入敏感信息
- 公网演示请加鉴权/网关限制
- `state.json` / `agents-state.json` 属于运行态文件，不建议提交
