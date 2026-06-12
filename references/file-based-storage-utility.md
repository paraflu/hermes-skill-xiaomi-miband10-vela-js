# File-Based Storage Utility

# 文件级键值存储工具（File-based Storage Utility）

> 来源：com.bandbbs.ebook 项目 `src/utils/storage.js`
> 官方文档状态：**官方未收录** — 官方有 `@system.storage` 接口，但本模式是基于 `@system.file` 实现的更可靠替代方案

## 背景

`@system.storage` 是官方提供的键值存储接口，但在实际 IoT 设备上存在以下问题：
- 存储容量有限
- 不同设备行为不一致
- 缺乏批量操作能力

参考项目实现了一个基于 `@system.file` 的键值存储工具，使用 JSON 文件持久化，具有更好的可控性。

## 完整实现代码

```javascript
import file from '@system.file'

const fileSavedPath = 'internal://files/books/storage-api/savedFile'

// 全局缓存，避免重复读取文件
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

// 读取
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

// 写入
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

// 批量保存整个对象
function save(data, param = {}) {
  loadIfNeeded(() => {
    global.__storage_cache__ = (data && typeof data === 'object') ? data : {}
    saveToFile()
    if (param.success) param.success()
    if (param.complete) param.complete()
  })
}

// 清空
function clear(param = {}) {
  global.__storage_cache__ = {}
  saveToFile()
  if (param.success) param.success()
  if (param.complete) param.complete()
}

// 删除单个键
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

## 使用示例

```javascript
import storage from '../../utils/storage'

// 读取
storage.get({
  key: 'fontSize',
  default: '16',
  success: (val) => {
    console.log('字体大小:', val)
  }
})

// 写入
storage.set({
  key: 'fontSize',
  value: '20',
  success: () => {
    console.log('保存成功')
  }
})

// 批量保存
storage.save({
  fontSize: '20',
  theme: 'dark',
  language: 'zh-CN'
})

// 删除单个键
storage.delete({ key: 'theme' })

// 清空所有
storage.clear()
```

## 设计要点

1. **全局缓存**：使用 `global.__storage_cache__` 避免每次读取都访问文件系统
2. **回调队列**：首次加载时，多个并发请求排队等待，只读一次文件
3. **延迟写入**：只有值实际变化时才写文件，减少 IO 操作
4. **JSON 持久化**：整个存储作为一个 JSON 文件，便于调试和备份

## 与官方 @system.storage 的对比

| 特性 | @system.storage | 文件级存储工具 |
|------|----------------|---------------|
| 底层实现 | 系统级 KV 存储 | @system.file + JSON |
| 容量限制 | 设备相关，通常较小 | 受限于文件系统空间 |
| 批量操作 | 不支持 | 支持 save() 批量写入 |
| 数据格式 | 仅 String | 任意 JSON 可序列化值 |
| 调试友好 | 不可直接查看 | 可直接读取 JSON 文件 |
| 缓存机制 | 无 | 内置全局缓存 |
| 删除单键 | 设空字符串 | 真正的 delete 操作 |
