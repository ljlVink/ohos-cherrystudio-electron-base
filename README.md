# HarmonyOS Electron HAP

English | [简体中文](./README-CN.md)

A HarmonyOS Application Package (HAP) project based on the HarmonyOS platform that enables running Electron applications on HarmonyOS devices.

## Project Structure

```
ohos_electron_hap/
├── AppScope/                    # Application scope configuration
├── chromium/                    # Chromium module
├── electron/                    # Electron main module
├── web_engine/                  # Web engine component
├── hvigor/                      # Build tool configuration
├── build-profile.json5          # Project build configuration
├── hvigorfile.ts                # Build script
└── oh-package.json5             # Project dependencies configuration
```

## Quick Start

### Environment Requirements

- **DevEco Studio**: 4.0 or higher
- **HarmonyOS SDK**: API Level 10 or higher  
- **Node.js**: 16.x or higher
- **HDC Tool**: For device debugging and installation

### 1. Prepare Resource Files

Before starting the build, you need to prepare the following resources:

#### Electron Application Code
Place your Electron application code (compiled artifacts) into:
```
web_engine/src/main/resources/resfile/resources/app/
```

### 2. Build HAP Package

#### Using DevEco Studio
1. Open the project with DevEco Studio
2. Select **Build** → **Build Hap(s)/APP(s)** → **Build Hap(s)**
3. Or click the run button in the top right corner to launch the application

After building, the unsigned HAP package will be saved at:
```
electron/build/default/outputs/default/electron-default-unsigned.hap
```

### 3. Application Signing

To run normally on devices, the HAP package needs to be signed:

> Recommend using automatic signature verification
1. Apply for Huawei Developer Certificate
2. Configure signing information in DevEco Studio
3. Rebuild to generate signed HAP package

For detailed signing process, please refer to: [Application/Service Signing-DevEco Studio](https://developer.huawei.com/)

### 4. Installation and Running

#### Via DevEco Studio
Click the run button directly to install on device

#### Via Command Line
```bash
hdc app install <signed-hap-package-path>
# Example: hdc app install electron-default-signed.hap
```

## Application Customization

### Modify Application Name
Edit file: `electron/src/main/resources/zh_CN/element/string.json`
```json
{
  "string": [
    {
      "name": "EntryAbility_label",
      "value": "Your Application Name"
    }
  ]
}
```

### Replace Application Icon
Place new icon file into: `AppScope/resources/base/media/`

### Configure Startup Window Size
Edit `electron/src/main/module.json5`, add metadata in abilities:
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

## Permission Configuration

Application permissions are configured in the `requestPermissions` field of the `web_engine/src/main/module.json5` file.

### Basic Permissions (No Special Application Required)
- `ohos.permission.INTERNET` - Network access
- `ohos.permission.GET_NETWORK_INFO` - Get network information
- `ohos.permission.RUNNING_LOCK` - Background running lock
- `ohos.permission.PREPARE_APP_TERMINATE` - Application termination preparation

### Permissions Requiring Application
- `ohos.permission.CAMERA` - Camera permission
- `ohos.permission.MICROPHONE` - Microphone permission
- `ohos.permission.LOCATION` - Location permission
- `ohos.permission.READ_WRITE_DOWNLOAD_DIRECTORY` - Download directory access

## HarmonyOS Specific Features

### Floating Window
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
  transparent: true,  // Transparent window
  opacity: 0.5       // Opacity
});
```

### System Permission Request
```javascript
const { systemPreferences } = require('electron');

// Request camera permission
systemPreferences.requestSystemPermission('camera').then(granted => {
  console.log('Camera permission:', granted);
});

// Request directory permission
systemPreferences.requestDirectoryPermission(null).then(granted => {
  console.log('Directory permission:', granted);
});
```

## Debugging

### Renderer Process Debugging
```javascript
const { BrowserWindow } = require('electron');
const win = new BrowserWindow();
win.webContents.openDevTools();
```

### Main Process Debugging
1. Add debugging parameters in `web_engine/src/main/ets/components/WebWindow.ets`:
```typescript
let inspect = '--inspect=9229';
let vec_args = [..., inspect];
```

2. Configure port forwarding:
```bash
hdc fport tcp:9229 tcp:9229
```

3. Access in Chrome browser: `chrome://inspect`

## Application Data Directory

- User data stored by default at: `/data/storage/el2/base/files`
- Application installation directory: `/data/storage/el1/bundle`
- Database directory: `/data/storage/el2/database`

## Common Issues

### Build Failure
1. Check if SO library files are complete
2. Confirm Electron application code is correctly placed
3. Verify permission configuration is correct

### Third-party Library Compatibility
- **C++ addon**: Need to recompile for HarmonyOS platform adaptation
- **Platform detection**: Need to adapt `process.platform === 'ohos'`
- **Binary files**: May need to replace with HarmonyOS versions

### Permission Issues
If certain ACL permissions cannot be obtained, you can temporarily comment out related permissions:
```json
// "requestPermissions": [
//   {
//     "name": "ohos.permission.SYSTEM_FLOAT_WINDOW"
//   }
// ]
```

## Contributing

1. Fork this repository
2. Create a feature branch: `git checkout -b feature/your-feature`
3. Commit your changes: `git commit -am 'Add some feature'`
4. Push to the branch: `git push origin feature/your-feature`
5. Submit a Pull Request

## Contact Us

If you encounter issues or need support, please submit an Issue or contact the maintenance team.
