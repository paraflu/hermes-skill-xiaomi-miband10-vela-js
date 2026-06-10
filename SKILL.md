---
name: xiaomi-miband10-vela-js
description: Sviluppo applicazioni per Xiaomi Mi Band 10 usando Vela JS Quick App framework
version: 1.0.0
tags: [xiaomi, miband, wearable, vela-js, quick-app, iot, javascript]
platforms: [linux, macos, windows]
---

# Xiaomi Mi Band 10 - Vela JS Development

Sviluppo di applicazioni Quick App per Xiaomi Mi Band 10 usando la piattaforma Xiaomi Vela JS (basata su OpenVela RTOS).

## When to use this skill

- Sviluppare nuove applicazioni per Mi Band 10
- Comunicare con app Android companion via BLE
- Usare sensori del dispositivo (accelerometro, barometro)
- Implementare UI su schermo AMOLED 1.72" (212×520px)
- Integrare con XMS Wearable SDK Android

## Stack tecnologico

| Layer | Tecnologia |
|-------|-----------|
| Runtime dispositivo | Xiaomi HyperOS 2 / OpenVela (RTOS basato su NuttX) |
| Linguaggio app | JavaScript (ES6+) |
| Template | File .ux (sintassi simile a Vue SFC: `<template>`, `<script>`, `<style>`) |
| IDE | AIoT-IDE (fork di VS Code) |
| Build output | .rpk (installabile via Mi Fitness / AIoT-IDE) |
| Comunicazione Android | system.interconnect (BLE automatico, tramite XMS Wearable SDK) |
| SDK Android companion | xms-wearable-lib_1.4_release.aar |

## Pre-requisiti

### 1. Installazione AIoT-IDE

```bash
# Download da https://iot.mi.com/vela/quickapp/en/guide/start/use-ide.html
# Estrai e avvia l'IDE (basato su VS Code)
# Configura l'emulatore Vela integrato per test
```

### 2. Clona repository esempi ufficiali

```bash
cd ~/projects
git clone --depth=1 --branch dev https://github.com/open-vela/packages_apps.git
cd packages_apps/wearable/
```

**Esempi chiave da studiare (in ordine):**

1. **`vela-sample/`** — Punto di partenza obbligatorio. Showcase completo di tutti componenti, API, pattern
2. **`interconnect_image_demo/`** — Critico per comunicazione Band↔Android. Include:
   - Codice watch (Quick App)
   - Codice Android (`android_program/`)
   - AAR SDK (`libs/xms-wearable-lib_1.4_release.aar`)
3. **`calendar/`**, **`settings/`** — Pattern app multi-pagina con routing
4. **`chart/`** — Uso del componente `<chart>` per grafici
5. **`eventBus/`**, **`parentChildComp/`** — Pattern comunicazione componenti

## Workflow di sviluppo

### Step 1: Setup progetto base

```bash
# In AIoT-IDE: File → New → Vela Quick App Project
# Oppure crea struttura manualmente:
mkdir my-band-app
cd my-band-app
touch manifest.json app.ux
mkdir -p pages/index common
```

**Struttura cartelle standard:**

```
my-band-app/
├── src/                          # Codice sorgente
│   ├── app.ux                   # Entry point globale (onCreate/onDestroy)
│   ├── manifest.json            # Configurazione app
│   ├── config-watch.json        # Configurazione watch mode
│   ├── pages/                   # Pagine dell'app
│   │   └── index/
│   │       └── index.ux         # Pagina principale
│   ├── common/                  # Risorse condivise
│   │   ├── logo.png             # Icona app (108×108px)
│   │   ├── logo.svg             # Versione vettoriale (opzionale)
│   │   ├── images/              # Altre immagini
│   │   └── utils.js             # Utility functions
│   └── i18n/                    # Internazionalizzazione (opzionale)
│       ├── en.json
│       ├── zh-CN.json
│       └── defaults.json
├── .vscode/                      # Configurazione VS Code
│   └── mcp.json                 # MCP server config (opzionale)
├── dist/                         # Build output (generato, .gitignore)
├── .gitignore
├── .npmignore                    # Esclusioni per npm publish
├── .eslintignore                # File da ignorare per ESLint
├── package.json                 # Dipendenze e scripts
├── package-lock.json            # Lock delle dipendenze
├── jsconfig.json                # Configurazione JavaScript/TypeScript
├── .prettierrc.js               # Formattazione codice
├── .stylelintrc.js              # Linting CSS/UX styles
├── .eslintrc.js                 # Linting JavaScript (se presente)
├── commitlint.config.js         # Conventional commits
├── husky.sh                     # Git hooks setup
├── build.sh                     # Script build helper
├── DEVELOPMENT.md               # Note tecniche e decisioni
└── README.md                    # Documentazione utente
```

