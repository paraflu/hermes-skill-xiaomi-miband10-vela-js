# Development Pitfalls & Tips

Merged from vela-quickapp-dev skill. Chinese content preserved for accuracy.

# Vela 快应用开发实战踩坑与最佳实践

> 本文档整理自多个真实项目的开发经验，已脱敏处理，适用于所有 Vela 快应用开发场景。

---

## 一、manifest.json 配置陷阱

### 1.1 deviceTypeList 只能用 "watch"

- **错误写法**：`"deviceTypeList": ["band"]`
- **正确写法**：`"deviceTypeList": ["watch"]`
- **现象**：RPK 构建成功，但安装到设备时报错 `miwear_install 失败: undefined`
- **原因**：官方文档明确说明可选值只有 `watch`、`tv`、`car`、`phone`，目前只支持 `watch`
- **教训**：`band` 仅在运行时 `device.getInfo().deviceType` 返回，不能写在 manifest 里

### 1.2 minPlatformVersion 推荐值

- 手环设备建议设为 `1100`（而非较低的 1070）
- 可参考已上架的同类项目确定合适的版本号

### 1.3 必须添加 simulationVersion

```json
"simulationVersion": "default"
```

缺少此字段会导致安装失败。

### 1.4 不要添加 minAPILevel

- **错误做法**：手动设置 `"minAPILevel": xxx`
- **正确做法**：不写此字段，让 aiot-toolkit 自动处理

### 1.5 display 配置需谨慎

- 部分设备上 `display` 配置可能导致安装失败
- 建议参考已验证的项目结构，不要随意添加 display 字段
- 如需全屏，使用：`"display": { "titleBar": false, "fullScreen": true }`

### 1.6 完整 manifest.json 参考模板

```json
{
  "package": "com.example.myapp",
  "name": "应用名",
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

**designWidth 参考**：
- 手环（窄长屏）：`192` 或 `212`
- 手表（方形/圆形）：`336` 或 `466`

---

## 二、项目结构规范

### 2.1 页面文件必须放在子目录中

```
src/pages/index/index.ux    # ✅ 正确
src/pages/index.ux           # ❌ 错误！会报 path does not exist
```

manifest.json 中 `"pages/index": { "component": "index" }` 对应的文件路径是 `pages/index/index.ux`，不是 `pages/index.ux`。

### 2.2 router 配置

```json
"router": {
  "entry": "pages/index",
  "pages": {
    "pages/index": { "component": "index" },
    "pages/detail": { "component": "detail" }
  }
}
```

- `entry` 不需要加 `/index` 后缀，直接写目录路径
- 页面目录名与 router 的 key 一致
- UX 文件名与 component 值一致

### 2.3 模块导入路径

```javascript
// 页面在 pages/index/index.ux 时，引用 common 下的模块：
import utils from '../../common/utils.js';
import router from '@system.router';
import vibrator from '@system.vibrator';
```

- 相对路径基于当前文件位置
- 所有系统模块都需要显式 import

### 2.4 移动页面后记得更新路径

页面文件位置变更后，所有 `require`/`import` 的相对路径都需要同步更新。Vela 构建时会将文件复制到临时目录，相对路径基于编译后的结构。

---

## 三、CSS 限制与替代方案

### 3.1 不支持的伪类选择器

| 不支持 | 说明 |
|--------|------|
| `:active` | 按下状态 |
| `:hover` | 悬停状态 |
| `:focus` | 聚焦状态 |
| `:disabled` | 禁用状态 |

**替代方案**：用 class 动态切换代替伪类

```html
<!-- ❌ 错误 -->
<div class="card:active">...</div>

