# web-version-checker

[![npm version](https://img.shields.io/npm/v/web-version-checker.svg)](https://www.npmjs.com/package/web-version-checker)
[![license](https://img.shields.io/npm/l/web-version-checker.svg)](https://github.com/yourusername/web-version-checker/blob/main/LICENSE)

一个纯前端实现的版本更新自动检测与提示刷新插件，无需后端配合。

简体中文 | [English](./README.en.md)

## ✨ 特性
- 纯前端实现，无需后端
- 自动或手动检测两种模式
- 可自定义提示UI（confirm或自定义弹窗）
- 页面隐藏时智能暂停（可配置）
- 支持 ESM / CJS / UMD，多框架适配

## 📦 安装
```bash
npm install web-version-checker
# 或
yarn add web-version-checker
# 或
pnpm add web-version-checker
```

## 🚀 使用示例（三种常见场景）

### 1) 原生 HTML + JS（UMD）
```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
  <meta charset="UTF-8">
  <title>版本更新检测示例</title>
</head>
<body>
  <script src="https://unpkg.com/web-version-checker/dist/index.umd.js"></script>
  <script>
    // 默认自动轮询：每分钟检测一次
    WebVersionChecker.createUpdateNotifier({ pollingInterval: 60000 });
  </script>
</body>
</html>
```

### 2) Vue + TypeScript（main.ts）
```ts
import { createApp } from 'vue';
import App from './App.vue';
import { createUpdateNotifier, type UpdateNotifierOptions } from 'web-version-checker';

createApp(App).mount('#app');

// 仅生产环境启用
if (import.meta.env.PROD) {
  const options: UpdateNotifierOptions = {
    pollingInterval: 60000,
    notifyType: 'custom',
    onUpdate: async () => {
      return confirm('发现新版本，是否立即刷新？');
    },
    onDetected: () => {
      console.log('检测到新版本');
    }
  };
  createUpdateNotifier(options);
}
```

### 3) React + TypeScript（index.tsx）
```tsx
import React from 'react';
import ReactDOM from 'react-dom/client';
import App from './App';
import { createUpdateNotifier, type UpdateNotifierOptions } from 'web-version-checker';

ReactDOM.createRoot(document.getElementById('root') as HTMLElement).render(
  <React.StrictMode>
    <App />
  </React.StrictMode>
);

if (process.env.NODE_ENV === 'production') {
  const options: UpdateNotifierOptions = {
    pollingInterval: 60000,
    debug: false
  };
  createUpdateNotifier(options);
}
```

## ⚙️ 参数说明（UpdateNotifierOptions）
- pollingInterval: number | null
  - 轮询间隔（毫秒）。默认 10000。
  - 设为 null 或 0：禁用自动轮询，需手动触发检测（例如自己写 setInterval 调用 checkUpdate）。
- notifyType: 'confirm' | 'custom'
  - 提示方式。默认 'confirm'（浏览器确认框）。
  - 设为 'custom' 时需提供 onUpdate。
- onUpdate: () => boolean | Promise<boolean>
  - 自定义提示函数，返回 true 表示确认刷新，false 表示取消。
  - 在 notifyType='custom' 下生效。
- onDetected: () => void
  - 检测到更新时触发的回调（不影响刷新流程）。
- pauseOnHidden: boolean
  - 页面隐藏时是否暂停检测。默认 true。
- immediate: boolean
  - 是否立即开始检测。默认 true（仅自动轮询模式下有效）。
- indexPath: string
  - 拉取页面内容的路径。默认 '/'
- scriptRegex: RegExp
  - 提取 script 的正则。默认 /\<script.*src=["'](?<src>[^"']+)/gm
- debug: boolean
  - 是否打印调试日志。默认 false。

## 🧩 API
- createUpdateNotifier(options?: UpdateNotifierOptions): WebVersionChecker
  - 创建并返回检测器实例。

- 实例方法：
  - start(): 开始检测（仅自动模式）
  - stop(): 停止检测（仅自动模式）
  - checkNow(): Promise<boolean>
    - 静默检测，仅返回是否有更新，不弹窗。
  - checkUpdate(): Promise<boolean>
    - 手动检测并弹窗提示用户，用户确认后刷新。
  - reset(): 重置状态并停止检测。

## 🔍 工作原理（简述）
1) 每次构建后，index.html 中的 script 文件名会变化（通常带 hash）。
2) 插件定期拉取最新的 index.html，提取其中的 script 列表。
3) 与上一次记录对比，若不同则判定为版本更新，并提示刷新。
