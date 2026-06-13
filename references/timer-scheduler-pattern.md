# Timer & Scheduler Pattern for Vela JS Apps

Validated pattern for apps that execute recurring time-based actions (hourly vibration, reminders, scheduled notifications).

## Architecture Overview

```
┌─────────────────────────────────┐
│  setInterval(1000ms)            │  Main timer
│  └─> updateTime()               │  Updates UI every second
│      └─> checkForEvent()        │  Checks trigger conditions
│          └─> performAction()    │  Performs action if match
│              └─> saveState()    │  Persists for anti-duplicate
└─────────────────────────────────┘
```

## Core Pattern: Hourly Event Trigger

### 1. Timer Setup in onInit()

```javascript
export default {
  data: {
    currentTime: '00:00:00',
    isEnabled: false,
    lastTriggeredHour: -1,
    checkIntervalId: null
  },
  
  onInit() {
    this.loadSettings()
    this.updateTime()
    this.startTimeCheck()
  },
  
  onDestroy() {
    if (this.checkIntervalId) {
      clearInterval(this.checkIntervalId)
    }
  },
  
  startTimeCheck() {
    // Update every second for precision and live UI
    this.checkIntervalId = setInterval(() => {
      this.updateTime()
      
      if (this.isEnabled) {
        this.checkForHourlyEvent()
      }
    }, 1000)
  }
}
```

**Rationale: 1 second vs 60 seconds**
- Pro: Precise capture of the :00 moment, UI always updated
- Con: Battery consumption ~2-3% per day (acceptable)
- Alternative `setInterval(60000)` risks missing the exact trigger

### 2. Event Detection with Capture Window

```javascript
checkForHourlyEvent() {
  const now = new Date()
  const currentHour = now.getHours()
  const currentMinute = now.getMinutes()
  const currentSecond = now.getSeconds()
  
  // 2-second window to compensate for timer drift
  if (currentMinute === 0 && 
      currentSecond <= 2 && 
      this.lastTriggeredHour !== currentHour) {
    
    this.performAction()
    this.lastTriggeredHour = currentHour
    this.saveLastTriggeredHour(currentHour)
    
    console.log(`Event triggered at ${this.currentTime}`)
  }
}
```

**Pitfall: setInterval drift**
- `setInterval(1000)` is not millisecond-precise
- Can trigger at :00:00.800 or :01:00.200
- **Solution**: 2-3 second window + duplicate protection via `lastTriggeredHour`

### 3. Multi-State Storage Pattern

```javascript
import storage from '@system.storage'

// State 1: Feature enabled/disabled
saveSettings() {
  storage.set({
    key: 'app_enabled',
    value: this.isEnabled.toString()
  })
}

loadSettings() {
  storage.get({
    key: 'app_enabled',
    success: (data) => {
      this.isEnabled = data === 'true'
    },
    fail: () => {
      this.isEnabled = false  // Safe default
    }
  })
}

// State 2: Daily counter with auto-reset
saveCounter() {
  const data = {
    date: new Date().toDateString(),
    count: this.counterToday
  }
  
  storage.set({
    key: 'app_counter',
    value: JSON.stringify(data)
  })
}

loadCounter() {
  storage.get({
    key: 'app_counter',
    success: (data) => {
      const saved = JSON.parse(data)
      const today = new Date().toDateString()
      
      if (saved.date === today) {
        this.counterToday = saved.count
      } else {
        this.counterToday = 0  // New day
        this.saveCounter()
      }
    },
    fail: () => {
      this.counterToday = 0
    }
  })
}

// State 3: Last triggered (anti-duplicate)
saveLastTriggeredHour(hour) {
  storage.set({
    key: 'app_lastHour',
    value: hour.toString()
  })
}

loadLastTriggeredHour() {
  storage.get({
    key: 'app_lastHour',
    success: (data) => {
      this.lastTriggeredHour = parseInt(data)
    },
    fail: () => {
      this.lastTriggeredHour = -1
    }
  })
}
```

**Why 3 separate keys**:
- Easier debugging (each state independent)
- Selective reset (e.g., counter without disabling feature)
- Backward compatibility (add state 4 without migrating everything)

### 4. UI Update Pattern

