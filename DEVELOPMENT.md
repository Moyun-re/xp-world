# XP World 开发文档

> 面向维护者与接手 AI。修改代码前先读完「维护原则」与「高风险区域」。

---

## 1. 概览

- **项目**：XP World v1.0
- **入口**：`index.html`（项目根目录，非 `src/`）
- **类型**：纯前端单文件应用
- **技术栈**：Vue 3 Composition API · Tailwind CSS CDN · localStorage · Blob/FileReader API
- **无构建工具**：不依赖 Node.js / npm / 任何打包器
- **运行**：浏览器直接打开（须通过 HTTP 服务，`file://` 下 `fetch('./entries.json')` 会失败）

---

## 2. 文件结构

```
XP World/
├── index.html       # 应用主程序（唯一改动文件）
├── entries.json     # 内置预设数据
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
├── header                顶部导航（汉堡 + 标题 + 版本 + 文件菜单 + 新建 + 暗色）
├── version banner        版本更新提醒（条件渲染）
├── main
│   ├── 遮罩              手机端侧边栏展开时可见
│   ├── aside             左侧目录（PC flex / 手机覆盖抽屉）
│   │   ├── 搜索 + 编号直达
│   │   ├── 分类分组（一级 -> 二级）
│   │   └── 底部批量操作悬浮栏
│   └── div               右侧编辑区（分类徽标 + 删除按钮 + 标题 + 正文）
├── Modal                 全局危险确认
├── New Entry Modal       新建条目
└── Toast                 提示
```

### 关键行为

| 区域 | 行为 |
|---|---|
| 侧边栏（PC ≥768px） | flex 并列，宽度 320px，可收起 |
| 侧边栏（手机 <768px） | 绝对定位覆盖抽屉，宽度 280px，编辑区宽度不受影响 |
| 选中条目 | 手机端 300ms 后自动收起侧边栏，PC 端保持展开 |
| 搜索框 | 模糊匹配标题/正文；输入如 `K29` 按回车 → 直达编号 |
| 新建 Modal | 强制选择一级 + 二级分类才可创建 |
| 文件菜单 | 顶部"文件"按钮下拉：导入… / 导出全部 / 重置为内置版本 |
| 导出全部 | 二次确认 Modal，导出 JSON 可直接导入墨韵 XP World 模组 |
| 导出选中 | 同上确认逻辑 |
| 删除（单条/批量） | 都走危险 Modal 确认 |
| 重置 | fetch 最新 entries.json 覆盖 localStorage，危险 Modal |

---

## 5. z-index 层级

| 区域 | z-index |
|---|---|
| 顶部导航 | 30 |
| 分类 sticky 头 | 20 |
| 底部批量操作栏 | 30 |
| 手机端侧边栏 | 40 |
| 手机端遮罩 | 30 |
| 新建条目 Modal | 50 |
| 全局确认 Modal | 60 |
| Toast | 50 |

不要随意调整，确认 / 新建 Modal 必须在所有内容之上。

---

## 6. 数据结构

### 内部条目
```js
[
  { id: '...', title: 'A1 - 现实世界', content: '...' }
]
```

### 分类映射（常量）
```js
const FIXED_LETTERS = ['A', ..., 'Z'];
const CATEGORY_DATA = { "A": { "name": "世界观-世界", "desc": "..." }, ... };
const PRIMARY_CATEGORIES = [
  { name: '世界观', letters: ['A', 'B', 'C'] },
  { name: '角色',   letters: ['D', 'E', 'F', 'G', 'H', 'I'] },
  { name: '恋物',   letters: ['J', 'K', 'L', 'M'] },
  { name: '剧情',   letters: ['N', 'O', 'P', 'Q', 'R', 'S'] },
  { name: 'SM',     letters: ['T', 'U', 'V'] },
  { name: '重口',   letters: ['W', 'X', 'Y', 'Z'] }
];
```

侧边栏分组、新建 Modal 二级下拉均来自 `PRIMARY_CATEGORIES`。`getCategoryName()` 返回二级名（如 "世界"，去掉一级前缀）。

### 标题分类规则
```js
/^([A-Za-z])\d+\s*-?/
```
不匹配的条目归入 `# 未分类`。

### ID 生成
```js
const genId = () => Date.now().toString() + Math.random().toString(36).slice(2, 7);
```
必须用此函数（纯时间戳连续创建会撞 ID）。

---

## 7. 数据加载与持久化

### 加载流程（`loadEntries()`）
1. 读 `localStorage.xp_entries`，有数据 → 用之
2. 无数据 → `fetch('./entries.json')`，装入并写入 `xp_last_reset_version`
3. fetch 失败 → entries 置空 + toast 报错

