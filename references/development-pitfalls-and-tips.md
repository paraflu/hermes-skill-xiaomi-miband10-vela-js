# Development Pitfalls & Tips

Merged from vela-quickapp-dev skill. Chinese content preserved for accuracy.

# Vela Quick App Development: Pitfalls & Best Practices

> This document is compiled from development experience across multiple real projects, with sensitive data removed. It applies to all Vela quick app development scenarios.

---

## 1. manifest.json Configuration Pitfalls

### 1.1 deviceTypeList Only Accepts "watch"

- **Incorrect**: `"deviceTypeList": ["band"]`
- **Correct**: `"deviceTypeList": ["watch"]`
- **Behavior**: RPK build succeeds, but installation on device fails with `miwear_install failed: undefined`
- **Reason**: The official documentation states that the only valid values are `watch`, `tv`, `car`, `phone`, and currently only `watch` is supported
- **Lesson**: `band` is only returned by `device.getInfo().deviceType` at runtime; it cannot be written in the manifest

### 1.2 Recommended minPlatformVersion

- It is recommended to set it to `1100` for band devices (instead of the lower 1070)
- Refer to similar published projects to determine the appropriate version number

### 1.3 Must Add simulationVersion

```json
"simulationVersion": "default"
```

Missing this field will cause installation failure.

### 1.4 Do Not Add minAPILevel

- **Incorrect**: Manually setting `"minAPILevel": xxx`
- **Correct**: Do not write this field; let aiot-toolkit handle it automatically

### 1.5 Be Cautious with display Configuration

- On some devices, the `display` configuration may cause installation failure
- It is recommended to refer to verified project structures and not arbitrarily add the display field
- For full screen, use: `"display": { "titleBar": false, "fullScreen": true }`

### 1.6 Complete manifest.json Reference Template

```json
{
  "package": "com.example.myapp",
  "name": "App Name",
  "icon": "/common/icon.png",
  "versionName": "1.0.0",
  "versionCode": 1,
  "minPlatformVersion": 1100,
  "simulationVersion": "default",
  "deviceTypeList": ["watch"],
  "features": [
    { "name": "system.router" },
    { "name": "system.storage" },
    { "name": "system.prompt" },
    { "name": "system.vibrator" },
    { "name": "system.device" }
  ],
  "permissions": [
    { "name": "hapjs.permission.DEVICE_INFO" }
  ],
  "config": {
    "logLevel": "log",
    "designWidth": 212
  },
  "router": {
    "entry": "pages/index",
    "pages": {
      "pages/index": { "component": "index" },
      "pages/detail": { "component": "detail" }
    }
  }
}
```

**designWidth Reference**:
- Band (narrow-tall screen): `192` or `212`
- Watch (square/round): `336` or `466`

---

## 2. Project Structure Standards

### 2.1 Page Files Must Be in Subdirectories

```
src/pages/index/index.ux    # ✅ Correct
src/pages/index.ux           # ❌ Incorrect! Will report path does not exist
```

In manifest.json, `"pages/index": { "component": "index" }` maps to the file path `pages/index/index.ux`, not `pages/index.ux`.

### 2.2 Router Configuration

```json
"router": {
  "entry": "pages/index",
  "pages": {
    "pages/index": { "component": "index" },
    "pages/detail": { "component": "detail" }
  }
}
```

- `entry` does not need a `/index` suffix; just write the directory path directly
- The page directory name must match the router key
- The UX filename must match the component value

### 2.3 Module Import Paths

```javascript
// When the page is at pages/index/index.ux, import modules from common:
import utils from '../../common/utils.js';
import router from '@system.router';
import vibrator from '@system.vibrator';
```

- Relative paths are based on the current file location
- All system modules require explicit import

### 2.4 Remember to Update Paths After Moving Pages

After changing the location of a page file, all relative paths in `require`/`import` must be updated accordingly. Vela copies files to a temporary directory during build, and relative paths are based on the compiled structure.

---

## 3. CSS Limitations & Alternatives

### 3.1 Unsupported Pseudo-class Selectors

| Unsupported | Description |
|--------|------|
| `:active` | Pressed state |
| `:hover` | Hover state |
| `:focus` | Focused state |
| `:disabled` | Disabled state |

**Alternative**: Use dynamic class switching instead of pseudo-classes

```html
<!-- ❌ Incorrect -->
<div class="card:active">...</div>

<!-- ✅ Correct -->
<div class="card {{isSelected ? 'card-selected' : ''}}" @click="toggle">
```

### 3.2 Units Must Use dp

