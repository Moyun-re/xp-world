# XP 大世界

XP 大世界是一个纯前端的设定 / 世界书条目管理工具。项目只有一个核心页面 `index.html`，可直接在浏览器中打开，也可作为静态站点部署到 GitHub Pages。

## 功能

- 新建、编辑、删除设定条目
- 按 A-Z 分类查看与折叠条目
- 搜索标题和正文
- 批量选择、批量导出、批量删除
- 导入 / 导出 JSON
- 暗色模式
- 自动保存到浏览器本地缓存

## 使用

### 在线访问

如果项目已部署到 GitHub Pages，直接打开站点地址即可使用：

```text
https://你的用户名.github.io/仓库名/
```

### 本地使用

1. 下载或克隆项目。
2. 用浏览器打开 `index.html`。
3. 开始编辑条目。

推荐使用 Chrome、Edge 或 Firefox。

## 数据导入导出

点击页面顶部的「导入」选择 `.json` 文件。当前支持：

1. 当前数组格式：

```json
[
  {
    "title": "A1 - 示例世界",
    "content": "这里是正文内容"
  }
]
```

2. 旧版 SillyTavern / 世界书风格格式：

```json
{
  "entries": {
    "0": {
      "comment": "A1 - 示例世界",
      "content": "这里是正文内容"
    }
  }
}
```

导入会替换当前列表。重要数据请先导出备份。

点击「导出」会下载当前条目 JSON；批量选择后也可以只导出选中条目。

## 部署

本项目不需要构建。GitHub Pages 可直接使用仓库根目录：

```text
index.html
README.md
DEVELOPMENT.md
```

推荐设置：

```text
Source: Deploy from a branch
Branch: main
Folder: /root
```

## 数据安全

本项目没有后端，也不会上传用户数据。数据只存在于：

- 当前浏览器页面内存
- 浏览器 `localStorage`
- 用户手动导出的 JSON 文件

注意：浏览器缓存可能因为清理数据、更换浏览器或更换设备而丢失。请在大量编辑后及时导出 JSON 备份。
