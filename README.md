# Xiaomi Mi Band 10 Vela JS Development Skill

Skill completa per sviluppare Quick Apps Vela JS su **Xiaomi Mi Band 10** usando Hermes Agent.

## 📋 Contenuto

- **Workflow completo**: setup AIoT-IDE, struttura progetto, build, deploy
- **Comunicazione Band↔Android**: `system.interconnect` + XMS Wearable SDK
- **Sensori**: accelerometro, barometro, GPS, heart rate
- **UI Components**: layout, swiper, list, chart, canvas
- **Storage & File System**: persistent storage, file operations
- **Esempi reali**: pattern da `open-vela/packages_apps`
- **Pitfalls & Troubleshooting**: problemi comuni e soluzioni

## 🎯 Quando usare questa skill

- Sviluppo di Quick Apps per Xiaomi Mi Band 10
- Integrazione sensori wearable
- Comunicazione tra band e smartphone Android
- UI design per display 212×520px AMOLED
- Ottimizzazione RAM/CPU su dispositivi embedded

## 📦 Installazione

### Per utenti Hermes Agent

Clona questa skill nella tua directory skills di Hermes:

```bash
# Naviga nella directory skills
cd ~/.hermes/skills/software-development/

# Clona il repository
git clone https://github.com/paraflu/hermes-skill-xiaomi-miband10-vela-js.git xiaomi-miband10-vela-js

# Verifica che sia stata installata
ls -la xiaomi-miband10-vela-js/
```

### Per profili multipli

Se usi più profili Hermes (es. `default`, `work`):

```bash
# Per il profilo "work"
cd ~/.hermes/profiles/work/skills/software-development/
git clone https://github.com/paraflu/hermes-skill-xiaomi-miband10-vela-js.git xiaomi-miband10-vela-js
```

### Aggiornamento

Per aggiornare la skill all'ultima versione:

```bash
cd ~/.hermes/skills/software-development/xiaomi-miband10-vela-js
git pull origin main
```

## 🚀 Utilizzo

Una volta installata, Hermes caricherà automaticamente la skill quando:
- Chiedi di sviluppare app per Mi Band 10
- Menzioni "Vela JS" o "Quick App"
- Lavori su progetti Xiaomi wearable

Puoi anche caricarla esplicitamente:

```
"Carica la skill xiaomi-miband10-vela-js e aiutami a creare un'app per monitorare i passi"
```

## 📁 Struttura

```
xiaomi-miband10-vela-js/
├── SKILL.md                          # Documentazione principale
├── references/
│   └── timer-scheduler-pattern.md    # Pattern timer/scheduler
├── README.md                          # Questo file
└── .gitignore
```

## 🔗 Risorse collegate

- **Skill originale**: Installata in `~/.hermes/skills/software-development/xiaomi-miband10-vela-js/`
- **Documentazione Vela JS**: https://iot.mi.com/vela/quickapp/en/guide/
- **OpenVela repository**: https://github.com/open-vela/packages_apps
- **Esempio app completa**: https://github.com/paraflu/miband10-hourly-vibrator

## 🤝 Contribuire

Hai scoperto nuovi pattern, API o pitfalls? I contributi sono benvenuti:

1. Fork del repository
2. Crea un branch per la tua feature (`git checkout -b feature/nuovo-pattern`)
3. Commit delle modifiche (`git commit -m 'Add: nuovo pattern per...'`)
4. Push del branch (`git push origin feature/nuovo-pattern`)
5. Apri una Pull Request

## 📄 Licenza

Questa skill è parte dell'ecosistema Hermes Agent ed è rilasciata sotto gli stessi termini.

## 👤 Autore

Creata da **Andrea Forlin** (@paraflu) - sviluppata con Hermes Agent

---

**Target**: Xiaomi Mi Band 10 (HyperOS 2, OpenVela RTOS)  
**Runtime**: Vela JS (ES6+, .ux template syntax)  
**Display**: 212×520px AMOLED (480px logici)
