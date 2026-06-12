---
name: xiaomi-miband10-vela-js
description: Xiaomi Mi Band 10 app development using the Vela JS Quick App framework
version: 1.0.0
tags: [xiaomi, miband, wearable, vela-js, quick-app, iot, javascript]
platforms: [linux, macos, windows]
---

# Xiaomi Mi Band 10 - Vela JS Development

Quick App development for Xiaomi Mi Band 10 using the Xiaomi Vela JS platform (based on OpenVela RTOS).

## When to use this skill

- Develop new applications for Mi Band 10
- Communicate with Android companion app via BLE
- Use device sensors (accelerometer, barometer)
- Implement UI on 1.72" AMOLED screen (212×520px)
- Integrate with XMS Wearable SDK Android

## Technology Stack

| Layer | Technology |
|-------|-----------|
| Device Runtime | Xiaomi HyperOS 2 / OpenVela (NuttX-based RTOS) |
| App Language | JavaScript (ES6+) |
| Template | .ux files (Vue SFC-like syntax: `<template>`, `<script>`, `<style>`) |
| IDE | AIoT-IDE (VS Code fork) |
| Build Output | .rpk (installable via Mi Fitness / AIoT-IDE) |
| Android Communication | system.interconnect (automatic BLE, via XMS Wearable SDK) |
| Android Companion SDK | xms-wearable-lib_1.4_release.aar |

## Platform Overview

### Supported APIs on Mi Band 10

| API Module | Status | Notes |
|-----------|--------|-------|
| `@system.battery` | ✅ Supported | Mi Band 10 is unique among bands to support this |
| `@system.sensor` | ✅ Supported | Accelerometer + Pressure (NOT compass) |
| `@system.vibrator` | ✅ Supported | `vibrate({mode:'short'\|'long'})` |
| `@system.device` | ✅ Supported | `getInfo()`, `getTotalStorage()`, `getAvailableStorage()` |
| `@system.storage` | ✅ Supported | Key-value string storage |
| `@system.file` | ✅ Supported | Full file operations (read/write/mkdir/list/delete) |
| `@system.router` | ✅ Supported | Page navigation |
| `@system.app` | ✅ Supported | `getInfo()`, `canIUse()`, `terminate()` |
| `@system.brightness` | ✅ Supported | `getValue()`, `setValue()`, `setKeepScreenOn()` |
| `@system.interconnect` | ✅ Supported | Phone communication (requires companion app) |
| `@system.configuration` | ✅ Supported | `getLocale()` |
| `@system.prompt` | ✅ Supported | `showToast()` |
| `@system.event` | ✅ Supported (API Level 4+) | Pub/sub event system |
| `@system.crypto` | ✅ Supported | Encryption |
| `@system.network` | ❌ Not supported | No WiFi on band |
| `@system.bluetooth` | ❌ Not supported | BLE only via interconnect |
| `@system.geolocation` | ❌ Not supported | No GPS |
| `@system.record` | ❌ Not supported | No microphone |
| `@system.volume` | ❌ Not supported | No speaker |
| `@system.fetch` | ❌ Not supported | HTTP via phone only |
| `@system.zip` | ❌ Not supported | Watch-only feature |

### Device Specs

| Spec | Value |
|------|-------|
| Screen | 1.72" AMOLED, 212×520px |
| Shape | Pill-shaped (`pill-shaped`) |
| PPI | 326 |
| DPR | 2.0 |
| DP | 106 |
| Sensors | Accelerometer (x/y/z), Barometer (hPa) |
| Connectivity | BLE (paired phone only) |
| Reserved Storage | ~90 MB for system |
| Available Storage | Varies (~100 MB app + data) |
| CPU/RAM | ARM low-power, ~64 MB RAM |
| Battery | ~300 mAh (14 days normal use) |

## Prerequisites

### 1. Install AIoT-IDE

```bash
# Download from https://iot.mi.com/vela/quickapp/en/guide/start/use-ide.html
# Extract and launch the IDE (VS Code-based)
# Configure the built-in Vela emulator for testing
```

### 2. Clone official example repositories

```bash
cd ~/projects
git clone --depth=1 --branch dev https://github.com/open-vela/packages_apps.git
cd packages_apps/wearable/
```

**Key examples to study (in order):**

1. **`vela-sample/`** — Mandatory starting point. Complete showcase of all components, APIs, patterns
2. **`interconnect_image_demo/`** — Critical for Band↔Android communication. Includes:
   - Watch code (Quick App)
   - Android code (`android_program/`)
   - SDK AAR (`libs/xms-wearable-lib_1.4_release.aar`)
3. **`calendar/`**, **`settings/`** — Multi-page app pattern with routing
4. **`chart/`** — Using the `<chart>` component for graphs
5. **`eventBus/`**, **`parentChildComp/`** — Component communication patterns

## Development Workflow

### Step 1: Setup base project

```bash
# In AIoT-IDE: File → New → Vela Quick App Project
# Or create structure manually:
mkdir my-band-app
cd my-band-app
touch manifest.json app.ux
mkdir -p pages/index common
```

**Standard folder structure:**

```
my-band-app/
├── src/                          # Source code
│   ├── app.ux                   # Global entry point (onCreate/onDestroy)
│   ├── manifest.json            # App configuration
│   ├── config-watch.json        # Watch mode configuration
│   ├── pages/                   # App pages
│   │   └── index/
│   │       └── index.ux         # Main page
│   ├── common/                  # Shared resources
│   │   ├── logo.png             # App icon (108×108px)
│   │   ├── logo.svg             # Vector version (optional)
│   │   ├── images/              # Other images
│   │   └── utils.js             # Utility functions
│   └── i18n/                    # Internationalization (optional)
│       ├── en.json
│       ├── zh-CN.json
│       └── defaults.json
├── .vscode/                      # VS Code configuration
│   └── mcp.json                 # MCP server config (optional)
├── dist/                         # Build output (generated, .gitignore)
├── .gitignore
├── .npmignore                    # Exclusions for npm publish
├── .eslintignore                # Files to ignore for ESLint
├── package.json                 # Dependencies and scripts
├── package-lock.json            # Dependency lockfile
├── jsconfig.json                # JavaScript/TypeScript configuration
├── .prettierrc.js               # Code formatting
├── .stylelintrc.js              # CSS/UX styles linting
├── .eslintrc.js                 # JavaScript linting (if present)
├── commitlint.config.js         # Conventional commits
├── husky.sh                     # Git hooks setup
├── build.sh                     # Build helper script
├── DEVELOPMENT.md               # Technical notes and decisions
└── README.md                    # User documentation
```

**Minimum essential files (for AIoT-IDE):**
- `src/manifest.json` — package configuration, features, router
- `src/app.ux` — global lifecycle
- `src/pages/index/index.ux` — main page
- `src/common/logo.png` — 108×108px icon

**Professional toolchain (optional but recommended):**
- `package.json` + `package-lock.json` — dependency management
- `.prettierrc.js`, `.stylelintrc.js`, `.eslintignore` — code quality
- `commitlint.config.js` + `husky.sh` — git hooks for conventional commits
- `jsconfig.json` — IntelliSense and autocompletion
- `.vscode/mcp.json` — MCP server integration for AI tools

