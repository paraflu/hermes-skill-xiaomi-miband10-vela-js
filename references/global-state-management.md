# Global State Management (globalThis Pattern)

# Cross-Page Global State Management (Global State via globalThis)

> Source: com.bandbbs.ebook project `src/app.ux` + multiple page files
> Official Documentation Status: **Not covered by official docs** — Vela quick apps have no built-in state management solution, and the official documentation does not cover cross-page data sharing

## Background

In Vela quick apps, each page has its own ViewModel. Data can only be passed between pages via `router.push({ params })` or global variables. The reference project uses `globalThis` to implement cross-page state sharing and event communication.

## Core Pattern

### Initialize global object in app.ux

```javascript
// src/app.ux
import interconn from '@system.interconnect'
import interconnfile from '../utils/interconnfile'

export default {
  onCreate() {
    // Global connection object
    globalThis.conn = new interconn()
    globalThis.conn.setHandshakeListener(() => {
      console.log('Phone connected')
    })

    // Global file transfer object
    globalThis.connfile = globalThis.conn.register(interconnfile)
  },
  onDestroy() {
    console.log('App exiting')
  }
}
```

### Using global state in pages

```javascript
// src/pages/detailsetting/detailsetting.ux
export default {
  data: {
    isKeepScreenOn: false
  },
  onShow() {
    // Read from global state
    if (typeof globalThis.isKeepScreenOn !== 'undefined') {
      this.isKeepScreenOn = globalThis.isKeepScreenOn
    }
  },
  toggleKeepScreenOn() {
    this.isKeepScreenOn = !this.isKeepScreenOn
    // Write back to global state so other pages can read it
    globalThis.isKeepScreenOn = this.isKeepScreenOn
  }
}
```

### Global event listening

```javascript
// src/pages/push/push.ux
export default {
  data: {
    connected: false
  },
  onInit() {
    // Check connection status
    this.connected = !!(globalThis.conn && globalThis.conn.connected)

    // Register event listener
    this.eventlistener = globalThis.conn.addEventListener((data) => {
      console.log('Received data:', data)
      this.handleData(data)
    })

    // Set file transfer callback
    globalThis.connfile.setCallback((data) => {
      this.handleFileData(data)
    })
  },
  onDestroy() {
    // Clean up event listeners (important to prevent memory leaks)
    if (globalThis.conn) {
      globalThis.conn.removeEventListener(this.eventlistener)
    }
  }
}
```

### Cross-page flags (navigation source tracking)

```javascript
// src/pages/detailsetting/detailsetting.ux
export default {
  onShow() {
    // Check which page the navigation came from
    if (globalThis.justJumpedFromChapter) {
      // From chapter navigation
      globalThis.justJumpedFromChapter = false
    }
    if (globalThis.justJumpedFromBookmark) {
      // From bookmark navigation
      globalThis.justJumpedFromBookmark = false
    }
    if (globalThis.justJumpedFromPageNumber) {
      // From page number navigation
      globalThis.justJumpedFromPageNumber = false
    }
  }
}

// Set flag when jumping from bookmarks
// src/pages/bookmarks/bookmarks.ux
jumpToBookmark(bookmark) {
  globalThis.justJumpedFromBookmark = true
  router.push({
    uri: 'pages/detail/detail',
    params: { position: bookmark.position }
  })
}
```

## Usage Notes

1. **Clean up in time**: Clean up event listeners and global references in `onDestroy()` to prevent memory leaks
2. **Type checking**: Check `typeof` before reading `globalThis` properties, as they may not have been initialized yet
3. **Avoid overuse**: `globalThis` is the global namespace. Too many global variables can lead to naming conflicts and debugging difficulties
4. **Thread safety**: Vela is single-threaded, so there are no concurrent modification issues, but asynchronous callback order still needs attention
5. **Page lifecycle**: The lifecycle of global objects is consistent with the application, but pages may be destroyed and recreated; do not cache page instances
