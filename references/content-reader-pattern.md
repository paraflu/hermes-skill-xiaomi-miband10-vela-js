# Content Reader Pattern for Vela JS Apps

Pattern for reader/book apps on Mi Band 10, based on verified code from real reading apps (ebook reader for Mi Band).

## Architecture

```
┌──────────────────────────────────────────┐
│  Content Reader App                       │
│                                           │
│  ┌──────────────────────────────────────┐│
│  │  Template (UI)                        ││
│  │  ┌─ Scroll view (virtual segments)   ││
│  │  ├─ Tap zones (forward/backward)     ││
│  │  ├─ Progress indicator               ││
│  │  └─ Settings panel (font/opacity)    ││
│  └──────────────────────────────────────┘│
│                                           │
│  ┌──────────────────────────────────────┐│
│  │  Core Reader Engine                   ││
│  │  ├─ File I/O (readArrayBuffer)        ││
│  │  ├─ Segment management (double)      ││
│  │  ├─ Pagination (scrollBy)            ││
│  │  └─ Bookmark save/load               ││
│  └──────────────────────────────────────┘│
│                                           │
│  ┌──────────────────────────────────────┐│
│  │  Auto-Read Mode                       ││
│  │  ├─ setInterval for scroll advance    ││
│  │  ├─ setKeepScreenOn                   ││
│  │  └─ Pause/Resume on tap              ││
│  └──────────────────────────────────────┘│
│                                           │
│  ┌──────────────────────────────────────┐│
│  │  Data Storage                         ││
│  │  ├─ Bookmarks: file-based            ││
│  │  ├─ Settings: file-based             ││
│  │  └─ Content cache: memory            ││
│  └──────────────────────────────────────┘│
└──────────────────────────────────────────┘
```

## Core Pattern: Virtual Scrolling

The 212×520px screen shows only a portion of the content. Use `<scroll>` with segments and `scrollBy()` for smooth navigation.

### 1. Template with Scroll + Tap Zones

```vue
<template>
  <div class="container">
    <!-- Scrollable reading area -->
    <scroll id="contentScroll"
      class="scroll-area"
      scroll-y="true"
      @scrolltop="onScrollTop"
      @scrollbottom="onScrollBottom">
      <div class="content">
        <text class="segment" for="{{ segments }}" tid="index">
          {{ $item }}
        </text>
      </div>
    </scroll>

    <!-- Tap zones for navigation (invisible, overlay) -->
    <div class="tap-zone-top" @click="prevPage"></div>
    <div class="tap-zone-bottom" @click="nextPage"></div>

    <!-- Progress bar -->
    <div class="progress-bar">
      <div class="progress-fill" style="width: {{ progressPercent }}%;"></div>
    </div>

    <!-- Current page / total -->
    <text class="page-info">{{ currentSegment }}/{{ totalSegments }}</text>
  </div>
</template>

<style>
.container {
  width: 100%;
  height: 100%;
  background-color: #000;
  position: relative;
}

.scroll-area {
  width: 100%;
  height: 100%;
  padding: 20px 30px;
}

.content {
  flex-direction: column;
}

.segment {
  font-size: {{ fontSize }}px;
  color: {{ textColor }};
  line-height: 1.5;
  margin-bottom: 20px;
  opacity: {{ textOpacity }};
}

.tap-zone-top {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 30%;
  opacity: 0; /* Invisible but clickable */
}

.tap-zone-bottom {
  position: fixed;
  bottom: 0;
  left: 0;
  width: 100%;
  height: 30%;
  opacity: 0;
}

.progress-bar {
  position: fixed;
  bottom: 10px;
  left: 30px;
  right: 30px;
  height: 4px;
  background-color: #333;
  border-radius: 2px;
}

.progress-fill {
  height: 100%;
  background-color: #007aff;
  border-radius: 2px;
}

.page-info {
  position: fixed;
  bottom: 20px;
  font-size: 20px;
  color: #666;
  text-align: center;
  width: 100%;
}
</style>
```

### 2. Segment Management (Double Buffer)

Keep only necessary segments in memory to save RAM:

