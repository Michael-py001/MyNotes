# 20220613-新版Electron如何使用remote模块

> [GitHub - electron/remote: Bridge JavaScript objects from the main process to the renderer process in Electron.](https://github.com/electron/remote)
>
> [重大更改 | Electron (electronjs.org)](https://www.electronjs.org/zh/docs/latest/breaking-changes#已废弃-remote-模块)

## 安装

```bash
$ npm install --save @electron/remote
```



## 主进程

### 初始化

```js
// main process
import remoteMain from '@electron/remote/main'
remoteMain.initialize()
```

根据你使用的electron版本进行配置

### electron >= 14.0.0

> **Note:** In `electron >= 14.0.0`, you must use the new `enable` API to enable the remote module for each desired `WebContents` separately: `require("@electron/remote/main").enable(webContents)`.

### electron < 14.0.0

> ​				In `electron < 14.0.0`, `@electron/remote` respects the `enableRemoteModule` WebPreferences value. You must pass `{ webPreferences: { enableRemoteModule: true } }` to the constructor of `BrowserWindow`s that should be granted permission to use `@electron/remote`.

我的版本是**electron19.0.5** ，写以下配置

```js
win = new BrowserWindow({
    title: "Li-Browser",
    frame:false,
		thickFrame: true,
    webPreferences: {
      nativeWindowOpen: false,
      preload: join(__dirname, "../preload/index.cjs"),
      nodeIntegration: true,
      contextIsolation: false,
      webviewTag : true,
			enableRemoteModule: true
    },
  })
remoteMain.enable(win.webContents) //开启允许remote
```

## 渲染进程

```js
import { getCurrentWindow } from '@electron/remote'
const BrowserWindow = getCurrentWindow()
```

以下是@electron/remote导出的所有模块

```js
import * as Electron from 'electron';

export {
  ClientRequest,
  CommandLine,
  Cookies,
  Debugger,
  Dock,
  DownloadItem,
  IncomingMessage,
  MessagePortMain,
  ServiceWorkers,
  TouchBarButton,
  TouchBarColorPicker,
  TouchBarGroup,
  TouchBarLabel,
  TouchBarOtherItemsProxy,
  TouchBarPopover,
  TouchBarScrubber,
  TouchBarSegmentedControl,
  TouchBarSlider,
  TouchBarSpacer,
  WebRequest,
} from 'electron/main';

// Taken from `RemoteMainInterface`
export var app: Electron.App;
export var autoUpdater: Electron.AutoUpdater;
export var BrowserView: typeof Electron.BrowserView;
export var BrowserWindow: typeof Electron.BrowserWindow;
export var clipboard: Electron.Clipboard;
export var contentTracing: Electron.ContentTracing;
export var crashReporter: Electron.CrashReporter;
export var desktopCapturer: Electron.DesktopCapturer;
export var dialog: Electron.Dialog;
export var globalShortcut: Electron.GlobalShortcut;
export var inAppPurchase: Electron.InAppPurchase;
export var ipcMain: Electron.IpcMain;
export var Menu: typeof Electron.Menu;
export var MenuItem: typeof Electron.MenuItem;
export var MessageChannelMain: typeof Electron.MessageChannelMain;
export var nativeImage: typeof Electron.nativeImage;
export var nativeTheme: Electron.NativeTheme;
export var net: Electron.Net;
export var netLog: Electron.NetLog;
export var Notification: typeof Electron.Notification;
export var powerMonitor: Electron.PowerMonitor;
export var powerSaveBlocker: Electron.PowerSaveBlocker;
export var protocol: Electron.Protocol;
export var screen: Electron.Screen;
export var session: typeof Electron.session;
export var ShareMenu: typeof Electron.ShareMenu;
export var shell: Electron.Shell;
export var systemPreferences: Electron.SystemPreferences;
export var TouchBar: typeof Electron.TouchBar;
export var Tray: typeof Electron.Tray;
export var webContents: typeof Electron.webContents;
export var webFrameMain: typeof Electron.webFrameMain;

// Taken from `Remote`
export function getCurrentWebContents(): Electron.WebContents;
export function getCurrentWindow(): Electron.BrowserWindow;
export function getGlobal(name: string): any;
export var process: NodeJS.Process;
export var require: NodeJS.Require;

```

