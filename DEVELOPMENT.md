# XP World 开发文档

> 面向维护者与接手 AI。修改代码前先读完「维护原则」与「高风险区域」。

---

## 1. 概览

- **项目**：XP World v1.1（数据格式：`title` / `human` / `ai`）
- **入口**：`index.html`（项目根目录，非 `src/`）
- **类型**：纯前端单文件应用
- **技术栈**：Vue 3 Composition API · Tailwind CSS CDN · localStorage · Blob/FileReader API
- **无构建工具**：不依赖 Node.js / npm / 任何打包器
- **运行**：须通过 HTTP 服务（`file://` 下 `fetch('./entries.json')` 会失败）

---

## 2. 文件结构

```
XP World/
├── index.html       # 应用主程序（唯一改动文件）
├── entries.json     # 内置：{ version, entries: [{title, human, ai}] }
├── README.md
└── DEVELOPMENT.md
```

---

## 3. 维护原则

- **只做必要改动**，不为工程化而重构
- **不破坏根节点**：所有 Vue 模板必须在 `<div id="app">` 内
- **不重新引入原生弹窗**：危险操作统一走 `openModal`，禁用 `window.confirm / alert / prompt`
- **不动 z-index 层级**：见 §5
- **新增功能优先局部修改**，验证通过再提交

---

## 4. UI 结构

```
#app
├── header                汉堡 + 标题 + 版本 + 文件菜单 + 新建 + 暗色
├── version banner        新版本提醒（条件渲染）
├── main
│   ├── 遮罩              手机侧边栏展开时
│   ├── aside             三级目录 + 搜索 + 批量栏
│   └── editor            标题 + 词条说明(human) + 预注入提示词(ai)
├── Modal                 全局确认
├── New Entry Modal       新建
└── Toast
```

### 关键行为

| 区域 | 行为 |
|---|---|
| 侧边栏 PC | flex，宽 320px，可收起 |
| 侧边栏手机 | 覆盖抽屉，编辑区宽度不受挤 |
| 三级目录 | 一级 `PRIMARY_CATEGORIES`（6）→ 二级 A–Z → 条目；一级/二级可独立折叠 |
| 搜索 | 匹配 `title` / `human` / `ai`；`K29` 回车编号直达 |
| 新建 | 一级 + 二级字母必选；`human`/`ai` 初始为空串 |
| 编辑区 | 标签：**词条说明**（`human`）、**预注入提示词**（`ai`）；两框透明底、同风格 |
| 文件菜单 | 导入… / **导出维护数据**（全量）/ 重置为内置版本 |
| 批量栏 | **提示词导出**（选中）+ 删除；**无**维护导出 |
| 危险操作 | 导入覆盖、删除、重置 → `openModal` |

### 界面文案 ↔ 字段

| 界面 | 字段 | 说明 |
|------|------|------|
| 词条说明 | `human` | 给人看 |
| 预注入提示词 | `ai` | 给 AI / 提示词导出 |
| 导出维护数据 | 全量 `title+human+ai` | 右上角文件菜单 |
| 提示词导出 | 选中 `title+content`（content=`ai`） | 底部批量栏 |

---

## 5. z-index 层级

| 区域 | z-index |
|---|---|
| 顶部导航 | 30 |
| 分类 sticky 头 | 20–21 |
| 底部批量操作栏 | 30 |
| 手机端侧边栏 | 40 |
| 手机端遮罩 | 30 |
| 新建条目 Modal | 50 |
| 全局确认 Modal | 60 |
| Toast | 50 |

确认 / 新建 Modal 必须在所有内容之上。

---

## 6. 数据结构

### 内部条目
```js
[
  { id: '...', title: 'A1 - 现实世界', human: '...', ai: '...' }
]
```