**Structure notes:**
- AIoT-IDE can also accept `app.ux` and `manifest.json` directly in the root (without `src/`)
- The `src/` folder is **strongly recommended** for professional projects
- `dist/` is auto-generated by the build (exclude from git)
- Config files in the root (`.*rc.js`, `commitlint.config.js`) are optional but improve quality and maintainability

**Minimal structure:**
```
my-band-app/
├── manifest.json       # App configuration
├── app.ux             # Global entry point
├── pages/
│   └── index/
│       └── index.ux   # Main page
└── common/
    └── logo.png       # App icon (108×108px)
```

### Step 2: Configure manifest.json

```json
{
  "package": "com.example.mybandapp",
  "name": "MyBandApp",
  "versionName": "1.0.0",
  "versionCode": 1,
  "minPlatformVersion": 1200,
  "icon": "/common/logo.png",
  "deviceTypeList": ["watch"],
  "features": [
    { "name": "system.sensor" },
    { "name": "system.interconnect" },
    { "name": "system.file" },
    { "name": "system.storage" },
    { "name": "system.vibrator" },
    { "name": "system.battery" }
  ],
  "config": {
    "logLevel": "log",
    "designWidth": 480
  },
  "router": {
    "entry": "pages/index",
    "pages": {
      "pages/index": { "component": "index" }
    }
  },
  "minAPILevel": 1
}
```

**All available features:**

| Feature | Description |
|---------|-------------|
| `system.router` | Page navigation |
| `system.storage` | Key-value storage (strings only) |
| `system.prompt` | Toast and dialogs |
| `system.file` | File system (text, ArrayBuffer) |
| `system.request` | HTTP requests (JSON) |
| `system.fetch` | Advanced HTTP requests (streaming, FormData) |
| `system.network` | Network status |
| `system.interconnect` | BLE communication band↔Android |
| `system.configuration` | Local device configuration |
| `system.app` | App info and lifecycle |
| `system.brightness` | Brightness and keepScreenOn |
| `system.sensor` | Accelerometer, barometer |
| `system.vibrator` | Vibration |
| `system.battery` | Battery status |
| `system.device` | Device info (IMEI, serial) |
| `system.volume` | Audio volume |
| `system.record` | Audio recording |
| `system.geolocation` | Location (if available) |
| `system.bluetooth` | Bluetooth low-level |
| `system.crypto` | Cryptography |
| `system.zip` | Zip compression/decompression |
| `system.audio` | Audio playback |
| `system.event` | System events |

**⚠️ Critical points:**
- `deviceTypeList: ["watch"]` — Do NOT use `"band"` for Mi Band 10
- `minPlatformVersion: 1200` — Minimum compatible version
- `designWidth: 480` — Logical resolution (real screen 212×520px).
  Alternative: `designWidth: 212` for 1:1 physical pixel mapping. Choose 480 for automatic scaling (recommended), 212 for pixel-perfect control.
- `package` — Must match Android package if using `system.interconnect`
- `display.backgroundColor` — Set global background color (e.g. `"#000000"`)
- `data` — `protected` object for parameters passed via router; `private` for internal non-overridable data
- `simulationVersion: "default"` — Required for emulator to work properly
- `minPlatformVersion` — Use `1100` for bands (SKILL.md uses 1200, which is fine; keep 1200 but add note that 1100+ works)
- NEVER add `minAPILevel` field — it can cause issues; use `minPlatformVersion` instead

### Step 3: Create base page (pages/index/index.ux)

```vue
<template>
  <div class="container">
    <text class="title">{{ message }}</text>
    <input type="button" class="btn" value="Vibrate" @click="vibrate" />
    <text class="info">Battery: {{ batteryLevel }}%</text>
  </div>
</template>

<script>
import vibrator from '@system.vibrator'
import battery from '@system.battery'

export default {
  data: {
    message: 'Hello Mi Band 10!',
    batteryLevel: 0
  },
  onInit() {
    this.getBatteryLevel()
  },
  vibrate() {
    vibrator.vibrate({
      mode: 'short'
    })
  },
  getBatteryLevel() {
    battery.getStatus({
      success: (data) => {
        this.batteryLevel = data.level
      }
    })
  }
}
</script>

<style>
.container {
  flex-direction: column;
  justify-content: center;
  align-items: center;
  width: 100%;
  height: 100%;
}
.title {
  font-size: 40px;
  color: #ffffff;
  margin-bottom: 40px;
}
.btn {
  width: 300px;
  height: 80px;
  font-size: 32px;
  background-color: #007aff;
  color: #ffffff;
}
.info {
  font-size: 28px;
  color: #cccccc;
  margin-top: 40px;
}
</style>
```

### Step 4: Build and deploy

**In AIoT-IDE:**
1. Build → Build RPK
2. Run → Run on Emulator / Real Device
3. Debug → View Console Logs

**From terminal (if configured):**
```bash
# Build
npm run build

# Output: dist/com.example.mybandapp.rpk
# Install via Mi Fitness app or AIoT-IDE
```

**Alternative: AIoT-toolkit CLI** (instead of AIoT-IDE):
```bash
# Create project
aiot create my-band-app --template wearable

# Build
aiot build

# Run on emulator
aiot start -d emulator

# Release build
aiot release

# Create emulator AVD
aiot crateVelaAvd
```

**Note**: For modern template syntax, migrate to AIoT-toolkit 2.0. AIoT-IDE supports `xiaomi10Band` as a built-in emulator preset and provides hot-reload (watches file changes and auto-pushes to emulator).

## Band ↔ Android Communication

### Established pattern (from interconnect_image_demo)

**On Band (Quick App):**

```javascript
import interconnect from '@system.interconnect'

export default {
  sendToPhone(data) {
    const message = {
      type: 'request_data',
      payload: data,
      timestamp: Date.now()
    }
    
    interconnect.send({
      message: JSON.stringify(message),
      success: () => {
        console.log('Message sent')
      },
      fail: (err) => {
        console.error('Send error:', err)
      }
    })
  },
  
  onInit() {
    // Listener for messages from Android
    interconnect.on({
      message: (data) => {
        const msg = JSON.parse(data.message)
        
        if (msg.type === 'response') {
          console.log('Received from Android:', msg.payload)
        }
      }
    })
  }
}
```

**On Android (Kotlin + XMS SDK):**

```kotlin
import com.xiaomi.xms.wearable.Wearable
import com.xiaomi.xms.wearable.message.MessageApi
import com.xiaomi.xms.wearable.message.MessageClient
import com.xiaomi.xms.wearable.message.MessageEvent

class BandCommunicationService {
    private lateinit var messageClient: MessageClient
    
    fun init(context: Context) {
        messageClient = Wearable.getMessageApi(context)
        messageClient.addListener(messageListener)
    }
    
    private val messageListener = MessageApi.MessageListener { messageEvent ->
        val receivedData = String(messageEvent.data)
        val json = JSONObject(receivedData)
        
        when (json.getString("type")) {
            "request_data" -> {
                val response = JSONObject().apply {
                    put("type", "response")
                    put("payload", "Processed data")
                }
                sendToBand(response.toString().toByteArray())
            }
        }
    }
    
    private fun sendToBand(data: ByteArray) {
        messageClient.sendMessage("com.example.mybandapp", data, null)
    }
}
```

