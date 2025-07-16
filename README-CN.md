# HarmonyOS Electron HAP

[English](./README.md) | 简体中文

这是一个基于 HarmonyOS 平台的 Electron 应用程序包（HAP）项目，支持在鸿蒙设备上运行 Electron 应用。

## 项目结构

```
ohos_electron_hap/
├── AppScope/                    # 应用范围配置
├── chromium/                    # Chromium 模块
├── electron/                    # Electron 主模块
├── web_engine/                  # Web 引擎组件
├── hvigor/                      # 构建工具配置
├── build-profile.json5          # 项目构建配置
├── hvigorfile.ts                # 构建脚本
└── oh-package.json5             # 项目依赖配置
```

## 快速开始

### 环境要求

- **DevEco Studio**: 4.0 或更高版本
- **HarmonyOS SDK**: API Level 10 或更高
- **Node.js**: 16.x 或更高版本
- **HDC工具**: 用于设备调试和安装

### 1. 准备资源文件

在开始构建之前，需要准备以下资源：

#### Electron 应用代码
将您的 Electron 应用代码（编译后的产物）放入：
```
web_engine/src/main/resources/resfile/resources/app/
```

### 2. 构建 HAP 包

#### 使用 DevEco Studio
1. 用 DevEco Studio 打开项目
2. 选择 **Build** → **Build Hap(s)/APP(s)** → **Build Hap(s)**
3. 或点击右上角的运行按钮启动应用

构建完成后，未签名的 HAP 包将保存在：
```
electron/build/default/outputs/default/electron-default-unsigned.hap
```

### 3. 应用签名

为了在设备上正常运行，需要对 HAP 包进行签名：

> 建议使用自动签名验证
1. 申请华为开发者证书
2. 在 DevEco Studio 中配置签名信息
3. 重新构建生成已签名的 HAP 包

详细签名流程请参考：[应用/服务签名-DevEco Studio](https://developer.huawei.com/)

### 4. 安装和运行

#### 通过 DevEco Studio
直接点击运行按钮安装到设备

#### 通过命令行
```bash
hdc app install <已签名hap包路径>
# 示例: hdc app install electron-default-signed.hap
```

## 应用定制

### 修改应用名称
编辑文件：`electron/src/main/resources/zh_CN/element/string.json`
```json
{
  "string": [
    {
      "name": "EntryAbility_label",
      "value": "您的应用名称"
    }
  ]
}
```

### 替换应用图标
将新图标文件放入：`AppScope/resources/base/media/`

### 配置启动窗口大小
编辑 `electron/src/main/module.json5`，在 abilities 中添加 metadata：
```json
"metadata": [
  {
    "name": "ohos.ability.window.height",
    "value": "800"
  },
  {
    "name": "ohos.ability.window.width",
    "value": "800"
  },
  {
    "name": "ohos.ability.window.left",
    "value": "center"
  },
  {
    "name": "ohos.ability.window.top",
    "value": "center"
  }
]
```

## 权限配置

应用权限在 `web_engine/src/main/module.json5` 文件的 `requestPermissions` 字段中配置。

### 基础权限（无需特殊申请）
- `ohos.permission.INTERNET` - 网络访问
- `ohos.permission.GET_NETWORK_INFO` - 获取网络信息
- `ohos.permission.RUNNING_LOCK` - 后台运行锁
- `ohos.permission.PREPARE_APP_TERMINATE` - 应用终止准备

### 需要申请的权限
- `ohos.permission.CAMERA` - 相机权限
- `ohos.permission.MICROPHONE` - 麦克风权限
- `ohos.permission.LOCATION` - 位置权限
- `ohos.permission.READ_WRITE_DOWNLOAD_DIRECTORY` - 下载目录访问

## HarmonyOS 特有功能

### 悬浮窗
```javascript
const { BrowserWindow } = require('electron');

let floatWindow = new BrowserWindow({
  windowInfo: {
    type: 'floatWindow'  // mainWindow, subWindow, floatWindow
  },
  parent: mainWindow,
  x: 100,
  y: 100,
  width: 800,
  height: 600,
  transparent: true,  // 透明窗口
  opacity: 0.5       // 透明度
});
```

### 系统权限请求
```javascript
const { systemPreferences } = require('electron');

// 请求相机权限
systemPreferences.requestSystemPermission('camera').then(granted => {
  console.log('Camera permission:', granted);
});

// 请求目录权限
systemPreferences.requestDirectoryPermission(null).then(granted => {
  console.log('Directory permission:', granted);
});
```

## 调试

### 渲染进程调试
```javascript
const { BrowserWindow } = require('electron');
const win = new BrowserWindow();
win.webContents.openDevTools();
```

### 主进程调试
1. 在 `web_engine/src/main/ets/components/WebWindow.ets` 中添加调试参数：
```typescript
let inspect = '--inspect=9229';
let vec_args = [..., inspect];
```

2. 配置端口转发：
```bash
hdc fport tcp:9229 tcp:9229
```

3. 在 Chrome 浏览器中访问：`chrome://inspect`

## 应用数据目录

- 用户数据默认存储在：`/data/storage/el2/base/files`
- 应用安装目录：`/data/storage/el1/bundle`
- 数据库目录：`/data/storage/el2/database`

## 常见问题

### 构建失败
1. 检查 SO 库文件是否完整
2. 确认 Electron 应用代码已正确放置
3. 验证权限配置是否正确

### 三方库兼容性
- **C++ addon**: 需要重新编译适配鸿蒙平台
- **平台检测**: 需要适配 `process.platform === 'ohos'`
- **二进制文件**: 可能需要替换为鸿蒙版本

### 权限问题
如果某些 ACL 权限无法获得，可以暂时注释掉相关权限：
```json
// "requestPermissions": [
//   {
//     "name": "ohos.permission.SYSTEM_FLOAT_WINDOW"
//   }
// ]
```

## 贡献指南

1. Fork 本仓库
2. 创建功能分支：`git checkout -b feature/your-feature`
3. 提交更改：`git commit -am 'Add some feature'`
4. 推送到分支：`git push origin feature/your-feature`
5. 提交 Pull Request

## 联系我们

如遇到问题或需要支持，请提交 Issue 或联系维护团队。
