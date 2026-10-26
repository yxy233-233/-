# WorkBuddy × Ardot 操作指南：拿到设计稿，一键转网页

> 示例主线：做一个**「灯泡开关」演示网页**，全程不离开 WorkBuddy 对话窗口。
> 目标闭环：一句话描述 → Ardot 出可编辑设计稿 → 一键转成 HTML/CSS/JS → 本地预览 / 发布上线。

---

## 0. 先搞清楚这套联动在做什么

Ardot 是腾讯的 AI 设计智能体平台，它对外提供 **MCP Server（18 个工具）**。WorkBuddy 通过 MCP 协议把这个 Server 挂进来之后，你对话里说的每一句设计需求，都会被翻译成对这些工具的调用。

```
┌──────────────┐   自然语言    ┌──────────────┐   MCP 协议   ┌──────────────┐
│              │ ───────────▶ │              │ ───────────▶ │              │
│  你（对话）   │              │  WorkBuddy   │              │ Ardot MCP    │
│              │ ◀─────────── │   Agent      │ ◀─────────── │  Server      │
└──────────────┘   设计稿预览   └──────────────┘   图层/截图   └──────┬───────┘
                                                                     │
                                                              ┌──────▼───────┐
                                                              │  .ardot 设计稿 │
                                                              │ （可编辑画布）  │
                                                              └──────┬───────┘
                                                                     │ 读结构+取色+出规范
                                                              ┌──────▼───────┐
                                                              │  HTML / React  │
                                                              └──────────────┘
```

**关键点**：Ardot 生成的是**带图层、组件、样式的可编辑文件**，不是一张死图。所以 AI 能读回图层结构、间距、色值，还原度远高于"截图识图转代码"。

### 两种接入方式怎么选

| | 云端 MCP（推荐先用） | 本地 MCP |
|---|---|---|
| 地址 | `https://ardot.tencent.com/mcp` | `http://127.0.0.1:50501/api/v1/mcp` |
| 客户端 | 不需要装 Ardot | 必须装 Ardot 桌面端并打开设计稿 |
| 授权 | OAuth 登录 | 客户端内启用 MCP |
| 限制 | 有调用频率限制、大文件响应慢 | 无频率限制、无文件大小限制 |
| 适合 | 轻量任务、远程/服务器环境 | 大设计稿、批量编辑 |

**不确定就从云端 MCP 开始。** 本指南以云端为例。

---

## 1. 前置条件

- [ ] 已安装并登录 WorkBuddy
- [ ] 有 Ardot 账号（微信登录即可，公测有额度）
- [ ] 网络能访问 `ardot.tencent.com`

> 快速自检：浏览器打开 `https://ardot.tencent.com/mcp`，返回 `401 Unauthorized ... missing bearer token` 是**正常的**（说明服务可达、只是在等你授权）。

---

## 2. 步骤一：在 WorkBuddy 里接入 Ardot MCP

Ardot **不在** WorkBuddy 内置连接器市场里，要走「自定义 MCP」通道。三种写法任选其一。

### 方式 A：让 WorkBuddy 自己写配置（最省事）

新建对话，直接粘贴：

```
帮我配置 Ardot 云端 MCP：服务名称 ardot-remote，类型 http，地址 https://ardot.tencent.com/mcp。
配置完成后如需信任或 OAuth 授权，请提示我按 WorkBuddy 的流程完成。
```

### 方式 B：手写配置文件

编辑 `C:\Users\china\.workbuddy\mcp.json`，把下面这段**合并进**已有的 `mcpServers`（不要覆盖别人的条目）：

```json
{
  "mcpServers": {
    "biomcp": {
      "command": "C:\\Users\\china\\.local\\bin\\biomcp.exe",
      "args": ["serve"]
    },
    "ardot-remote": {
      "type": "http",
      "url": "https://ardot.tencent.com/mcp",
      "timeout": 120000
    }
  }
}
```

> ⚠️ 路径是 `~/.workbuddy/mcp.json`，**不是** `~/.workbuddy/.mcp.json`（带点前缀的那个是别的 Agent 用的）。
> ⚠️ JSON 里不能有多余逗号、不能写注释。写错会报 `CONFIG_PARSE_ERROR`。

**本次会话我已经帮你把这段写进去了**，你只需要重启 + 授权（见下）。

### 方式 C：从连接器面板添加

WorkBuddy 左下角 ➜ **连接器管理** ➜ 右上角**自定义 MCP** 入口 ➜ 新增，`type` 填 `http`，URL 填 `https://ardot.tencent.com/mcp`，服务名 `ardot-remote`。

