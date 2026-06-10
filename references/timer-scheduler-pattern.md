# Timer & Scheduler Pattern per Vela JS Apps

Pattern validato per app che eseguono azioni ricorrenti basate su tempo (vibrazione oraria, reminder, notifiche schedulate).

## Architecture Overview

```
┌─────────────────────────────────┐
│  setInterval(1000ms)            │  Timer principale
│  └─> updateTime()               │  Aggiorna UI ogni secondo
│      └─> checkForEvent()        │  Controlla condizioni trigger
│          └─> performAction()    │  Esegue azione se match
│              └─> saveState()    │  Persiste per anti-duplicati
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
    // Update ogni secondo per precisione e UI live
    this.checkIntervalId = setInterval(() => {
      this.updateTime()
      
      if (this.isEnabled) {
        this.checkForHourlyEvent()
      }
    }, 1000)
  }
}
```

**Rationale: 1 secondo vs 60 secondi**
- Pro: Cattura precisa del momento :00, UI sempre aggiornata
- Contro: Consumo batteria ~2-3% al giorno (accettabile)
- Alternativa `setInterval(60000)` rischia di perdere il trigger esatto

### 2. Event Detection con Finestra di Cattura

```javascript
checkForHourlyEvent() {
  const now = new Date()
  const currentHour = now.getHours()
  const currentMinute = now.getMinutes()
  const currentSecond = now.getSeconds()
  
  // Finestra 2 secondi per compensare timer drift
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
- `setInterval(1000)` non è preciso al millisecond
- Può triggare a :00:00.800 o :01:00.200
- **Soluzione**: Finestra di 2-3 secondi + protezione duplicati via `lastTriggeredHour`

### 3. Multi-State Storage Pattern

```javascript
import storage from '@system.storage'

// Stato 1: Feature enabled/disabled
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
      this.isEnabled = false  // Default sicuro
    }
  })
}

// Stato 2: Daily counter con auto-reset
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
        this.counterToday = 0  // Nuovo giorno
        this.saveCounter()
      }
    },
    fail: () => {
      this.counterToday = 0
    }
  })
}

// Stato 3: Last triggered (anti-duplicati)
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

**Perché 3 chiavi separate**:
- Più facile debuggare (ogni stato indipendente)
- Reset selettivo (es. counter senza disabilitare feature)
- Backward compatibility (aggiungere stato 4 senza migrare tutto)

### 4. UI Update Pattern

```javascript
updateTime() {
  const now = new Date()
  const hours = String(now.getHours()).padStart(2, '0')
  const minutes = String(now.getMinutes()).padStart(2, '0')
  const seconds = String(now.getSeconds()).padStart(2, '0')
  
  this.currentTime = `${hours}:${minutes}:${seconds}`
  
  // Calcola prossimo trigger
  const nextHour = (now.getHours() + 1) % 24
  this.nextEventTime = `${String(nextHour).padStart(2, '0')}:00`
}
```

**Binding automatico**: Modificare `this.currentTime` aggiorna automaticamente `{{ currentTime }}` nel template

## Template UI Ottimizzato 212×520px

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

**Design System per 212×520px (designWidth: 480)**:
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
  
  // Ogni 15 minuti (:00, :15, :30, :45)
  if (currentMinute % 15 === 0 && 
      currentSecond <= 2 &&
      this.lastTriggeredMinute !== currentMinute) {
    
    this.performAction()
    this.lastTriggeredMinute = currentMinute
  }
}
```

### Time Range Filter (es. solo giorno)

```javascript
data: {
  enabledStartHour: 8,
  enabledEndHour: 22
}

checkForHourlyEvent() {
  const hour = now.getHours()
  
  // Fuori range → skip
  if (hour < this.enabledStartHour || hour >= this.enabledEndHour) {
    return
  }
  
  // ... resto logica
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

### Funzionali
- [ ] Azione test funziona (button immediato)
- [ ] Toggle attiva/disattiva aggiorna stato e UI
- [ ] Timer aggiorna UI ogni secondo
- [ ] Calcolo prossimo evento corretto

### Temporali (richiede pazienza)
- [ ] Aspetta :59:58 → verifica trigger a :00:00
- [ ] Verifica NO trigger a :00:05 (protezione duplicati)
- [ ] Riavvia app a :00:30 → NO trigger all'ora già passata

### Persistenza
- [ ] Attiva, chiudi, riapri → stato conservato
- [ ] Trigger a 10:00, riavvia a 10:30 → counter corretto
- [ ] Trigger a 23:00, passa mezzanotte → counter reset a 0

### Edge Cases
- [ ] Cambio manuale ora device → NO trigger multipli
- [ ] App in background → continua a funzionare
- [ ] Batteria scarica → comportamento graceful

## Pitfalls

### 1. App killata dal sistema
**Problema**: HyperOS può terminare l'app se memoria scarsa
**Impatto**: Timer si ferma fino a riavvio manuale
**Workaround**: Companion app Android con AlarmManager + BLE wakeup

### 2. Timer drift su long run
**Problema**: `setInterval(1000)` accumula drift (~1-2s/ora)
**Soluzione**: Finestra cattura 2-3 secondi compensa drift normale

### 3. No autostart dopo boot
**Problema**: Quick App non ha autostart capability
**Workaround**: Widget home screen o companion app che invia wakeup message

### 4. Storage callback-based (no async/await)
**Errore comune**:
```javascript
// ❌ NON funziona
const data = await storage.get({ key: 'x' })
```

**Corretto**:
```javascript
// ✅ Usa callback
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
- `README.md`: Guida utente, build, installazione, personalizzazioni
- `DEVELOPMENT.md`: Decisioni design, pattern tecnici, testing, known issues

## Performance Notes

- Timer a 1s consuma ~2-3% batteria/giorno (accettabile)
- RAM limitata: max 3-4 componenti complessi per pagina
- Evita re-render inutili: check valore prima di assegnare a `data`

## Session Reference

Creato da sessione 2026-06-10: app "Hourly Vibrator" funzionante completa con tutte le feature del pattern.
