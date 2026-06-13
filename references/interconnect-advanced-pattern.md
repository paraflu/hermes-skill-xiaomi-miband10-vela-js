# Interconnect Advanced Pattern

Advanced pattern for structured BLE communication between Mi Band 10 and Android companion app, based on code verified from real apps.

## Architecture

```
┌─────────────────────────────────────────┐
│  Interconn (main class)                 │
│  ├── Tag dispatching (_tagMap)           │
│  ├── send(tag, data) → Promise           │
│  └── on(tag, handler) / off(tag)         │
│                                          │
│  ┌─────────────────────────────────────┐ │
│  │  InterconnModule (base module)       │ │
│  │  ├── install(interconn)              │ │
│  │  ├── uninstall()                     │ │
│  │  └── InterconnModule.define(name, cl)| │
│  └─────────────────────────────────────┘ │
│                                          │
│  ┌─────────────────────────────────────┐ │
│  │  Handshake (connection detection)    │ │
│  │  ├── connect() → Promise<bool>       │ │
│  │  ├── retryCount / maxRetries         │ │
│  │  └── handshake_req / handshake_ack   │ │
│  └─────────────────────────────────────┘ │
│                                          │
│  ┌─────────────────────────────────────┐ │
│  │  FileTransfer (file over BLE)        │ │
│  │  ├── sendFile(name, data) → Promise  │ │
│  │  ├── chunkSize: 1024                │ │
│  │  └── file_meta / file_chunk          │ │
│  └─────────────────────────────────────┘ │
└─────────────────────────────────────────┘
```

## Components

### 1. Interconn Class (tag dispatching)

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
    this._tagMap = {}
    this._listening = false
  }

  /**
   * Register handler for a specific tag
   * @param {string} tag - Message tag name
   * @param {Function} handler - Callback(data)
   */
  on(tag, handler) {
    this._tagMap[tag] = handler
  }

  /**
   * Remove handler for a tag
   */
  off(tag) {
    delete this._tagMap[tag]
  }

  /**
   * Start listening for interconnect messages
   */
  startListening() {
    if (this._listening) return
    this._listening = true
    interconnect.on({
      message: (data) => this._onInterconnectMessage(data)
    })
  }

  /**
   * Stop listening
   */
  stopListening() {
    this._listening = false
    // Note: interconnect has no off() API, so
    // after stopListening messages arrive but are ignored
  }

  /**
   * Send message with tag
   * @returns {Promise}
   */
  send(tag, data) {
    const message = JSON.stringify({
      tag,
      data,
      timestamp: Date.now()
    })
    return runAsyncFunc(interconnect.send.bind(interconnect), { message })
  }

  /**
   * Internal dispatch: received message → handler for tag
   */
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

// Singleton
export const interconn = new Interconn()
```

### 2. InterconnModule (registerable modules)

Pattern to organize features into independent modules:

```javascript
// interconn-module.js
export default class InterconnModule {
  constructor() {
    this.interconn = null
    this._installed = false
  }

  /**
   * Install module: register tag handlers on Interconn
   */
  install(interconn) {
    this.interconn = interconn
    this._installed = true
    this._registerHandlers()
  }

  /**
   * Uninstall module: remove handlers
   */
  uninstall() {
    if (this._installed && this.interconn) {
      this._unregisterHandlers()
      this.interconn = null
      this._installed = false
    }
  }

  /**
   * Override in subclasses to register handlers
   */
  _registerHandlers() {
    // this.interconn.on('my_tag', (data) => this._handle(data))
  }

  _unregisterHandlers() {
    // this.interconn.off('my_tag')
  }

  /**
   * Register a class as a module
   */
  static define(name, ModuleClass) {
    InterconnModule.registry = InterconnModule.registry || {}
    InterconnModule.registry[name] = ModuleClass
  }

  static create(name, interconn) {
    const ModuleClass = InterconnModule.registry && InterconnModule.registry[name]
    if (!ModuleClass) throw new Error(`Module ${name} not defined`)
    const instance = new ModuleClass()
    instance.install(interconn)
    return instance
  }
}

InterconnModule.registry = {}
```

### 3. Handshake Protocol

Connection detection and confirmation with retry:

```javascript
// handshake.js
const TAG = 'Handshake'