### 使配置生效（必做）

1. **完全退出并重启 WorkBuddy**（只关窗口不够，托盘图标也要退）
2. 打开 **连接器管理** 页面 ➜ 右上角 **自定义连接器** 入口
3. 找到 `ardot-remote`，点击 **「信任」**
4. 浏览器弹出 Ardot 授权页 ➜ 微信/账号登录 ➜ 同意授权
5. 回到 WorkBuddy，状态应为「已连接」

### 验证连通

**新开一个对话**（旧会话有缓存，工具列表不会刷新），输入：

```
使用 ardot-remote MCP，创建一个设计文件
```

能成功创建文件并弹出设计稿面板 → 接入成功。

想确认工具都在，再问一句：

```
列出 ardot-remote 提供的所有工具
```

应该能看到 18 个，见文末附录。

---

## 3. 步骤二：一句话让 Ardot 出设计稿（灯泡开关）

### 3.1 写提示词的三个要点

| 要点 | 说明 | 反例 → 正例 |
|---|---|---|
| **给足上下文** | 说明这是什么产品、给谁用、用在哪 | ❌「做个灯泡开关」→ ✅「做一个智能家居 App 里的灯泡开关控件演示页」 |
| **指定技术规格** | 画布尺寸、风格、明暗主题 | ❌「好看点」→ ✅「移动端 390×844，现代简约风，亮色主题」 |
| **说清交互状态** | 有几种状态、切换后发生什么 | ❌「能开关」→ ✅「关闭态=灰暗房间，开启态=暖黄光晕+光芒射线，带亮度滑杆」 |

### 3.2 直接复制的提示词

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

### 3.3 Ardot 背后会干什么

```
你说完需求
   │
   ├─▶ search_style_guide   搜索配色/字体/布局风格候选
   ├─▶ build_style_guide    构建设计系统（颜色/字号/间距/效果）
   ├─▶ fetch_guidelines     取 landing-page / web-app 设计规范
   ├─▶ locate_available_space  在画布上找空白区域
   ├─▶ batch_edit           批量 Insert 节点，搭出页面结构
   └─▶ capture_screenshot   截图自查，不对就再改一轮
```

设计稿会直接以 **MCP App 面板**形式出现在对话里，可以直接在面板上看、点、改。

### 3.4 迭代修改（接着聊就行）

```
灯泡的玻璃罩再圆润一点，光晕范围缩小 20%
开关的开启色改成 #ff8a00，更暖一些
在卡片右上角加一个「自动模式」的小标签
顶部留白从 40px 改成 32px
```

每次改完它会自动截图回看，不满意继续说，直到定稿。

---

## 4. 步骤三：一键把设计稿转成网页

### 4.1 核心提示词

```
基于当前设计稿，生成完整的网页代码。要求：
- 单个 HTML 文件，内联 CSS 和 JS，不依赖任何外部库
- 严格还原设计稿的色值、圆角、间距、字体层级
- 保留全部交互：点击灯泡/开关/空格键切换、亮度滑杆联动
- 使用 CSS 变量管理设计 Token
- 动画过渡 0.55s，并支持 prefers-reduced-motion
- 输出后直接保存到当前工作区，并在预览面板打开
```

### 4.2 转代码时 Ardot 在做什么

```
fetch_editor_state   →  拿到当前活跃画布、选区、打开的文件
capture_screenshot   →  截图，作为视觉参照
batch_read           →  按节点读回图层树、坐标、尺寸、色值、字号
fetch_variables      →  读设计变量（Design Token），映射成 CSS 自定义属性
fetch_guidelines     →  取 code / tailwind 主题的代码生成规范
        ↓
   生成 HTML/CSS/JS
```

> 💡 想要 React / Vue / Tailwind 版本，只要在提示词里说清楚：
> `改成 React 函数组件 + Tailwind CSS v4，导出 .tsx 文件`

### 4.3 导出图片资源

页面里如果有图标、插画等图片资源：

```
扫描当前设计稿里可导出的资源，导成 SVG 和 2x PNG，保存到 ./assets 目录
```

走的是 `scan_exportable_resources` → `export_nodes` 两个工具。

---

## 5. 跑通之后的收尾动作

| 想做什么 | 直接说 |
|---|---|
| 本地预览 | `在预览面板打开这个页面` |
| 发布上线 | `把这个页面发布成在线链接，别人能访问` |
| 导出资源 | `把灯泡插画导出成 SVG` |
| 出设计系统 | `为这个项目创建一套亮色/暗色主题的设计变量` |
| 设计走查 | `检查当前页面有没有对齐或溢出问题` |