**File essenziali minimi (per AIoT-IDE):**
- `src/manifest.json` — configurazione package, features, router
- `src/app.ux` — lifecycle globale
- `src/pages/index/index.ux` — pagina principale
- `src/common/logo.png` — icona 108×108px

**Toolchain professionale (opzionale ma raccomandato):**
- `package.json` + `package-lock.json` — gestione dipendenze
- `.prettierrc.js`, `.stylelintrc.js`, `.eslintignore` — code quality
- `commitlint.config.js` + `husky.sh` — git hooks per commit convenzionali
- `jsconfig.json` — IntelliSense e completamento automatico
- `.vscode/mcp.json` — integrazione MCP server per AI tools

**Note sulla struttura:**
- AIoT-IDE può accettare anche `app.ux` e `manifest.json` direttamente nella root (senza `src/`)
- La cartella `src/` è **fortemente raccomandata** per progetti professionali
- `dist/` è generata automaticamente dal build (escludere da git)
- I file di configurazione nella root (`.*rc.js`, `commitlint.config.js`) sono opzionali ma migliorano qualità e manutenibilità

**Struttura minima:**
```
my-band-app/
├── manifest.json       # Configurazione app
├── app.ux             # Entry point globale
├── pages/
│   └── index/
│       └── index.ux   # Pagina principale
└── common/
    └── logo.png       # Icona app (108×108px)
```

### Step 2: Configura manifest.json

```json
{
  "package": "com.example.mybandapp",
  "name": "MyBandApp",
  "versionName": "1.0.0",
  "versionCode": 1,
  "minPlatformVersion": 1200,
  "icon": "/common/logo.png",
  "deviceTypeList": ["watch"],
  "features": [
    { "name": "system.sensor" },
    { "name": "system.interconnect" },
    { "name": "system.file" },
    { "name": "system.storage" },
    { "name": "system.vibrator" },
    { "name": "system.battery" }
  ],
  "config": {
    "logLevel": "log",
    "designWidth": 480
  },
  "router": {
    "entry": "pages/index",
    "pages": {
      "pages/index": { "component": "index" }
    }
  },
  "minAPILevel": 1
}
```

**⚠️ Punti critici:**
- `deviceTypeList: ["watch"]` — NON usare `"band"` per Mi Band 10
- `minPlatformVersion: 1200` — Versione minima compatibile
- `designWidth: 480` — Risoluzione logica (schermo reale 212×520px)
- `package` — Deve coincidere con package Android se usi `system.interconnect`

### Step 3: Crea pagina base (pages/index/index.ux)

```vue
<template>
  <div class="container">
    <text class="title">{{ message }}</text>
    <input type="button" class="btn" value="Vibra" @click="vibrate" />
    <text class="info">Batteria: {{ batteryLevel }}%</text>
  </div>
</template>

<script>
import vibrator from '@system.vibrator'
import battery from '@system.battery'

export default {
  data: {
    message: 'Hello Mi Band 10!',
    batteryLevel: 0
  },
  onInit() {
    this.getBatteryLevel()
  },
  vibrate() {
    vibrator.vibrate({
      mode: 'short'
    })
  },
  getBatteryLevel() {
    battery.getStatus({
      success: (data) => {
        this.batteryLevel = data.level
      }
    })
  }
}
</script>

<style>
.container {
  flex-direction: column;
  justify-content: center;
  align-items: center;
  width: 100%;
  height: 100%;
}
.title {
  font-size: 40px;
  color: #ffffff;
  margin-bottom: 40px;
}
.btn {
  width: 300px;
  height: 80px;
  font-size: 32px;
  background-color: #007aff;
  color: #ffffff;
}
.info {
  font-size: 28px;
  color: #cccccc;
  margin-top: 40px;
}
</style>
```