```javascript
updateTime() {
  const now = new Date()
  const hours = String(now.getHours()).padStart(2, '0')
  const minutes = String(now.getMinutes()).padStart(2, '0')
  const seconds = String(now.getSeconds()).padStart(2, '0')
  
  this.currentTime = `${hours}:${minutes}:${seconds}`
  
  // Calculate next trigger
  const nextHour = (now.getHours() + 1) % 24
  this.nextEventTime = `${String(nextHour).padStart(2, '0')}:00`
}
```

**Automatic binding**: Changing `this.currentTime` automatically updates `{{ currentTime }}` in the template

## Optimized UI Template 212×520px

```vue
<template>
  <div class="container">
    <!-- Header -->
    <text class="title">App Title</text>
    
    <!-- Time Display (focal point) -->
    <div class="time-display">
      <text class="current-time">{{ currentTime }}</text>
    </div>
    
    <!-- Status Row -->
    <div class="status-section">
      <text class="status-label">Stato:</text>
      <text class="status-value" style="color: {{ isEnabled ? '#00ff00' : '#ff6b6b' }}">
        {{ isEnabled ? 'ATTIVO' : 'DISATTIVO' }}
      </text>
    </div>
    
    <!-- Info Section -->
    <div class="info-section">
      <text class="info-text">Prossimo: {{ nextEventTime }}</text>
      <text class="info-text">Oggi: {{ counterToday }}</text>
    </div>
    
    <!-- Buttons -->
    <div class="button-section">
      <input 
        type="button" 
        class="btn btn-primary" 
        value="{{ isEnabled ? 'Disattiva' : 'Attiva' }}" 
        @click="toggleService" 
      />
      <input 
        type="button" 
        class="btn btn-test" 
        value="Test" 
        @click="testAction" 
      />
    </div>
    
    <text class="footer">Footer text</text>
  </div>
</template>

<style>
.container {
  flex-direction: column;
  justify-content: flex-start;
  align-items: center;
  width: 100%;
  height: 100%;
  background-color: #000000;
  padding-top: 60px;
}

.title {
  font-size: 38px;
  color: #ffffff;
  font-weight: bold;
  margin-bottom: 40px;
}

.time-display {
  width: 400px;
  height: 120px;
  background-color: #1a1a1a;
  border-radius: 20px;
  justify-content: center;
  align-items: center;
  margin-bottom: 30px;
}

.current-time {
  font-size: 56px;
  color: #00ffff;
  font-weight: bold;
  font-family: monospace;
}

.status-section {
  flex-direction: row;
  align-items: center;
  margin-bottom: 30px;
}

.status-label {
  font-size: 32px;
  color: #cccccc;
  margin-right: 15px;
}

.status-value {
  font-size: 32px;
  font-weight: bold;
}

.info-section {
  flex-direction: column;
  align-items: center;
  margin-bottom: 40px;
}

.info-text {
  font-size: 26px;
  color: #999999;
  margin-bottom: 10px;
}

.button-section {
  flex-direction: column;
  align-items: center;
  width: 100%;
}

.btn {
  width: 360px;
  height: 70px;
  font-size: 32px;
  color: #ffffff;
  border-radius: 35px;
  margin-bottom: 20px;
}

.btn-primary {
  background-color: #007aff;
}

.btn-test {
  background-color: #ff9500;
}

.footer {
  font-size: 24px;
  color: #666666;
  margin-top: 30px;
}
</style>
```

**Design System for 212×520px (designWidth: 480)**:
- Font title: 38-40px
- Font primary: 32px
- Font secondary: 26-28px
- Font footer: 24px
- Button height: 70px (touch target)
- Button width: 360px (80% screen width)
- Margin standard: 20-40px
- Border radius: 20-35px

## Variations

### Every N Minutes

```javascript
checkForIntervalEvent() {
  const now = new Date()
  const currentMinute = now.getMinutes()
  const currentSecond = now.getSeconds()
  
  // Every 15 minutes (:00, :15, :30, :45)
  if (currentMinute % 15 === 0 && 
      currentSecond <= 2 &&
      this.lastTriggeredMinute !== currentMinute) {
    
    this.performAction()
    this.lastTriggeredMinute = currentMinute
  }
}
```

### Time Range Filter (e.g., daytime only)

```javascript
data: {
  enabledStartHour: 8,
  enabledEndHour: 22
}

checkForHourlyEvent() {
  const hour = now.getHours()
  
  // Out of range → skip
  if (hour < this.enabledStartHour || hour >= this.enabledEndHour) {
    return
  }
  
  // ... rest of logic
}
```