---

## 6. 常见坑与排查

### 连不上

按顺序过一遍：

1. **配置生效了吗** — 改完 `mcp.json` 必须**完全重启** WorkBuddy
2. **点信任了吗** — 连接器管理 ➜ 自定义 MCP ➜ `ardot-remote` ➜ 信任 + OAuth
3. **网络通吗** — 浏览器开 `https://ardot.tencent.com/mcp`，应返回 401 而不是超时
4. **工具列出来了吗** — 让 WorkBuddy `列出 ardot-remote 的工具`，列不出来就是没生效
5. **新开对话了吗** — 旧会话有缓存，**一定要新开一个对话**
6. 还不行就：重新授权 → 删掉重建 MCP 条目 → 新开对话重试

### 报错对照表

| 报错 | 原因 | 处理 |
|---|---|---|
| `401 missing bearer token` | 正常，未授权 | 完成 OAuth 即可 |
| `CONFIG_PARSE_ERROR` | JSON 语法错误 | 检查多余逗号/缺引号，或让 WorkBuddy 帮你修 |
| `PERMISSION_DENIED` | 配置文件属主是 root/管理员 | 把文件属主改回当前用户 |
| `NO_ADAPTER` | 本地 MCP 下没打开设计稿 | 打开 Ardot 客户端并打开设计文件 |
| 截图空白 | 节点不在可视区 / 是空 Frame | 检查 `nodeId`，确认节点有内容 |
| `batch_read` 返回信息少 | 一次读太多节点被精简 | 分批读，控制层级深度 |

### 使用限制

- `batch_edit` 单次不超过 **25 个操作**
- `batch_read` 一次别读 500+ 节点，会撑爆上下文
- `capture_screenshot` 单次建议不超过 **10 个**
- 云端 MCP 有调用频率限制，大稿子切本地 MCP
- 刚复制出来的节点，其子节点 ID 会重新生成，**别立刻对子节点做 Update**

---

## 7. 附录：Ardot MCP 的 18 个工具

### 查询类

| 工具 | 作用 |
|---|---|
| `fetch_editor_state` | 获取编辑器状态：活跃画布、选区、打开的文件 |
| `fetch_file_info` | 文件信息：名称、链接、ID、你的权限 |
| `batch_read` | 按 name/type 正则或节点 ID 批量检索节点属性 |
| `capture_screenshot` | 截取节点/区域图，验证布局与视觉效果 |
| `capture_layout` | 获取布局结构树，可只返回有重叠/裁剪问题的节点 |

### 设计系统

| 工具 | 作用 |
|---|---|
| `fetch_variables` | 读取设计变量（颜色/间距/字号 Token） |
| `apply_variables` | 创建/更新设计变量，支持 Light/Dark 多模式 |
| `fetch_component_lib` | 获取本地组件库，便于复用 |
| `fetch_guidelines` | 取 7 类规范：`table` `landing-page` `web-app` `mobile-app` `slides` `code` `tailwind` |
| `search_style_guide` | 搜索风格/配色/字体/布局的风格指南候选 |
| `build_style_guide` | 基于选择结果构建完整设计系统（Markdown 输出） |

### 编辑操作

| 工具 | 作用 |
|---|---|
| `batch_edit` | 批量执行 Insert / Copy / Update / Move / Delete / Image |
| `locate_available_space` | 按方向和尺寸找画布空白区 |
| `create_new_page` | 在当前 .ardot 文件里新增空白页面 |

### 资源与导出

| 工具 | 作用 |
|---|---|
| `scan_exportable_resources` | 扫描可导出节点（image / svg） |
| `export_nodes` | 导出为 PNG / JPEG / WEBP / SVG / PDF |
| `upload_images` | 批量上传本地图片并填充到节点（单次 ≤20） |

### 辅助

| 工具 | 作用 |
|---|---|
| `get_available_fonts` | 查可用字体列表与样式，确保 fontName 组合正确 |

---

## 8. 本次会话已完成的事

- ✅ 探测 `https://ardot.tencent.com/mcp` 连通性 —— 可达（401，待授权）
- ✅ 已写入 `C:\Users\china\.workbuddy\mcp.json` 的 `ardot-remote` 条目
- ✅ 产出示例目标页面 `lightbulb-switch/index.html`（手写对照版，展示"一键转网页"后应达到的效果）

**你接下来只需**：重启 WorkBuddy → 连接器管理里点「信任」→ 完成 OAuth → 新开对话开干。

---

*参考文档：<https://docs.ardot.tencent.com/ardot-mcp/quick-start.html>*