**⚠️ Signature constraints:**
- Quick App Package = Android App Package
- Same signature (certificate.pem) for both
- Online tool: https://cdn.hybrid.xiaomi.com/aiot-ide/signature-generate-tool/v2/index.html

## Sensor Usage

### Accelerometer

```javascript
import sensor from '@system.sensor'

export default {
  onInit() {
    sensor.subscribeAccelerometer({
      interval: 'normal', // 'game' for high frequency
      success: (data) => {
        console.log(`x: ${data.x}, y: ${data.y}, z: ${data.z}`)
      }
    })
  },
  onDestroy() {
    sensor.unsubscribeAccelerometer()
  }
}
```

### Barometer

```javascript
import sensor from '@system.sensor'

export default {
  data: {
    pressure: 0
  },
  onInit() {
    sensor.subscribeBarometer({
      success: (data) => {
        this.pressure = data.pressure // hPa
      }
    })
  },
  onDestroy() {
    sensor.unsubscribeBarometer()
  }
}
```

**⚠️ Sensors NOT supported on Mi Band 10:**
- Compass / magnetometer
- Gyroscope
- GPS

## Main UI Components

### Layout containers

```vue
<template>
  <!-- Stack: element overlay -->
  <stack class="container">
    <image src="/common/bg.png" class="background"></image>
    <text class="overlay-text">Text above image</text>
  </stack>
  
  <!-- Swiper: page carousel -->
  <swiper class="swiper" indicator="true">
    <div class="page1">Page 1</div>
    <div class="page2">Page 2</div>
  </swiper>
  
  <!-- List: vertical scroll list -->
  <list class="list">
    <list-item for="{{items}}" class="item">
      <text>{{$item.name}}</text>
    </list-item>
  </list>
</template>
```

### Chart

\`\`\`vue
<template>
  <chart class="chart" type="line" options="{{chartOptions}}"></chart>
</template>

<script>
export default {
  data: {
    chartOptions: {
      xAxis: {
        min: 0,
        max: 10
      },
      yAxis: {
        min: 0,
        max: 100
      },
      series: [{
        data: [10, 20, 30, 50, 80]
      }]
    }
  }
}
</script>

### Marquee

Scrolling text with start/stop/bounce events:

\`\`\`vue
<marquee class="marquee" start="true" @start="onStart" @bounce="onBounce">Scrolling text</marquee>
\`\`\`

### Progress

Horizontal and arc progress bars:

\`\`\`vue
<progress type="horizontal" percent="{{ val }}" color="#007aff"></progress>
<progress type="arc" percent="{{ val }}" stroke-width="10" background-color="#333" color="#007aff"></progress>
\`\`\`

### Image Animator (API Level 2+)

Frame-based animation:

\`\`\`vue
<image-animator images="{{ frames }}" duration="500" iterations="1"></image-animator>
\`\`\`

### QR Code / Barcode (API Level 2+)

\`\`\`vue
<qrcode value="{{ text }}" type="qr" width="200" height="200"></qrcode>
<barcode value="{{ text }}" type="code128" width="300" height="100"></barcode>
\`\`\`

### Checkbox & Radio

\`\`\`vue
<input type="checkbox" checked="{{ isChecked }}" @change="onChange" />
<input type="radio" name="group" value="a" checked="{{ isA }}" />
<input type="radio" name="group" value="b" checked="{{ isB }}" />
\`\`\`

### Picker

Text and time picker modes:

\`\`\`vue
<picker type="text" range="{{ options }}" selected="{{ index }}" @change="onChange" />
<picker type="time" selected="{{ time }}" @change="onTimeChange" />
\`\`\`

### Slider

Range selector with `isFromUser` in change event:

\`\`\`vue
<slider min="0" max="100" value="{{ val }}" @change="onChange" />
\`\`\`

### Switch

Toggle switch with thumb/track colors:

\`\`\`vue
<switch checked="{{ enabled }}" thumb-color="#007aff" track-color="#444" @change="onToggle" />
\`\`\`

## Storage and File

### Key-value storage

```javascript
import storage from '@system.storage'

export default {
  saveData() {
    storage.set({
      key: 'userSettings',
      value: JSON.stringify({ theme: 'dark', lang: 'it' }),
      success: () => console.log('Saved')
    })
  },
  
  loadData() {
    storage.get({
      key: 'userSettings',
      success: (data) => {
        const settings = JSON.parse(data)
        console.log('Theme:', settings.theme)
      }
    })
  }
}
```

### File system

```javascript
import file from '@system.file'

export default {
  writeFile() {
    file.writeText({
      uri: 'internal://cache/data.txt',
      text: 'File content',
      success: () => console.log('File written')
    })
  },
  
  readFile() {
    file.readText({
      uri: 'internal://cache/data.txt',
      success: (data) => {
        console.log('Content:', data.text)
      }
    })
  }
}

// Additional file operations
import file from '@system.file'

// Create directory (recursive)
file.mkdir({
  uri: 'internal://app/books/',
  recursive: true,
  success: () => console.log('Directory created')
})

// Check if file exists
file.access({
  uri: 'internal://app/data.txt',
  success: () => console.log('File exists'),
  fail: () => console.log('File not found')
})

// List directory contents
file.list({
  uri: 'internal://app/',
  success: (data) => {
    console.log('Files:', data.fileList) // Array of {uri, lastModifiedTime, length, mode}
  }
})

// Delete file
file.delete({
  uri: 'internal://app/old_data.txt',
  success: () => console.log('File deleted')
})

// Copy file
file.copy({
  srcUri: 'internal://cache/temp.txt',
  dstUri: 'internal://app/data.txt',
  success: () => console.log('File copied')
})

// Move file
file.move({
  srcUri: 'internal://cache/temp.txt',
  dstUri: 'internal://app/data.txt',
  success: () => console.log('File moved')
})

// Read file partially (position + length)
file.readArrayBuffer({
  uri: 'internal://app/large_file.bin',
  position: 1024,  // Start at byte 1024
  length: 4096,     // Read 4096 bytes
  success: (data) => {
    const buffer = data.buffer // ArrayBuffer
  }
})

// Write with append mode
file.writeArrayBuffer({
  uri: 'internal://app/log.txt',
  buffer: logBuffer,
  append: true,     // Append to existing file
  success: () => console.log('Data appended')
})
```

### Advanced storage (file-based for complex data)

When data exceeds key-value simplicity (nested objects, long texts), use `@system.file` instead of `@system.storage`:

```javascript
import file from '@system.file'

const STORAGE_FILE = 'internal://cache/app_data.json'