- **Incorrect**: `font-size: 24px; padding: 16px;`
- **Correct**: `font-size: 24dp; padding: 16dp;`
- `line-height` must also use dp

### 3.3 Scroll Component Limitations

- The `scroll-direction` attribute is not supported; vertical scrolling is the default
- For horizontal scrolling, use the `swiper` component

---

## 4. Narrow-Tall Screen (Band) Layout Tips

Band screens are typically 212dp wide narrow-tall screens. Pay special attention to layout:

### 4.1 Width Control

- Root div fixed width: `width: 212dp`
- Prevent horizontal overflow that causes the page to scroll left/right
- Vela's div does not clip overflow content by default; width must be manually constrained

### 4.2 Font Size Reference (212dp Wide Screen)

| Purpose | Font Size |
|------|------|
| Page title | 15-16dp |
| Menu/Body text | 13-14dp |
| Button text | 11-12dp |
| Status hint | 10dp |
| Auxiliary info | 8-9dp |

**Lesson**: Font sizes on bands should be 2-3 levels smaller than on phones; do not copy phone designs directly.

### 4.3 Horizontal Layout Tips

- Menu items: icon (20dp) + text (13dp) arranged horizontally, total width ~170dp, within 212dp
- Settings items: split into two rows, top row for icon + label, bottom row for control
- Grid layout: 3 columns (60dp each, 3 columns = 180dp), with `flex-wrap: wrap`
- Avoid placing too many child elements with `flex-direction: row`

### 4.4 Preventing Horizontal Scrolling

```css
/* Root container fixed width */
.page-root {
  width: 212dp;
  flex-direction: column;
}

/* Grid layout */
.grid {
  flex-wrap: wrap;
  width: 180dp;  /* 3 columns x 60dp */
}
```

---

## 5. JavaScript Engine Notes

### 5.1 Variable Scope

- Properties defined in `data` must be accessed with `this.xxx` in methods
- **Incorrect**: `count = count + 1` (count is not defined)
- **Correct**: `this.count = this.count + 1`

### 5.2 System Modules Must Be Explicitly Imported

```javascript
import router from '@system.router';
import vibrator from '@system.vibrator';
import prompt from '@system.prompt';

// or use require
var storage = require('@system.storage');
```

### 5.3 Storage API Is Asynchronous

```javascript
// ❌ Wrong: synchronous thinking
var value = storage.get({ key: 'xxx' });

// ✅ Correct: callback style
storage.get({
  key: 'xxx',
  success: function(val) {
    console.log(val);  // val is a string
  },
  fail: function(d, code) {
    console.error('Read failed', code);
  }
});
```

- The success callback parameter of `storage.get` is a value string, not an object
- Serialize complex data with `JSON.stringify()` / `JSON.parse()`
- Cannot serialize functions, circular references, or undefined values

### 5.4 Avoid Unused Variable Warnings

```javascript
// ❌ Assigning but not using will produce a warning
var chosenColor = 'red';
// ... ultimately the code uses decision.chosenColor

// ✅ Ensure assigned variables are actually used, or remove unused assignments
```

---

## 6. Performance Optimization

### 6.1 Avoid Frequent DOM Operations

- **Incorrect approach**: Modifying data properties one by one in a loop
- **Correct approach**: Calculate all data first, build the complete array, then assign once

```javascript
// ❌ Modify items one by one
for (var i = 0; i < items.length; i++) {
  this.list[i].name = 'new name';  // triggers render each time
}

// ✅ Update in a single assignment
var newList = items.map(function(item) {
  return { name: 'new name', ...item };
});
this.list = newList;
```

### 6.2 Timer Management

- **Single delay**: Use `setTimeout` (e.g., AI decision delay 500-800ms)
- **Periodic execution**: Use `setInterval` (e.g., countdown at 1000ms intervals)
- **Must clean up**: Clear timers in `onDestroy` when the page is destroyed

```javascript
export default {
  onDestroy() {
    if (this.timer) clearTimeout(this.timer);
    if (this.countdown) clearInterval(this.countdown);
  }
}
```

- Do not use `setInterval` for high-frequency animations; it will lag
- Do not use `setInterval` for high-frequency polling

### 6.3 Use \<list\> for List Rendering

Large data lists must use the virtual scrolling of the `<list>` component; do not render everything with `<div>` + `for`:

```html
<!-- ❌ Incorrect: render everything -->
<div for="{{items}}">
  <text>{{$item.name}}</text>
</div>

<!-- ✅ Correct: virtual scrolling -->
<list>
  <list-item for="{{items}}">
    <text>{{$item.name}}</text>
  </list-item>
</list>
```