### 持久化
```js
watch(entries, saveEntries, { deep: true });
```
- `deep: true` 必须（v-model 直接改对象字段）
- 保存失败抛 `QuotaExceededError` 时 toast 提示导出备份

### 版本检查
```js
onMounted(() => { loadEntries(); checkVersion(); });
```
`checkVersion()` 带时间戳参数 fetch 内置 json 比对 `version`，不一致则顶部横幅提醒，点查看 → 危险 Modal → 用户确认 → 走 `doReset()`。

### 重置
```js
fetchBuiltIn() → entries.value = arrayTo内部(arr) → 更新 localStorage + VERSION_KEY
```
重置 = 完全覆盖本地数据，不可撤销。

---

## 8. 导入逻辑

### 支持格式
1. **纯数组** `[{title, content}]`
2. **内置包装** `{ "version": "...", "entries": [...] }`
3. **旧版 SillyTavern** `{ "entries": { "0": { comment, content } } }`

### 覆盖保护
- 当前已有数据 → 弹危险 Modal 确认
- 当前空 → 直接覆盖

不支持合并导入（合并需处理 ID/标题去重，未实现）。

---

## 9. 导出逻辑

```js
executeExport(dataToExport, filenamePrefix)
```
- 导出格式：`[{title, content}]`，不含 `id`
- 文件名：`XP大世界_全量_YYYY-MM-DD.json` 或 `XP大世界_选中导出_YYYY-MM-DD.json`
- UTF-8 BOM 头，可直接被墨韵模组识别

---

## 10. 高风险区域

| 改动 | 风险 |
|---|---|
| `#app` 根节点 | Modal/Toast 出去会编译失败 |
| z-index | Modal 必须永远在内容之上 |
| `STORAGE_KEY = 'xp_entries'` | 改 key 老用户丢数据，须写迁移 |
| `genId` | 改纯时间戳会撞 key |
| 删除操作 | 须同步维护 `entries / selectedEntry / selectedIds` |
| `watch` 去掉 `deep:true` | 编辑不保存 |
| `entries.json` 路径 | 必须与 `index.html` 同目录，GitHub Pages 需要一起部署 |

---

## 11. 测试清单

每次改动后过一遍：

- [ ] `python -m http.server 8000` 下首次加载（清 localStorage）
- [ ] 有 localStorage 时优先用本地数据
- [ ] 侧边栏手机抽屉遮罩 click 能关
- [ ] 文件菜单点外部自动关闭
- [ ] 导入旧版 JSON 会弹覆盖确认
- [ ] 新建未选全分类时创建按钮 disabled
- [ ] 删除单条 / 批量都有 Modal
- [ ] 导出全部 / 导出选中都有 Modal
- [ ] 搜索 K29 回车直达，未匹配有提示
- [ ] 暗色模式切换 + 刷新保持
- [ ] 编辑后刷新数据仍在
- [ ] 新版推送后版本横幅出现 + 重置流程正常

---

## 12. AI 接手 Prompt

```
你在维护 XP World v1.0。
纯前端单文件 HTML 项目，入口 index.html，技术栈 Vue 3 + Tailwind CDN + localStorage。
内置预设数据在 entries.json（带 version 字段）。
不要大面积重构，不要破坏 #app 根节点 / Modal / Toast / z-index / 暗色模式。
危险操作统一走 openModal，禁用 window.confirm。
修改后跑测试清单（见 DEVELOPMENT.md §11）。
```

---

## 13. 历史变更记录

### v1.0（2026-07-19）
- 数据体系重构：26 类三级目录（一级分组 → A-Z → 条目），全部连续编号
- 内置预设：`entries.json` 带 version，与 index.html 同目录部署
- 加载三级回退：localStorage 优先 → fetch entries.json → 空 + 报错
- 版本管理：版本不一致时顶部横幅提醒 + 确认 Modal 重置
- 顶部 UI：`文件` 下拉菜单（导入/导出全部/重置为内置版本）+ 新建 + 暗色
- 新建条目：两级下拉（一级分类 -> 二级 A-Z），自动编号，只填名称
- 手机端：侧边栏改为覆盖抽屉，编辑区宽度不受影响；选中条目 300ms 后自动收起
- 搜索：输入编号（如 K29）按回车直达
- 危险确认：所有导入覆盖、删除、重置均走自定义 Modal
- `content` 初始为空，不再注入旧版 ST 酒馆四段模板
- 旧版 ST 格式导入 fallback：`comment || key`

### v0.9（2026-06-20，原作者 crazyWq）
- 初版：纯前端工具，A-Z 二级分类，localStorage 持久化
- 旧版 ST 格式导入 + 数组格式导出