export default {
  // Save complex data structure
  saveAppData(data) {
    file.writeText({
      uri: STORAGE_FILE,
      text: JSON.stringify(data),
      success: () => console.log('Data saved'),
      fail: (err) => console.error('Save failed:', err)
    })
  },

  // Load with fallback to defaults
  loadAppData(callback) {
    file.readText({
      uri: STORAGE_FILE,
      success: (data) => {
        try {
          callback(JSON.parse(data.text))
        } catch (e) {
          callback(this.getDefaultData())
        }
      },
      fail: () => callback(this.getDefaultData())
    })
  },

  getDefaultData() {
    return { books: [], settings: {}, lastRead: null }
  }
}
```

**Advantages over `@system.storage`:**
- No key size limit (key-value storage has implicit limits)
- Readable/debuggable via file system
- Easier backup/export
- Binary file support (ArrayBuffer) with `readArrayBuffer`/`writeArrayBuffer`

**⚠️ URI scheme:**
- `internal://cache/` — Temporary files (system can delete)
- `internal://app/` — Persistent files (not deleted)
- Use `internal://app/` for important user data

### Promise wrapper for callback-based APIs

All Vela JS APIs use callbacks (no native async/await). Pattern to wrap them in Promises:

```javascript
// runAsyncFunc.js — Utility to convert callback APIs to Promises
function runAsyncFunc(func, ...args) {
  return new Promise((resolve, reject) => {
    try {
      func({
        ...args[0],
        success: (data) => resolve(data),
        fail: (err) => reject(err),
        complete: () => {} // optional
      })
    } catch (e) {
      reject(e)
    }
  })
}

// Example: storage.get with Promise
async function loadUserSettings() {
  try {
    const data = await runAsyncFunc(storage.get.bind(storage), { key: 'settings' })
    return JSON.parse(data)
  } catch (err) {
    console.error('Failed to load settings:', err)
    return null
  }
}
```

### Internationalization (i18n)

```javascript
// src/i18n/defaults.json
{
  "app_name": "My App",
  "hello": "Ciao",
  "settings": "Impostazioni",
  "confirm": "Conferma",
  "cancel": "Annulla"
}

// src/i18n/en.json
{ "app_name": "My App", "hello": "Hello", "settings": "Settings", "confirm": "Confirm", "cancel": "Cancel" }

// src/i18n/zh-CN.json
{ "app_name": "我的应用", "hello": "你好", "settings": "设置", "confirm": "确认", "cancel": "取消" }
```

Pattern for loading languages with fallback:

```javascript
import configuration from '@system.configuration'
import file from '@system.file'

const DEFAULT_LANG = 'defaults'
const LANG_MAP = { 'zh-CN': 'zh-CN', 'zh': 'zh-CN', 'en-US': 'en', 'en': 'en' }

export default {
  data: {
    i18n: {}
  },

  loadI18n() {
    const lang = configuration.getLocale().language || 'en'
    const langFile = LANG_MAP[lang] || DEFAULT_LANG

    file.readText({
      uri: `internal://app/i18n/${langFile}.json`,
      success: (data) => {
        this.i18n = JSON.parse(data.text)
      },
      fail: () => {
        // Fallback to defaults
        file.readText({
          uri: 'internal://app/i18n/defaults.json',
          success: (data) => { this.i18n = JSON.parse(data.text) }
        })
      }
    })
  },

  // Helper in templates: {{ i18n.hello }}
  t(key) {
    return this.i18n[key] || key
  }
}
```

## Publishing Vela JS skills to GitHub

When documenting project structure:
1. **Check the real project first** — don't rely on AIoT-IDE defaults alone
2. **Include all toolchain config files** — `.prettierrc.js`, `.stylelintrc.js`, `.eslintignore`, `commitlint.config.js`, `husky.sh`, `jsconfig.json`
3. **Document optional vs essential** — separate "works in AIoT-IDE" from "professional setup"
4. **MIT License** — use `@username` in copyright if user prefers privacy over real name
5. **Multi-agent installation** — document setup for Hermes, Claude Code, Pi Agent, OpenCode, and generic AI agents

Standard README sections for published skills:
- Features & when to use
- Installation (Hermes + other agents)
- Usage examples
- Structure overview
- Resources & links
- Contributing guidelines
- License (MIT recommended)

## Band ↔ Android Communication (Advanced)

Advanced pattern for structured BLE communication. See also `references/interconnect-advanced-pattern.md`.

### Interconn class with tag dispatching

```javascript
// interconn.js
import interconnect from '@system.interconnect'
import { runAsyncFunc } from './runAsyncFunc'

const TAG = 'Interconn'

export default class Interconn {
  constructor() {
    this._onMessage = null
    this._onError = null
    this._onSend = null
    this._tagMap = {} // { tag: handler(data) }
    this._listening = false
  }

  on(tag, handler) {
    this._tagMap[tag] = handler
  }

  off(tag) {
    delete this._tagMap[tag]
  }

  startListening() {
    if (this._listening) return
    this._listening = true
    interconnect.on({
      message: (data) => this._onInterconnectMessage(data)
    })
  }

  stopListening() {
    if (!this._listening) return
    this._listening = false
    // interconnect has no native off()
  }

  send(tag, data) {
    const message = JSON.stringify({ tag, data, timestamp: Date.now() })
    return runAsyncFunc(interconnect.send.bind(interconnect), { message })
  }

  _onInterconnectMessage(data) {
    try {
      const msg = JSON.parse(data.message)
      const { tag, data: payload } = msg

      if (tag && this._tagMap[tag]) {
        this._tagMap[tag](payload)
      } else if (this._onMessage) {
        this._onMessage(msg)
      }
    } catch (e) {
      if (this._onError) this._onError(e)
    }
  }

  setOnMessage(handler) { this._onMessage = handler }
  setOnError(handler) { this._onError = handler }
}
```

### Handshake protocol

```javascript
export default class Handshake {
  constructor(interconn) {
    this.interconn = interconn
    this.connected = false
    this.retryCount = 0
    this.maxRetries = 3
  }

  async connect() {
    this.interconn.on('handshake_ack', () => {
      this.connected = true
      console.log('Handshake completed')
    })

    while (this.retryCount < this.maxRetries && !this.connected) {
      try {
        await this.interconn.send('handshake_req', { version: '1.0' })
        await this._wait(2000)
      } catch (e) {
        console.error('Handshake failed, retrying...', e)
      }
      this.retryCount++
    }
    return this.connected
  }

  _wait(ms) {
    return new Promise(resolve => setTimeout(resolve, ms))
  }
}
```

### File transfer over BLE (chunked)

```javascript
export default class FileTransfer {
  constructor(interconn) {
    this.interconn = interconn
    this.chunkSize = 1024 // bytes per chunk
    this.buffer = {}
  }

  async sendFile(fileName, data) {
    const totalChunks = Math.ceil(data.length / this.chunkSize)
    const fileId = Date.now().toString()

    // Send metadata
    await this.interconn.send('file_meta', { fileId, fileName, totalChunks, size: data.length })

    // Send chunks
    for (let i = 0; i < totalChunks; i++) {
      const start = i * this.chunkSize
      const end = Math.min(start + this.chunkSize, data.length)
      const chunk = data.slice(start, end)
      await this.interconn.send('file_chunk', { fileId, index: i, data: chunk, final: i === totalChunks - 1 })
    }
  }