### Step 4: Build e deploy

**In AIoT-IDE:**
1. Build → Build RPK
2. Run → Run on Emulator / Real Device
3. Debug → View Console Logs

**Da terminale (se configurato):**
```bash
# Build
npm run build

# Output: dist/com.example.mybandapp.rpk
# Installa via Mi Fitness app o AIoT-IDE
```

## Comunicazione Band ↔ Android

### Pattern consolidato (da interconnect_image_demo)

**Sul Band (Quick App):**

```javascript
import interconnect from '@system.interconnect'

export default {
  sendToPhone(data) {
    const message = {
      type: 'request_data',
      payload: data,
      timestamp: Date.now()
    }
    
    interconnect.send({
      message: JSON.stringify(message),
      success: () => {
        console.log('Messaggio inviato')
      },
      fail: (err) => {
        console.error('Errore invio:', err)
      }
    })
  },
  
  onInit() {
    // Listener per messaggi da Android
    interconnect.on({
      message: (data) => {
        const msg = JSON.parse(data.message)
        
        if (msg.type === 'response') {
          console.log('Ricevuto da Android:', msg.payload)
        }
      }
    })
  }
}
```

**Su Android (Kotlin + XMS SDK):**

```kotlin
import com.xiaomi.xms.wearable.Wearable
import com.xiaomi.xms.wearable.message.MessageApi
import com.xiaomi.xms.wearable.message.MessageClient
import com.xiaomi.xms.wearable.message.MessageEvent

class BandCommunicationService {
    private lateinit var messageClient: MessageClient
    
    fun init(context: Context) {
        messageClient = Wearable.getMessageApi(context)
        messageClient.addListener(messageListener)
    }
    
    private val messageListener = MessageApi.MessageListener { messageEvent ->
        val receivedData = String(messageEvent.data)
        val json = JSONObject(receivedData)
        
        when (json.getString("type")) {
            "request_data" -> {
                val response = JSONObject().apply {
                    put("type", "response")
                    put("payload", "Dati elaborati")
                }
                sendToBand(response.toString().toByteArray())
            }
        }
    }
    
    private fun sendToBand(data: ByteArray) {
        messageClient.sendMessage("com.example.mybandapp", data, null)
    }
}
```

**⚠️ Vincoli firma:**
- Package Quick App = Package Android app
- Stessa firma (certificate.pem) per entrambe
- Tool online: https://cdn.hybrid.xiaomi.com/aiot-ide/signature-generate-tool/v2/index.html

## Uso sensori

### Accelerometro

```javascript
import sensor from '@system.sensor'

export default {
  onInit() {
    sensor.subscribeAccelerometer({
      interval: 'normal', // 'game' per frequenza alta
      success: (data) => {
        console.log(`x: ${data.x}, y: ${data.y}, z: ${data.z}`)
      }
    })
  },
  onDestroy() {
    sensor.unsubscribeAccelerometer()
  }
}
```

### Barometro

```javascript
import sensor from '@system.sensor'

export default {
  data: {
    pressure: 0
  },
  onInit() {
    sensor.subscribeBarometer({
      success: (data) => {
        this.pressure = data.pressure // hPa
      }
    })
  },
  onDestroy() {
    sensor.unsubscribeBarometer()
  }
}
```

**⚠️ Sensori NON supportati su Mi Band 10:**
- Bussola / magnetometro
- Giroscopio
- GPS

## Componenti UI principali

### Layout container

```vue
<template>
  <!-- Stack: sovrapposizione elementi -->
  <stack class="container">
    <image src="/common/bg.png" class="background"></image>
    <text class="overlay-text">Testo sopra immagine</text>
  </stack>
  
  <!-- Swiper: carousel pagine -->
  <swiper class="swiper" indicator="true">
    <div class="page1">Pagina 1</div>
    <div class="page2">Pagina 2</div>
  </swiper>
  
  <!-- List: lista scroll verticale -->
  <list class="list">
    <list-item for="{{items}}" class="item">
      <text>{{$item.name}}</text>
    </list-item>
  </list>
</template>
```

