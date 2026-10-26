# 灯泡开关 · FastAPI + Vue 2.0 全栈 VibeCoding 实战教程

> **写给谁**：第一次做全栈项目的开发者，会写一点 Python 和 HTML，但没把「前端 + 后端 + 数据库」串起来过。
>
> **你会做出什么**：一个能真实开关电灯的网页。点一下，灯亮，数据库留下记录，后端控制台打印一行日志，浏览器按 F5 刷新页面，灯还亮着。
>
> **怎么学**：本教程不背知识点，只背一个循环 —— **说需求 → AI 写 → 验收**。VibeCoding 的核心不是"让 AI 帮你写代码"，而是**你先想清楚验收标准，再让 AI 写**。

---

## 目录

- [第 0 章　先看一眼成品](#第-0-章先看一眼成品)
- [第 1 章　VibeCoding 到底是怎么回事](#第-1-章vibecoding-到底是怎么回事)
- [第 2 章　工具准备](#第-2-章工具准备)
- [第 3 章　项目结构总览](#第-3-章项目结构总览)
- [第 4 章　后端实战：FastAPI + SQLite](#第-4-章后端实战fastapi--sqlite)
- [第 5 章　接口调试：Thunder Client](#第-5-章接口调试thunder-client)
- [第 6 章　前端设计过程（Ardot + 自主设计）](#第-6-章前端设计过程ardot--自主设计)
- [第 7 章　前端实现：Vue 2.0 + Axios](#第-7-章前端实现vue-20--axios)
- [第 8 章　前后端联调与跨域](#第-8-章前后端联调与跨域)
- [第 9 章　提示词清单（可直接复制）](#第-9-章提示词清单可直接复制)
- [第 10 章　常见报错速查表](#第-10-章常见报错速查表)
- [第 11 章　验收总表与课后练习](#第-11-章验收总表与课后练习)

---

## 第 0 章　先看一眼成品

### 0.1 最终效果

打开 `FrontEnd/index.html`，你会看到一个卡片：

- 上半部分是「房间」：一个吊线灯泡。**关着的时候**房间是深灰蓝 `#232a38`，灯泡玻璃暗沉、灯丝不发光；**开着的时候**房间变成暖黄渐变，灯泡玻璃透亮 `#fff6dc`，灯丝发出橙光 `#ff9d2e`，灯泡周围有径向光晕、七条光芒射线、地面还有一圈椭圆光圈。
- 下半部分是控制区：状态文案、胶囊开关、亮度滑杆、今日次数、**查询服务器状态按钮**。
- 鼠标点灯泡、点开关、按空格键，都能开关。

### 0.2 技术栈一览

| 层 | 选型 | 版本 | 为什么选它 |
|---|---|---|---|
| 后端框架 | FastAPI | ≥0.115 | 自动生成 API 文档，代码量少 |
| 后端语言 | Python | 3.12.14 | 由 uv 创建的虚拟环境 |
| 数据库 | SQLite | Python 标准库 `sqlite3` | 零配置，存成一个文件，不用装服务 |
| 包管理 | uv | 0.12.6 | 比 pip 快一个数量级 |
| 接口调试 | Thunder Client | VS Code 插件 | 免装 Postman，编辑器里直接发请求 |
| 前端框架 | Vue | 2.7.16（CDN） | 符合题目要求；2.x 已停止维护，仅用于学习 |
| HTTP 客户端 | Axios | 1.7.9（CDN） | 浏览器里发请求的事实标准 |

> ⚠️ **关于 Vue 2.0**：Vue 2 已于 2023 年 12 月 31 日停止官方维护（EOL）。这里按题目要求使用 Vue 2 学习，它的语法思想在 Vue 3 里依然成立（`data` / `computed` / `methods` 三大块完全一样），只是模板语法细节和 CDN 引入方式有差别。**真实的新项目请直接上 Vue 3。**

---

## 第 1 章　VibeCoding 到底是怎么回事

### 1.1 一句话定义

**VibeCoding = 你负责定义"做成什么样算对"，AI 负责写"怎么做到"。**

名字来自 2025 年初 Andrej Karpathy 的一条推文，大意是：新一代 AI 编程工具让人可以"靠感觉编程"（vibe coding），不再手写每一行代码，直接对 AI 描述需求，让它生成。

### 1.2 初学者最大的误解

新手常见的错误姿势是：

```
❌ 错误：帮我写一个 FastAPI 项目，要什么功能你自己看着办
```

AI 会给你一堆花哨但完全不是你想要的东西。因为你没有告诉它**判断对错的标准**。

正确的姿势是**先写验收标准，再让 AI 动手**：

```
✅ 正确：
   我要做一个电灯开关后端。
   【必须满足】
   1. POST /api/light，body 传 {"action":"On"}，写入 SQLite 并记录时间
   2. 同一次操作要打印到控制台，且追加到 logs/app.log
   3. GET /api/light/status 返回当前状态，状态 = 最后一条记录的 action
   4. action 只能是 On 或 Off，别的值返回 422，且不写数据库
   5. 不要引入 ORM，只用标准库 sqlite3
   【技术约束】
   Python 3.12，FastAPI，只依赖 fastapi + uvicorn
   【验收】
   用 curl 能打出 {"is_on":true}，light.db 里有对应行，app.log 里有对应行
```

**区别在于**：第二种写法里，AI 生成的代码你可以直接对着验收标准逐条检查，而不是靠"看着像对的"。

### 1.3 本教程的节奏

每一章都是同一个四步循环：

```
   ┌──────────────┐
   │ 1. 想清楚    │  先明确这一章要做成什么样
   └──────┬───────┘
          ▼
   ┌──────────────┐
   │ 2. 给提示词  │  带着约束和验收标准去问 AI
   └──────┬───────┘
          ▼
   ┌──────────────┐
   │ 3. 验收      │  逐条打勾，不通过就改
   └──────┬───────┘
          │ 不通过
          └────────► 回到第 1 步，说清哪里不对
```

---

## 第 2 章　工具准备

### 2.1 需要的工具清单

| 工具 | 是否必装 | 检查命令 |
|---|---|---|
| Visual Studio Code | ✅ 必装 | — |
| VS Code 插件：Python | ✅ 必装 | 编辑器扩展面板搜索 |
| VS Code 插件：Thunder Client | ✅ 必装 | 编辑器扩展面板搜索 |
| Python（由 uv 管理） | ✅ 必装 | `uv python list` |
| uv | ✅ 必装 | `uv --version` |
| Ardot（可选） | 想做设计稿才装 | 见 `Design-Ardot/` |

### 2.2 安装 uv

uv 是 Rust 写的 Python 包管理器，比 pip 快很多，并且能自己下载管理 Python 版本 —— 你不需要单独装 Python。

**Windows（PowerShell 或 CMD）**：

```powershell
powershell -ExecutionPolicy ByPass -c "irm https://astral.sh/uv/install.ps1 | iex"
```

**macOS / Linux**：

```bash
curl -LsSf https://astral.sh/uv/install.sh | sh
```

验证：

```bash
uv --version
# uv 0.12.6 (7938ca5d5 2026-08-25 x86_64-pc-windows-msvc)
```

### 2.3 让 uv 装一个 Python 3.12

```bash
uv python install 3.12
uv python list
```

你会看到类似输出，`download available` 变成路径就说明装好了：

```
cpython-3.12.14-windows-x86_64-none   C:\Users\<你>\AppData\Roaming\uv\python\cpython-3.12.14-windows-x86_64-none\python.exe
```

> 💡 **为什么锁定 3.12？** 新手不要追新版本。3.12 生态最稳、教程最多。而 3.13/3.14 的一些第三方库还没跟上，容易踩坑。

### 2.4 VS Code 设置建议

按下 `Ctrl + ,` 打开设置，搜索并修改这几项：

| 设置项 | 改成 | 原因 |
|---|---|---|
| `python.defaultInterpreterPath` | `Backend/.venv/Scripts/python.exe` | VS Code 自动用虚拟环境，代码提示才不会飘红 |
| `python.terminal.activateEnvironment` | `true` | 新开终端自动激活 venv，命令行前出现 `(bulbswitch)` |
| `files.autoSave` | `afterDelay` | 忘记保存导致调试白忙一场 |
| `editor.formatOnSave` | `true` | 保持代码整洁 |

再装两个推荐的免费插件（可选）：

- **Chinese (Simplified) 中文语言包** —— 让 VS Code 变中文
- **Error Lens** —— 把报错直接显示在代码行尾，不用鼠标悬停去看

### 2.5 ⚠️ 一个必须知道的坑：别把 `.venv` 提交到 Git

`.venv` 里有几万个文件，是**你本机生成的**，换台电脑要重新装，绝不能提交到版本库。

在项目根目录建一个 `.gitignore`：

```gitignore
# Python
__pycache__/
*.py[cod]
.venv/
venv/

# 运行时产物
Backend/light.db
Backend/logs/

# 编辑器
.vscode/
.DS_Store
```

> **验收条件 2.1**：
> - [ ] `uv --version` 能打印版本号
> - [ ] `uv python list` 里 3.12 那行后面是路径而不是 `download available`
> - [ ] VS Code 里 Python 插件已安装且无报错
> - [ ] Thunder Client 插件已安装
> - [ ] 项目根目录已有 `.gitignore` 且包含 `.venv/`

---

## 第 3 章　项目结构总览

### 3.1 目录结构

```
BulbSwitch/
├── README.md                          ← 你正在读的这份教程
├── .gitignore                         ← 第 2.5 节创建
│
├── Design-Ardot/                      ← 设计资源（只读，不改）
│   ├── Ardot接入WorkBuddy操作指南.md    ← Ardot 完整操作手册
│   └── lightbulb-switch/
│       └── index.html                 ← Ardot 导出的原始静态页面（对照用）
│
├── Backend/                           ← 后端
│   ├── .venv/                         ← uv 创建的虚拟环境（Python 3.12.14）
│   ├── main.py                        ← 后端全部代码，单文件
│   ├── requirements.txt               ← 依赖清单
│   ├── light.db                       ← SQLite 数据库（运行后生成）
│   └── logs/
│       └── app.log                    ← 日志文件（运行后生成）
│
└── FrontEnd/                          ← 前端
    └── index.html                     ← Vue 2.0 + Axios 单页
```

### 3.2 请求与响应全流程

理解这张图比记住任何语法都重要：

```
   浏览器 (FrontEnd/index.html)
        │
        │  用户点击开关
        ▼
   ┌──────────────────────────────────────┐
   │ Vue 组件                                │
   │  data.on = true                ← 先本地响应 │
   │  调 sendAction('On')                   │
   └───────────────┬──────────────────────┘
                   │  axios.post('/api/light', {action:'On'})
                   │  POST http://127.0.0.1:8000/api/light
                   ▼
   ┌──────────────────────────────────────┐
   │ FastAPI (Backend/main.py)│
   │  ① Pydantic 校验 action ∈ {On, Off}        │  ← 不合法直接 422，不落库
   │  ② INSERT INTO light_records              │
   │  ③ logger.info(...)  → 控制台 + app.log   │
   │  ④ 返回 { id, action, created_at, source } │
   └───────────────┬──────────────────────┘
                   ▼
              light.db（多了 1 行）
```

反向查询时：

```
   页面加载 / 点击「查询服务器当前状态」
        │
        │  axios.get('/api/light/status')
        ▼
   ┌──────────────────────────────────────┐
   │ SELECT ... ORDER BY id DESC LIMIT 1        │  ← 最后一条即当前状态
   │ 返回 { is_on, action, updated_at, ... }     │
   └───────────────┬──────────────────────┘
                   ▼
   Vue 更新 data.on → 页面灯泡重新渲染
```

### 3.3 核心设计决策：为什么"当前状态"不单独存？

很多新手会犯这个错：在数据库里建两个表，`light_state` 存当前状态，`light_logs` 存历史。然后某次忘了更新 `light_state`，于是**状态表说灯亮着，日志表说最后关的**，数据打架了。

本项目的做法是：**只有一张表，当前状态 = 最后一条记录的 action**。

```
  id | action | created_at          当前状态 = ORDER BY id DESC LIMIT 1
----+--------+---------------------  =  id=5 那一行 = Off
  1 | On     | 2026-10-15 09:00:00
  2 | Off    | 2026-10-15 09:05:12
  3 | On     | 2026-10-15 10:22:40
  4 | Off    | 2026-10-15 10:31:03
  5 | Off    | 2026-10-15 14:02:55  ← 取这一行
```

**只有一份真相，不可能出现矛盾。** 这是本教程最想让你带走的设计思想。

---

## 第 4 章　后端实战：FastAPI + SQLite

### 4.1 第 1 步：创建虚拟环境

在 VS Code 里打开 `BulbSwitch` 文件夹 → 终端（`` Ctrl+` ``）：

```bash
cd Backend
uv venv --python 3.12 .venv
```

输出：

```
Using CPython 3.12.14
Creating virtual environment at: .venv
Activate with .venv\Scripts\activate
```

**这行输出每个参数的含义**：

| 部分 | 含义 |
|---|---|
| `Using CPython 3.12.14` | uv 用了3.12.14 这个解释器 |
| `Creating virtual environment at: .venv` | 虚拟环境建在当前目录的 `.venv` 文件夹 |
| `Activate with .venv\Scripts\activate` | 激活命令（可选，见下） |

> 💡 **虚拟环境是什么？** 想象每个项目有自己的「工具箱」。项目 A 需要 fastapi 2.0，项目 B 需要 3.0，互相不打架。`.venv` 就是项目 A 的专属工具箱。
>
> **要不要激活？** 两种方式：
> - **方式 A（推荐新手）**：不激活，每次命令前面加 `.venv\Scripts\` 前缀
>   ```bash
>   .venv\Scripts\python.exe -m uvicorn main:app --reload
>   ```
> - **方式 B**：激活后命令更短
>   ```bash
>   .venv\Scripts\activate
>   # 之后直接用 python / uvicorn
>   ```
>   VS Code 用户其实不用管激活 —— 第 2.4 节配好了 `python.defaultInterpreterPath`，编辑器会自动用对。

### 4.2 第 2 步：安装依赖

```bash
uv pip install "fastapi>=0.115" "uvicorn[standard]>=0.32"
```

**为什么不写 requirements.txt 再安装？** 因为 uv 比 pip 快，直接装最省时间；装完再导出清单用于备份。

导出清单：

```bash
uv pip freeze > requirements.txt
```

本项目实际装到的版本（可作为参考）：

```
fastapi==0.143.1
uvicorn==0.54.0
pydantic==2.14.1
starlette==1.8.0
...
```

**只有两个直接依赖**：`fastapi` 和 `uvicorn`。其余 17 个是它们自己拖下来的传递依赖。`sqlite3` 和 `logging` 是 Python 标准库，不需要装。

> 💡 **`uvicorn[standard]` 的方括号是什么？** 「可选依赖组」。`standard` 组包含性能相关的包（uvloop、httptools、watchfiles）。其中 **`watchfiles` 很关键：它就是让 `--reload` 文件热重载变快的那个包**。不装也能跑，但改代码重启会慢一点。

### 4.3 第 3 步：给 AI 的提示词（生成后端代码）

这一步你**不用手写代码**，让 AI 写。但要给出明确约束。

```
在 Backend/main.py 写一个 FastAPI 后端，用于管理一个智能电灯的使用记录。

【场景】
模拟管理一盏智能电灯。每次开关都要留下带时间戳的记录。

【必须实现的接口】
1. POST /api/light
   - 请求体 JSON：{"action": "On" | "Off"}
   - 行为：写入 SQLite，同时记录当前时间
   - 同一次操作必须做两件事：
     a) 打印到控制台
     b) 追加到日志文件 Backend/logs/app.log
   - 返回：{"id": int, "action": str, "created_at": str, "source": str}

2. GET /api/light/status
   - 返回：{"is_on": bool, "action": str, "updated_at": str, "record_id": int, "total_records": int}
   - 当前状态 = 数据库里最后一条记录的 action（不要单独存状态字段）
   - 数据库为空时返回 is_on=false

【技术约束】
- Python 3.12
- 只依赖 fastapi + uvicorn，不要引入 SQLAlchemy 等 ORM
- 数据库用标准库 sqlite3，文件放在 Backend/light.db
- action 用 Literal["On","Off"] 做校验，传错值自动返回 422 且不写数据库
- 时间格式固定为 "%Y-%m-%d %H:%M:%S"
- 用 pathlib 构造绝对路径，不要用相对路径
- 开启 CORS，允许所有来源（这是教学项目）

【代码质量要求】
- 每一节加中文注释，说明"为什么这么写"而不只是"这是什么"
- logging 用 RotatingFileHandler（maxBytes=1MB, backupCount=3）
- 用 logger.info("...%s", var) 而不是 f-string 或 % 格式化
  （说明原因：logging 是惰性求值，被过滤掉的日志不做无意义的字符串拼接）
- FastAPI 的 lifespan 里建表，不要在每个请求里建
- 每条 SQL 用参数化查询（?占位符），不要字符串拼接——防 SQL 注入

【验收标准】
1. curl -X POST http://127.0.0.1:8000/api/light -H "Content-Type: application/json" -d "{\"action\":\"On\"}" 返回 200 且含 id 和 created_at
2. 控制台出现"收到控制指令"日志行
3. Backend/logs/app.log 里有一行时间格式的日志
4. 用 SQLite 打开 light.db，light_records 表里多了一行
5. GET /api/light/status 返回 is_on=true
6. 传 {"action":"open"} 返回 422，且数据库不新增行
7. GET / 后能看到接口清单
```

AI 生成后，**逐条对着验收标准检查**，而不是看一眼觉得"对"。

### 4.4 第 4 步：读懂 main.py

`main.py` 是单文件，按 7 节组织。下面逐节讲**为什么**这么写。

#### 4.4.1 路径与常量

```python
from pathlib import Path

BASE_DIR = Path(__file__).resolve().parent
DB_PATH = BASE_DIR / "light.db"
LOG_PATH = BASE_DIR / "logs" / "app.log"
TIME_FORMAT = "%Y-%m-%d %H:%M:%S"
Action = Literal["On", "Off"]
```

**关键点**：`Path(__file__).resolve().parent` 得到的是**这个文件所在目录的绝对路径**。

为什么不用相对路径？相对路径是相对于「你执行命令时所在的目录」。如果你在 `BulbSwitch` 根目录启动服务，而代码在 `Backend/main.py`，那 `light.db` 就会跑到 `BulbSwitch/` 下面去 —— 到处散落，且换个启动目录就找不到。**用绝对路径就没有这个问题。**

**`Literal["On","Off"]`** 是给类型检查器（也就是编辑器）看的。写上之后：
- 编辑器会自动补全 `"On"` 和 `"Off"`
- 传错值 FastAPI 自动返回 422，不用自己写 `if action not in ...`

#### 4.4.2 配置日志

```python
def setup_logger() -> logging.Logger:
    LOG_DIR.mkdir(parents=True, exist_ok=True)
    logger = logging.getLogger("light")
    logger.setLevel(logging.INFO)

    logger.handlers.clear()          # ← 关键！

    file_handler = RotatingFileHandler(
        LOG_PATH, maxBytes=1024*1024, backupCount=3, encoding="utf-8")
    console_handler = logging.StreamHandler()
    # ... 设置格式并 addHandler
    return logger
```

**为什么要自己写这段？** uvicorn 自带的日志只有「谁访问了哪个 URL」这类访问日志，不包含你的业务信息。「用户把灯关了」这种业务日志必须自己写。

**两个 Handler 的分工**：

| Handler | 去向 | 格式 |
|---|---|---|
| `StreamHandler` | 控制台 | `2026-10-15 14:02:55  收到控制指令 \| ...` |
| `RotatingFileHandler` | `logs/app.log` | `2026-10-15 14:02:55 \| INFO    \| light \| ...` |

控制台用简洁格式（时间 + 消息），文件用完整格式（含级别和 logger 名）。因为控制台要给人看，文件是给排查问题用的，要信息全。

**`logger.handlers.clear()` 这一行为什么必须有？**

`--reload` 模式下，改一次代码 uvicorn 会重新导入本模块。如果不先清空，handlers 会一次次累加，变成打印 3 遍、5 遍。这是新手最常见的日志问题。

**为什么用 `RotatingFileHandler` 不用 `FileHandler`？**

`FileHandler` 会无限往一个文件里追加，跑一个月可能变成几百 MB。`RotatingFileHandler` 超过 `maxBytes`（本项目 1MB）就自动切一个备份文件，`backupCount=3` 表示最多留 3 个备份，旧的自动删。

**日志格式里的 `%s` 写法（重点）**：

```python
# ✅ 正确
logger.info("收到控制指令 | action=%s | time=%s", action, now)

# ❌ 错误：f-string
logger.info(f"收到控制指令 | action={action} | time={now}")

# ❌ 错误：% 格式化
logger.info("收到控制指令 | action=%s | time=%s" % (action, now))
```

原因：logging 是**惰性求值**的 —— 只有这条日志真的会被输出时，才去格式化字符串。如果日志级别设成 WARNING 而你打的是 INFO，f-string 版本仍然会白白拼好字符串再丢掉，白费性能。而且 f-string 在多线程下有隐蔽的顺序错乱问题。

#### 4.4.3 数据库

```python
def get_db() -> sqlite3.Connection:
    conn = sqlite3.connect(DB_PATH)
    conn.row_factory = sqlite3.Row
    return conn
```

**为什么每次请求都新建连接，不用全局变量？**

SQLite 的连接对象**不能被多个线程同时使用**。FastAPI 的接口跑在线程池里，一个全局连接用着用着就会报 `ProgrammingError: SQLite objects created in a thread can only be used in that same thread`。

每次新建、用完就关，是最简单可靠的做法。SQLite 建连接极快（微秒级），性能完全够用。

**`row_factory = sqlite3.Row` 的好处**：

```python
# 不设的话
row[0]      # 靠位置，字段一改顺序就错
row["action"]

# 设了之后
row["action"]     # 靠名字，一目了然
```

#### 建表语句

```python
CREATE TABLE IF NOT EXISTS light_records (
    id         INTEGER PRIMARY KEY AUTOINCREMENT,
    action     TEXT    NOT NULL,
    created_at TEXT    NOT NULL,
    source     TEXT    NOT NULL DEFAULT 'unknown'
)
```

| 字段 | 类型 | 说明 |
|---|---|---|
| `id` | INTEGER PK AUTOINCREMENT | 自增流水号。用 AUTOINCREMENT 而不是单纯 INTEGER PRIMARY KEY，是为了保证删除记录后 id 不会重复复用 |
| `action` | TEXT NOT NULL | `'On'` 或 `'Off'`，不允许空 |
| `created_at` | TEXT NOT NULL | 操作时间 |
| `source` | TEXT NOT NULL DEFAULT | 来源标记，`thunder-client` / `frontend` / `manual`，排查问题时能立刻知道是谁操作的 |

```python
CREATE INDEX IF NOT EXISTS idx_created_at ON light_records(created_at)
```

**索引是什么？** 像书的目录。没有索引，数据库要查「最新一条」就得逐行扫描全部数据；有索引，直接翻目录。数据量小的时候没区别，但这是**从第一天就建立好习惯**的地方 —— 等你有 100 万条记录再想起加索引，就晚了。

#### lifespan 建表

```python
@asynccontextmanager
async def lifespan(app: FastAPI):
    init_db()      # 启动时执行一次
    yield# 服务运行期间停在这里
    logger.info("服务已关闭")
```

为什么不在接口里建表？因为接口可能被调用一万次，建表只需要一次。

#### 4.4.4 请求与响应结构（Pydantic）

```python
class LightActionRequest(BaseModel):
    action: Action = Field(..., description="开灯传 On，关灯传 Off",
                           examples=["On"])
    source: str = Field(default="manual", description="操作来源，可忽略")
```

`...`（Ellipsis）是 FastAPI 的惯用写法，意思是「这个字段必填」。

`description` 和 `examples` 不是装饰，它们会出现在 `/docs` 文档里 —— 这就是 FastAPI 的核心价值：**你写的类型定义，自动变成别人能看懂能试的 API 文档**。

#### 4.4.5 CORS 跨域

```python
app.add_middleware(
    CORSMiddleware,
    allow_origins=["*"],
    allow_credentials=False,
    allow_methods=["*"],
    allow_headers=["*"],
)
```

四个参数的含义：

| 参数 | 作用 |
|---|---|
| `allow_origins` | 允许哪些网页来源。当前是 `["*"]` = 全部 |
| `allow_methods` | 允许哪些 HTTP 方法。GET/POST/PUT/DELETE |
| `allow_headers` | 允许请求头携带什么。`Content-Type` 需要 |
| `allow_credentials` | 是否允许带 Cookie。**注意 `["*"]` 和 `True` 不能共存** |

**为什么需要 CORS？** 你的页面从 `file://` 或某个端口打开，接口在另一个端口。浏览器认为这是两个不同的「源」（源 = 协议 + 域名 + 端口，三者任一不同就是不同源）。跨源请求浏览器默认拦截，**即使服务器愿意配合**，浏览器也不发 —— 这叫「同源策略」，是浏览器的安全底线。CORS 头是服务器在说「我允许你来」。

> ⚠️ `allow_origins=["*"]` 只用于教学。生产环境务必改成具体域名。

#### 4.4.6 四个接口的实现

**接口 1：POST /api/light（控制电灯）**

```python
@app.post("/api/light", response_model=LightRecordOut)
def control_light(payload: LightActionRequest) -> LightRecordOut:
    action = payload.action
    now = datetime.now().strftime(TIME_FORMAT)

    with get_db() as conn:
        cursor = conn.execute(
            "INSERT INTO light_records (action, created_at, source) VALUES (?, ?, ?)",
            (action, now, payload.source),
        )
        conn.commit()
        record_id = cursor.lastrowid

    logger.info("收到控制指令 | action=%s | time=%s | source=%s | record_id=%s",
                action, now, payload.source, record_id)
    logger.info("电灯已%s", "打开" if action == "On" else "关闭")

    return LightRecordOut(id=record_id, action=action,
                          created_at=now, source=payload.source)
```

**三个关键点**：

1. **参数化查询**：`VALUES (?, ?, ?)` + `(action, now, payload.source)`。**永远不要用 f-string 或 `%` 拼 SQL**：
   ```python
   # ❌ SQL 注入漏洞：用户传 action = "On'; DROP TABLE light_records; --" 就把表删了
   conn.execute(f"INSERT INTO ... VALUES ('{action}', ...)")
   ```
   参数化查询让数据库把 `action` 永远当**数据**处理，永远不当**代码**执行。

2. **`with get_db() as conn:`** 上下文管理器，`with` 块结束时自动提交或回滚，异常时也不会让连接泄漏。

3. **`logger.info("电灯已%s", "打开" if action == "On" else "关闭")`** —— 把逻辑判断放进日志的参数里，日志文本本身保持不变。日志最好写成可被机器检索的固定格式。

**接口 2：GET /api/light/status（查询状态）**

```python
with get_db() as conn:
    total = conn.execute("SELECT COUNT(*) AS c FROM light_records").fetchone()["c"]
    latest = conn.execute(
        "SELECT id, action, created_at FROM light_records ORDER BY id DESC LIMIT 1"
    ).fetchone()

if latest is None:
    return LightStatusOut(is_on=False, action="Off",
                          updated_at=None, record_id=None, total_records=0)

return LightStatusOut(
    is_on=latest["action"] == "On",
    action=latest["action"],
    updated_at=latest["created_at"],
    record_id=latest["id"],
    total_records=total,
)
```

`ORDER BY id DESC LIMIT 1` 就是「按 id 倒序取第一条」= **最新一条**。这就是第 3.3 节说的「当前状态 = 最后一条记录」。

数据库为空时返回 `is_on=False`，符合直觉：没开过灯就是关着。

**接口 3、4** 同理，`records` 是查历史（带 `limit` 参数校验，防止一次拉太多），`stats` 是统计今日次数（用 `LIKE '2026-10-15%'` 前缀匹配当天）。

### 4.5 第 5 步：启动服务

```bash
cd Backend
.venv\Scripts\python.exe -m uvicorn main:app --reload --port 8000
```

**把这条命令拆开看**：

| 部分 | 含义 |
|---|---|
| `.venv\Scripts\python.exe` | 用虚拟环境里的 Python（不要直接写 `python`，可能用到系统版本） |
| `-m uvicorn` | 用 Python 运行 uvicorn 模块 |
| `main:app` | **冒号是分界线**：左边 `main` 是文件名（对应 `main.py`），右边 `app` 是文件里的变量名（对应 `app = FastAPI(...)`） |
| `--reload` | 代码改了自动重启，开发必开 |
| `--port 8000` | 监听 8000 端口 |

**常见的启动失败原因**：

| 报错 | 原因 | 解决 |
|---|---|---|
| `No module named uvicorn` | 用了系统 Python，不是虚拟环境的 | 命令里写全路径 `.venv\Scripts\python.exe` |
| `Address already in use` | 8000 端口被占用 | `--port 8001`，同时改前端 `API_BASE` |
| `Error loading ASGI app` | `main:app` 写错了 | 确认在 `Backend` 目录下执行 |

### 4.6 自动生成的 API 文档（FastAPI 的杀手锏）

启动成功后，浏览器打开：

| 地址 | 内容 |
|---|---|
| `http://127.0.0.1:8000/docs` | **Swagger UI**，可视化文档，可以直接在网页上发请求测试 |
| `http://127.0.0.1:8000/redoc` | ReDoc，更漂亮的阅读版文档 |
| `http://127.0.0.1:8000/` | 本项目的根路径，返回接口清单 |

`/docs` 里能看到每个接口的参数表和响应模型 —— 这些都是从第 4.4.4 节那些类定义自动生成的。**这就是为什么 FastAPI 值得学：写好类型，文档就有了。**

### 4.7 后端验收条件

服务启动后，逐条打勾：

| # | 验收条件 | 怎么验 | 通过 |
|---|---|---|---|
| B1 | 服务正常启动 | 终端显示 `Uvicorn running on http://127.0.0.1:8000` | ☐ |
| B2 | 数据库文件生成 | `Backend/light.db` 存在 | ☐ |
| B3 | 表结构正确 | 用 SQLite 工具打开，`light_records` 表有 id/action/created_at/source 四列 | ☐ |
| B4 | 根路径可访问 | 浏览器开 `http://127.0.0.1:8000/`，返回接口清单 JSON | ☐ |
| B5 | 文档页可访问 | 浏览器开 `http://127.0.0.1:8000/docs`，能看到 4 个接口 | ☐ |
| B6 | 控制台打印 | 发一次 POST，终端出现「收到控制指令」行 | ☐ |
| B7 | 日志落盘 | `Backend/logs/app.log` 里出现同一时间戳的行 | ☐ |
| B8 | 参数校验生效 | 发 `{"action":"open"}` 返回 **422**，且数据库**不新增行** | ☐ |
| B9 | 空库默认状态 | 数据库为空时 GET status 返回 `is_on: false` | ☐ |
| B10 | 状态取最新 | 依次发 On→Off→On，GET status 返回 `is_on: true` | ☐ |

> 💡 **B8 是最重要的一条。** 它验证了"校验发生在写库之前"。很多初学者的代码是「先写库再校验」，或者压根没校验，导致数据库里出现 `"ON"`、`"open"`、`"1"` 各种脏数据。**校验必须在最前面。**

---

## 第 5 章　接口调试：Thunder Client

### 5.1 为什么用 Thunder Client 而不是 Postman？

- **不用装** —— VS Code 插件，装完即用
- **不用切窗口** —— 写代码和调接口在同一个界面
- **能存成文件** —— 请求保存为 `light.http`，跟着项目走，换电脑还在
- **免费够用** —— 完全够做本教程的所有调试

### 5.2 安装与基本操作

1. VS Code 扩展面板（`` Ctrl+Shift+X ``）→ 搜索 `Thunder Client` → Install
2. 快捷键 `` Ctrl+Alt+R `` 打开 Thunder Client 面板
3. 点左上角 `+` 新建请求，请求会保存成 `.http` 文件

### 5.3 环境变量（先做这一步，省掉后面一堆麻烦）

在 Thunder Client 面板点选请求旁边的 **`Env`**（或右键 → `Manage Environment`），新建一个环境 `local`，加两个变量：

| 变量名 | 值 | 用途 |
|---|---|---|
| `baseUrl` | `http://127.0.0.1:8000` | 后端地址 |

以后所有请求里写 `{{baseUrl}}`。换端口时**只改这一个地方**，所有请求自动跟着变 —— 这是很基础但很有用的习惯。

### 5.4 三个接口的完整配置

#### 请求 1：开灯

- **Method**：`POST`
- **URL**：`{{baseUrl}}/api/light`
- **Body** → 选 `raw` → 格式选 `JSON`（不是 Form、不是 x-www-form-urlencoded）：

```json
{
  "action": "On",
  "source": "thunder-client"
}
```

- **Headers**（Body 选 raw JSON 时必须手动加）：

```
Content-Type: application/json
```

点 **Send**，预期响应：

```json
{
  "id": 1,
  "action": "On",
  "created_at": "2026-10-15 14:02:55",
  "source": "thunder-client"
}
```

#### 请求 2：关灯

复制请求 1，只改 Body 的 `"action": "Off"`。

#### 请求 3：查询当前状态

- **Method**：`GET`
- **URL**：`{{baseUrl}}/api/light/status`
- 无 Body，无 Headers

预期响应：

```json
{
  "is_on": true,
  "action": "On",
  "updated_at": "2026-10-15 14:02:55",
  "record_id": 1,
  "total_records": 1
}
```

#### 请求 4（选做）：查历史

- **Method**：`GET`
- **URL**：`{{baseUrl}}/api/light/records?limit=5`

#### 请求 5（选做）：看统计

- **Method**：`GET`
- **URL**：`{{baseUrl}}/api/light/stats`

点 Thunder Client 的 **Save As**，把这批请求保存为 `Backend/light.http`。

### 5.5 校验参数错误的返回（重点）

故意发一个错误请求，把 Body 改成：

```json
{
  "action": "open"
}
```

预期：**422 Unprocessable Entity**

```json
{
  "detail": [
    {
      "type": "literal_error",
      "loc": ["body", "action"],
      "msg": "Input should be 'On' or 'Off'",
      "input": "open",
      "ctx": {"expected": "'On' or 'Off'"}
    }
  ]
}
```

**怎么读这个错误**：

| 字段 | 含义 |
|---|---|
| `type` | 错误类型，`literal_error` = 不在允许的字面量列表里 |
| `loc` | 出错位置：`["body", "action"]` = 请求体的 action 字段 |
| `msg` | 人类可读的说明 |
| `input` | 你实际传的值 |
| `ctx.expected` | 期望的值 |

**然后立刻查数据库，确认没有新增行。** 这一步不能省 —— 它验证了「校验在写库之前」。

### 5.6 Thunder Client 验收条件

| # | 验收条件 | 怎么验 | 通过 |
|---|---|---|---|
| T1 | 三个请求已保存 | `Backend/light.http` 存在，含 3+ 个请求 | ☐ |
| T2 | 环境变量生效 | 所有请求用 `{{baseUrl}}`，无硬编码 IP | ☐ |
| T3 | 开灯成功 | POST `{"action":"On"}` 返回 200，body 含 `id` 和 `created_at` | ☐ |
| T4 | 关灯成功 | POST `{"action":"Off"}` 返回 200，body 的 `action` 是 `Off` | ☐ |
| T5 | 查询正确 | GET status 的 `action` 等于最后一次 POST 的值 | ☐ |
| T6 | **控制台有输出** | 终端出现 `收到控制指令 \| action=On \| time=... \| source=thunder-client` | ☐ |
| T7 | **日志文件有输出** | `Backend/logs/app.log` 内容与控制台时间戳一致 | ☐ |
| T8 | 校验生效 | POST `{"action":"open"}` 返回 422，body 的 `loc` 指向 `body.action` | ☐ |
| T9 | **校验未污染数据库** | T8 之后查 `records?limit=5`，条数不变 | ☐ |
| T10 | 序号自增 | 连续发 3 次，`id` 依次为 1、2、3 | ☐ |

> 💡 **T6 和 T7 是本教程的核心验收点。** 你的题目明确要求「在控制台打印传入参数和把时间记录在日志文件中」，这两条就是直接对应的验证。一定要**打开日志文件亲眼看**，而不是觉得"代码写了就等于生效了"。

---

## 第 6 章　前端设计过程（Ardot + 自主设计）

这一章讲**设计稿是怎么来的**。有两种路线：

- **路线 A**：用 Ardot 一句话生成设计稿，再转成代码（快，风格统一）
- **路线 B**：自己画 / 自己描述（慢，但完全可控）

两条路线都给你现成的提示词。

### 6.1 路线 A：用 Ardot 生成设计稿

完整操作步骤见 **`Design-Ardot/Ardot接入WorkBuddy操作指南.md`**，这里只讲设计**思路**。

#### 6.1.1 核心链路

```
   自然语言需求
        │
        ▼
   ┌─────────────────────────┐
   │  Ardot MCP Server        │  ← WorkBuddy 通过 MCP 协议调用
   │  （18 个工具）             │
   └────────────┬────────────┘
                │
   ┌────────────▼────────────────────────────┐
   │ search_style_guide  找配色/字体风格候选   │
   │ build_style_guide   构建设计系统          │
   │ fetch_guidelines    取 web-app 设计规范    │
   │ locate_available_space  在画布找空白      │
   │ batch_edit          批量插入节点搭结构     │
   │ capture_screenshot  截图自查，不对再改     │
   └────────────┬────────────────────────────┘
                ▼
        .ardot 可编辑设计稿
       （带图层、组件、样式，不是死图）
                │
                │ fetch_variables  读设计变量（Design Token）
                │ batch_read       读图层树、坐标、色值
                ▼
          HTML / CSS / JS
```

**为什么 Ardot 生成的代码还原度高？** 因为它输出的是**带图层结构的可编辑文件**，AI 能读回图层名、坐标、间距、色值，而不是只对着截图猜。「截图识图转代码」只能还原个大概，「读图层转代码」能还原 90% 以上。

#### 6.1.2 接入 Ardot MCP（三步）

1. 配置 MCP：编辑 `C:\Users\<你>\.workbuddy\mcp.json`，合并进（**不要覆盖别人的条目**）：
   ```json
   {
     "mcpServers": {
       "ardot-remote": {
         "type": "http",
         "url": "https://ardot.tencent.com/mcp",
         "timeout": 120000
       }
     }
   }
   ```
   > ⚠️ 路径是 `~/.workbuddy/mcp.json`，**不是** `~/.workbuddy/.mcp.json`（带点前缀的是别的 Agent 用的）。JSON 里不能有多余逗号、不能写注释。

2. **完全退出并重启 WorkBuddy**（只关窗口不够，托盘图标也要退）→ 连接器管理 → 右上角「自定义连接器」→ 找到 `ardot-remote` → 点「信任」→ 浏览器弹 Ardot 授权页 → 登录并同意授权

3. **新开一个对话**（旧会话有缓存，工具列表不会刷新），输入：
   ```
   使用 ardot-remote MCP，创建一个设计文件
   ```
   能成功创建并弹出设计稿面板 → 接入成功

#### 6.1.3 出设计稿的提示词

写提示词有三个要点，缺一个效果就会打折：

| 要点 | 说明 | 反例 → 正例 |
|---|---|---|
| **给足上下文** | 什么产品、给谁用、用在哪 | ❌「做个灯泡开关」→ ✅「智能家居App 里的灯泡开关控件演示页」 |
| **指定技术规格** | 画布尺寸、风格、明暗主题 | ❌「好看点」→ ✅「移动端 390×844，现代简约风，亮色主题」 |
| **说清交互状态** | 有几种状态、切换后发生什么 | ❌「能开关」→ ✅「关闭态=灰暗房间，开启态=暖黄光晕+光芒射线，带亮度滑杆」 |

本项目用的完整提示词：

```
使用 Ardot，设计一个「灯泡开关」演示页面。

【画布】移动端 390×844，亮色主题，现代简约风格。
【页面结构】从上到下：
  1. 顶部卡片区（高约 320px）：一个吊线灯泡的插画，
     - 关闭态：深灰蓝房间背景（#232a38），灯泡玻璃为暗色，灯丝不发光
     - 开启态：暖黄渐变房间背景，灯泡玻璃变为 #fff6dc，灯丝发橙光，
       灯泡外围有径向光晕，四周有 7 条短射线，地面有椭圆投影光圈
  2. 控制区（白底卡片，圆角 26px）：
     - 标题「客厅吊灯」
     - 状态文案「当前状态：已关闭 / 已点亮 · 70%」
     - 一个胶囊开关（宽 64 高 34，关闭 #ccd3de / 开启 #ffb01f，滑块带阴影）
     - 亮度滑杆（10%-100%，默认 70%，关闭态置灰不可拖）
     - 底部一行「今日开关次数：0」
【交互】点击灯泡本体、点击开关、按空格键都能切换状态；
       开启后亮度滑杆可拖动，且实时联动房间亮度与光晕强度。
【风格】圆角大、阴影柔和、过渡动画 0.55s，整体偏温暖。
```

#### 6.1.4 迭代修改的提示词

Ardot 出稿后，**直接接着聊就行**，不用重新生成：

```
灯泡的玻璃罩再圆润一点，光晕范围缩小 20%
开关的开启色改成 #ff8a00，更暖一些
在卡片右上角加一个「自动模式」的小标签
顶部留白从 40px 改成 32px
```

每次改完 Ardot 会自动截图回看，不满意继续说，直到定稿。

#### 6.1.5 设计稿转网页的提示词

```
基于当前设计稿，生成完整的网页代码。要求：
- 单个 HTML 文件，内联 CSS 和 JS，不依赖任何外部库
- 严格还原设计稿的色值、圆角、间距、字体层级
- 保留全部交互：点击灯泡/开关/空格键切换、亮度滑杆联动
- 使用 CSS 变量管理设计 Token
- 动画过渡 0.55s，并支持 prefers-reduced-motion
- 输出后直接保存到当前工作区，并在预览面板打开
```

想生成 Vue 版本，只要在提示词里说清楚：`改成 Vue 2 完整版 + CDN 引入 + new Vue 单文件组件写法`。

本项目的 `Design-Ardot/lightbulb-switch/index.html` 就是这么出来的 —— **373 行，包含完整 CSS 和内联 SVG**，可以直接双击打开看效果。

#### 6.1.6 设计 Token 对照表

Ardot 的设计变量（Design Token）最终会变成 CSS 自定义属性。这张表值得记住 —— 它让你知道**改哪个值能影响什么**：

| 设计 Token | CSS 变量 | 值 | 用在哪 |
|---|---|---|---|
| 页面背景 | `--page` | `#eef1f6` | body 背景 |
| 卡片背景 | `--card` | `#ffffff` | 卡片底色 |
| 主文字 | `--ink` | `#1b2333` | 标题 |
| 次文字 | `--muted` | `#6d7789` | 状态文案、提示 |
| 分割线 | `--line` | `#e3e8f0` | 边框、分隔线 |
| 主色（琥珀） | `--amber` | `#ffb01f` | 开关开启、滑杆填充 |
| 主色深 | `--amber-deep` | `#ff8a00` | 状态强调文字 |
| 关闭色 | `--switch-off` | `#ccd3de` | 开关关闭态 |

**为什么用变量而不是写死？** 想换主题色，只改 `:root` 里的一行，全页跟着变。写死的话要改几十处，漏一处就出 bug。

### 6.2 路线 B：自主设计前端页面

不想用 Ardot，或者想自己折腾，**也完全可以**。设计是自由的，只要满足**第 7 章的功能需求**。

#### 6.2.1 自主设计的需求提示词

```
设计一个智能电灯控制页面，要求：

【功能】
- 图形化展示电灯开关状态，一眼能看出开还是关
- 提供开关控件
- 提供「查询服务器状态」按钮
- 显示今日开关次数

【设计要求】
- 桌面端浏览器，宽度自适应，最大 420px 居中
- 亮色主题、现代简约、圆角大、阴影柔和
- 状态要有强烈的视觉对比：关闭时冷色调，开启时暖色调
- 动画过渡 0.3~0.6 秒，不要生硬跳变
- 支持 prefers-reduced-motion（无障碍：有些人对动画敏感）

【不用纠结的地方】
- 灯泡插画可以直接用 emoji（💡）或纯 CSS 圆形，不用画 SVG
- 图标用文字或简单形状代替
- 重点是状态清晰、操作顺手，不是视觉华丽

【交付】
单个 HTML 文件，内联 CSS，无外部依赖
```

#### 6.2.2 三个可选设计方向

**方向 1：极简卡片**（难度 ★☆☆）
```
┌────────────────────────┐
│                        │
│         💡             │  ← emoji 或 CSS 圆形
│      （亮/暗靠滤镜）      │
│                        │
├────────────────────────┤
│ 客厅吊灯                │
│ 当前状态：已点亮 · 70%   │
│ ┌──┐  开    空格键切换   │
│ │  │                   │
│ └──┘                   │
│ 亮度 ──────●────       │
│ 今日开关次数     3      │
│ ┌───────────────────┐  │
│ │  查询服务器当前状态  │  │
│ └───────────────────┘  │
└────────────────────────┘
```
用 emoji 💡 + CSS `filter: brightness()` 控制亮暗，最省事。

**方向 2：房间场景**（难度 ★★☆，本项目采用）
```
┌────────────────────────┐
│  ▓▓▓▓ 深色房间 ▓▓▓▓    │  ← 关：#232a38
│      ╱ 吊线 ╲           │     开：暖黄渐变
│     （玻璃罩）           │     光晕 + 射线 + 地面光圈
│      灯丝                │
│  ▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓    │
├────────────────────────┤
│ 客厅吊灯  [服务器 14:02] │
│ 当前状态：已点亮 · 70%    │
│ ...
```
用内联 SVG 画灯泡，靠 CSS class 切换 `.is-on` 改变所有子元素的填充色。视觉冲击最强。

**方向 3：仪表盘**（难度 ★★☆）
```
┌────────────────────────┐
│  智能灯控制              │
│  ┌──────┐┌──────┐     │
│  │ 客厅  ││ 卧室  │     │  ← 多设备网格
│  │  ON  ││  OFF │     │
│  └──────┘└──────┘     │
│  ┌──────┐┌──────┐     │
│  │ 厨房  ││ 玄关  │     │
│  │  OFF ││  ON  │     │
│  └──────┘└──────┘     │
└────────────────────────┘
```
如果你想把后端扩展成多设备，这是最自然的演进方向。

#### 6.2.3 用 AI 生成自主设计稿的提示词

```
我要设计一个智能电灯控制页面。请先用文字描述你的设计方案，
包括：整体布局、每个区域的尺寸比例、配色方案（给出具体色值）、
状态切换时的视觉变化、动画时长。

要求：
- 单页面，最大宽度 420px 居中，亮色主题
- 必须包含：灯泡可视化、开关控件、状态文案、亮度滑杆、
  今日次数、查询服务器状态按钮
- 状态对比要强：关闭冷色调，开启暖色调
- 动画 0.3~0.6 秒
- 不要用图片，用 CSS 和内联 SVG

描述完先别写代码，等我确认方案。
```

> 💡 **"描述完先别写代码，等我确认方案"** 这句话很重要。先看方案再动手，避免生成一堆不要的东西。这是 VibeCoding 的正确节奏 —— **人掌握方向，AI 执行细节**。

---

## 第 7 章　前端实现：Vue 2.0 + Axios

### 7.1 30 秒搞懂 Vue 和 Axios

#### Vue 是什么

Vue 是一个**前端框架**，解决的核心问题是：**数据变了，界面怎么自动跟着变？**

不用 Vue 的写法（原始设计稿就是这样）：

```javascript
let on = false;

function render() {
  scene.classList.toggle('is-on', on);
  label.textContent = on ? '开' : '关';
  status.innerHTML = '当前状态：<b>' + (on ? '已点亮' : '已关闭') + '</b>';
  // ... 每次改 on 都要手动调用 render()，漏了就出bug
}

function toggle() {
  on = !on;
  render();
}
```

用 Vue 的写法：

```javascript
new Vue({
  el: '#app',
  data: { on: false },
  methods: {
    toggle() { this.on = !this.on; }   // 改完就结束了，不用调render()
  }
});
```

```html
<div :class="{ 'is-on': on }">
  <span>{{ on ? '开' : '关' }}</span>
</div>
```

**改 `this.on`，DOM 自动更新。** 这就是 Vue 的核心价值。

#### Vue 的三个核心概念

```
   data       会变化的状态（单一数据源）
     ↓
   computed   由 data 推导出来的值（自动缓存）
     ↓
   methods    会改变 data 的动作
     ↓
   模板 {{ }} :prop @event   把结果渲染到页面
```

| 概念 | 用来放什么 | 例子 |
|---|---|---|
| `data` | 单纯的状态值 | `on: false`、`brightness: 70` |
| `computed` | **由其他数据推导出来的**值 | `statusText`（由 `on` 和 `brightness` 推出） |
| `methods` | 会**修改**数据的动作 | `toggle()`、`fetchStatus()` |

**`computed` 和 `methods` 的关键区别**：

| | computed | methods |
|---|---|---|
| 有没有括号 | **没有**：`{{ statusText }}` | **有**：`{{ greet() }}` |
| 何时重新算 | 依赖变化时算一次，**结果缓存** | 每次渲染都重算 |
| 适合什么 | 纯计算，无副作用 | 有网络请求、有副作用的操作 |

**判断标准很简单：只是「根据现有数据算个值」→ `computed`；要「做点什么」（发请求、改状态）→ `methods`。**

#### Axios 是什么

Axios 是一个 **HTTP 客户端** —— 让 JavaScript 能发网络请求。

浏览器的 `fetch` 原生就能发请求，但 Axios 更好用：

| 能力 | fetch | Axios |
|---|---|---|
| 写法 | `fetch(url, {method, body})` | `axios.post(url, data)` |
| JSON 处理 | 要手动 `.json()` | **自动序列化/反序列化** |
| 状态码 | 不抛异常，要自己判断 | **非 2xx 自动进 catch** |
| 浏览器兼容 | 需要 polyfill | 自带兼容处理 |
| 超时设置 | 不支持（要手写 AbortController） | `timeout: 8000` 一行搞定 |

本项目里对应的三句：

```javascript
// 原来（fetch）
fetch('http://127.0.0.1:8000/api/light', {
  method: 'POST',
  headers: {'Content-Type': 'application/json'},
  body: JSON.stringify({action: 'On'})
}).then(r => r.json()).then(d => console.log(d));
// ↑ 要手动 stringify、要手动 .json()、要手动判断状态码

// 现在（axios）
const res = await axios.post(
  'http://127.0.0.1:8000/api/light',
  { action: 'On', source: 'frontend' },
  { timeout: 8000 }
);
console.log(res.data);
// ↑ 一行搞定，自动序列化、自动解析、错误自动进 catch
```

### 7.2 第 1 步：复制设计稿到 FrontEnd

```bash
mkdir FrontEnd
cp Design-Ardot/lightbulb-switch/index.html FrontEnd/index.html
```

Windows CMD 写法：
```cmd
mkdir FrontEnd
copy Design-Ardot\lightbulb-switch\index.html FrontEnd\index.html
```

**为什么要复制而不是直接引用 `Design-Ardot/` 里的文件？**

|理由 | 说明 |
|---|---|
| 生产代码和设计稿分离 | `Design-Ardot/` 是设计资源，应该保持干净不被污染 |
| 改造有据可查 | 原始版本留着，随时能对比「改了什么」 |
| 目录职责清晰 | 前端代码就放 `FrontEnd/`，一眼找到 |

### 7.3 第 2 步：给 AI 的提示词（静态页改造成 Vue）

这一步是**改造**，不是从零写。要在提示词里说清「保留什么、改什么」。

```
把 FrontEnd/index.html 从「静态 HTML + 原生 JS」改造成「Vue 2.0 + Axios 单页应用」。

【必须保留（不要动）】
- <style> 里所有 CSS，一个字都不要改
- 所有内联 SVG（灯泡、光晕、射线）
- 视觉设计：圆角、色值、阴影、过渡时长全部保持一致

【CDN 引入】
- Vue 2.7.16 完整版（vue.min.js，不是 runtime-only，因为要编译 DOM 内模板）
- Axios 1.7.9

【改造要求】

1. 根元素加 id="app" 和 v-cloak，加 CSS [v-cloak]{display:none}
   避免 JS 加载完之前用户看到 {{ }} 原始语法

2. 把所有 addEventListener 改成 Vue 模板语法：
   - class="scene"      → :class="{ 'is-on': on }"
   - sw.addEventListener('click', toggle)        → @click="toggle"
   - scene.addEventListener('keydown', ...)      → @keydown.space.prevent="toggle"
   - range.addEventListener('input', render)    → v-model.number="brightness"

3. 把全局变量 on / count 收进 data，render() 拆成 computed：
   - on           → data
   - brightness   → data
   - statusText   → computed（原来 render() 里拼的字符串）
   - roomStyle    → computed（原来 render() 里设的 scene.style.background）
   - haloStyle    → computed（原来设的 .halo opacity）
   - floorStyle   → computed（原来设的 .floor opacity）
   - rangeStyle   → computed（原来设的 --pct CSS 变量）

4. 加后端联调功能：
   - data 里加 API_BASE: 'http://127.0.0.1:8000'、loading、serverTime、connected、feedback
   - 新增按钮「查询服务器当前状态」，调 GET /api/light/status
   - 切换开关时调 POST /api/light，body {action:'On'|'Off', source:'frontend'}
   - 乐观更新：先改本地状态（点击立刻有反应），请求失败要回滚
   - loading 期间禁用按钮，防重复点击
   - mounted() 里自动调一次 fetchStatus()，实现刷新页面状态不丢
   - 新增「今日开关次数」显示，调 GET /api/light/stats
   - 错误提示翻译成人话，区分「连不上后端」和「参数错误」

5. 代码注释用中文，重点解释「为什么这么写」而不只是「这是什么」

【验收标准】
1. 双击打开页面（不报错），灯泡初始为关闭态
2. 点灯泡/点开关/按空格键都能切换，动画和原来一样流畅
3. 后端未启动时，点开关有明确的错误提示，且界面会回滚
4. 后端启动后，点开关能看到绿色提示条显示写入的时间和动作
5. 页面加载时自动显示服务器当前状态
6. 「查询服务器当前状态」按钮能手动刷新，点多次不会发多次请求
7. 亮度滑杆拖动时房间底色和光晕实时联动
8. 关闭状态下亮度滑杆置灰不可拖
```

### 7.4 第 3 步：静态写法 → Vue 写法对照表

这张表是本章最有价值的部分。**看懂它，你就看懂 Vue 的核心。**

| 原始静态写法 | Vue 2.0 写法 | 说明 |
|---|---|---|
| `class="scene"` | `:class="{ 'is-on': on }"` | 动态类名，on 为 true 时加 `is-on` |
| `style="background: ..."` | `:style="roomStyle"` | 动态样式，值来自 computed |
| `sw.setAttribute('aria-checked', String(on))` | `:aria-checked="String(on)"` | 动态属性，模板里不用 setAttribute |
| `sw.addEventListener('click', toggle)` | `@click="toggle"` | 事件绑定，`@` 是 `v-on` 的简写 |
| `scene.addEventListener('keydown', e => {...})` | `@keydown.space.prevent="toggle"` | `.space` 监听空格，`.prevent` 阻止滚动 |
| `range.addEventListener('input', render)` | `v-model.number="brightness"` | 双向绑定 + `.number` 自动转数字 |
| `let on = false` | `data: { on: false }` | 状态收进 data，Vue 才能追踪 |
| `countEl.textContent = count` | `{{ count }}` | 插值，直接渲染 |
| `box.classList.toggle('disabled', !on)` | `:class="{ disabled: !on }"` | 条件类名 |
| `range.disabled = !on` | `:disabled="!on"` | 动态禁用 |
| `if (on) {...} else {...}` 分支 | `v-if="on"` / `v-else` | 条件渲染 |
| `status.innerHTML = '...<b>' + x + '</b>'` | `<b>{{ statusText }}</b>` | **不要用 innerHTML**，用插值更安全 |

**两个必须记住的安全点**：

1. **永远不要用 `v-html`**（相当于 `innerHTML`）。用户输入会被当成 HTML 执行，XSS 攻击就是这么来的。要显示纯文本就用 `{{ }}`。

2. **`v-model` 只用在表单控件上**。它的本质是 `:value` + `@input` 的语法糖。

**常用修饰符速查**：

| 修饰符 | 作用 |
|---|---|
| `.number` | 自动转数字（`v-model.number="n"`） |
| `.trim` | 自动去首尾空格 |
| `.prevent` | 阻止默认行为（如 `a` 标签跳转） |
| `.stop` | 阻止事件冒泡 |
| `.space` / `.enter` | 只响应特定按键（keyCode 别名） |
| `.once` | 只执行一次 |

### 7.5 第 4 步：五个核心代码块

#### 7.5.1 模板：根元素和动态绑定

```html
<div id="app" v-cloak>
  <div class="scene"
       :class="{ 'is-on': on }"
       :style="roomStyle"
       @click="toggle"
       @keydown.space.prevent="toggle">
    <div class="halo" :style="haloStyle"></div>
    <div class="floor" :style="floorStyle"></div>
    <!-- SVG 原样保留 -->
  </div>

  <div class="status">当前状态：<b>{{ statusText }}</b></div>

  <button class="switch"
          :aria-checked="String(on)"
          :disabled="loading"
          @click="toggle"></button>
  <span>{{ on ? '开' : '关' }}</span>

  <input type="range" :disabled="!on" :style="rangeStyle" v-model.number="brightness">
  <span>{{ count }}</span>
</div>
```

**注意 `v-cloak`**：它配合 `[v-cloak]{display:none}`，在 Vue 挂载完成前隐藏整个模板。否则用户会先看到一闪而过的 `{{ on }}` 和空荡荡的页面。

#### 7.5.2 data：状态集中管理

```javascript
data: {
  API_BASE: 'http://127.0.0.1:8000',   // ← 后端地址，换机器改这里

  on: false,
  brightness: 70,
  count: 0,

  loading: false,
  serverTime: '',
  connected: null,     // null=未查询 true=已连接 false=连不上
  feedback: null,      // { type: 'ok'|'err', text: '提示文案' }
}
```

**`loading` 为什么必须有？** 防止用户疯狂点击，同时后端收到十几个请求。加了 `:disabled="loading"` 之后，请求期间按钮变灰点不动，用户也不会困惑「怎么点了没反应」。

**`connected` 为什么设计成三个状态而不是布尔？** 因为「还没试过」和「试过了连不上」是两回事：
- `null`：页面刚打开，还没查询过 → 灰色圆点 + "尚未查询后端"
- `true`：连上了 → 绿色圆点 + "已连接后端 http://..."
- `false`：连不上 → 红色圆点 + "无法连接后端，请确认服务已启动"

如果只用布尔，初次加载会显示"连不上"，给用户错误的挫败感。

#### 7.5.3 computed：自动推导的样式

```javascript
computed: {
  statusText() {
    if (!this.on) return '已关闭';
    return '已点亮 · ' + this.brightness + '%';
  },

  roomStyle() {
    if (!this.on) return {};           // 关闭时返回空对象
    const k = this.brightness / 100;
    const a = (0.35 + 0.65 * k).toFixed(3);
    const b = (0.30 + 0.70 * k).toFixed(3);
    return {
      background:
        'linear-gradient(180deg,' +
        'rgba(74,68,54,' + a + ') 0%,' +
        'rgba(107,95,69,' + a + ') 55%,' +
        'rgba(138,122,88,' + b + ') 100%)'
    };
  },

  haloStyle() {
    if (!this.on) return { opacity: 0 };
    const k = this.brightness / 100;
    return { opacity: (0.35 + 0.65 * k).toFixed(3) };
  },

  floorStyle() {
    if (!this.on) return { opacity: 0 };
    const k = this.brightness / 100;
    return { opacity: (0.25 + 0.75 * k).toFixed(3) };
  },

  rangeStyle() {
    return { '--pct': this.brightness + '%' };
  },
}
```

**这段逻辑是从原始 `render()` 函数原样搬过来的**，只是从「手动调用」变成了「自动推导」。`data` 一变，这些 computed 自动重算，DOM 自动更新 —— 你什么都不用调。

**`rangeStyle` 返回 CSS 变量**：`{'--pct': '70%'}` 让 CSS 里的 `linear-gradient(90deg, var(--amber) var(--pct,70%), #e3e8f0 var(--pct,70%))` 能画进度条。这是CSS 和 JS 通信的一个实用技巧。

**注意 `if (!this.on) return {}`**：关闭时返回**空对象**，而不是 `{background: ''}`。返回空对象表示"不覆盖任何样式"，让 CSS 类里的默认值生效；返回空字符串则会强制清空样式，可能产生意外。

#### 7.5.4 methods：三个核心方法

**方法 1：toggle —— 乐观更新 + 失败回滚**

```javascript
async toggle() {
  if (this.loading) return;              // 正在请求，忽略连点

  const next = !this.on;

  // 1) 立刻改本地状态，用户马上看到效果
  this.on = next;
  this.feedback = null;

  // 2) 调后端写入记录
  const ok = await this.sendAction(next ? 'On' : 'Off');

  // 3) 后端写失败就回滚
  if (!ok) this.on = !next;
}
```

**为什么要「先改后请求」？** 想象改成「先请求，成功后再改」。网络请求要 100~300ms，用户点了按钮后毫无反应，然后灯突然亮了 —— 感觉卡顿、怀疑没点上。

**乐观更新**（Optimistic UI）：先假设会成功，立刻更新；失败了再回滚。这是现代前端处理这类交互的标准做法，Twitter、Facebook 的点赞都是这么做的。

**`if (this.loading) return;`** 是第一道防线，防止连点。`:disabled="loading"` 是第二道（视觉 + 交互双重禁用）。

**方法 2：sendAction —— POST 写入**

```javascript
async sendAction(action) {
  this.loading = true;
  try {
    const res = await axios.post(
      this.API_BASE + '/api/light',
      { action: action, source: 'frontend' },
      { headers: { 'Content-Type': 'application/json' }, timeout: 8000 }
    );

    this.serverTime = res.data.created_at;
    this.connected = true;
    this.feedback = {
      type: 'ok',
      text: '已写入服务器 · 动作 ' + res.data.action + ' · 时间 ' + res.data.created_at,
    };

    this.refreshCount();
    return true;

  } catch (err) {
    this.connected = false;
    this.feedback = { type: 'err', text: this.errText(err) };
    return false;

  } finally {
    this.loading = false;
  }
}
```

**`try / catch / finally` 的分工**：

| 块 | 作用 | 本例 |
|---|---|---|
| `try` | 放可能出错的代码 | 发请求 |
| `catch` | 出错时的补救 | 显示错误、回滚 |
| `finally` | **无论成败都执行** | 恢复 `loading = false` |

**`finally` 为什么关键？** 如果只在 `try` 成功后写 `this.loading = false`，一旦请求失败，按钮会永远卡在"请求中…"状态，再也点不动了。**清理代码永远放 `finally`。**

**`{ headers: {...}, timeout: 8000 }` 是 Axios 的第三个参数**（第二个是请求体）。`timeout` 很重要 —— 没有它，后端挂了的话请求会一直挂着，按钮永远转圈。

**方法 3：fetchStatus —— 查询服务器状态**

```javascript
async fetchStatus() {
  this.loading = true;
  this.feedback = null;
  try {
    const res = await axios.get(this.API_BASE + '/api/light/status', { timeout: 8000 });

    this.on         = res.data.is_on;
    this.serverTime = res.data.updated_at || '';
    this.connected  = true;
    this.feedback = {
      type: 'ok',
      text: res.data.updated_at
        ? ('服务器记录：' + res.data.action
           + ' · 最后更新 ' + res.data.updated_at
           + ' · 第 ' + res.data.record_id + ' 号记录'
           + ' · 共 ' + res.data.total_records + ' 条')
        : '服务器暂无任何记录，当前按「关闭」处理',
    };

    this.refreshCount();
  } catch (err) {
    this.connected = false;
    this.feedback = { type: 'err', text: this.errText(err) };
  } finally {
    this.loading = false;
  }
}
```

**`res.data.updated_at || ''` 的 `||` 兜底**：数据库为空时 `updated_at` 是 `null`，`null || ''` 得到空字符串。少了这个兜底，模板里会显示 "null"，很难看。

**三元运算符处理空数据**：

```javascript
res.data.updated_at
  ? ('服务器记录：...')        // 有数据：显示详情
  : '服务器暂无任何记录...'     // 没数据：显示友好提示
```

**方法 4：errText —— 把报错翻译成人话**

```javascript
errText(err) {
  if (err.code === 'ECONNABORTED') return '请求超时，后端可能没启动';
  if (!err.response)              return '连不上后端 ' + this.API_BASE + '，请先启动 Backend 服务';
  const d = err.response.data;
  if (err.response.status === 422 && Array.isArray(d.detail)) {
    return '参数不对：' + d.detail.map(x => x.msg).join('; ');
  }
  return '接口报错 ' + err.response.status + '：' + (d.detail || d.error || '未知错误');
}
```

**Axios 错误的两种形态**：

| 形态 | 特征 | 含义 |
|---|---|---|
| **网络错误** | `err.response` 是 `undefined` | 请求根本没发出去（后端没启动、地址错、被拦截） |
| **HTTP 错误** | `err.response` 有值，`status` 是 4xx/5xx | 请求发出去了，服务端返回错误 |

**这两类错误要用完全不同的话术。** 新手最常见的问题是把网络错误也显示成"参数错误"，用户完全看不懂。

#### 7.5.5 mounted：刷新页面状态不丢

```javascript
mounted() {
  this.fetchStatus();
}
```

**`mounted` 是什么？** Vue 实例创建并挂载到页面**之后**自动调用的钩子函数。

为什么在这里查状态？这样你点开电灯之后按 F5 刷新页面，灯泡依然是亮的 —— 因为状态真存在服务器上，不在浏览器里。**这才是"和后端同步"的真正意义。**

### 7.6 Vue 2.0 必须注意的三个坑

### 坑 1：必须用完整版，不能用 runtime-only

```html
<!-- ✅ 正确：完整版，自带模板编译器 -->
<script src="https://cdn.jsdelivr.net/npm/vue@2.7.16/dist/vue.min.js"></script>

<!-- ❌ 错误：runtime 版无法编译 DOM 内模板 -->
<script src="https://cdn.jsdelivr.net/npm/vue@2.7.16/dist/vue.runtime.min.js"></script>
```

用 runtime 版的话，你写在 HTML 里的 `{{ }}` 和 `@click` 会被当成普通文本，页面一片空白，控制台报 `You are using the runtime-only build of Vue`。

如果你想用 runtime 版（比如配合 webpack 打包），就必须用 `render` 函数写渲染逻辑：

```javascript
new Vue({
  el: '#app',
  render: h => h('div', this.statusText)
});
```

**本项目用 CDN + DOM 内模板，必须用完整版。**

### 坑 2：Vue 2 用 Options API，不是 Composition API

```javascript
// ✅ Vue 2 写法（本项目使用）
new Vue({
  el: '#app',
  data: { on: false },
  computed: { statusText() { return this.on ? '亮' : '灭' } },
  methods: { toggle() { this.on = !this.on } }
});

// ❌ Vue 3 写法，在 Vue 2 里会报错
const app = Vue.createApp({
  setup() { const on = ref(false); return { on } }
});
```

**`data` 必须是函数吗？** 只有组件的 `data` 必须是函数（因为组件可能被创建多次，每次要独立的 data）。**根实例的 `data` 可以直接是对象**，本项目就是根实例。

### 坑 3：`v-model` 和 `v-bind` 的配合

```html
<!-- v-model 已经包含了 :value -->
<input v-model.number="brightness">
```

如果同时写 `:value="brightness"`，两者会打架，行为不可预测。**记住：`v-model` 和 `:value` 二选一。**

同理，`<input type="radio">` 和 `<input type="checkbox">` 用 `v-model` 时，**不要**再加 `:checked`。

### 7.7 前端验收条件

前端不依赖后端也能验证大部分功能。分两组：

#### A 组：不启动后端，验证视觉与交互

| # | 验收条件 | 怎么验 | 通过 |
|---|---|---|---|
| F1 | 页面正常加载 | 双击 `FrontEnd/index.html`，控制台无红色报错 | ☐ |
| F2 | 无闪烁 | 加载时看不到 `{{ }}` 原始语法（v-cloak 生效） | ☐ |
| F3 | 点击切换 | 点灯泡 → 变亮；再点 → 变暗 | ☐ |
| F4 | 开关联动 | 点右下角胶囊开关，灯泡同步变化 | ☐ |
| F5 | 空格键 | 点灯泡让页面获得焦点后按空格，灯泡切换 | ☐ |
| F6 | 动画流畅 | 切换时房间底色、光晕、射线是渐变过渡（0.55s），不是瞬间跳变 | ☐ |
| F7 | 亮度联动 | 开灯后拖滑杆，房间底色明暗和光晕强度实时变化 | ☐ |
| F8 | 滑杆禁用 | 关灯时滑杆置灰、拖不动 | ☐ |
| F9 | 文案正确 | 开灯显示「已点亮 · 70%」，关灯显示「已关闭」 | ☐ |
| F10 | 降级动画 | 系统开启「减少动态效果」时动画关闭 | ☐ |

#### B 组：启动后端，验证联调

| # | 验收条件 | 怎么验 | 通过 |
|---|---|---|---|
| F11 | 初次加载查状态 | 打开页面，状态圆点为绿，显示「已连接后端」 | ☐ |
| F12 | 状态从服务器来 | 先在 Thunder Client 关灯 → 刷新页面 → 灯泡显示为灭 | ☐ |
| F13 | 写入成功提示 | 点开灯，出现绿色提示条，含动作和时间 | ☐ |
| F14 | 数据库真的写入 | 点完开关，查 `GET /api/light/records`，条数增加了 | ☐ |
| F15 | 失败回滚 | **关掉后端服务**，点开关 → 灯闪一下又弹回来 + 红色提示 | ☐ |
| F16 | 错误提示友好 | F15 时提示是「连不上后端...请先启动 Backend 服务」 | ☐ |
| F17 | 查询按钮 | 点「查询服务器当前状态」，信息更新且按钮防连点 | ☐ |
| F18 | 次数统计 | 今日开关次数 = Thunder Client 里的操作次数 + 页面操作次数 | ☐ |
| F19 | 无控制台报错 | 浏览器 F12 Console 全程无红色 error | ☐ |
| F20 | 无跨域报错 | F12 Network 里请求状态是 200，不是 blocked | ☐ |

> 💡 **F15 是最能检验代码质量的一条。** 它验证了「失败回滚」逻辑真的работает —— 不是只在成功路径上好看。很多初学者的代码在网络错误时界面会永久卡在错误状态，点一下就再也不动了。**一定要亲手关掉后端试一次。**

---

## 第 8 章　前后端联调与跨域

### 8.1 完整联调流程

```
   ① 启动后端
      cd Backend
      .venv\Scripts\python.exe -m uvicorn main:app --reload --port 8000
      → 看到 Uvicorn running on http://127.0.0.1:8000

   ② 先用 Thunder Client 验证接口
      → 确认 4 个接口都正常（这一步过了再弄前端）

   ③ 打开前端页面
      右键 FrontEnd/index.html → Open with Live Server
      （或直接双击，但建议用 Live Server，见 8.3）

   ④ 看页面状态
      → 绿灯 + "已连接后端"
      → 灯泡显示服务器上的真实状态

   ⑤ 试交互
      → 点开关，看后端终端有没有打印日志
      → 刷新页面，状态还在
```

### 8.2 跨域：为什么能通

浏览器控制台如果出现这个：

```
Access to XMLHttpRequest at 'http://127.0.0.1:8000/api/light/status'
from origin 'null' has been blocked by CORS policy
```

说明后端的 CORS 没生效。本项目在 `main.py` 里已经配好了：

```python
app.add_middleware(
    CORSMiddleware,
    allow_origins=["*"],
    allow_methods=["*"],
    allow_headers=["*"],
    allow_credentials=False,
)
```

**从 `file://` 打开页面时，`origin` 是 `null`**，而不是具体域名。所以 `allow_origins=["*"]` 能覆盖它。

> ⚠️ `allow_credentials=True` 和 `allow_origins=["*"]` **不能共存** —— 这是浏览器安全规范的硬性要求，同时开启会直接报错。要带 Cookie 就必须写具体来源。

### 8.3 建议用 Live Server 而不是双击

双击打开是 `file://` 协议，虽然本项目能跑，但有两个问题：

1. 部分浏览器对 `file://` 的请求有额外限制
2. Console 里的报错没有源文件定位，调试困难

**装 Live Server 插件**（VS Code 扩展面板搜索，作者是 Ritwick Dey），然后：

```
右键 index.html → Open with Live Server
```

页面会跑在 `http://127.0.0.1:5501`（端口可改），这是标准的 HTTP 环境，和线上环境一致。**建议养成这个习惯。**

### 8.4 数据流全链路验证

按这个顺序走一遍，能完整验证所有环节：

| 步骤 | 操作 | 预期结果 |
|---|---|---|
| 1 | 后端启动，记录当前记录数 N | `GET /api/light/stats` 返回 `total_count: N` |
| 2 | 页面点一下开关 | 控制台打印新日志，页面次数 +1 |
| 3 | `GET /api/light/stats` | `total_count` 变成 N+1 |
| 4 | `GET /api/light/records?limit=1` | 最新一条的 `action` = 刚点的动作 |
| 5 | 打开 `logs/app.log` | 有一行时间戳对应的日志 |
| 6 | F5 刷新页面 | 灯泡状态不变（来自服务器） |
| 7 | Thunder Client 发 `{"action":"Off"}` | 页面点「查询服务器当前状态」→ 灯泡变灭 |

**第 7 步是最关键的**：它证明前端能正确读取后端状态，实现了真正的「双向同步」。

---

## 第 9 章　提示词清单（可直接复制）

### 9.1 后端生成提示词

见 [4.3 节](#43-第-3-步给-ai-的提示词生成后端代码)。

### 9.2 前端改造提示词

见 [7.3 节](#73-第-2-步给-ai-的提示词静态页改造成-vue)。

### 9.3 自主设计提示词

见 [6.2.1 节](#621-自主设计的需求提示词) 和 [6.2.3 节](#623-用-ai-生成自主设计稿的提示词)。

### 9.4 报错求助提示词模板

出bug时，把这些信息一次性说清楚，能省掉大量来回：

```
【错误信息】
（把浏览器 Console 或终端的完整报错粘贴过来，不要截图）

【复现步骤】
1. 在哪个页面/什么命令
2. 做了什么操作
3. 预期什么结果，实际什么结果

【环境】
- Python 版本：.venv\Scripts\python.exe --version 的输出
- 后端依赖：Backend/requirements.txt 内容
- 浏览器：Chrome，Console 里的错误
- 后端是否启动：是/否，端口 8000

【已尝试】
（你已经试过什么方法，避免 AI 重复建议）
```

### 9.5 代码审查提示词

让 AI 当 reviewer，比自己看不出来更快：

```
请审查 Backend/main.py，重点检查：
1. SQL 注入风险：有没有用字符串拼接 SQL
2. 资源泄漏：数据库连接、文件句柄是否正确关闭
3. 异常处理：有没有裸 except，是否吞掉了异常
4. 并发安全：SQLite 连接在多线程下安全吗
5. 边界处理：数据库为空、参数缺失、超长字符串
6. 命名与注释：是否清晰表达了「为什么」
```

### 9.6 VibeCoding 的五条实践原则

1. **先写验收标准，再让 AI 动手。** 没有验收标准 = 无法判断 AI 写得对不对。
2. **一次只改一件事。** 同时提五个需求，出了问题根本定位不到是哪条引起的。
3. **报错就贴原文。** 截图 AI 看不清，必须复制文字。
4. **AI 给的代码要能解释。** 解释不了就说明你没真的理解，下次还会踩同样的坑。
5. **验收不通过就说清哪里不对。** 「不对」AI 猜不到，「点了开关灯泡没变，控制台报 500」才行。

---

## 第 10 章　常见报错速查表

### 10.1 环境与启动类

| 报错 | 原因 | 解决 |
|---|---|---|
| `'uv' 不是内部或外部命令` | uv 没装或 PATH 没刷新 | 重装 uv，重启终端 |
| `No virtual environment found` | 没建 venv 就想装依赖 | 先跑 `uv venv --python 3.12 .venv` |
| `No module named uvicorn` | 用了系统 Python | 命令写全路径 `.venv\Scripts\python.exe -m uvicorn` |
| `Address already in use` | 8000 被占用 | 换端口：`--port 8001`，前端 `API_BASE` 同步改 |
| `Error loading ASGI app` | 不在 Backend 目录，或 `main:app` 写错 | `cd Backend` 后再执行 |
| 编辑器代码飘红但能运行 | VS Code 用错解释器 | 设 `python.defaultInterpreterPath` 为 `.venv\Scripts\python.exe` |
| `uv: command not found` 装完 | macOS/Linux 需要刷新 shell | `source ~/.bashrc` 或 `~/.zshrc` |

### 10.2 后端接口类

| 现象 | 原因 | 解决 |
|---|---|---|
| 返回 422 | `action` 不是 On/Off | 检查大小写，`on` 小写也不行 |
| 返回 422 但看着没错 | Body 格式不是 JSON | Headers 必须有 `Content-Type: application/json` |
| 返回 404 | URL 打错 | 注意末尾斜杠：`/api/light` 不是 `/api/light/` |
| 数据库没变化 | 连接没提交 | 忘了 `conn.commit()` |
| 找不到 light.db | 相对路径问题 | 用 `Path(__file__).resolve().parent` |
| 日志重复打印多遍 | handlers 累加 | `logger.handlers.clear()` |
| 日志文件是空的 | FileHandler 没加进去 | 检查是否 `addHandler` |

### 10.3 前端类

| 现象 | 原因 | 解决 |
|---|---|---|
| 页面一片空白，Console 报 runtime-only | 用了 runtime 版 | 换成 `vue.min.js` 完整版 |
| 显示 `{{ on }}` 原文 | 没加 v-cloak | 加 `v-cloak` 和 `[v-cloak]{display:none}` |
| 点击没反应 | Vue 没挂载上 | 检查根元素有没有 `id="app"`，`el` 对不对 |
| 颜色全变默认 | CSS 被破坏 | 检查 `<style>` 是否完整 |
| 进度条不动 | `--pct` 没绑定 | `:style="rangeStyle"` 少了冒号 |
| **Network 里 blocked** | CORS | 确认 `main.py` 有 `add_middleware` 且服务已重启 |
| 请求一直 pending | 没设 timeout | Axios 加 `{ timeout: 8000 }` |
| 按钮转圈不停 | `finally` 没写 | `loading = false` 必须在 `finally` |
| 后端关了界面卡死 | 没做错误处理 | `try/catch/finally` 补全 |
| Network 显示 200 但界面没变 | `res.data` 取错层级 | Axios 返回值在 `res.data`，不是 `res` |

### 10.4 Thunder Client 类

| 现象 | 原因 | 解决 |
|---|---|---|
| 变量显示红色未定义 | 环境没选中 | 右上角下拉选中 `local` |
| 响应 404 | 后端没启动或 URL 错 | 先访问 `http://127.0.0.1:8000/` |
| Body 发不出去 | 选了 Form 而不是 raw JSON | Body → raw → JSON |
| Content-Type 缺失 | 选了 raw 但没手动加 Header | 手动加 `Content-Type: application/json` |
| 保存后找不到请求 | 没指定路径 | Save As 时存到 `Backend/light.http` |

---

## 第 11 章　验收总表与课后练习

### 11.1 完整验收总表

所有条件都打上勾，这个项目就完成了。

#### 环境（2.x）

- [ ] 2.1 uv 可用、Python 3.12 已装、两个插件已装、`.gitignore` 已建

#### 后端（4.7 + 5.6）

- [ ] B1~B10 全部通过
- [ ] T1~T10 全部通过

#### 前端（7.7）

- [ ] F1~F10 全部通过（视觉与交互）
- [ ] F11~F20 全部通过（前后端联调）

#### 核心功能对照（回到你的题目）

| 题目要求 | 对应实现 | 验收编号 |
|---|---|---|
| 开发工具 VS Code | 全教程基于 VS Code + Python 插件 | 2.4 |
| 接口调试 Thunder Client | 3 个请求 + `light.http` | T1~T10 |
| 后端在 Backend 子文件夹 | `Backend/main.py` | 3.1 |
| uv 创建 Python 3.12 的 .venv | `uv venv --python 3.12 .venv` | 4.1 |
| SQLite 数据库 | `Backend/light.db` | B2、B3 |
| 接口：传入 On/Off 并记录时间 | `POST /api/light` | T3、T4、B6 |
| 控制台打印传入参数 | `logger.info` + StreamHandler | B6、T6 |
| 时间记录到日志文件 | `RotatingFileHandler` → `logs/app.log` | B7、T7 |
| 接口：查询当前状态 | `GET /api/light/status` | T5、B9、B10 |
| 写清楚验收条件 | 本章 + 4.7 + 5.6 + 7.7 | — |
| 前端设计过程 + 提示词 | 第 6 章，Ardot 路线 + 自主设计路线 | 6.1、6.2 |
| 前端：图形化模拟开关 | SVG 灯泡 + 开关控件 | F3~F6 |
| 按钮：查询服务器状态 | 「查询服务器当前状态」按钮 | F17、F12 |
| 介绍 Vue 和 Axios | 7.1 节，含 fetch 对比 | — |
| FrontEnd 子文件夹 + 复制 index.html | `FrontEnd/index.html` | 7.2 |
| 单网页、Vue2.0 CDN | `vue.min.js` + `axios.min.js` | 7.3 |
| Vue 对象管理开关和数据同步 | `new Vue({ data, computed, methods })` | 7.5 |
| 生成代码的提示词 | 4.3、7.3 | — |

### 11.2 课后练习（按难度排序）

#### 入门

1. **加一个设备选择器**：页面顶部加下拉框，可选「客厅 / 卧室 / 厨房」，不同设备的状态独立存储。
   *提示*：数据库加一个 `device` 字段，接口加一个 `device` 参数。

2. **扩展日志**：给 `main.py` 加一个 `GET /api/light/logs` 接口，返回日志文件的最后 N 行。
   *提示*：读文件用 `pathlib.read_text(encoding='utf-8')`，注意编码。

3. **前端暗色模式**：加一个切换按钮，切换 `[data-theme="dark"]`，跟第 6.1.6 节的 Token 表对着改。

#### 进阶

4. **定时开关**：新增 `POST /api/light/schedule`，设定"每天 22:00 自动关灯"，用 `asyncio` 任务实现。
   *提示*：FastAPI 的 `lifespan` 里可以启动后台任务，`asyncio.create_task`。

5. **操作历史列表**：页面下方加一个列表，显示最近 10 条操作记录（时间 + 动作 + 来源），调 `GET /api/light/records`。

6. **WebSocket 实时推送**：后端状态变化时主动推给前端，前端自动更新，不用手动点查询。
   *提示*：FastAPI 用 `@app.websocket`，前端用 `new WebSocket('ws://...')`。

#### 挑战

7. **多设备仪表盘**：实现第 6.2.2 节的「方向 3」，支持管理 8 个设备，每格显示开关状态。

8. **用电量统计**：按"开着的时长 × 功率"算日用电量（度）。需要记录每次操作的时刻，统计相邻两次的间隔。
   *提示*：`WITH RECURSATE` 或直接用 SQL 的 `LAG()` 窗口函数。

9. **重构为 Vue 3**：把这个项目迁移到 Vue 3，用 `createApp` + `setup()` + `ref`，感受两代 API 的差异。

10. **部署上线**：把这个项目部署到公网，绑定真实域名，加 HTTPS。
    *提示*：后端用 `uvicorn main:app --host 0.0.0.0 --port 8000`（注意 `allow_origins` 改成实际域名），前端用静态托管。

### 11.3 一句话总结

这个项目真正想教你的不是 FastAPI 或 Vue，而是**「把一件事想清楚」的方法**：

```
   需求  →  想清楚做成什么样  →  写验收标准  →  AI 实现  →  逐条验收  →  不通过就改
```

这个循环适用于你学任何新技术。先想清楚要什么，再让 AI 帮你写。**你永远是那个定义标准的人。**

---

## 附录　相关文档

|资源 | 地址 |
|---|---|
| FastAPI 官方文档 | https://fastapi.tiangolo.com/zh/ |
| Pydantic 文档 | https://docs.pydantic.dev/ |
| Vue 2 官方文档 | https://v2.cn.vuejs.org/ |
| Axios 文档 | https://axios-http.com/zh/ |
| uv 官方文档 | https://docs.astral.sh/uv/ |
| Thunder Client | https://www.thunderclient.com/ |
| Ardot 快速开始 | https://docs.ardot.tencent.com/ardot-mcp/quick-start.html |
| SQLite 语法 | https://www.sqlite.org/lang.html |

---

*本教程随 BulbSwitch 项目一同维护。设计稿来自 `Design-Ardot/`，Ardot 完整操作流程见 `Design-Ardot/Ardot接入WorkBuddy操作指南.md`。*