  onFileReceived(callback) {
    this.interconn.on('transfer_ack', (data) => callback(data))
  }
}
```

## Complete Lifecycle

### App lifecycle (app.ux)

```javascript
// app.ux
export default {
  onCreate() {
    console.log('App created')
    // Global initializations: storage, i18n, interconnect
  },
  onShow() {
    console.log('App shown')
    // App returns to foreground
  },
  onHide() {
    console.log('App hidden')
    // App goes to background — save state, clean resources
  },
  onDestroy() {
    console.log('App destroyed')
    // Final cleanup: clearInterval, unsubscribe sensors
  },
  onError(error) {
    console.error('App error:', error)
    // Log uncaught errors
  }
}
```

### Page lifecycle (pages/index/index.ux)

```javascript
export default {
  // Loaded with parameters from router
  protected: {
    bookId: '',
    page: 1
  },

  data: {
    title: '',
    content: ''
  },

  onInit() {
    console.log('Page init')
    // Read parameters, initial setup
    this.bookId = this.bookId // from protected
  },

  onReady() {
    console.log('Page ready')
    // DOM ready, elements rendered
  },

  onShow() {
    console.log('Page shown')
    // Page visible — resume timers, update UI
  },

  onHide() {
    console.log('Page hidden')
    // Page hidden — save reading progress, stop animations
  },

  onDestroy() {
    console.log('Page destroyed')
    // Cleanup: clearInterval, unsubscribe
  },

  onBackPress() {
    console.log('Back pressed')
    // Handle physical back button
    // return true to prevent default navigation
    return false
  },

  onRefresh() {
    console.log('Page refreshed')
    // Pull-to-refresh or data reload
  },

  onConfigurationChanged(config) {
    console.log('Config changed:', config)
    // Language, theme, orientation changed
  }
}
```

**Execution order:**
1. `onInit()` — Data available, DOM not yet
2. `onReady()` — DOM ready (similar to mounted)
3. `onShow()` — Page visible
4. `onHide()` — Page hidden (overlaid by another page)
5. `onDestroy()` — Page destroyed
- `onBackPress()` — Interceptable at any time

### onBackPress pattern for custom navigation

```javascript
onBackPress() {
  if (this.view !== 'main') {
    this.switchView('main') // Return to main view
    return true              // Prevents app close
  }
  return false // Closes app (default)
}
```

## Data: protected vs private

### protected (overridable via router)

Data in `protected` are parameters passed from other pages via `router.push()`:

```javascript
// manifest.json — routing
"router": {
  "entry": "pages/index",
  "pages": {
    "pages/index": { "component": "index" },
    "pages/detail": { "component": "detail" }
  }
}

// Source page
import router from '@system.router'
router.push({
  uri: 'pages/detail',
  params: { bookId: '123', page: 42 }
})

// Destination page (pages/detail/detail.ux)
export default {
  protected: {
    bookId: '',
    page: 1
  },
  onInit() {
    console.log(this.bookId) // '123'
    console.log(this.page)   // 42
  }
}
```

### private (not overridable)

Data in `data` are **private** by default — they cannot be overwritten by external parameters. Use `protected` only for parameters that must come from the router.

```javascript
export default {
  // Input parameters (overridable by router.push)
  protected: {
    bookId: '',
    page: 1
  },
  // Internal data (never overridable)
  data: {
    items: [],
    isLoading: false
  }
}
```

## Brightness and KeepScreenOn

For apps that require the screen to stay on (reading, timer, monitoring):

```javascript
import brightness from '@system.brightness'

export default {
  data: {
    keepScreenOn: false,
    brightnessValue: 100
  },

  enableKeepScreenOn() {
    brightness.setKeepScreenOn({
      keepScreenOn: true,
      success: () => {
        this.keepScreenOn = true
        console.log('Screen always on')
      },
      fail: (err) => console.error('KeepScreenOn failed:', err)
    })
  },

  disableKeepScreenOn() {
    brightness.setKeepScreenOn({
      keepScreenOn: false,
      success: () => {
        this.keepScreenOn = false
      }
    })
  },

  setBrightness(brightness) {
    brightness.setBrightness({
      brightness: brightness, // 0-255
      success: () => { this.brightnessValue = brightness }
    })
  },

  // Use in combination with auto-read mode
  startAutoRead() {
    this.enableKeepScreenOn()
    this.setBrightness(80) // Reduced brightness for reading
  }
}
```

⚠️ **Battery**: KeepScreenOn increases consumption. Use only when necessary and disable in `onHide()`.

## Inter-Page State Management

### globalThis Pattern

Vela JS has no built-in state management. Use `globalThis` for cross-page state sharing:

```javascript
// app.ux — initialize global state
export default {
  onCreate() {
    // Only initialize if not already set
    if (typeof globalThis.__appState === 'undefined') {
      globalThis.__appState = {
        settings: {},
        userData: null,
        isLoggedIn: false
      }
    }
  },

  onDestroy() {
    // Cleanup global state
    delete globalThis.__appState
  }
}

// Any page — read/write global state
export default {
  onInit() {
    if (globalThis.__appState && globalThis.__appState.settings) {
      this.theme = globalThis.__appState.settings.theme
    }
  },

  updateSetting(key, value) {
    if (globalThis.__appState) {
      globalThis.__appState.settings[key] = value
    }
  },

  onDestroy() {
    // Cleanup page-specific listeners from globalThis
    // Delete only what this page added
  }
}
```

### Router Stack Inspection

```javascript
import router from '@system.router'

// Get page stack length
const stackLength = router.getLength()

// Get current page state
const state = router.getState()
console.log('Current page:', state.name, state.path, state.index)

// Get full page stack
const pages = router.getPages()
pages.forEach((page, i) => {
  console.log(i + ': ' + page.name + ' (' + page.path + ')')
})

// Clear entire stack (e.g., on logout)
router.clear()
```

### Launch Modes

Configure page launch behavior in manifest.json:

```json
{
  "router": {
    "entry": "pages/index",
    "pages": {
      "pages/index": {
        "component": "index",
        "launchMode": "standard"
      },
      "pages/detail": {
        "component": "detail",
        "launchMode": "singleTask"
      }
    }
  }
}
```

- `standard` (default): Creates new page instance each time
- `singleTask`: Reuses existing page instance, calls `onRefresh()` instead

## Memory Optimization

### 12 Best Practices for Mi Band 10 (64MB RAM)

1. **Keep non-UI data out of ViewModel** — Use module-level `const` instead of `data{}` for static data
2. **Mutate objects in-place** — Avoid creating new object references; modify existing properties
3. **Don't cache page methods globally** — Can prevent garbage collection
4. **Clear timers in onDestroy()** — Always `clearInterval`/`clearTimeout`
5. **Release resources after use** — File handles, storage references, large data arrays
6. **Call `global.runGC()`** — After heavy operations (file loading, data processing, UI refresh)
7. **Use `static` attribute** — Prevents data binding updates on elements that don't need them
8. **Use `<block static>`** — Groups static elements together for rendering optimization
9. **Reduce dependencies** — Import only what you need; avoid large utility libraries
10. **Use global methods** — Reduce per-page import overhead
11. **Compress images** — Use tinypng or similar; prefer PNG8 over PNG24
12. **Minimize page count** — Use view switching (`data.view`) instead of extra pages

### Static Attribute Example

```vue
<!-- Elements with static don't re-render when data changes -->
<text static>{{ fixedTitle }}</text>