### Grafico

```vue
<template>
  <chart class="chart" type="line" options="{{chartOptions}}"></chart>
</template>

<script>
export default {
  data: {
    chartOptions: {
      xAxis: {
        min: 0,
        max: 10
      },
      yAxis: {
        min: 0,
        max: 100
      },
      series: [{
        data: [10, 20, 30, 50, 80]
      }]
    }
  }
}
</script>
```

## Storage e file

### Storage key-value

```javascript
import storage from '@system.storage'

export default {
  saveData() {
    storage.set({
      key: 'userSettings',
      value: JSON.stringify({ theme: 'dark', lang: 'it' }),
      success: () => console.log('Salvato')
    })
  },
  
  loadData() {
    storage.get({
      key: 'userSettings',
      success: (data) => {
        const settings = JSON.parse(data)
        console.log('Tema:', settings.theme)
      }
    })
  }
}
```

### File system

```javascript
import file from '@system.file'

export default {
  writeFile() {
    file.writeText({
      uri: 'internal://cache/data.txt',
      text: 'Contenuto del file',
      success: () => console.log('File scritto')
    })
  },
  
  readFile() {
    file.readText({
      uri: 'internal://cache/data.txt',
      success: (data) => {
        console.log('Contenuto:', data.text)
      }
    })
  }
}
```

## Publishing Vela JS skills to GitHub

When documenting project structure:
1. **Check the real project first** — don't rely on AIoT-IDE defaults alone
2. **Include all toolchain config files** — `.prettierrc.js`, `.stylelintrc.js`, `.eslintignore`, `commitlint.config.js`, `husky.sh`, `jsconfig.json`
3. **Document optional vs essential** — separate "works in AIoT-IDE" from "professional setup"
4. **MIT License** — use `@username` in copyright if user prefers privacy over real name
5. **Multi-agent installation** — document setup for Hermes, Claude Code, Pi Agent, OpenCode, and generic AI agents

Standard README sections for published skills:
- Features & when to use
- Installation (Hermes + other agents)
- Usage examples
- Structure overview
- Resources & links
- Contributing guidelines
- License (MIT recommended)

## Pitfalls comuni

### 1. Display troppo piccolo (212×520px)

Il framework scala automaticamente. Progetta sempre per 480px di larghezza logica:
- Font size: 28–40px (leggibile)
- Button height: 60–80px (tocco facile)
- Margin/padding: 20–40px

### 2. RAM/CPU molto limitata

- Max 3–4 componenti complessi per pagina
- Niente animazioni full-screen
- Minimizza immagini (preferisci SVG/icon font)
- Usa `list` invece di tanti `div` ripetuti

### 3. interconnect richiede firma condivisa

Se `interconnect.send()` fallisce:
1. Verifica package name uguale (manifest.json ↔ AndroidManifest.xml)
2. Verifica certificato firmato con stessa chiave
3. Usa tool online Xiaomi per generare firma valida

### 4. Sensori non sempre disponibili

```javascript
sensor.subscribeAccelerometer({
  success: (data) => { /* OK */ },
  fail: (err) => {
    console.error('Sensore non disponibile:', err)
    // Fallback UI senza sensore
  }
})
```

### 5. BLE solo verso telefono accoppiato

Nessun WiFi proprio — usa `system.interconnect` per dati esterni. L'app Android fa da bridge.

### 6. Emulatore vs dispositivo reale

Testa sempre su Mi Band 10 reale per:
- Performance reali
- Touch/gesture
- Sensori hardware
- Consumo batteria

## Risorse interne

- `references/timer-scheduler-pattern.md` — Pattern timer vs scheduler per task periodici
- `references/data-tracking-app-pattern.md` — Pattern multi-view per app di data tracking (habit/mood/spesa tracker). Include: navigation state, storage con array, streak calc, cleanup, statistics, bottom nav UI, form input
- `references/publishing-hermes-skills.md` — Workflow completo pubblicazione skill Hermes su GitHub

