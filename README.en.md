# web-version-checker

[![npm version](https://img.shields.io/npm/v/web-version-checker.svg)](https://www.npmjs.com/package/web-version-checker)
[![license](https://img.shields.io/npm/l/web-version-checker.svg)](https://github.com/yourusername/web-version-checker/blob/main/LICENSE)

A pure front-end solution for automatic version update detection and refresh notification.

[简体中文](./README.md) | English

## Features
- Front-end only, no backend needed
- Auto or manual detection modes
- Customizable UI (confirm or custom dialog)
- Smart pause when page hidden
- Supports ESM / CJS / UMD

## Install
```bash
npm install web-version-checker
# or
yarn add web-version-checker
# or
pnpm add web-version-checker
```

## Usage (3 common cases)

### 1) Native HTML + JS (UMD)
```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Version Check Demo</title>
</head>
<body>
  <script src="https://unpkg.com/web-version-checker/dist/index.umd.js"></script>
  <script>
    WebVersionChecker.createUpdateNotifier({ pollingInterval: 60000 });
  </script>
</body>
</html>
```

### 2) Vue + TypeScript (main.ts)
```ts
import { createApp } from 'vue';
import App from './App.vue';
import { createUpdateNotifier, type UpdateNotifierOptions } from 'web-version-checker';

createApp(App).mount('#app');

if (import.meta.env.PROD) {
  const options: UpdateNotifierOptions = {
    pollingInterval: 60000,
    notifyType: 'custom',
    onUpdate: async () => confirm('New version found, refresh now?'),
    onDetected: () => console.log('New version detected')
  };
  createUpdateNotifier(options);
}
```

### 3) React + TypeScript (index.tsx)
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
  const options: UpdateNotifierOptions = { pollingInterval: 60000 };
  createUpdateNotifier(options);
}
```

## Options (UpdateNotifierOptions)
- pollingInterval: number | null
  - Polling interval in ms. Default 10000.
  - Set null or 0 to disable auto mode and use manual checks.
- notifyType: 'confirm' | 'custom'
  - Notification type. Default 'confirm'.
- onUpdate: () => boolean | Promise<boolean>
  - Custom prompt function. Return true to refresh.
- onDetected: () => void
  - Called when an update is detected.
- pauseOnHidden: boolean
  - Pause when page hidden. Default true.
- immediate: boolean
  - Start immediately. Default true (auto mode only).
- indexPath: string
  - Path to fetch page HTML. Default '/'.
- scriptRegex: RegExp
  - Regex to match script tags. Default /\<script.*src=["'](?<src>[^"']+)/gm
- debug: boolean
  - Print debug logs. Default false.

## API
- createUpdateNotifier(options?): WebVersionChecker

Instance methods:
- start(): start detection (auto)
- stop(): stop detection (auto)
- checkNow(): Promise<boolean> (silent check)
- checkUpdate(): Promise<boolean> (check and prompt)
- reset(): reset and stop

## How it works
1) Build changes script filenames in index.html (with hash)
2) Fetch latest index.html and extract script list
3) Compare lists; if changed, prompt to refresh