<!-- Static block groups multiple elements -->
<block static>
  <text>Version 1.0.0</text>
  <text>Copyright 2026</text>
</block>
```

### Performance Audit Checklist

- [ ] FMP (First Meaningful Paint) <= 2000ms (mandatory for publishing)
- [ ] No duplicate code in pages
- [ ] No large dependencies
- [ ] No unused imports
- [ ] Network requests use HTTPS
- [ ] Debounce button handlers
- [ ] Batch DOM updates where possible

## Binary File I/O (ArrayBuffer)

For reading binary files (ebooks, images, raw data) with UTF-16 encoding:

```javascript
import file from '@system.file'

export default {
  // Read file as ArrayBuffer
  readBinaryFile(uri) {
    return new Promise((resolve, reject) => {
      file.readArrayBuffer({
        uri: uri,
        success: (data) => {
          const buffer = data.buffer // ArrayBuffer
          // UTF-16 conversion (typical for ebooks/Chinese texts)
          const text = this.utf16ArrayBufferToString(buffer)
          resolve(text)
        },
        fail: (err) => reject(err)
      })
    })
  },

  // Write ArrayBuffer
  writeBinaryFile(uri, buffer) {
    return new Promise((resolve, reject) => {
      file.writeArrayBuffer({
        uri: uri,
        buffer: buffer,
        success: () => resolve(),
        fail: (err) => reject(err)
      })
    })
  },

  // ArrayBuffer → UTF-16 string
  utf16ArrayBufferToString(buffer) {
    let result = ''
    const view = new Uint8Array(buffer)
    for (let i = 0; i < view.length; i += 2) {
      const charCode = view[i] | (view[i + 1] << 8)
      if (charCode === 0) break
      result += String.fromCharCode(charCode)
    }
    return result
  },

  // String → UTF-16 ArrayBuffer
  stringToUtf16ArrayBuffer(str) {
    const buffer = new ArrayBuffer(str.length * 2)
    const view = new Uint8Array(buffer)
    for (let i = 0; i < str.length; i++) {
      const charCode = str.charCodeAt(i)
      view[i * 2] = charCode & 0xFF
      view[i * 2 + 1] = (charCode >> 8) & 0xFF
    }
    return buffer
  }
}
```

## Feature Detection

### Runtime API detection with canIUse()

Use `app.canIUse()` (API Level 3+) to check if a feature is available at runtime:

```javascript
import app from '@system.app'

export default {
  onInit() {
    // Check if a module exists
    if (app.canIUse('@system.bluetooth')) {
      console.log('Bluetooth API available')
    }

    // Check if a specific method exists
    if (app.canIUse('@system.router.push')) {
      console.log('router.push available')
    }

    // Check if a component exists
    if (app.canIUse('scroll')) {
      console.log('<scroll> component available')
    }

    // Check if a component attribute exists
    if (app.canIUse('scroll.attr.scroll-x')) {
      console.log('scroll-x attribute available')
    }
  }
}
```

### Device Info API

```javascript
import device from '@system.device'

export default {
  getDeviceInfo() {
    device.getInfo({
      success: (info) => {
        console.log('Brand:', info.brand)           // e.g. "Xiaomi"
        console.log('Model:', info.model)            // e.g. "Mi Band 10"
        console.log('Screen shape:', info.screenShape) // 'pill-shaped'
        console.log('Device type:', info.deviceType)   // 'band'
        console.log('API Level:', info.apiLevel)       // e.g. 4
        console.log('Screen:', info.screenWidth + 'x' + info.screenHeight)
        console.log('Density:', info.screenDensity)
      }
    })
  },

  getStorage() {
    device.getTotalStorage({
      success: (data) => {
        console.log('Total storage:', data.size, 'bytes')
      }
    })
    device.getAvailableStorage({
      success: (data) => {
        console.log('Available storage:', data.size, 'bytes')
      }
    })
  }
}
```

### App Launch Source Detection

`app.getInfo()` returns how the app was launched:

```javascript
import app from '@system.app'

const appInfo = app.getInfo()
// appInfo.source = { packageName: '...', type: 'shortcut'|'push'|'url'|'barcode'|'nfc'|'bluetooth'|'other' }
```

## Custom Components

Create reusable `.ux` components:

```vue
<!-- common/components/book-card/book-card.ux -->
<template>
  <div class="card" @click="onClick">
    <image src="{{ cover }}" class="cover"></image>
    <div class="info">
      <text class="title">{{ title }}</text>
      <text class="author">{{ author }}</text>
    </div>
  </div>
</template>

<script>
export default {
  data: {
    title: '',
    author: '',
    cover: ''
  },
  onClick() {
    this.$emit('cardClick', { title: this.title })
  }
}
</script>