<!-- ✅ 正确 -->
<div class="card {{isSelected ? 'card-selected' : ''}}" @click="toggle">
```

### 3.2 单位必须用 dp

- **错误**：`font-size: 24px; padding: 16px;`
- **正确**：`font-size: 24dp; padding: 16dp;`
- `line-height` 也需要用 dp

### 3.3 scroll 组件限制

- 不支持 `scroll-direction` 属性，默认纵向滚动
- 如需横向滚动，使用 `swiper` 组件

---

## 四、窄长屏（手环）布局要点

手环屏幕通常为 212dp 宽的窄长屏，布局需特别注意：

### 4.1 宽度控制

- 根 div 固定宽度：`width: 212dp`
- 防止水平溢出导致页面可左右滚动
- Vela 的 div 默认不会裁剪溢出内容，必须手动限制宽度

### 4.2 字号参考（212dp 宽屏幕）

| 用途 | 字号 |
|------|------|
| 页面标题 | 15-16dp |
| 菜单/正文 | 13-14dp |
| 按钮文字 | 11-12dp |
| 状态提示 | 10dp |
| 辅助信息 | 8-9dp |

**教训**：手环字号要比手机小 2-3 级，不能照搬手机版设计。

### 4.3 横向排列技巧

- 菜单项：图标(20dp) + 文字(13dp) 横向排列，总宽约 170dp，在 212dp 内
- 设置项：拆成上下两行，上行放图标+标签，下行放控件
- 网格布局：3 列（每列 60dp，3列=180dp），配合 `flex-wrap: wrap`
- 避免使用 `flex-direction: row` 放太多子元素

### 4.4 防止左右滚动

```css
/* 根容器固定宽度 */
.page-root {
  width: 212dp;
  flex-direction: column;
}

/* 网格布局 */
.grid {
  flex-wrap: wrap;
  width: 180dp;  /* 3列 x 60dp */
}
```

---

## 五、JavaScript 引擎注意事项

### 5.1 变量作用域

- `data` 中定义的属性在方法中必须用 `this.xxx` 访问
- **错误**：`count = count + 1`（count 未定义）
- **正确**：`this.count = this.count + 1`

### 5.2 系统模块必须显式导入

```javascript
import router from '@system.router';
import vibrator from '@system.vibrator';
import prompt from '@system.prompt';

// 或使用 require
var storage = require('@system.storage');
```

### 5.3 storage API 是异步的

```javascript
// ❌ 错误：同步思维
var value = storage.get({ key: 'xxx' });

// ✅ 正确：回调方式
storage.get({
  key: 'xxx',
  success: function(val) {
    console.log(val);  // val 是字符串
  },
  fail: function(d, code) {
    console.error('读取失败', code);
  }
});
```

- `storage.get` 的 success 回调参数是 value 字符串，不是对象
- 复杂数据用 `JSON.stringify()` / `JSON.parse()` 序列化
- 不能序列化函数、循环引用、undefined 值

### 5.4 避免未使用变量警告

```javascript
// ❌ 赋值但未使用会产生警告
var chosenColor = 'red';
// ... 最终用的是 decision.chosenColor

// ✅ 确保变量赋值后有实际使用，或删除无用赋值
```

---

## 六、性能优化

### 6.1 避免频繁 DOM 操作

- **错误做法**：在循环中逐个修改 data 属性
- **正确做法**：先计算好所有数据，构建完整数组，一次性赋值

```javascript
// ❌ 逐个修改
for (var i = 0; i < items.length; i++) {
  this.list[i].name = 'new name';  // 每次都触发渲染
}

// ✅ 一次性更新
var newList = items.map(function(item) {
  return { name: 'new name', ...item };
});
this.list = newList;
```

### 6.2 定时器管理

- **单次延迟**：用 `setTimeout`（如 AI 决策延迟 500-800ms）
- **周期执行**：用 `setInterval`（如倒计时 1000ms 间隔）
- **必须清理**：页面销毁时在 `onDestroy` 中清除定时器

```javascript
export default {
  onDestroy() {
    if (this.timer) clearTimeout(this.timer);
    if (this.countdown) clearInterval(this.countdown);
  }
}
```

- 不要用 `setInterval` 做高频动画，会卡顿
- 不要用 `setInterval` 做高频轮询

### 6.3 列表渲染用 <list>

大量数据列表必须使用 `<list>` 组件的虚拟滚动，不要用 `<div>` + `for` 渲染全部：

```html
<!-- ❌ 错误：全部渲染 -->
<div for="{{items}}">
  <text>{{$item.name}}</text>
</div>

<!-- ✅ 正确：虚拟滚动 -->
<list>
  <list-item for="{{items}}">
    <text>{{$item.name}}</text>
  </list-item>
</list>
```

---

## 七、构建与签名

### 7.1 aiot-toolkit 没有 init 命令

- `aiot init` 会报错 `unknown command`
- 需手动创建项目结构（manifest.json、app.ux、pages 目录等）
- 或使用 `npm create aiot` 创建项目

### 7.2 构建命令

```bash
# 开发构建（debug，使用默认签名）
aiot build

