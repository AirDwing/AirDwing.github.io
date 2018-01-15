---
title: "Willin: 使用 OS X 进行高效 JS 前后端开发"
date: 2018-01-15 07:45:53
categories: Dev
tags: [node.js, fe, be, dev]
---

## 高效工具

- Brew： 用于安装各类 *nix 依赖包和应用
- dnsmasq： Localhost下的泛域名指定（Brew 安装）
- LaunchRocket： Brew 应用 GUI 管理工具（Brew Cask 安装）
- iTerm + Oh My Zsh!： 高效 Shell 工具
- iStat Menus：系统监控
- CheatSheet： 按住 `⌘ command` 不放弹出快捷键菜单
- Moom： 窗口分屏工具
- iHosts： 本地域名解析管理工具
- RescueTime： 记录每天你到底只tm专注工作了几分钟（胆小勿入）

<!--more-->

## IDE 配置

### VS Code 插件

- Auto Close Tag：用于前端 HTML 模板
- Auto Rename Tag：用于前端 HTML 模板
- Can I Use：前端兼容性检测
- EditorConfig for VS Code： 必装
- ESlint： 必装
- Git Lens： git 辅助工具
- Path Intellisense： 引用路径智能提示
- TODO Highlight： 见字如面
- Vetur： 用于前端 Vue 项目
- vscode-proto3： 用于 ProtoBuf 项目

不推荐安装过多插件，多余插件不仅影响 IDE 速度，更可能造成冲突。建议只开启几个必须的插件，其他如 Vetur 只在特定项目内开启即可。

### 配置参考

只提出了一些关键的配置：

```js
// 将设置放入此文件中以覆盖默认设置
{
  // 编辑器自带的配置也需要设置，避免与 EditorConfig 冲突
  "editor.tabSize": 2,
  "editor.insertSpaces": true,
  // 实时格式化
  "editor.formatOnType": true,
  // 迷你地图
  "editor.minimap.enabled": true,
  // 折行
  "editor.wordWrap": "on",
  "extensions.autoUpdate": true,
  // 文件自动保存
  "files.autoSave": "afterDelay",
  "files.autoSaveDelay": 5000,
  // 左侧资源管理器隐藏以下文件/目录
  "files.exclude": {
    "**/.git": true,
    "**/.svn": true,
    "**/.hg": true,
    "**/.DS_Store": true,
    "**/bower_components": true,
    "**/tmp": true,
    "**/vendor": true,
    "**/node_modules": true,
    "**/dist": true,
    "**/.cache": true
  },
  // 默认 cmd 终端
  "terminal.external.osxExec": "iTerm.app",
  "telemetry.enableTelemetry": false,
  "telemetry.enableCrashReporter": false,
  "window.zoomLevel": 1,
  // 插件定义
  // 关闭自带 js 校验，避免与 ESLint 插件冲突
  "javascript.validate.enable": false,
  // ESLint 自动修复
  "eslint.autoFixOnSave": true,
  // ESLint 加入 Vue 格式支持
  "eslint.validate": [
    "javascript",
    "javascriptreact",
     { "language": "vue", "autoFix": true },
     { "language": "html", "autoFix": true }
  ]
}
```

### 键位设置

根据个人习惯及键盘布局进行优化，示例：

```js
[
  // 针对 HHKB 键盘优化
  { "key": "cmd+escape", "command": "workbench.action.terminal.toggleTerminal" },
  { "key": "cmd+shift+escape", "command": "workbench.action.showErrorsWarnings"},
  { "key": "alt+i", "command": "cursorUp", "when": "editorTextFocus" },
  { "key": "alt+j", "command": "cursorLeft", "when": "editorTextFocus" },
  { "key": "alt+k", "command": "cursorDown", "when": "editorTextFocus" },
  { "key": "alt+l", "command": "cursorRight", "when": "editorTextFocus" }
]
```

## 项目配置

创建项目应养成的几个习惯步骤：

1. 创建 .editorconfig 配置
2. 创建 .eslintrc 配置
3. 创建 .gitignore 配置
4. 创建 .babelrc 配置（*）

### .editorconfig 配置

通用，所有项目应都保持该配置一致：

```yml
#  See http://editorconfig.org/
root = true
[*]
indent_style = space
indent_size = 2
end_of_line = lf
charset = utf-8
trim_trailing_whitespace = true
insert_final_newline = true
```

### .eslintrc.yml 配置

通用，后端可直接使用：

```yml
root: true
extends: 'dwing'
```

在这个基础上可以进行一些扩展，如：

```yml
root: true
extends:
  - 'plugin:vue/recommended'
  - 'dwing'
rules:
  import/extensions: [2, 'always', { js: 'never', vue: 'always' }]
settings:
  import/resolver:
    webpack:
      config: 'build/webpack.config.js'
```

### .gitignore 配置

默认配置：

```
node_modules/
.nyc_output/
.cache/
demo/
dist/

*.log
*.log.*
.DS_Store
dump.rdb
coverage.lcov
```

### .babelrc 配置

默认配置：

```json
{
  "presets": [
    ["env", {
      "targets": {
        "browsers": ["last 2 versions"]
      }
    }]
  ],
  "plugins": [
    ["transform-runtime", {
      "helpers": false,
      "polyfill": false,
      "regenerator": true,
      "moduleName": "babel-runtime"
    }]
  ]
}
```

默认需要安装 `babel-preset-env`、`babel-plugin-transform-runtime`、`babel-runtime`。

添加插件务必谨慎，任何一个插件都可能导致一个几百字节的 `Hello World` 变成几十甚至几百 KB。
