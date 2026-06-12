# Global State Management (globalThis Pattern)

# 跨页面全局状态管理（Global State via globalThis）

> 来源：com.bandbbs.ebook 项目 `src/app.ux` + 多个页面文件
> 官方文档状态：**官方未收录** — Vela 快应用没有内置的状态管理方案，官方文档未涉及跨页面数据共享

## 背景

Vela 快应用中，每个页面有独立的 ViewModel，页面间传递数据只能通过 `router.push({ params })` 或全局变量。参考项目使用 `globalThis` 实现了跨页面状态共享和事件通信。

## 核心模式

### 在 app.ux 中初始化全局对象

```javascript
// src/app.ux
import interconn from '@system.interconnect'
import interconnfile from '../utils/interconnfile'

export default {
  onCreate() {
    // 全局连接对象
    globalThis.conn = new interconn()
    globalThis.conn.setHandshakeListener(() => {
      console.log('手机端已连接')
    })

    // 全局文件传输对象
    globalThis.connfile = globalThis.conn.register(interconnfile)
  },
  onDestroy() {
    console.log('应用退出')
  }
}
```

### 在页面中使用全局状态

```javascript
// src/pages/detailsetting/detailsetting.ux
export default {
  data: {
    isKeepScreenOn: false
  },
  onShow() {
    // 从全局状态读取
    if (typeof globalThis.isKeepScreenOn !== 'undefined') {
      this.isKeepScreenOn = globalThis.isKeepScreenOn
    }
  },
  toggleKeepScreenOn() {
    this.isKeepScreenOn = !this.isKeepScreenOn
    // 写回全局状态，其他页面可读取
    globalThis.isKeepScreenOn = this.isKeepScreenOn
  }
}
```

### 全局事件监听

```javascript
// src/pages/push/push.ux
export default {
  data: {
    connected: false
  },
  onInit() {
    // 检查连接状态
    this.connected = !!(globalThis.conn && globalThis.conn.connected)

    // 注册事件监听
    this.eventlistener = globalThis.conn.addEventListener((data) => {
      console.log('收到数据:', data)
      this.handleData(data)
    })

    // 设置文件传输回调
    globalThis.connfile.setCallback((data) => {
      this.handleFileData(data)
    })
  },
  onDestroy() {
    // 清理事件监听（重要！防止内存泄漏）
    if (globalThis.conn) {
      globalThis.conn.removeEventListener(this.eventlistener)
    }
  }
}
```

### 跨页面标记（页面跳转来源追踪）

```javascript
// src/pages/detailsetting/detailsetting.ux
export default {
  onShow() {
    // 检查是从哪个页面跳转来的
    if (globalThis.justJumpedFromChapter) {
      // 来自章节切换
      globalThis.justJumpedFromChapter = false
    }
    if (globalThis.justJumpedFromBookmark) {
      // 来自书签跳转
      globalThis.justJumpedFromBookmark = false
    }
    if (globalThis.justJumpedFromPageNumber) {
      // 来自页码跳转
      globalThis.justJumpedFromPageNumber = false
    }
  }
}

// 在书签页面跳转时设置标记
// src/pages/bookmarks/bookmarks.ux
jumpToBookmark(bookmark) {
  globalThis.justJumpedFromBookmark = true
  router.push({
    uri: 'pages/detail/detail',
    params: { position: bookmark.position }
  })
}
```

## 使用注意事项

1. **及时清理**：在 `onDestroy()` 中清理事件监听器和全局引用，防止内存泄漏
2. **类型检查**：读取 `globalThis` 属性前先检查 `typeof`，因为可能尚未初始化
3. **不要过度使用**：`globalThis` 是全局命名空间，过多的全局变量会导致命名冲突和调试困难
4. **线程安全**：Vela 是单线程的，不存在并发修改问题，但异步回调顺序仍需注意
5. **页面生命周期**：全局对象的生命周期与应用一致，但页面可能被销毁重建，不要缓存页面实例
