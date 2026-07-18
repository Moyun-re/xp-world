<p align="center">
  <h1 align="center">XP World</h1>
  <p align="center"><em>纯前端 · 本地运行 · 零依赖部署 的 XP 设定条目管理工具</em></p>
  <p align="center">
    <img src="https://img.shields.io/badge/version-1.0-3B82F6?style=flat-square" alt="version">
    <img src="https://img.shields.io/badge/license-CC%20BY--NC%204.0-EF4444?style=flat-square" alt="license">
    <img src="https://img.shields.io/badge/Vue-3-42B883?style=flat-square&logo=vue.js&logoColor=white" alt="Vue 3">
    <img src="https://img.shields.io/badge/Tailwind-CSS-38BDF8?style=flat-square&logo=tailwindcss&logoColor=white" alt="Tailwind">
    <img src="https://img.shields.io/badge/entries-2773+-F59E0B?style=flat-square" alt="entries">
    <img src="https://img.shields.io/badge/platform-GitHub%20Pages-181717?style=flat-square&logo=github&logoColor=white" alt="platform">
  </p>
  <p align="center">
    <a href="#快速开始">🚀 快速开始</a> ·
    <a href="#使用说明">📖 使用说明</a> ·
    <a href="#数据格式">📦 数据格式</a> ·
    <a href="./DEVELOPMENT.md">🛠️ 开发文档</a>
  </p>
</p>


---

## 简介

**XP World** 用于浏览、编辑、导出 XP 设定条目集合。内置 2700+ 条目的预设数据，打开即用，无需后端、无需安装。所有数据保存在浏览器本地，刷新不丢。可部署到任意静态托管（推荐 GitHub Pages）。

## 特性

- 纯前端单文件 HTML，部署到 GitHub Pages 即可访问
- 内置 2700+ 条目离线可用，所有编辑自动保存在浏览器本地
- 三级分类浏览（世界观/角色/恋物/剧情/SM/重口 → A-Z → 条目）
- 支持暗色模式与手机端抽屉式侧边栏
- 搜索框输入编号（如 `K29`）按回车直达条目

## 文件结构

```
XP World/
├── index.html      # 应用主程序
├── entries.json    # 内置预设数据（2773 条 + version 字段）
├── README.md       # 本文件
└── DEVELOPMENT.md  # 开发者文档
```

## 快速开始

### 本地开发
```bash
# 在 XP World/ 目录下起一个本地 HTTP 服务
python -m http.server 8000
# 浏览器访问 http://localhost:8000
```

> 直接双击 `index.html` 用 `file://` 打开会因 CORS 拦截 `fetch` 失败，必须走 HTTP。

### 部署到 GitHub Pages
1. Fork 仓库
2. 在仓库 Settings → Pages → Source 选择 `Deploy from a branch` → `main` / `/root`
3. 等待构建，访问 `https://<用户名>.github.io/<仓库名>/`

## 使用说明

| 操作 | 说明 |
|---|---|
| **左侧目录** | 点击分类展开/折叠，点击条目进入编辑 |
| **顶部 ⌄ 目录** | 手机端点汉堡按钮展开/收起侧边栏（覆盖抽屉，编辑区不被挤压） |
| **文件** | 下拉菜单：导入 JSON / 导出全部 / 重置为内置版本 |
| **新建** | 选一级 + 二级分类，自动编号，只填名称即可 |
| **搜索** | 输入编号（如 `K29`）按回车直达；或输入关键词模糊匹配标题/正文 |
| **批量操作** | 侧边栏勾选多条 → 底部出现导出/删除按钮 |
| **暗色** | 右上角图标切换，跟随系统首次进入 |

## 数据格式

### `entries.json`
```json
{
  "version": "2026-07-19",
  "entries": [
    { "title": "A1 - 现实世界", "content": "..." },
    ...
  ]
}
```

### 用户导出文件
```json
[
  { "title": "K29 - 巨乳/爆乳", "content": "..." }
]
```

导出文件可直接导入墨韵 XP World 模组使用。

## 版本管理

- 内置 `entries.json` 带 `version` 字段
- 用户首次加载数据时记录 `xp_last_reset_version` 到 localStorage
- 当服务器新版推送后，用户进入页面会在顶部看到"检测到新版"横幅，点击查看后可选择重置为最新内置版本（需在确认弹窗中二次确认，会放弃本地所有修改）

## 预设数据

预设数据来自 [XP大世界 / crazyWq](https://github.com/crazyWq) 的整理归档，由本项目筛选、整理、重新编号为 26 类连续序号体系。

## 维护

- 二改 & 维护：@葱油卷香辣
- 原工具作者：@crazyWq

详细开发说明见 [`DEVELOPMENT.md`](./DEVELOPMENT.md)。

## License

[CC BY-NC 4.0](./LICENSE) · 署名 + 非商业性使用