```javascript
export default {
  data: {
    segments: [],
    currentSegment: 0,
    totalSegments: 0,
    allContent: [],
    segmentSize: 5, // rows per segment
    bufferSegments: 4, // segments to keep in RAM
    fontSize: 28,
    textColor: '#cccccc',
    textOpacity: 0.9,
    progressPercent: 0,
    isLoading: false,
    pageLock: false
  },

  onInit() {
    this.loadBook()
  },

  loadBook() {
    this.readBookFile('internal://app/books/book.txt')
      .then((content) => {
        this.allContent = this.splitIntoLines(content)
        this.totalSegments = Math.ceil(this.allContent.length / this.segmentSize)
        this.loadSegment(0)
      })
  },

  splitIntoLines(text) {
    return text.split('\n').filter(line => line.trim())
  },

  loadSegment(index) {
    if (index < 0 || index >= this.totalSegments) return
    if (this.pageLock) return

    this.pageLock = true
    this.currentSegment = index

    const start = index * this.segmentSize
    const end = Math.min(start + this.segmentSize * this.bufferSegments, this.allContent.length)
    const visibleLines = this.allContent.slice(start, end)

    // Rebuilds segments with numbered lines
    this.segments = visibleLines.map((line, i) => {
      return `[${start + i + 1}] ${line}`
    })

    this.updateProgress()
    this.saveReadingProgress()

    setTimeout(() => { this.pageLock = false }, 300)
  },

  updateProgress() {
    this.progressPercent = Math.round(
      (this.currentSegment / this.totalSegments) * 100
    )
  }
}
```

### 3. Navigation with scrollBy

```javascript
nextPage() {
  if (this.currentSegment < this.totalSegments - 1) {
    this.loadSegment(this.currentSegment + 1)
    this.scrollToTop()
  }
},

prevPage() {
  if (this.currentSegment > 0) {
    this.loadSegment(this.currentSegment - 1)
    this.scrollToTop()
  }
},

scrollToTop() {
  const scrollElement = this.$element('contentScroll')
  if (scrollElement) {
    scrollElement.scrollBy({ dx: 0, dy: -9999 }) // Scroll back to top
  }
},

onScrollTop() {
  // If scrolled to top, load previous segment
  if (this.currentSegment > 0) {
    this.loadSegment(this.currentSegment - 1)
  }
},

onScrollBottom() {
  // If scrolled to bottom, load next segment
  if (this.currentSegment < this.totalSegments - 1) {
    this.loadSegment(this.currentSegment + 1)
  }
}
```

### 4. File Reading (Binary I/O)

Reads text files with UTF-16 encoding (used for Chinese/Asian ebooks):

```javascript
import file from '@system.file'

export default {
  readBookFile(uri) {
    return new Promise((resolve, reject) => {
      file.readArrayBuffer({
        uri: uri,
        success: (data) => {
          const text = this.arrayBufferToUtf16(data.buffer)
          resolve(text)
        },
        fail: (err) => reject(err)
      })
    })
  },

  arrayBufferToUtf16(buffer) {
    let result = ''
    const view = new Uint8Array(buffer)
    for (let i = 0; i < view.length; i += 2) {
      const charCode = view[i] | (view[i + 1] << 8)
      if (charCode === 0) break // Null terminator
      result += String.fromCharCode(charCode)
    }
    return result
  },

  splitIntoLines(text) {
    // Supports both \r\n and \n
    return text.split(/\r?\n/).filter(line => line.trim())
  }
}
```

## Auto-Read Mode

Automatic scrolling with brightness control:

```javascript
import brightness from '@system.brightness'

export default {
  data: {
    autoRead: false,
    autoReadSpeed: 3, // seconds per segment
    autoReadTimer: null
  },

  toggleAutoRead() {
    if (this.autoRead) {
      this.stopAutoRead()
    } else {
      this.startAutoRead()
    }
  },

  startAutoRead() {
    this.autoRead = true
    brightness.setKeepScreenOn({ keepScreenOn: true })

    this.autoReadTimer = setInterval(() => {
      if (this.currentSegment < this.totalSegments - 1) {
        this.nextPage()
      } else {
        this.stopAutoRead() // End of book
      }
    }, this.autoReadSpeed * 1000)
  },

  stopAutoRead() {
    this.autoRead = false
    if (this.autoReadTimer) {
      clearInterval(this.autoReadTimer)
      this.autoReadTimer = null
    }
    brightness.setKeepScreenOn({ keepScreenOn: false })
  },

  // Pause on tap during auto-read
  onTapDuringAutoRead() {
    if (this.autoRead) {
      this.stopAutoRead()
      prompt.showToast({ message: 'Auto-read paused' })
    }
  },

  onHide() {
    this.stopAutoRead() // Always stop when hidden
  },

  onDestroy() {
    this.stopAutoRead()
  }
}
```

## Bookmarks & Reading Progress

Save and load reading progress and bookmarks:

```javascript
import file from '@system.file'

const BOOKMARKS_FILE = 'internal://app/bookmarks.json'

export default {
  data: {
    bookmarks: [],
    lastPosition: 0
  },

  // Save progress
  saveReadingProgress() {
    file.writeText({
      uri: 'internal://app/current_book.json',
      text: JSON.stringify({
        bookId: this.bookId,
        segment: this.currentSegment,
        timestamp: Date.now()
      }),
      fail: (err) => console.error('Save progress failed:', err)
    })
  },

  // Load progress
  loadReadingProgress() {
    return new Promise((resolve) => {
      file.readText({
        uri: 'internal://app/current_book.json',
        success: (data) => {
          try {
            const saved = JSON.parse(data.text)
            this.currentSegment = saved.segment || 0
            resolve(true)
          } catch (e) { resolve(false) }
        },
        fail: () => resolve(false)
      })
    })
  },

  // Add bookmark
  addBookmark() {
    const bookmark = {
      id: Date.now().toString(),
      segment: this.currentSegment,
      text: this.segments[0] || '',
      timestamp: Date.now()
    }
    this.bookmarks.push(bookmark)
    this.saveBookmarks()
    prompt.showToast({ message: 'Bookmark added' })
  },

  // Remove bookmark
  removeBookmark(id) {
    this.bookmarks = this.bookmarks.filter(b => b.id !== id)
    this.saveBookmarks()
  },

  // Go to bookmark
  goToBookmark(id) {
    const bookmark = this.bookmarks.find(b => b.id === id)
    if (bookmark) {
      this.loadSegment(bookmark.segment)
      this.scrollToTop()
    }
  },

  saveBookmarks() {
    file.writeText({
      uri: BOOKMARKS_FILE,
      text: JSON.stringify(this.bookmarks),
      fail: (err) => console.error('Save bookmarks failed:', err)
    })
  },

  loadBookmarks() {
    file.readText({
      uri: BOOKMARKS_FILE,
      success: (data) => {
        try { this.bookmarks = JSON.parse(data.text) }
        catch (e) { this.bookmarks = [] }
      },
      fail: () => { this.bookmarks = [] }
    })
  }
}
```

## Reading Settings

```javascript
export default {
  data: {
    settings: {
      fontSize: 28,
      lineHeight: 1.5,
      textColor: '#cccccc',
      bgColor: '#000000',
      opacity: 0.9,
      autoReadSpeed: 3
    },
    fontSizeOptions: [24, 28, 32, 36, 40],
    opacityOptions: [0.6, 0.7, 0.8, 0.9, 1.0]
  },

  changeFontSize(delta) {
    const index = this.fontSizeOptions.indexOf(this.settings.fontSize)
    const newIndex = Math.max(0, Math.min(this.fontSizeOptions.length - 1, index + delta))
    this.settings.fontSize = this.fontSizeOptions[newIndex]
    this.saveSettings()
  },

  changeOpacity(delta) {
    const index = this.opacityOptions.indexOf(this.settings.opacity)
    const newIndex = Math.max(0, Math.min(this.opacityOptions.length - 1, index + delta))
    this.settings.opacity = this.opacityOptions[newIndex]
    this.saveSettings()
  },

  saveSettings() {
    file.writeText({
      uri: 'internal://app/reader_settings.json',
      text: JSON.stringify(this.settings)
    })
  },

  loadSettings() {
    file.readText({
      uri: 'internal://app/reader_settings.json',
      success: (data) => {
        try { Object.assign(this.settings, JSON.parse(data.text)) }
        catch (e) {}
      }
    })
  }
}
```