export default class Handshake extends InterconnModule {
  constructor() {
    super()
    this.connected = false
    this.retryCount = 0
    this.maxRetries = 3
    this.timeout = 3000
    this._connectResolver = null
  }

  _registerHandlers() {
    this.interconn.on('handshake_ack', (data) => this._onAck(data))
  }

  _unregisterHandlers() {
    this.interconn.off('handshake_ack')
  }

  /**
   * Start handshake connection
   * @returns {Promise<boolean>} true if connected
   */
  connect() {
    return new Promise((resolve) => {
      this._connectResolver = resolve
      this._tryConnect()
    })
  }

  _tryConnect() {
    if (this.retryCount >= this.maxRetries) {
      this.connected = false
      if (this._connectResolver) this._connectResolver(false)
      return
    }

    this.interconn.send('handshake_req', {
      version: '1.0',
      retry: this.retryCount
    }).catch(() => {})

    setTimeout(() => {
      if (!this.connected) {
        this.retryCount++
        this._tryConnect()
      }
    }, this.timeout)
  }

  _onAck(data) {
    this.connected = true
    console.log(`Handshake OK: ${JSON.stringify(data)}`)
    if (this._connectResolver) this._connectResolver(true)
  }

  disconnect() {
    this.connected = false
    this.retryCount = 0
  }
}

InterconnModule.define('handshake', Handshake)
```

### 4. FileTransfer Module

Send chunked files over BLE:

```javascript
// interconn-file.js
const TAG = 'FileTransfer'

export default class FileTransfer extends InterconnModule {
  constructor() {
    super()
    this.chunkSize = 1024
    this._pending = {} // fileId -> { chunks[], totalChunks, resolver }
  }

  _registerHandlers() {
    this.interconn.on('transfer_ack', (data) => this._onAck(data))
  }

  _unregisterHandlers() {
    this.interconn.off('transfer_ack')
  }

  /**
   * Send file in chunks
   * @param {string} fileName
   * @param {Uint8Array|string} data
   * @returns {Promise}
   */
  sendFile(fileName, data) {
    return new Promise((resolve, reject) => {
      const fileId = Date.now().toString()
      const totalChunks = Math.ceil(data.length / this.chunkSize)

      this._pending[fileId] = {
        totalChunks,
        received: 0,
        resolver: resolve,
        rejecter: reject
      }

      // Send metadata
      this.interconn.send('file_meta', {
        fileId,
        fileName,
        totalChunks,
        size: data.length
      }).then(() => {
        // Send chunks
        for (let i = 0; i < totalChunks; i++) {
          const start = i * this.chunkSize
          const end = Math.min(start + this.chunkSize, data.length)
          const chunk = data.slice(start, end)
          this.interconn.send('file_chunk', {
            fileId,
            index: i,
            data: Array.from(chunk), // Uint8Array → array for JSON
            final: i === totalChunks - 1
          }).catch(reject)
        }
      }).catch(reject)
    })
  }

  _onAck(data) {
    const pending = this._pending[data.fileId]
    if (pending) {
      console.log(`File ${data.fileId} transferred (${data.size} bytes)`)
      pending.resolver(data)
      delete this._pending[data.fileId]
    }
  }

  /**
   * Handler for file reception (to implement in app)
   */
  onFileReceived(callback) {
    this.interconn.on('transfer_ack', (data) => callback(data))
  }
}

InterconnModule.define('fileTransfer', FileTransfer)
```

### 5. Promise Wrapper (runAsyncFunc)

Transforms any callback-based API into Promise:

```javascript
// runAsyncFunc.js
export function runAsyncFunc(func, ...args) {
  return new Promise((resolve, reject) => {
    try {
      const options = {
        ...args[0],
        success: (data) => resolve(data),
        fail: (err, code) => reject(err || { code, message: 'API call failed' }),
        complete: () => {}
      }
      func(options)
    } catch (e) {
      reject(e)
    }
  })
}

// Usage examples:
import storage from '@system.storage'
import vibrator from '@system.vibrator'
import sensor from '@system.sensor'

// Storage get
const data = await runAsyncFunc(storage.get.bind(storage), { key: 'settings' })

// Vibrator
await runAsyncFunc(vibrator.vibrate.bind(vibrator), { mode: 'short' })

// Sensor accelerometer
const accel = await runAsyncFunc(sensor.subscribeAccelerometer.bind(sensor), {
  interval: 'normal'
})