## Risorse ufficiali

### Documentazione
- Quick Start: https://iot.mi.com/vela/quickapp/en/guide/start/use-ide.html
- Componenti UI: https://iot.mi.com/vela/quickapp/en/components/
- JS API: https://iot.mi.com/vela/quickapp/en/features/
- Interconnect: https://iot.mi.com/vela/quickapp/en/features/network/interconnect.html

### Repository esempi
- GitHub: https://github.com/open-vela/packages_apps
- Cartella wearable: `/wearable/vela-sample/`, `/wearable/interconnect_image_demo/`

### Tool
- AIoT-IDE download: https://iot.mi.com/vela/quickapp/en/guide/start/use-ide.html
- Firma online: https://cdn.hybrid.xiaomi.com/aiot-ide/signature-generate-tool/v2/index.html

## Checklist prima di iniziare

- [ ] AIoT-IDE installato e funzionante con emulatore Vela
- [ ] Clonato `open-vela/packages_apps` e studiato `wearable/vela-sample/`
- [ ] Letto codice di `interconnect_image_demo/` (watch + Android)
- [ ] Compreso pattern manifest.json + app.ux + pagine .ux
- [ ] Definito package name (coincidente con app Android companion)
- [ ] Generata coppia firma (private.pem / certificate.pem)
- [ ] Verificato quali sensori servono (accel/baro disponibili, bussola NO)

## Schema architetturale

```
┌─────────────────────────────────┐
│  Mi Band 10                     │
│  Quick App (.rpk)               │
│  - Vela JS / .ux files          │
│  - system.interconnect (send)   │
│  - system.sensor (accel/baro)   │
│  - system.vibrator              │
└────────────┬────────────────────┘
             │ BLE automatico
             │ (gestito da Mi Fitness)
┌────────────▼────────────────────┐
│  Android companion app          │
│  - xms-wearable-lib_1.4.aar     │
│  - MessageApi (byte[])          │
│  - NodeApi (device state)       │
│  - NotifyApi (notifiche)        │
└────────────┬────────────────────┘
             │ (se Flutter/altro)
             │ MethodChannel / REST API
┌────────────▼────────────────────┐
│  App mobile / Backend           │
│  - UI framework (Flutter/React) │
│  - Business logic               │
└─────────────────────────────────┘
```

## Vincoli hardware Mi Band 10

| Caratteristica | Valore |
|----------------|--------|
| Schermo | 1.72" AMOLED, 212×520px |
| Sensori | Accelerometro (x/y/z), Barometro (hPa) |
| Connettività | BLE (solo verso telefono accoppiato) |
| Storage | Limitato (~100 MB app + dati) |
| CPU/RAM | ARM low-power, ~64 MB RAM |
| Batteria | ~300 mAh (durata 14 giorni uso normale) |

## Next steps dopo skill loaded

1. **Clona repo esempi**: `git clone https://github.com/open-vela/packages_apps.git`
2. **Studia vela-sample**: Apri `packages_apps/wearable/vela-sample/` in AIoT-IDE
3. **Testa interconnect_image_demo**: Build + deploy su emulatore
4. **Crea progetto base**: Usa template minimal sopra
5. **Implementa feature**: Inizia con UI semplice, aggiungi sensori/interconnect gradualmente

## Pattern & Templates

Questa skill include reference file con pattern validati:

- **`references/timer-scheduler-pattern.md`** — Pattern completo per app timer-based (vibrazione oraria, reminder, scheduler). Include: gestione timer con anti-drift, storage multi-stato, protezione anti-duplicati, UI ottimizzata 212×520px, testing checklist. Validato in produzione.
- **`references/data-tracking-app-pattern.md`** — Pattern multi-view per app di data tracking (habit, mood, spesa, studio timer). Include: navigation state con view switcher, storage con array di date, streak calculation, cleanup automatico, statistics aggregation, bottom nav UI, form input con icon/color picker. Validato con Habit Tracker.

---

*Skill creata per sviluppo Quick App Xiaomi Mi Band 10 con Vela JS framework. Basata su documentazione ufficiale Xiaomi IoT e repository open-vela.*