### 分类常量
```js
const FIXED_LETTERS = ['A', /* ... */, 'Z'];
const CATEGORY_DATA = { "A": { "name": "世界观-世界", "desc": "..." }, /* ... */ };
const PRIMARY_CATEGORIES = [
  { name: '世界观', letters: ['A', 'B', 'C'] },
  { name: '角色',   letters: ['D', 'E', 'F', 'G', 'H', 'I'] },
  { name: '恋物',   letters: ['J', 'K', 'L', 'M'] },
  { name: '剧情',   letters: ['N', 'O', 'P', 'Q', 'R', 'S'] },
  { name: 'SM',     letters: ['T', 'U', 'V'] },
  { name: '重口',   letters: ['W', 'X', 'Y', 'Z'] }
];
```

侧边栏由 `primaryTree` 计算：`PRIMARY_CATEGORIES` × `groupedAndSortedData`。  
`getCategoryName(letter)` 返回二级短名（去掉一级前缀）。

### 标题分类规则
```js
/^([A-Za-z])\d+\s*-?/
```
不匹配 → `# 未分类`。

### ID
```js
const genId = () => Date.now().toString() + Math.random().toString(36).slice(2, 7);
```

---

## 7. 数据加载与持久化

### `loadEntries()`
1. `localStorage.xp_entries`
   - 合法新格式 → 用之
   - 整表为旧 `title/content` → 清空 + toast，再走内置
2. 否则 `fetch('./entries.json')`，写入 `xp_last_reset_version`
3. 失败 → 空 + toast

### 持久化
```js
watch(entries, saveEntries, { deep: true });
// 存：id, title, human, ai
```
`QuotaExceededError` 时 toast 提示先做维护导出备份。

### 版本 / 重置
- `checkVersion()` 比对内置 `version` 与本地 `VERSION_KEY`
- `doReset()`：`arrayToEntries` 覆盖本地，更新版本，关横幅

---

## 8. 导入

### 支持
1. `[{ title, human, ai }]`
2. `{ version, entries: [{ title, human, ai }] }`（可写 `VERSION_KEY`）

### 拒绝
- SillyTavern `entries` 对象 map（已删除）
- 旧 `[{ title, content }]` 主格式

有现网数据时 Modal 覆盖确认。不支持合并导入。

---

## 9. 导出

```js
executeExport(dataToExport, filenamePrefix, mode)
// mode: 'maintain' | 'ai'
```

| 入口 | mode | 形状 | 文件名前缀 |
|------|------|------|------------|
| 文件 → 导出维护数据 | `maintain` | 全量 `[{ title, human, ai }]` | `XP大世界_维护全量_` |
| 批量 → 提示词导出 | `ai` | 选中 `[{ title, content }]`，`content` = `ai` | `XP大世界_提示词选中_` |

- 不含运行时 `id`
- UTF-8 BOM
- 弹窗文案面向小白，避免堆 JSON 术语

---

## 10. 高风险区域

| 改动 | 风险 |
|---|---|
| `#app` 根节点 | Modal/Toast 出去会编译失败 |
| z-index | Modal 须在最上层 |
| `STORAGE_KEY = 'xp_entries'` | 改 key 须迁移 |
| `genId` | 纯时间戳会撞 ID |
| 删除 | 同步 `entries / selectedEntry / selectedIds` |
| `watch` 去掉 `deep:true` | 编辑不保存 |
| `entries.json` 路径 | 与 `index.html` 同目录部署 |
| 主库字段 | 勿再把 `content` 写回主库；`content` 仅出现在提示词导出 |

---

## 11. 测试清单

- [ ] HTTP 下清 localStorage 首次加载 → 内置条数与 version 正确
- [ ] 合法本地数据优先；旧 content localStorage 提示后回落
- [ ] 三级目录：一级展开后见 A–Z，再展开见条目
- [ ] 编辑区可见「词条说明」「预注入提示词」，样式一致，刷新仍在
- [ ] 导入维护数组 / 带 version 包装成功
- [ ] 导入旧 ST / 旧 content 失败且有提示
- [ ] 导出维护数据：全量 title+human+ai
- [ ] 提示词导出：仅选中，title+content，content===ai
- [ ] 文件菜单无「全量提示词导出」；批量栏无维护导出
- [ ] 编号直达、删除 Modal、重置、版本横幅、暗色

