# Chunked File Reading

## Chunked File Reading Pattern

> Source: com.bandbbs.ebook project `src/pages/detail/detail.ux`
> Official documentation status: **The API exists, but no practical chunked-reading pattern is provided** — `file.readArrayBuffer` supports `position` and `length`, but official examples only show single-shot reads

## Background

IoT devices have limited memory (typically 128MB–512MB) and cannot load large files into memory at once. E-book applications need to read multi-megabyte text files; a chunked reading strategy using `file.readArrayBuffer` is used.

## Core implementation

```javascript
import file from '@system.file'

/**
 * Read a large file in chunks
 * @param {string} uri - file path
 * @param {number} fileSize - total file size (bytes)
 * @param {number} chunkSize - chunk size (bytes), recommended 8192-65536
 * @param {Function} onChunk - chunk callback (chunkText, offset, isLast)
 * @param {Function} onComplete - called when all chunks are read
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

        // Decode to text (UTF-16LE encoded text file)
        const text = decodeUTF16LE(buffer, bytesToUse)

        readOffset += bytesToUse
        const isLast = readOffset >= fileSize

        onChunk(text, readOffset, isLast)

        if (!isLast) {
          // Continue reading next chunk
          attemptRead(chunkSize, 3)
        } else {
          if (onComplete) onComplete()
        }
      },
      fail: (data, code) => {
        if (attemptsLeft > 0) {
          // Retry
          setTimeout(() => attemptRead(len, attemptsLeft - 1), 100)
        } else {
          console.error('Read failed:', code)
          if (onComplete) onComplete()
        }
      }
    })
  }

  attemptRead(chunkSize, 3)
}

/**
 * UTF-16LE decoding
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

## Paragraph truncation prevention (optimization)

Chunked reads may split paragraphs in the middle. The reference project implements paragraph integrity protection:

```javascript
/**
 * Chunked reading with paragraph protection
 *
 * Key idea: after reading a chunk, search backwards for the last newline,
 * process only up to that newline and merge the remainder with the next chunk.
 */
function readWithParagraphProtection(uri, fileSize, chunkSize, onChunk, onComplete) {
  let readOffset = 0
  let remainder = ''  // Unprocessed tail from previous chunk

  function readNext(len) {
    file.readArrayBuffer({
      uri: uri,
      position: readOffset,
      length: len,
      success: (data) => {
        if (!data.buffer || data.buffer.byteLength === 0) {
          // Handle final remainder
          if (remainder) onChunk(remainder, readOffset, true)
          if (onComplete) onComplete()
          return
        }

        const buffer = new Uint8Array(data.buffer)
        let bytesToUse = buffer.length
        const isLast = (readOffset + buffer.length) >= fileSize

        if (!isLast) {
          // Find last newline from the end (newline is 0x000A in UTF-16LE)
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
          // Save unprocessed tail
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

## Example usage

```javascript
readWithParagraphProtection(
  'internal://files/books/novel.txt',
  1024 * 1024,  // file size 1MB
  16384,         // chunk 16KB
  (text, offset, isLast) => {
    // Append text to the display area
    this.content += text
    // Update progress
    this.readProgress = Math.floor(offset / (1024 * 1024) * 100)
  },
  () => {
    console.log('Read complete')
  }
)
```

## Performance recommendations

1. **Chunk size**: 8KB–64KB recommended; too small increases IO, too large uses excessive memory
2. **Use `position` parameter**: avoid reading the whole file and slicing
3. **UTF-16LE considerations**: Vela system text files may be UTF-16LE encoded (2 bytes per character); align chunks to 2-byte boundaries
4. **Retry mechanism**: IoT device IO can be unstable; implement 3 retries
5. **Progress feedback**: reading large files takes time; show progress to the user
