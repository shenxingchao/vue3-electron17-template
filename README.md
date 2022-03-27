# Vue 3 + Typescript + Vite + electron17
**需要的拿走**

nodejs版本 > 16.3.2

[快速开始](https://www.electronjs.org/zh/docs/latest/tutorial/quick-start)

[参考搭建](https://dev.to/brojenuel/vite-vue-3-electron-5h4ohttps://dev.to/brojenuel/vite-vue-3-electron-5h4o)

注意运行electron需要等待vite运行完毕，不然白屏

**创建vite app**

```
yarn create @vitejs/app  project-name --template vue-ts
```

**安装依赖**

```
cd project-name
yarn
```

**启动**

```
yarn dev
```

**安装electron**

```
yarn add --dev electron
```

**依次安装依赖**

```
yarn add -D concurrently cross-env electron electron-builder wait-on
```

concurrently：阻塞运行多个命令，`-k`参数用来清除其它已经存在或者挂掉的进程

cross-env：跨环境设置环境变量包

wait-on：等待资源，此处用来等待url可访问

**src文件夹下依次建立文件background.ts 和 preload.ts**

background.ts

```ts
//主进程
const { app, BrowserWindow } = require("electron");
const path = require("path");
const loadUrl =
  process.env.NODE_ENV == "development"
    ? "http://localhost:3000"
    : `file://${path.join(__dirname, "../dist/index.html")}`;

function createWindow() {
  const win = new BrowserWindow({
    width: 800,
    height: 600,
    webPreferences: {
      preload: path.join(__dirname, "preload.js"),
    },
  });

  win.loadURL(loadUrl);
}

app.whenReady().then(() => {
  createWindow();

  app.on("activate", () => {
    if (BrowserWindow.getAllWindows().length === 0) {
      createWindow();
    }
  });
});

app.on("window-all-closed", () => {
  if (process.platform !== "darwin") {
    app.quit();
  }
});

```

preload.js

```ts
window.addEventListener('DOMContentLoaded', () => {
    const replaceText = (selector, text) => {
      const element = document.getElementById(selector)
      if (element) element.innerText = text
    }

    for (const type of ['chrome', 'node', 'electron']) {
      replaceText(`${type}-version`, process.versions[type])
    }
  })
```

**配置文件vite.config.ts和package.json**

```ts
import { defineConfig } from 'vite'
import vue from '@vitejs/plugin-vue'

// https://vitejs.dev/config/
export default defineConfig({
  base: './',//新增
  plugins: [vue()],
})
```

```json
{
  "name": "vue3-electron17-template",
  "private": true,
  "version": "0.0.0",
  "main": "src/background.ts",
  "scripts": {
    "dev": "vite",
    "build": "vue-tsc --noEmit && vite build",
    "preview": "vite preview",
    "electron": "wait-on tcp:3000 && cross-env NODE_ENV=development electron .",
    "electron:serve": "concurrently -k \"yarn dev\"  \"yarn electron\"",
    "electron:builder": "electron-builder",
    "electron:build": "yarn build && yarn electron:builder"
  },
  "dependencies": {
    "vue": "^3.2.25"
  },
  "devDependencies": {
    "@vitejs/plugin-vue": "^2.2.0",
    "concurrently": "^7.0.0",
    "cross-env": "^7.0.3",
    "electron": "^17.2.0",
    "electron-builder": "^22.14.13",
    "typescript": "^4.5.4",
    "vite": "^2.8.0",
    "vue-tsc": "^0.29.8",
    "wait-on": "^6.0.1"
  },
  "build": {
    "appId": "com.org.vue3-electron17-template",
    "productName": "vue3-electron17-template",
    "copyright": "Copyright © sxc",
    "directories": {
      "buildResources": "./assets_electron",
      "output": "./dist_electron"
    },
    "win": {
      "target": [
        "nsis"
      ],
      "icon": "./public/favicon.ico"
    },
    "nsis": {
      "oneClick": false,
      "allowToChangeInstallationDirectory": true,
      "perMachine": true,
      "allowElevation": true,
      "installerIcon": "./public/favicon.ico",
      "uninstallerIcon": "./public/favicon.ico",
      "installerHeaderIcon": "./public/favicon.ico",
      "deleteAppDataOnUninstall": true,
      "createDesktopShortcut": true,
      "createStartMenuShortcut": false,
      "shortcutName": "vue3-electron17-template"
    },
    "publish": [
      {
        "provider": "generic",
        "url": "http://xxx.xxx.com/download/"
      }
    ]
  }
}
```