---

## 12. 内置数据同步工作流（维护 JSON → entries.json）

优化版维护文件（如仓库根目录 `XP大世界_优化版_ABC.json`）**不会**在编辑后自动进入前端。  
同步内置必须走脚本，且 **默认 dry-run，只有 `--apply` 才写盘**。

### 原则

| 规则 | 说明 |
|------|------|
| 源格式 | `[{ title, human, ai }]` 或 `{ version, entries: [...] }` |
| 目标 | `XP World/entries.json` → `{ version, entries: [{title,human,ai}] }` |
| 默认 | 只校验 + diff 预览，**不修改任何文件** |
| 写入 | 必须显式 `--apply` |
| version | 每次正式发布内置应 bump，便于用户端版本横幅 |
| 文档 | 可选 `--update-docs` 替换 README/DEVELOPMENT 中的旧 version 字符串 |
| AI/人工 | **禁止**在未获用户明确同意时执行 `--apply` |

### 脚本

```
XP World/scripts/sync_builtin_entries.py
```

### 命令（在仓库根目录）

```bash
# 1) 仅预览（应先跑）
python "XP World/scripts/sync_builtin_entries.py" \
  --source "XP大世界_优化版_ABC.json" \
  --version "2026-07-20-abc"

# 2) 用户确认 diff 后，再写入
python "XP World/scripts/sync_builtin_entries.py" \
  --source "XP大世界_优化版_ABC.json" \
  --version "2026-07-20-abc" \
  --apply

# 3) 写入并同步文档中的 version 字样
python "XP World/scripts/sync_builtin_entries.py" \
  --source "XP大世界_优化版_ABC.json" \
  --version "2026-07-20-abc" \
  --apply --update-docs
```

### 发布后用户侧

- 已有 localStorage 的用户：靠 `version` 横幅，或「重置为内置版本」
- 开发预览：清 `xp_entries` / 重置后刷新

### 后续扩类（如 ABCD）

- 源文件换成新的优化版合并 JSON 即可，流程相同
- 建议 version 体现范围，例如 `2026-08-01-abcd`

---

## 13. AI 接手 Prompt

```
你在维护 XP World v1.1。
纯前端单文件 HTML，入口 index.html，Vue 3 + Tailwind CDN + localStorage。
内置 entries.json：{ version, entries: [{title, human, ai}] }。
运行时：{ id, title, human, ai }。
界面：词条说明=human，预注入提示词=ai；侧边栏三级（6 一级→A-Z→条目）。
导出：右上角仅维护全量 title/human/ai；批量栏「提示词导出」选中 title/content(content=ai)。
同步内置数据：只用 XP World/scripts/sync_builtin_entries.py；默认 dry-run；
未获用户明确同意禁止 --apply。见 DEVELOPMENT.md §12。
危险操作走 openModal。已删 SillyTavern 与旧 content 正式导入。
改后跑 DEVELOPMENT.md §11。
```

---

## 14. 历史变更

### v1.1（2026-07-19）— 主数据 human/ai + 三级目录 + 导出收口
- 主字段：`title` / `human` / `ai`；内置由优化版 ABC 维护文件经同步脚本生成
- 删除 SillyTavern 导入；拒绝旧 content 主格式正式导入
- 维护导出：右上角全量；提示词导出：批量选中 only
- 编辑区双字段，标签「词条说明」「预注入提示词」
- 侧边栏真正三级：`PRIMARY_CATEGORIES` → 字母 → 条目
- 旧 localStorage content 检测后清空提示，不静默迁移
- 新增 `scripts/sync_builtin_entries.py`：维护 JSON → 内置 entries 的 dry-run / --apply 工作流

### v1.0（2026-07-19）
- 26 类体系、version 内置、文件菜单、新建两级、手机抽屉、编号直达、自定义 Modal

### v0.9（2026-06-20，crazyWq）
- 初版 A–Z + localStorage；ST 导入 + 数组导出