## Multi-Page Routing for Reader

For readers with separate pages (library, reading, bookmarks, settings):

```json
// manifest.json
{
  "router": {
    "entry": "pages/library",
    "pages": {
      "pages/library": { "component": "library" },
      "pages/reader": { "component": "reader" },
      "pages/bookmarks": { "component": "bookmarks" },
      "pages/settings": { "component": "settings" }
    }
  }
}
```

```javascript
// Navigation
import router from '@system.router'

export default {
  openBook(bookId) {
    router.push({
      uri: 'pages/reader',
      params: { bookId: bookId }
    })
  },

  showBookmarks() {
    router.push({ uri: 'pages/bookmarks' })
  },

  onBackPress() {
    // Reader → library
    router.replace({ uri: 'pages/library' })
    return true
  }
}
```

## Book List (Library Page)

```vue
<template>
  <div class="container">
    <text class="header">Library</text>
    <list class="book-list">
      <list-item for="{{ books }}" tid="id" class="book-item" @click="openBook($item.id)">
        <image src="{{ $item.cover }}" class="cover"></image>
        <div class="book-info">
          <text class="book-title">{{ $item.title }}</text>
          <text class="book-author">{{ $item.author }}</text>
          <text class="book-progress">{{ $item.progress }}%</text>
        </div>
      </list-item>
    </list>
  </div>
</template>
```

## Performance Optimization

### For large files
- Load only visible segments + buffer (not the entire file)
- Use `readArrayBuffer` instead of `readText` for encoding control
- Segment size: 5-10 rows per 212×520px screen
- Buffer: 4-6 segments forward/backward
- Call `global.runGC()` after file loading

### For smooth scrolling
- `scrollBy()` for programmatic navigation
- Anti-duplicate lock (`pageLock`) to prevent double loads
- `tid` attribute on list-item for efficient rendering
- `@scrolltop` / `@scrollbottom` for continuous lazy loading

### For memory
- Use `file-based storage` not `@system.storage` for large content
- Clean segments outside buffer
- Limit bookmarks to 50 max

## UI Pattern for Narrow Screen

### Tap Navigation
- Top 30% → Previous page
- Bottom 30% → Next page
- Center 40% → Menu/Settings

### Reader Controls (on center tap)
```vue
<template>
  <!-- Overlay menu (visible on center tap) -->
  <div class="reader-menu" show="{{ showMenu }}">
    <div class="menu-row">
      <text class="menu-btn" @click="changeFontSize(-1)">A−</text>
      <text class="menu-btn" @click="toggleAutoRead">
        {{ autoRead ? '⏸' : '▶' }}
      </text>
      <text class="menu-btn" @click="changeFontSize(1)">A+</text>
    </div>
    <div class="menu-row">
      <text class="menu-btn" @click="addBookmark">🔖</text>
      <text class="menu-btn" @click="showBookmarks">📑</text>
      <text class="menu-btn" @click="changeOpacity(-1)">☀−</text>
    </div>
  </div>
</template>
```

## Testing Checklist

- [ ] Load text file (.txt) with UTF-16 encoding
- [ ] Load text file with UTF-8 encoding (fallback)
- [ ] Virtual scroll: next/previous segments
- [ ] Tap zones: top=prev, bottom=next
- [ ] Progress bar updates correctly
- [ ] Auto-read mode: start/pause/stop
- [ ] `setKeepScreenOn` activates/deactivates
- [ ] Bookmark add/remove/go-to
- [ ] Progress save persists after closing
- [ ] Settings (font, opacity) persist
- [ ] Library with navigable book list
- [ ] Multi-page navigation (library ↔ reader ↔ bookmarks)
- [ ] `onBackPress()` handles navigation correctly
- [ ] Large files (1MB+): no crash, stable memory
- [ ] `global.runGC()` after heavy operations
