<p align="center">
  <h1 align="center">XP World</h1>
  <p align="center"><em>纯前端 · 本地运行的 XP 设定条目管理工具</em></p>
  <p align="center">
    <a href="https://moyun-re.github.io/xp-world/">在线体验</a> ·
    <a href="#使用说明">说明</a> ·
    <a href="./DEVELOPMENT.md">开发文档</a>
  </p>
  <p align="center">
    <img src="https://img.shields.io/badge/license-CC%20BY--NC%204.0-EF4444?style=flat-square" alt="license">
    <img src="https://img.shields.io/badge/Vue-3-42B883?style=flat-square&logo=vue.js&logoColor=white" alt="Vue 3">
    <img src="https://img.shields.io/badge/Tailwind-CSS-38BDF8?style=flat-square&logo=tailwindcss&logoColor=white" alt="Tailwind">
  </p>
</p>

---

## 简介

**XP World** 用于浏览、编辑、导出 XP 设定条目集合。内置优化版条目离线可用，所有编辑自动保存在浏览器本地，刷新不丢。

已在 GitHub Pages 部署：**[https://moyun-re.github.io/xp-world/](https://moyun-re.github.io/xp-world/)**

## 特性

- 纯前端单文件 HTML，部署到 GitHub Pages 即可访问
- 内置预设数据离线可用，编辑自动保存在浏览器本地
- **三级分类**：一级 6 类（世界观 / 角色 / 恋物 / 剧情 / SM / 重口）→ 二级 A–Z → 条目
- 每条含：**标题**、**词条说明**、**预注入提示词**（JSON 字段为 `title` / `human` / `ai`）
- 维护备份导出 + 勾选「提示词导出」（给 AI Novel）
- 支持暗色模式与手机端抽屉式侧边栏
- 搜索：编号（如 `K29`）回车直达，或关键词匹配标题 / 说明 / 提示词

## 文件结构

```
XP World/
├── index.html      # 应用主程序
├── entries.json    # 内置预设（version + entries）
├── README.md       # 本文件
└── DEVELOPMENT.md  # 开发者文档
```

## 快速开始

### 本地开发
```bash
# 在 XP World/ 目录下起本地 HTTP 服务
cd "XP World"
python -m http.server 5173 --bind 127.0.0.1
# 浏览器访问 http://127.0.0.1:5173/
```

> 直接双击 `index.html` 用 `file://` 打开会因 CORS 拦截 `fetch` 失败，必须走 HTTP。

### 部署到 GitHub Pages
1. Fork 仓库
2. Settings → Pages → Source：`Deploy from a branch` → `main` / `/root`
3. 访问 `https://<用户名>.github.io/<仓库名>/`

## 使用说明

| 操作 | 说明 |
|---|---|
| **左侧目录** | 三级折叠：一级大类 → 字母分类 → 条目 |
| **顶部 ⌄ 目录** | 手机端汉堡按钮开关侧边栏 |
| **文件** | 导入… / 导出维护数据 / 重置为内置版本 |
| **新建** | 选一级 + 二级字母，自动编号，只填名称 |
| **编辑区** | 标题 · 词条说明 · 预注入提示词（两框同风格） |
| **搜索** | 编号回车直达；或模糊匹配标题/说明/提示词 |
| **勾选多条** | 底部栏：**提示词导出**（仅选中）/ 删除 |
| **暗色** | 右上角切换，首次跟随系统 |

## 数据格式

### 条目字段（逻辑名 → JSON 键）

| 界面名称 | JSON 键 | 用途 |
|----------|---------|------|
| 标题 | `title` | 索引与展示，含分类编号（如 `A1 - 现实世界`） |
| 词条说明 | `human` | 给人速览：场景 / 用途 |
| 预注入提示词 | `ai` | 给 AI 的正文；「提示词导出」只导出此内容 |

```json
{
  "title": "A1 - 现实世界",
  "human": "用一两句话说明适用场景…",
  "ai": "将注入 AI 的提示词正文…"
}
```

### 内置 `entries.json`（版本包装）

```json
{
  "version": "2026-07-19-abc-r2",
  "entries": [
    { "title": "A1 - 现实世界", "human": "...", "ai": "..." }
  ]
}
```

用于首次加载、版本横幅、**重置为内置版本**。

### 维护导入 / 导出

```json
[
  { "title": "K29 - …", "human": "...", "ai": "..." }
]
```

- **导出**：右上角「文件 → 导出维护数据」= 当前全部条目的完整备份（标题 + 说明 + 提示词）
- **导入**：接受上列纯数组，或带 `version` 的包装格式
- 适合备份、换电脑、再次导入本工具

### 提示词导出（勾选后）

```json
[
  { "title": "K29 - …", "content": "<该条预注入提示词原文>" }
]
```

- 入口：侧边栏勾选 → 底部 **提示词导出**
- 只含「标题 + 预注入提示词」；JSON 里正文键名为 `content`（值来自 `ai`），**不含词条说明**
- 适合导入 AI Novel 等写作工具

### 不再支持

- 旧版 SillyTavern `entries` 对象 map
- 旧主格式 `[{ "title", "content" }]` 作为正式导入

## 版本与本地数据

- 内置 `entries.json` 带 `version`
- 本地键：`xp_entries`（条目）、`xp_last_reset_version`、`theme`
- 服务器推新 `version` 后顶部横幅提示；点查看可重置（二次确认，会丢本地未备份修改）
- **清本地条目**：页面内「重置为内置版本」，或开发者工具 Local Storage 删除 `xp_entries` 后刷新

## 预设数据

当前内置为优化版 **A / B / C 类**（以 `entries.json` 的 `version` 为准），由维护文件（如仓库根目录 `XP大世界_优化版_ABC.json`）经同步脚本写入。

**不会**在改优化版 JSON 后自动更新内置。需要发布新内置时：

```bash
# 先预览
python "XP World/scripts/sync_builtin_entries.py" --source "XP大世界_优化版_ABC.json" --version "新版本号"
# 确认后再 --apply（详见 DEVELOPMENT.md §12）
```

## 维护

- 二改 & 维护：@葱油卷香辣
- 原工具作者：@crazyWq

详细开发说明见 [`DEVELOPMENT.md`](./DEVELOPMENT.md)。

## License

[CC BY-NC 4.0](./LICENSE) · 署名 + 非商业性使用