---

## 7. Build & Signing

### 7.1 aiot-toolkit Has No init Command

- `aiot init` will throw an error `unknown command`
- Create the project structure manually (manifest.json, app.ux, pages directory, etc.)
- Or use `npm create aiot` to create the project

### 7.2 Build Commands

```bash
# Dev build (debug, uses default signing)
aiot build

# Production build (release, uses custom signing)
aiot release
```

### 7.3 Signing Certificate Generation

```bash
mkdir -p sign
openssl req -newkey rsa:2048 -nodes -keyout sign/private.pem \
  -x509 -days 3650 -out sign/certificate.pem \
  -subj "/CN=Name/O=Org/C=CN"
```

- The `sign` directory must be at the project root; aiot will find it automatically
- Debug builds use aiot's built-in default signing and generate `.debug.rpk`
- Release builds use custom signing and generate `.release.rpk`

### 7.4 Build Output Location

- aiot-toolkit creates a temporary directory `.temp_<project_name>/` in the workspace
- After the build completes, the output is in `dist/` at the project root
- For example: `dist/com.example.myapp.release.1.0.0.rpk`

### 7.5 Installing aiot-toolkit via Domestic Mirror

```bash
npm install -g aiot-toolkit@2.0.5 --registry=https://registry.npmmirror.com/
```

### 7.6 Build Warning Handling

- Warnings like `unsupport attribute` do not affect RPK generation, but should be noted
- Warnings like `'xxx' is not defined` will cause runtime errors and must be fixed
- First run `aiot build` (debug) to confirm functionality, then `aiot release`

---

## 8. Real Device Debugging

### 8.1 Installation Method

Installing RPK on wearable devices requires the **official App's Debug menu**:

1. Install the corresponding brand's Health App on your phone (beta version)
2. Go to: My → About → Debug → Third-party Apps
3. Enter the package name → Select RPK file to install

### 8.2 Debugging Tips

1. First verify basic functionality with the emulator
2. Debug builds are faster than release builds
3. Check the `dist/` directory to confirm the RPK file was generated
4. After connecting the device, use `aiot getConnectedDevices` to check
5. Hardware APIs like vibration may not be supported on the emulator; wrap with try-catch

```javascript
try { vibrator.vibrate({ mode: 'short' }); } catch(e) {}
```

---

## 9. UI/UX Practical Tips

### 9.1 Enhancing Visual Hierarchy in Text-Only Interfaces

Even without image resources, you can enhance visual hierarchy through:
- Colored icon boxes (rounded squares) + single character identifiers
- Color coding for different types (e.g., blue = Action A, orange = Action B, red = Warning)
- Border highlighting for states (e.g., yellow = currently active)
- Size variation for importance

### 9.2 Popups & Interactions

- Countdown using `setInterval(1000ms)` + manual counting
- Progress bar using dynamic `width` calculation
- For multi-step interactions, use intermediate state variables

### 9.3 Emoji Compatibility

Vela has poor emoji support; use SVG icons or font icons instead:

```html
<!-- ❌ Incorrect -->
<text>❤️ Favorite</text>

<!-- ✅ Correct -->
<image src="/common/icons/heart.svg" style="width: 24dp; height: 24dp;"></image>
<text>Favorite</text>
```

---

## 10. Common Error Quick Reference

| Error | Cause | Solution |
|----------|------|----------|
| RPK build succeeds but installation fails | deviceTypeList used "band" | Change to ["watch"] |
| Build error PseudoClassSelector | Used pseudo-classes like :active | Use dynamic class switching |
| path does not exist | Page file not in subdirectory | Move to pages/xxx/xxx.ux |
| 'router' is not defined | System module not imported | Add import statement |
| 'gs' is not defined | data property not accessed with this | Change to this.gs |
| unsupport attribute scroll-direction | scroll doesn't support this attribute | Remove scroll-direction |
| Page scrolls left/right | Child element overflow | Add width: 212dp to root div |
| Font too large | Copied phone font sizes | Use recommended band font sizes |
| Storage read failed | Synchronous thinking calling async API | Use callback style |
| Cannot find RPK after build | Looked in temp directory | Look in project dist/ directory |
| npm install stuck | Network issue | Use npmmirror mirror |
| aiot init error | Command doesn't exist | Use npm create aiot or create manually |

---

*This document is compiled from development experience across multiple real Vela quick app projects and is continuously updated.*