<style>
.card {
  flex-direction: row;
  height: 120px;
  padding: 10px;
}
.cover {
  width: 80px;
  height: 100px;
}
.info {
  flex-direction: column;
  margin-left: 10px;
}
.title { font-size: 28px; color: #fff; }
.author { font-size: 24px; color: #999; }
</style>
```

```vue
<!-- pages/index/index.ux — usage -->
<template>
  <div class="container">
    <import name="bookCard" src="../../common/components/book-card/book-card.ux"></import>
    <bookCard
      title="{{ book.title }}"
      author="{{ book.author }}"
      cover="{{ book.cover }}"
      @cardClick="handleCardClick"
    ></bookCard>
  </div>
</template>
```

**Common patterns:**
- `import name="componentName" src="path"` to import components
- `this.$emit('eventName', data)` to communicate with parent
- `@eventName="handler"` to listen for events from child
- Components in `common/components/` for cross-page reuse

## Advanced Template

### show vs if

- **`if`/`elif`/`else`**: Removes/adds DOM on change. Use for conditions that change rarely.
- **`show`**: Hides/shows with `display: none`. Use for frequent toggles (e.g. popups).

```vue
<!-- if: destroys/creates DOM -->
<text if="{{ isLoggedIn }}">Welcome!</text>
<text elif="{{ isGuest }}">Guest mode</text>
<text else>Please login</text>

<!-- show: only hides -->
<text show="{{ showPopup }}">Popup content</text>
```

### tid attribute

Optimizes list rendering. `tid` uniquely identifies each item to avoid unnecessary re-renders:

```vue
<list>
  <list-item for="{{ books }}" tid="id">
    <!-- tid="id" → uses book.id as unique key -->
    <text>{{ $item.title }}</text>
  </list-item>
</list>
```

### `<block>` element

Groups elements without adding DOM nodes:

```vue
<block for="{{ items }}">
  <text>{{ $item.name }}</text>
  <text>{{ $item.value }}</text>
</block>
```

### static attribute

`static` prevents data binding updates (performance optimization):

```vue
<text static>{{ fixedText }}</text>
<!-- Does not update even if fixedText changes -->
```

### Native events

| Event | Description |
|--------|-------------|
| `@click` | Tap/press |
| `@longpress` | Long press |
| `@swipe` | Swipe (with `direction`, `distance`) |
| `@change` | Input/picker/switch value change |
| `@scroll` | Scroll in progress |
| `@scrolltop` | Scroll reached top |
| `@scrollbottom` | Scroll reached bottom |
| `@focus` / `@blur` | Input focus/blur |

### this.$element()

Direct access to DOM element:

```javascript
// Template: <scroll id="contentScroll" ...>
// Script:
this.$element('contentScroll') // Returns reference to scroll node
```

## CSS Subset (Vela limitations)

CSS in Vela JS is a **subset** of browser CSS. Does not support:

```css
/* ❌ NOT supported: */
.class1 .class2 {}        /* Descendant selectors */
.class1 > .class2 {}      /* Child selector */
.class1 + .class2 {}      /* Sibling selector */
.class1:hover {}          /* Pseudo-classes (hover, active, focus) */
.class1::before {}        /* Pseudo-elements (before, after) */
* { }                     /* Universal selector */

/* ✅ Supported: */
.class-name {}             /* Class selector (one class per element) */
#id {}                     /* ID selector */
.class1.class2 {}          /* Multi-class (same element) */
tag {}                     /* Tag selector (div, text, image...) */
```

**Supported CSS properties:**
- `width`, `height`, `margin`, `padding`, `border` (with `border-radius`)
- `flex-direction`, `justify-content`, `align-items`, `flex-wrap`
- `font-size`, `color`, `font-weight`, `font-family`, `text-align`
- `background-color`, `background-image` (.png/.svg only)
- `display` (flex/none), `visibility`
- `opacity`
- `position` (static/fixed/absolute), `left`, `top`, `right`, `bottom`
- `transform` (translate, scale, rotate)
- `animation` (simple keyframes)

**⚠️ Differences from browser CSS:**
- `position: fixed` is relative to the current scroll viewport (not the window)
- `background-image` only accepts local URIs (no data URI)
- `animation` does not support `infinite` on all components
- `z-index` not supported; use DOM order for stacking
- `overflow: hidden` only works on specific containers (`list`, `scroll`)

## CSS Media Queries for Screen Shapes

Adapt UI to different screen shapes using media queries (API Level 2+):

```css
/* Pill-shaped (Mi Band 10) */
@media (screen-shape: pill-shaped) {
  .container { padding: 20dp; }
}

/* Round watches */
@media (screen-shape: circle) {
  .container { padding: 30dp; }
}

/* Rectangular watches */
@media (screen-shape: rect) {
  .container { padding: 15dp; }
}
```

### Screen-specific dimensions

| Screen Type | designWidth | Typical Devices |
|-------------|-------------|-----------------|
| Band (pill) | 192 or 212 | Mi Band 10, Band 9 |
| Watch (round) | 336 or 466 | Xiaomi Watch S4/S5 |
| Watch (rect) | 336 or 480 | Redmi Watch |

### Background Image Support

| Device | Background Images |
|--------|-------------------|
| Mi Band 10 | ✅ Supported |
| Watch S4, S5 | ✅ Supported |
| Redmi Watch 5, 6 | ✅ Supported |
| Watch S1 Pro, S3 | ❌ Not supported |
| Redmi Watch 4 | ❌ Not supported |
| Band 8 Pro, 9, 9 Pro | ❌ Not supported |

## Transition & Animation Limitations

### transition-property (limited)

Only these properties support CSS transitions:
- `width`, `height`
- `opacity`, `visibility`
- `border-width`, `border-color`
- `background-color`, `background-position`

### @keyframes Requirements

- Both `0%` and `100%` keyframes MUST be explicitly specified
- Supported animatable properties: `background-color`, `background-position`, `opacity`, `width`, `height`, `transform`
- `infinite` may not work on all components
- Multiple animations supported from API 1070+:

```css
.animated-element {
  animation-name: fadeIn, slideIn;
  animation-duration: 300ms, 500ms;
}
```

## Common Pitfalls

### 1. Display too small (212×520px)

The framework scales automatically. Always design for 480px logical width:
- Font size: 28–40px (readable)
- Button height: 60–80px (easy touch)
- Margin/padding: 20–40px

### 2. Very limited RAM/CPU

- Max 3–4 complex components per page
- No full-screen animations
- Minimize images (prefer SVG/icon fonts)
- Use `list` instead of many repeated `div`s

### 3. interconnect requires shared signature

If `interconnect.send()` fails:
1. Verify matching package name (manifest.json ↔ AndroidManifest.xml)
2. Verify certificate signed with same key
3. Use Xiaomi online tool to generate valid signature

### 4. Sensors not always available

```javascript
sensor.subscribeAccelerometer({
  success: (data) => { /* OK */ },
  fail: (err) => {
    console.error('Sensor not available:', err)
    // Fallback UI without sensor
  }
})
```

### 5. BLE only to paired phone

No WiFi of its own — use `system.interconnect` for external data. The Android app acts as a bridge.

### 6. Emulator vs real device

Always test on a real Mi Band 10 for:
- Real performance
- Touch/gesture
- Hardware sensors
- Battery consumption

### 7. Manual garbage collection

Vela JS does not do aggressive automatic GC. Call `global.runGC()` after heavy operations:

```javascript
// After loading/processing large datasets
this.loadLargeFile(() => {
  // Processing completed
  global.runGC() // Force garbage collection
})
```

### 8. Cleanup in onHide/onDestroy

Always clean up timers and listeners in exit lifecycle methods:

```javascript
onShow() {
  this.startTimer()
},
onHide() {
  this.stopTimer() // Save state, stop animations
},
onDestroy() {
  clearInterval(this.intervalId)
  sensor.unsubscribeAccelerometer()
  global.runGC()
}
```

### 9. Signature mismatch for interconnect

Common error: Android app and Quick App have different signatures.
- Use SAME `certificate.pem` for both projects
- The package name in `manifest.json` must match Android `applicationId`
- Test on real device (emulator does not handle interconnect)

### 10. Timer does not survive onDestroy

`setInterval` and `setTimeout` do not persist when the app is killed. For timers that must survive:
- Android companion app with `AlarmManager` + BLE wakeup
- Home screen widget that reactivates the app

### 11. Limited key-value storage

`@system.storage` has implicit size limits per key. For data > 10KB use `@system.file` (file-based storage).

### 12. No native async/await

All Vela JS APIs are callback-based. Always use the Promise wrapper pattern:

```javascript
// ❌ Does NOT work
const data = await storage.get({ key: 'x' })

// ✅ Use callbacks or runAsyncFunc
storage.get({ key: 'x', success: (data) => { /* ... */ } })
// Or with Promise wrapper
const data = await runAsyncFunc(storage.get.bind(storage), { key: 'x' })
```

### 13. Emoji Not Supported

Vela renderer has poor emoji support. ALWAYS use SVG icons or font icons instead:

```vue
<!-- ❌ BAD: Emoji may not render -->
<text>❤️ Save</text>

<!-- ✅ GOOD: SVG icon -->
<image src="/common/icons/heart.svg" class="icon"></image>
<text class="icon-label">Save</text>
```

### 14. CSS media queries required for multi-screen

Always use `@media (screen-shape: pill-shaped)` for Mi Band 10-specific styles. Generic styles may break on other screen shapes.

### 15. Storage Budget Management

Mi Band 10 reserves ~90MB for system. Always check available storage before writing large files:
- Use `device.getAvailableStorage()` at startup
- Warn user if storage < 2MB
- Auto-clean old data (e.g., keep max 90 days of logs)

### 16. Button Debounce Required

`onShow()` fires when the screen wakes up. If you have a fetch in `onShow()`, it will trigger on every wake. Always debounce button handlers:
```javascript
data: { buttonLocked: false }

handleClick() {
  if (this.buttonLocked) return
  this.buttonLocked = true
  // ... action ...
  setTimeout(() => { this.buttonLocked = false }, 500)
}
```

## Internal Resources

- `references/timer-scheduler-pattern.md` — Timer vs scheduler pattern for periodic tasks
- `references/data-tracking-app-pattern.md` — Multi-view pattern for data tracking apps (habit/mood/spending tracker). Includes: navigation state, array-based storage, streak calc, cleanup, statistics, bottom nav UI, form input
- `references/interconnect-advanced-pattern.md` — Advanced BLE communication pattern: Interconn class with tag dispatching, Handshake protocol, FileTransfer chunked, Promise wrapper
- `references/content-reader-pattern.md` — Complete pattern for reader/book apps: virtual scrolling, pagination, binary file I/O, bookmarks, auto-read mode, tap navigation
- `references/publishing-hermes-skills.md` — Complete workflow for publishing Hermes skills on GitHub
- `references/development-pitfalls-and-tips.md` — Comprehensive troubleshooting guide from real Vela projects (manifest, CSS, narrow-screen, build/signing)
- `references/device-storage-reservation.md` — Device storage reservation patterns (Mi Band 10 reserves ~90MB)
- `references/global-state-management.md` — Cross-page state sharing using `globalThis` pattern
- `references/file-based-storage-utility.md` — Robust file-based storage utility with caching, callback queue, CRUD operations
- `references/chunked-file-reading.md` — Chunked reading for large files with paragraph boundary protection

## Official Resources

### Documentation
- Quick Start: https://iot.mi.com/vela/quickapp/en/guide/start/use-ide.html
- UI Components: https://iot.mi.com/vela/quickapp/en/components/
- JS API: https://iot.mi.com/vela/quickapp/en/features/
- Interconnect: https://iot.mi.com/vela/quickapp/en/features/network/interconnect.html

### Example repositories
- GitHub: https://github.com/open-vela/packages_apps
- Wearable folder: `/wearable/vela-sample/`, `/wearable/interconnect_image_demo/`

### Tools
- AIoT-IDE download: https://iot.mi.com/vela/quickapp/en/guide/start/use-ide.html
- Online signature: https://cdn.hybrid.xiaomi.com/aiot-ide/signature-generate-tool/v2/index.html
- Vela Toolkit CLI: `aiot start`, `aiot build`, `aiot release`, `aiot crateVelaAvd`
- AIoT-toolkit 2.0 migration for modern template syntax
- AIoT-IDE supports `xiaomi10Band` as a built-in emulator preset
- Hot-reload: AIoT-IDE watches file changes and auto-pushes to emulator

## Checklist before starting

- [ ] AIoT-IDE installed and working with Vela emulator
- [ ] Cloned `open-vela/packages_apps` and studied `wearable/vela-sample/`
- [ ] Read `interconnect_image_demo/` code (watch + Android)
- [ ] Understood manifest.json + app.ux + .ux pages pattern
- [ ] Defined package name (matching Android companion app)
- [ ] Generated signature pair (private.pem / certificate.pem)
- [ ] Verified which sensors are needed (accel/baro available, compass NO)

## Architecture diagram

```
┌─────────────────────────────────┐
│  Mi Band 10                     │
│  Quick App (.rpk)               │
│  - Vela JS / .ux files          │
│  - system.interconnect (send)   │
│  - system.sensor (accel/baro)   │
│  - system.vibrator              │
└────────────┬────────────────────┘
             │ Automatic BLE
             │ (managed by Mi Fitness)
┌────────────▼────────────────────┐
│  Android companion app          │
│  - xms-wearable-lib_1.4.aar     │
│  - MessageApi (byte[])          │
│  - NodeApi (device state)       │
│  - NotifyApi (notifications)    │
└────────────┬────────────────────┘
             │ (if Flutter/other)
             │ MethodChannel / REST API
┌────────────▼────────────────────┐
│  Mobile app / Backend           │
│  - UI framework (Flutter/React) │
│  - Business logic               │
└─────────────────────────────────┘
```

## Mi Band 10 Hardware Constraints

| Feature | Value |
|----------------|--------|
| Screen | 1.72" AMOLED, 212×520px |
| Sensors | Accelerometer (x/y/z), Barometer (hPa) |
| Connectivity | BLE (only to paired phone) |
| Storage | Limited (~100 MB app + data) |
| CPU/RAM | ARM low-power, ~64 MB RAM |
| Battery | ~300 mAh (14 days normal use) |

## Next steps after skill loaded

1. **Clone example repo**: `git clone https://github.com/open-vela/packages_apps.git`
2. **Study vela-sample**: Open `packages_apps/wearable/vela-sample/` in AIoT-IDE
3. **Test interconnect_image_demo**: Build + deploy on emulator
4. **Create base project**: Use minimal template above
5. **Implement feature**: Start with simple UI, add sensors/interconnect gradually

## Pattern & Templates

This skill includes reference files with validated patterns:

- **`references/timer-scheduler-pattern.md`** — Complete pattern for timer-based apps (hourly vibration, reminders, scheduler). Includes: timer management with anti-drift, multi-state storage, anti-duplicate protection, 212×520px optimized UI, testing checklist. Production-validated.
- **`references/data-tracking-app-pattern.md`** — Multi-view pattern for data tracking apps (habit, mood, spending, study timer). Includes: navigation state with view switcher, date-array-based storage, streak calculation, automatic cleanup, statistics aggregation, bottom nav UI, form input with icon/color picker. Validated with Habit Tracker.
- **`references/interconnect-advanced-pattern.md`** — Advanced pattern for BLE band↔Android communication with tag dispatching, handshake, file transfer.
- **`references/content-reader-pattern.md`** — Pattern for reader/book apps: virtual scrolling, binary file I/O, bookmarks, auto-read mode.

---

*Skill created for Xiaomi Mi Band 10 Quick App development with Vela JS framework. Based on official Xiaomi IoT documentation and open-vela repository.*