// File read
const fileData = await runAsyncFunc(file.readText.bind(file), {
  uri: 'internal://app/data.txt'
})
```

### 6. Combined Usage in App

```javascript
// app.ux — setup interconnect
import Interconn from './common/interconn'
import Handshake from './common/handshake'
import FileTransfer from './common/interconn-file'
import { InterconnModule } from './common/interconn-module'

const interconn = new Interconn()

export default {
  onCreate() {
    interconn.startListening()

    // Create modules
    const handshake = InterconnModule.create('handshake', interconn)
    const fileTransfer = InterconnModule.create('fileTransfer', interconn)

    // Register handlers for specific features
    interconn.on('sync_settings', (data) => {
      console.log('Settings from phone:', data)
      // Apply settings
    })

    interconn.on('navigate', (data) => {
      // Navigate to page requested by Android
      router.push({ uri: data.page, params: data.params })
    })

    // Start handshake
    handshake.connect().then((connected) => {
      if (connected) {
        console.log('Connected to phone!')
        interconn.send('app_ready', { version: '1.0' })
      } else {
        console.log('Offline mode')
      }
    })
  },

  onDestroy() {
    interconn.stopListening()
  }
}
```

## Android Side (Kotlin + XMS SDK)

```kotlin
import com.xiaomi.xms.wearable.Wearable
import com.xiaomi.xms.wearable.message.MessageApi
import com.xiaomi.xms.wearable.message.MessageClient

class BandInterconnectService(private val context: Context) {
    private lateinit var messageClient: MessageClient

    fun init() {
        messageClient = Wearable.getMessageApi(context)
        messageClient.addListener { messageEvent ->
            val json = String(messageEvent.data)
            handleMessage(json)
        }
    }

    private fun handleMessage(json: String) {
        val msg = JSONObject(json)
        val tag = msg.getString("tag")
        val data = msg.getJSONObject("data")

        when (tag) {
            "handshake_req" -> sendToBand(JSONObject(mapOf(
                "tag" to "handshake_ack",
                "data" to mapOf("status" to "ok", "version" to "1.0")
            )))
            "app_ready" -> println("Band app ready")
            "file_meta" -> handleFileMeta(data)
            "file_chunk" -> handleFileChunk(data)
        }
    }

    fun sendToBand(data: Any) {
        val payload = if (data is String) data.toByteArray()
                      else JSONObject(data as Map<*, *>).toString().toByteArray()
        messageClient.sendMessage("com.example.mybandapp", payload, null)
    }

    private fun handleFileMeta(data: JSONObject) {
        println("Receiving file: ${data.getString("fileName")} (${data.getInt("size")} bytes)")
    }

    private fun handleFileChunk(data: JSONObject) {
        val index = data.getInt("index")
        val isFinal = data.getBoolean("final")
        println("Chunk $index received${if (isFinal) " — COMPLETE" else ""}")
    }
}
```

## Best Practices

### Message Protocol
- Always use format `{ tag, data, timestamp }`
- Tags in `snake_case` (e.g., `handshake_req`, `file_meta`)
- Data always JSON-serializable
- Timestamp for ordering and anti-duplicate

### Error Handling
```javascript
// Every send should have a catch
interconn.send('data_sync', payload)
  .then(() => console.log('Sent OK'))
  .catch((err) => {
    console.error('Send failed:', err)
    // Show error UI
    prompt.showToast({ message: 'Sync failed' })
  })
```

### Connection Management
- Call `startListening()` in `onCreate()` of app.ux
- Stop in `onDestroy()`
- Handshake at startup + periodic retry
- If handshake fails, work in offline mode

### Performance
- Chunk file size: 512-1024 bytes (BLE MTU limit ~512 bytes)
- Do not send more than 5 messages/second
- Large files: progress bar + final confirmation
- Use `global.runGC()` after heavy transfers

## Testing Checklist

- [ ] Handshake connect/disconnect/reconnect
- [ ] Simple message round-trip
- [ ] Small file transfer (< 10 chunks)
- [ ] Large file transfer (100+ chunks)
- [ ] Reconnection after BLE loss
- [ ] Offline mode without crashes
- [ ] Invalid JSON payload → error handling
- [ ] Handshake timeout → correct retry
- [ ] Module install/uninstall without leaks
- [ ] Test with real Android app (not emulator)
