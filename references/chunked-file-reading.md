# Chunked File Reading

# 大文件分块读取模式（Chunked File Reading）

> 来源：com.bandbbs.ebook 项目 `src/pages/detail/detail.ux`
> 官方文档状态：**官方接口已有，但未提供大文件分块读取的实践模式** — `file.readArrayBuffer` 支持 `position` 和 `length` 参数，但官方示例仅展示单次读取

## 背景

IoT 设备内存有限（通常 128MB-512MB），不能一次性将大文件读入内存。电子书应用需要读取几 MB 的文本文件，采用了基于 `file.readArrayBuffer` 的分块读取策略。

## 核心实现

```javascript
import file from '@system.file'

/**
 * 分块读取大文件
 * @param {string} uri - 文件路径
 * @param {number} fileSize - 文件总大小（字节）
 * @param {number} chunkSize - 每块大小（字节），建议 8192-65536
 * @param {Function} onChunk - 每块回调 (chunkText, offset, isLast)
 * @param {Function} onComplete - 全部读取完成回调
 */
function readLargeFile(uri, fileSize, chunkSize, onChunk, onComplete) {
  let readOffset = 0

  function attemptRead(len, attemptsLeft) {
    file.readArrayBuffer({
      uri: uri,
      position: readOffset,
      length: len,
      success: (data) => {
        if (!data.buffer || data.buffer.byteLength === 0) {
          if (onComplete) onComplete()
          return
        }

        const buffer = new Uint8Array(data.buffer)
        let bytesToUse = buffer.length

        // 解码为文本（UTF-16LE 编码的文本文件）
        const text = decodeUTF16LE(buffer, bytesToUse)

        readOffset += bytesToUse
        const isLast = readOffset >= fileSize

        onChunk(text, readOffset, isLast)

        if (!isLast) {
          // 继续读取下一块
          attemptRead(chunkSize, 3)
        } else {
          if (onComplete) onComplete()
        }
      },
      fail: (data, code) => {
        if (attemptsLeft > 0) {
          // 重试
          setTimeout(() => attemptRead(len, attemptsLeft - 1), 100)
        } else {
          console.error('读取失败:', code)
          if (onComplete) onComplete()
        }
      }
    })
  }

  attemptRead(chunkSize, 3)
}

/**
 * UTF-16LE 解码
 */
function decodeUTF16LE(buffer, length) {
  let result = ''
  for (let i = 0; i < length - 1; i += 2) {
    const charCode = buffer[i + 1] * 256 + buffer[i]
    result += String.fromCharCode(charCode)
  }
  return result
}
```

## 防止段落被截断的优化

分块读取可能在段落中间截断。参考项目实现了段落完整性保护：

```javascript
/**
 * 带段落保护的分块读取
 * 
 * 关键思想：读取一块数据后，从末尾向前查找最后一个换行符，
 * 只处理到换行符为止的内容，剩余部分与下一块合并。
 */
function readWithParagraphProtection(uri, fileSize, chunkSize, onChunk, onComplete) {
  let readOffset = 0
  let remainder = ''  // 上一块未处理的尾部

  function readNext(len) {
    file.readArrayBuffer({
      uri: uri,
      position: readOffset,
      length: len,
      success: (data) => {
        if (!data.buffer || data.buffer.byteLength === 0) {
          // 处理最后的余留内容
          if (remainder) onChunk(remainder, readOffset, true)
          if (onComplete) onComplete()
          return
        }

        const buffer = new Uint8Array(data.buffer)
        let bytesToUse = buffer.length
        const isLast = (readOffset + buffer.length) >= fileSize

        if (!isLast) {
          // 从末尾找最后一个换行符（UTF-16LE 中换行是 0x000A）
          let lastNewlinePos = -1
          for (let i = buffer.length - 2; i >= 0; i -= 2) {
            const charCode = buffer[i + 1] * 256 + buffer[i]
            if (charCode === 10) { // '\n'
              lastNewlinePos = i
              break
            }
          }
          if (lastNewlinePos >= 0) {
            bytesToUse = lastNewlinePos + 2
          }
        }

        const text = remainder + decodeUTF16LE(buffer, bytesToUse)
        remainder = ''

        if (!isLast && bytesToUse < buffer.length) {
          // 保存未处理的尾部
          remainder = decodeUTF16LE(
            buffer.slice(bytesToUse),
            buffer.length - bytesToUse
          )
        }

        readOffset += buffer.length
        onChunk(text, readOffset, isLast)

        if (!isLast) {
          readNext(chunkSize)
        } else if (onComplete) {
          onComplete()
        }
      },
      fail: () => {
        setTimeout(() => readNext(len), 100)
      }
    })
  }

  readNext(chunkSize)
}
```

## 使用示例

```javascript
readWithParagraphProtection(
  'internal://files/books/novel.txt',
  1024 * 1024,  // 文件大小 1MB
  16384,         // 每块 16KB
  (text, offset, isLast) => {
    // 将文本追加到显示区域
    this.content += text
    // 更新进度
    this.readProgress = Math.floor(offset / (1024 * 1024) * 100)
  },
  () => {
    console.log('读取完成')
  }
)
```

## 性能建议

1. **块大小选择**：8KB-64KB 为宜，太小会增加 IO 次数，太大占用过多内存
2. **使用 `position` 参数**：避免读取整个文件再截取
3. **UTF-16LE 注意事项**：Vela 系统的文本文件可能是 UTF-16LE 编码，每个字符占 2 字节，分块时需要对齐到 2 字节边界
4. **重试机制**：IoT 设备 IO 可能不稳定，建议实现 3 次重试
5. **进度反馈**：大文件读取耗时较长，应向用户显示进度
