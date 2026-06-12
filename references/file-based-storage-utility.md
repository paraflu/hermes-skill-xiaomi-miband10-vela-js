# File-Based Storage Utility

## File-level key-value storage utility

> Source: com.bandbbs.ebook project `src/utils/storage.js`
> Official documentation status: **Not listed** — `@system.storage` exists, but this pattern is a more reliable alternative built on `@system.file`

## Background

`@system.storage` is the official KV storage API, but on real IoT devices it has issues:
- limited capacity
- inconsistent behavior across devices
- lacks batch operations

The reference project implements a file-based KV storage utility using `@system.file` and JSON persistence for better control.

## Full implementation code

```javascript
import file from '@system.file'

const fileSavedPath = 'internal://files/books/storage-api/savedFile'

// Global cache to avoid repeated file reads
if (typeof global.__storage_cache__ === 'undefined') {
  global.__storage_cache__ = null
}
if (typeof global.__storage_loading__ === 'undefined') {
  global.__storage_loading__ = false
}
if (typeof global.__storage_callbacks__ === 'undefined') {
  global.__storage_callbacks__ = []
}

function processCallbacks() {
  global.__storage_loading__ = false
  const callbacks = global.__storage_callbacks__
  global.__storage_callbacks__ = []
  callbacks.forEach(cb => {
    try {
      cb(global.__storage_cache__)
    } catch (e) {
      console.error('Storage callback error:', e)
    }
  })
}

function loadIfNeeded(callback) {
  if (global.__storage_cache__ !== null) {
    callback(global.__storage_cache__)
    return
  }
  global.__storage_callbacks__.push(callback)
  if (global.__storage_loading__) return

  global.__storage_loading__ = true
  file.readText({
    uri: fileSavedPath,
    success: function (data) {
      try {
        global.__storage_cache__ = JSON.parse(data.text) || {}
        if (typeof global.__storage_cache__ !== 'object') {
          global.__storage_cache__ = {}
        }
      } catch (e) {
        global.__storage_cache__ = {}
      }
      processCallbacks()
    },
    fail: function () {
      global.__storage_cache__ = {}
      processCallbacks()
    }
  })
}

function saveToFile() {
  const toWrite = (global.__storage_cache__ && typeof global.__storage_cache__ === 'object')
    ? global.__storage_cache__
    : {}
  file.writeText({
    uri: fileSavedPath,
    text: JSON.stringify(toWrite)
  })
}

// Read
function get(param = {}) {
  loadIfNeeded(data => {
    const key = param.key
    let val = (data && key !== undefined) ? data[key] : undefined
    if (val === undefined) {
      val = param.default !== undefined ? param.default : ''
    }
    if (param.success) param.success(val)
    if (param.complete) param.complete()
  })
}

// Write
function set(param = {}) {
  loadIfNeeded(data => {
    const safeData = data || {}
    if (safeData[param.key] !== param.value) {
      safeData[param.key] = param.value
      global.__storage_cache__ = safeData
      saveToFile()
    }
    if (param.success) param.success()
    if (param.complete) param.complete()
  })
}

// Save entire object in batch
function save(data, param = {}) {
  loadIfNeeded(() => {
    global.__storage_cache__ = (data && typeof data === 'object') ? data : {}
    saveToFile()
    if (param.success) param.success()
    if (param.complete) param.complete()
  })
}

// Clear
function clear(param = {}) {
  global.__storage_cache__ = {}
  saveToFile()
  if (param.success) param.success()
  if (param.complete) param.complete()
}

// Delete single key
function del(param = {}) {
  loadIfNeeded(data => {
    if (data && param.key in data) {
      delete data[param.key]
      global.__storage_cache__ = data
      saveToFile()
    }
    if (param.success) param.success()
    if (param.complete) param.complete()
  })
}

export default { get, set, clear, delete: del, save }
```

## Example usage

```javascript
import storage from '../../utils/storage'

// Read
storage.get({
  key: 'fontSize',
  default: '16',
  success: (val) => {
    console.log('Font size:', val)
  }
})

// Write
storage.set({
  key: 'fontSize',
  value: '20',
  success: () => {
    console.log('Save successful')
  }
})

// Save in batch
storage.save({
  fontSize: '20',
  theme: 'dark',
  language: 'zh-CN'
})

// Delete single key
storage.delete({ key: 'theme' })

// Clear all
storage.clear()
```

## Design highlights

1. **Global cache**: use `global.__storage_cache__` to avoid filesystem reads on every access
2. **Callback queue**: concurrent requests queue during first load so the file is read once
3. **Deferred writes**: write only when values actually change to reduce IO
4. **JSON persistence**: store entire data as a JSON file for easier debugging and backups

## Comparison with official `@system.storage`

| Feature | @system.storage | File-based utility |
|------|----------------|---------------|
| Implementation | system-level KV | @system.file + JSON |
| Capacity limits | device-dependent, usually small | limited by filesystem space |
| Batch operations | not supported | supports `save()` batch write |
| Data format | strings only | any JSON-serializable value |
| Debug-friendly | not directly viewable | can read the JSON file directly |
| Caching | none | built-in global cache |
| Delete single key | set empty string | actual delete operation |