### Multi-Pattern Actions

```javascript
data: {
  actionPattern: 'single'  // 'single', 'double', 'triple'
}

performAction() {
  switch(this.actionPattern) {
    case 'double':
      vibrator.vibrate({ mode: 'short' })
      setTimeout(() => vibrator.vibrate({ mode: 'short' }), 500)
      break
    case 'triple':
      vibrator.vibrate({ mode: 'short' })
      setTimeout(() => vibrator.vibrate({ mode: 'short' }), 300)
      setTimeout(() => vibrator.vibrate({ mode: 'short' }), 600)
      break
    default:
      vibrator.vibrate({ mode: 'long' })
  }
}
```

## Testing Checklist

### Functional
- [ ] Test action works (immediate button)
- [ ] Enable/disable toggle updates state and UI
- [ ] Timer updates UI every second
- [ ] Next event calculation correct

### Temporal (requires patience)
- [ ] Wait for :59:58 → verify trigger at :00:00
- [ ] Verify NO trigger at :00:05 (duplicate protection)
- [ ] Restart app at :00:30 → NO trigger for already passed hour

### Persistence
- [ ] Enable, close, reopen → state preserved
- [ ] Trigger at 10:00, restart at 10:30 → counter correct
- [ ] Trigger at 23:00, past midnight → counter reset to 0

### Edge Cases
- [ ] Manual device time change → NO multiple triggers
- [ ] App in background → continues working
- [ ] Low battery → graceful behavior

## Pitfalls

### 1. App killed by system
**Problem**: HyperOS can terminate the app if memory is low
**Impact**: Timer stops until manual restart
**Workaround**: Android companion app with AlarmManager + BLE wakeup

### 2. Timer drift on long run
**Problem**: `setInterval(1000)` accumulates drift (~1-2s/hour)
**Solution**: 2-3 second capture window compensates for normal drift

### 3. No autostart after boot
**Problem**: Quick App has no autostart capability
**Workaround**: Home screen widget or companion app that sends wakeup message

### 4. Storage callback-based (no async/await)
**Common mistake**:
```javascript
// ❌ Does NOT work
const data = await storage.get({ key: 'x' })
```

**Correct**:
```javascript
// ✅ Use callback
storage.get({
  key: 'x',
  success: (data) => {
    this.value = data
  }
})
```

## Project Structure Best Practice

```
my-scheduler-app/
├── manifest.json
├── app.ux
├── pages/index/index.ux      # Main UI + timer logic
├── common/
│   └── logo.png
├── README.md                  # User-facing doc
├── DEVELOPMENT.md             # Technical notes
└── build.sh
```

**Documentation split**:
- `README.md`: User guide, build, installation, customizations
- `DEVELOPMENT.md`: Design decisions, technical patterns, testing, known issues

## Advanced: onBackPress Pattern

```javascript
onBackPress() {
  if (this.isEnabled) {
    // Ask for confirmation before exiting
    prompt.showToast({ message: 'Disabilita prima di uscire' })
    return true // Block exit
  }
  this.saveSettings() // Save before exiting
  return false // Allow exit
}
```

## Advanced: onHide Lifecycle

Saves state when the app goes to background (e.g., notification arrives):

```javascript
onShow() {
  if (this.isEnabled && !this.checkIntervalId) {
    this.startTimeCheck() // Resume timer if it was active
  }
},

onHide() {
  // Save state before hiding
  this.saveSettings()
  this.saveCounter()
},

onDestroy() {
  // Full cleanup
  if (this.checkIntervalId) {
    clearInterval(this.checkIntervalId)
    this.checkIntervalId = null
  }
  global.runGC() // Force garbage collection
}
```

## Advanced: global.runGC()

After heavy operations (e.g., batch calculation, UI refresh):

```javascript
refreshUI() {
  this.updateTime()
  this.checkForHourlyEvent()
  this.updateStats()
  global.runGC() // Free memory after refresh
}
```

## Performance Notes

- 1s timer consumes ~2-3% battery/day (acceptable)
- Limited RAM: max 3-4 complex components per page
- Avoid unnecessary re-renders: check value before assigning to `data`
- Call `global.runGC()` after UI refresh or heavy calculations
- Use `onHide()` to save state and stop timer when the app goes to background
- Use `onBackPress()` to handle clean exit

## Session Reference

Created from session 2026-06-10: fully working "Hourly Vibrator" app with all pattern features.