# 生产构建（release，使用自定义签名）
aiot release
```

### 7.3 签名证书生成

```bash
mkdir -p sign
openssl req -newkey rsa:2048 -nodes -keyout sign/private.pem \
  -x509 -days 3650 -out sign/certificate.pem \
  -subj "/CN=Name/O=Org/C=CN"
```

- `sign` 目录必须在项目根目录下，aiot 会自动查找
- debug 构建使用 aiot 自带的默认签名，生成 `.debug.rpk`
- release 构建使用自定义签名，生成 `.release.rpk`

### 7.4 构建产物位置

- aiot-toolkit 会在工作区创建临时目录 `.temp_<项目名>/`
- 构建完成后产物在项目根目录的 `dist/` 下
- 例如：`dist/com.example.myapp.release.1.0.0.rpk`

### 7.5 安装 aiot-toolkit 国内源

```bash
npm install -g aiot-toolkit@2.0.5 --registry=https://registry.npmmirror.com/
```

### 7.6 构建警告处理

- `unsupport attribute` 等警告不影响 RPK 生成，但需关注
- `'xxx' is not defined` 警告会导致运行时错误，必须修复
- 先 `aiot build`（debug）确认功能正常，再 `aiot release`

---

## 八、真机调试

### 8.1 安装方式

RPK 安装到穿戴设备需通过**官方 App 的 Debug 菜单**：

1. 手机安装对应品牌的运动健康 App（测试版）
2. 进入 我的 → 关于 → Debug → 第三方应用
3. 输入包名 → 选择 RPK 文件安装

### 8.2 调试技巧

1. 先用模拟器验证基本功能
2. debug 构建比 release 构建更快
3. 检查 `dist/` 目录确认 RPK 文件生成
4. 设备连接后用 `aiot getConnectedDevices` 查看
5. 振动等硬件 API 在模拟器上可能不支持，需 try-catch 包裹

```javascript
try { vibrator.vibrate({ mode: 'short' }); } catch(e) {}
```

---

## 九、UI/UX 实用技巧

### 9.1 纯文字界面增强视觉层次

即没有图片资源，也可以通过以下方式增加视觉层次：
- 彩色图标盒（圆角方块）+ 单字符标识
- 颜色编码表示不同类型（如蓝色=操作A，橙色=操作B，红色=警告）
- 边框高亮表示状态（如黄色=当前活跃）
- 大小变化表示重要程度

### 9.2 弹窗与交互

- 倒计时用 `setInterval(1000ms)` + 手动计数
- 进度条用 `width` 动态计算
- 需要多步交互的操作，用中间状态变量暂存

### 9.3 emoji 兼容性

Vela 系统对 emoji 支持差，必须使用 SVG 图标或字体图标替代：

```html
<!-- ❌ 错误 -->
<text>❤️ 收藏</text>

<!-- ✅ 正确 -->
<image src="/common/icons/heart.svg" style="width: 24dp; height: 24dp;"></image>
<text>收藏</text>
```

---

## 十、常见错误速查

| 错误现象 | 原因 | 解决方案 |
|----------|------|----------|
| RPK 构建成功但安装失败 | deviceTypeList 用了 "band" | 改为 ["watch"] |
| 构建报错 PseudoClassSelector | 使用了 :active 等伪类 | 用 class 动态切换 |
| path does not exist | 页面文件未放在子目录 | 移到 pages/xxx/xxx.ux |
| 'router' is not defined | 未 import 系统模块 | 添加 import 语句 |
| 'gs' is not defined | data 属性未用 this 访问 | 改为 this.gs |
| unsupport attribute scroll-direction | scroll 不支持该属性 | 移除 scroll-direction |
| 页面可左右滚动 | 子元素溢出 | 根 div 加 width: 212dp |
| 字号太大 | 照搬手机端字号 | 使用手环推荐字号 |
| storage 读取失败 | 同步思维调用异步 API | 使用回调方式 |
| 构建后找不到 RPK | 查找了临时目录 | 在项目 dist/ 目录下找 |
| npm install 卡死 | 网络问题 | 使用 npmmirror 镜像源 |
| aiot init 报错 | 该命令不存在 | 用 npm create aiot 或手动创建 |

---

*本文档基于多个真实 Vela 快应用项目开发经验整理，持续更新中。*
