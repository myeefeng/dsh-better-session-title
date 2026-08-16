# Better Session Title（dsh-better-session-title）

[English](README.en.md) · [简体中文](README.md)

![npm version](https://img.shields.io/npm/v/dsh-better-session-title) ![npm downloads](https://img.shields.io/npm/dw/dsh-better-session-title) ![license](https://img.shields.io/npm/l/dsh-better-session-title)

> 面向 **DeepSeek Harness** 的会话顶栏增强插件：把顶栏中央的原生会话标题替换为「工作区 / 会话」两级面包屑，直接在顶栏切换工作区与会话——**纯 UI 增强，不引入新能力，不改任何系统源码**。

支持 **Windows · macOS · Linux**（DSH Web GUI）——纯 JavaScript，无原生依赖。

```sh
dsh plugin --profile web add dsh-better-session-title
```

**快速上手** · [为什么做这个插件](#为什么做这个插件) · [主要功能](#主要功能) · [安装](#安装) · [使用](#使用) · [与 basrs-sidebar 的关系](#与-basrs-sidebar-的关系) · [二次开发](#二次开发) · [许可证](#许可证)

---

## 为什么做这个插件

DSH 原生顶栏中央显示会话标题（层级面包屑），但工作区/会话的切换入口在左侧边栏——侧边栏收起时，切换会话或工作区需要额外的点击才能触达。Better Session Title 把切换入口直接放到顶栏：

- **工作区下拉** —— 展开可见全部工作区及其会话数，一键切换；可「添加工作区」（目录选择器）或删除。
- **会话下拉** —— 展开可见当前工作区全部会话及相对更新时间，一键切换；可「新建会话」。

并且顶栏的行内操作全部复用 DSH 原生能力：重命名工作区/会话、删除工作区、分支会话、归档会话，与原生侧边栏行为完全一致。

## 主要功能

**两级面包屑** —— 顶栏左侧以「工作区 / 会话」形式展示当前位置，两侧各自展开下拉列表；空间不足时自动折叠为图标，悬停/点击仍可用。

**工作区下拉** —— 工作区列表（含会话数元信息）、添加工作区（原生目录选择器）、行内更多：重命名 / 删除工作区。

**会话下拉** —— 当前工作区的会话列表（含相对时间元信息）、新建会话、行内更多：重命名 / 分支 / 归档会话。

**原生一致的交互细节**：

- 行内元信息靠行尾，悬停行时被「更多」按钮覆盖；触屏设备常显按钮、隐藏元信息。
- 下拉列表为自绘层，样式逐值复刻原生 `Menu.module.css`（行高 40、padding 8/10、圆角 10、同款 shadow/边框 token）。
- 更多菜单、重命名/删除对话框全部使用原生 `Menu` / `Modal` / `Button` / `Input`（autoFocus、全选、输入法组合 Enter 防误触、工作区重复名校验、错误行）。
- 更多菜单跟随其所在下拉同生命周期：下拉关闭（点击外部 / Escape / 选择 / 切换）时菜单一并关闭，不残留孤立菜单。
- 内置 zh/en 双语言字典，跟随 DSH 语言切换实时生效。

## 安装

`dsh-better-session-title` 是一个**组合包（bundle）**——以 npm 包形式分发的配置层（见[官方插件文档](https://deepseek-harness.github.io/deepseek-harness/develop/basic/publish)）。用 `dsh plugin` 命令安装进 profile：它会自动链接包并追加到 `dsh.profile.bundles`：

```sh
# 从 npm 注册表
dsh plugin --profile web add dsh-better-session-title

# 或本地目录 / 打包好的 tarball
dsh plugin --profile web add ./dsh-better-session-title
dsh plugin --profile web add ./dsh-better-session-title-1.0.0.tgz

# 或直接从 git（纯 JS 包无需构建步骤）
dsh plugin --profile web add github:xinspark/dsh-better-session-title#<sha>
```

然后启动 Web UI：`dsh web`（或 `dsh --profile web`）。首次 `add` 会自动用 `@deepseek-ai/dsh-base` 初始化 profile。

卸载：`dsh plugin --profile web remove dsh-better-session-title`（同时移除依赖与组合层）。

插件的 bundle 补丁（`cordis.patch.yml`）会自动注册 host 与 client 两半。客户端按请求加载：客户端改动刷新页面即可，host 改动需要重启。

## 使用

安装后，打开任意会话，顶栏中央即显示「工作区 / 会话」面包屑：

- 点击**工作区**一侧：展开工作区下拉（顶部为「添加工作区」）。
- 点击**会话**一侧：展开会话下拉（顶部为「新建会话」）。
- 悬停列表行：行尾显示「更多」按钮（重命名 / 分支 / 归档等，按行类型而定）。
- 选择行即切换；点击空白处或按 `Esc` 关闭下拉。

## 与 basrs-sidebar 的关系

[basrs-sidebar](https://github.com/xinspark/DSH-Basic-Right-Sidebar) 也提供会话顶栏的工作区/会话面包屑。**两个插件都注册同一顶栏槽位，请二选一使用**：同时安装时，可在 basrs-sidebar 的设置中关闭其面包屑，仅保留本插件的顶栏面包屑。

## 二次开发

- **`index.js`** —— host 半：最小空实现（纯 UI 插件，所有逻辑在浏览器半）。
- **`lib/client.js`** —— 浏览器半（UMD，注册进 DSH `clientModules`）。结构自上而下：
  1. 本地基础工具：样式注入 / 定时器。
  2. **双语字典**（`BST_ZH` / `BST_EN`）—— 注册进 `ctx.locale`，跟随语言切换；`bstT()` 带 fallback。
  3. **自绘下拉列表** `BstList` —— 样式复刻原生 Menu；定位避让视口边缘；pointerdown 外部关闭（不用 mousedown，避免吃掉行内 click）。
  4. `WorkspaceTitleBreadcrumb` —— 注册进 `conversation.session.header.actions`（`order: -20`，顶栏最左）；紧凑模式自动折叠；更多菜单与下拉同生命周期；重命名/删除走原生 Modal。
  5. `apply(ctx)` —— 样式注入 + 槽位注册；服务调用与 ui-workspace 一致（`ctx.workspaces` / `ctx.sessions`）。

扩展速查：

- **新增行操作** —— 在 `wsListItems` / `ssListItems` 的 `actions` 数组加一项（`{ id, label, icon?, danger?, action }`）。
- **新增文案** —— 在 `BST_ZH` / `BST_EN` 字典加 key。
- **调整下拉样式** —— 修改 `insertStyle` 中的 `.bst-item*` 规则（token 与原生 Menu 一致）。

## 许可证

[MIT](LICENSE) —— 完全开源免费。

Better Session Title 是 DeepSeek Harness 的社区插件，**并非 DeepSeek 官方产品**。
