# Plugin for Media Sniffer Pro

媒体嗅探器 Pro 的官方插件仓库。脚本的「插件市场」会自动读取本仓库的 `plugins.json`。

## 使用方式

脚本 → 设置 → 插件 → 插件市场，首次打开会自动拉取；点右上角「刷新」可强制重新拉取。
找到想要的插件点「安装」即可，安装后立即生效，可在「已安装」里启停或编辑配置。

## 仓库结构

**一个插件一个文件**，放在 `plugins/` 目录下，文件名就是插件 id：

```
plugins/
  only-video-ext.json
  dedup-normalized.json
  skip-small-images.json
  ...
```

单个文件的内容就是一个插件对象（不是数组）：

```json
{
  "id": "only-video-ext",
  "name": "只要指定视频格式",
  "version": "1.0.0",
  "hooks": ["resource:keep"],
  "config": {},
  "code": "MS.hook('resource:keep', function (kind, url) { ... });"
}
```

脚本的读取顺序：

1. `plugins.json` / `manifest.json` / `index.json`（本仓库不使用）
2. `plugins/` 目录下的所有 `.json`（本仓库走这条路）
3. 根目录下的所有 `.json`

新增插件只需往 `plugins/` 里加一个 `<id>.json`，不用改任何索引文件。

## 插件格式

```json
{
  "id": "only-video-ext",
  "name": "只要指定视频格式",
  "description": "视频列表只保留指定扩展名的直链",
  "version": "1.0.0",
  "author": "zhjich123",
  "category": "enhance",
  "hooks": ["resource:keep"],
  "config": { "exts": [".mp4", ".webm"] },
  "code": "MS.hook('resource:keep', function (kind, url) { /* ... */ });"
}
```

- `id` 唯一，安装后靠它判断是否已装、是否要更新
- `version` 用 `1.2.3` 形式，仓库版本更高时市场会显示「更新」
- `category`：`enhance` / `download` / `parse` / `ui` / `tool`
- `config` 是插件自有配置，插件内用 `MS.config.get/set` 读写
- `code` 上限 64KB，全部插件合计上限 1MB

## 插件能用的接口

```js
MS.hook(name, fn)          // 注册钩子
MS.unhook(name, fn)        // 注销
MS.config.get(k, def)      // 读本插件配置
MS.config.set(k, v)        // 写并保存
MS.config.all()            // 整个配置副本
MS.toast(msg, type)        // info / success / warn / error
MS.log(...)                // 输出到脚本日志
MS.lang()                  // 当前界面语言
MS.version()               // 脚本版本号
MS.plugin()                // { id, version, name }
MS.media.list(kind)        // kind: img / video / audio / m3u8，返回地址数组
MS.media.count(kind)       // 该类型数量
MS.hooks()                 // 可用钩子名数组
```

> 不暴露 `GM_*`、`fetch`、`document.cookie`、`localStorage`、`eval` 以及脚本内部对象。

## 钩子一览

| 钩子 | 签名 | 说明 |
|---|---|---|
| `resource:keep` | `(kind, url, meta) → boolean\|undefined` | 返回 `false` 丢弃该资源 |
| `resource:seen` | `(key, url) → void` | 资源入库后通知 |
| `render:begin` | `() → void` | 列表开始渲染 |
| `parse:result` | `(result) → result\|undefined` | 改写解析结果 |
| `download:filename` | `(name, url, idx) → string\|undefined` | 改写下载文件名 |
| `download:quality` | `(pref) → pref\|undefined` | 改写码率偏好（high / medium / low / auto） |
| `download:progress` | `(progress) → void` | 下载进度 |
| `download:end` | `() → void` | 下载结束 |
| `render:cardName` | `(cur, url) → string\|undefined` | 改写卡片标题 |
| `render:translate` | `(kind, list) → void` | 渲染后可做翻译 |
| `render:layout` | `(currentLayout, kind) → {minCardWidth}\|null\|undefined` | 网格布局 |
| `render:footerButtons` | `(kind, list) → [{label, icon, onClick}]` | 追加底部按钮 |
| `theme:colors` | `(colors) → colors\|undefined` | 改写配色 |
| `shortcuts` | `() → [{mod, key, run}]` | 追加快捷键 |

**链式钩子**（`parse:result` / `download:filename` / `download:quality` / `render:cardName` / `theme:colors` / `render:layout`）统一是「第一个参数为当前值」，返回 `undefined` 表示不改变。

`kind` 取值为 `img` / `video` / `audio` / `m3u8`。

## 本仓库现有插件

| 插件 | 分类 | 作用 |
|---|---|---|
| 只要指定视频格式 | 增强 | 视频列表只保留指定扩展名 |
| 按地址去重 | 增强 | 丢弃归一化后重复的地址 |
| 过滤小图与占位图 | 增强 | 剔除小图、占位图、埋点像素 |
| 屏蔽指定域名 | 增强 | 命中域名（含子域）的资源不显示 |
| 文件名加站点前缀 | 下载 | 文件名前加来源域名 |
| 文件名加序号 | 下载 | 文件名前加补齐序号 |
| 合并下载优先高清 | 下载 | m3u8 合并选最高码率 |
| 更紧凑的卡片网格 | 界面 | 缩小卡片最小宽度 |
| 底部复制文件名按钮 | 工具 | 一键复制当前列表文件名 |
| 更深一档的暗色 | 界面 | 更深的面板暗色配色 |
| 复制链接快捷键 | 工具 | Alt+E 复制全部资源链接 |
| 过滤水印变体地址 | 解析 | 剔除带 watermark/logo 的地址 |

## 安全说明

插件代码只在你点击「安装 / 导入」后执行，脚本不会自动从网络下载并运行代码。
安装前脚本会扫描 `GM_`、`document.cookie`、`localStorage`、`eval(`、`fetch(` 等风险字样并提示。
单个插件连续出错 5 次会自动停用，不影响其它插件与主脚本。

## 许可证

GPL-3.0-or-later，与主脚本一致。
