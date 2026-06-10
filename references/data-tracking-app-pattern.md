# Data-Tracking App Pattern (Multi-View)

Pattern per app Vela JS che raccolgono e visualizzano dati strutturati (habit tracker, mood tracker, spesa giornaliera, studio timer). Validato con l'app Habit Tracker (com.forlin.habittracker).

## Quando usare questo pattern

- App con 2+ viste navigationabili (today, add, stats)
- Dati persistenti con strutture complesse (array di date, oggetti annidati)
- Statistiche calcolate da dati stored
- UI con lista interattiva + form di input + dashboard

## Architettura

```
src/pages/index/index.ux
├── <template>
│   ├── Header (titolo + data)
│   ├── Content (view switcher)
│   │   ├── View: today      — lista interattiva con checkbox
│   │   ├── View: add        — form input + picker
│   │   └── View: stats      — dashboard con card + classifica
│   └── Bottom Nav (3 item: list, +, stats)
└── <script>
    ├── data: { view, items[], form fields, computed stats }
    ├── Storage: load/save con JSON.stringify/parse
    ├── Business logic: CRUD, streak calc, cleanup
    └── Navigation: switchView()
```

## Navigation Pattern

Usare `data.view` per switchare tra viste (non routing multi-pagina):

```javascript
data: {
  view: 'today',  // 'today' | 'add' | 'stats'
}

switchView(v) {
  this.view = v
  if (v === 'today') this.pageTitle = `Habits · ${this.completedToday}/${this.totalHabits}`
  else if (v === 'stats') this.pageTitle = 'Statistics'
}
```

**Perché non routing?** Su schermo 212×520px, navigazione instantanea senza transizioni è più fluida. Il routing multi-pagina è meglio per app con pagine indipendenti.

## Storage Pattern (Array di date)

Per app che tracciano completamenti giornalieri:

```javascript
// Struttura dati
habits = [
  {
    id: '1718000000000',     // Date.now() come stringa
    name: 'Exercise',
    icon: '💪',
    color: '#22c55e',
    completions: ['2026-06-10', '2026-06-09', '2026-06-08'],
    streak: 3,
    createdAt: '2026-06-01'
  }
]

// Salva
storage.set({ key: 'habits_data', value: JSON.stringify(this.habits) })

// Carica
storage.get({
  key: 'habits_data',
  success: (data) => { if (data) this.habits = JSON.parse(data) }
})
```

**Pitfall:** `storage.set` accetta solo stringhe. Usare sempre `JSON.stringify/parse`.

## Streak Calculation

Algoritmo per calcolare giorni consecutivi:

```javascript
calcStreak(completions, todayStr, yestStr) {
  if (!completions || completions.length === 0) return 0
  if (!completions.includes(todayStr) && !completions.includes(yestStr)) return 0

  let streak = 0
  let checkDate = new Date()
  if (!completions.includes(todayStr)) checkDate.setDate(checkDate.getDate() - 1)

  while (true) {
    const dateStr = `${checkDate.getFullYear()}-${String(checkDate.getMonth()+1).padStart(2,'0')}-${String(checkDate.getDate()).padStart(2,'0')}`
    if (completions.includes(dateStr)) {
      streak++
      checkDate.setDate(checkDate.getDate() - 1)
    } else break
  }
  return streak
}
```

**Nota:** Usare stringhe ISO (`YYYY-MM-DD`) per confronti, non oggetti Date.

## Cleanup Old Data

Evitare crescita incontrollata dello storage:

```javascript
cleanOldCompletions() {
  const cutoff = new Date()
  cutoff.setDate(cutoff.getDate() - 90)
  const cutoffStr = `${cutoff.getFullYear()}-${String(cutoff.getMonth()+1).padStart(2,'0')}-${String(cutoff.getDate()).padStart(2,'0')}`

  this.habits.forEach(habit => {
    if (habit.completions) {
      habit.completions = habit.completions.filter(d => d >= cutoffStr)
    }
  })
}
```

## Statistics Aggregation

Pattern per calcolare statistiche da dati stored:

```javascript
updateStats() {
  this.completedToday = this.habits.filter(h => this.isCompletedToday(h.id)).length
  this.totalHabits = this.habits.length

  this.bestStreak = 0
  this.habits.forEach(h => { if (h.streak > this.bestStreak) this.bestStreak = h.streak })

  this.totalCompletions = 0
  this.habits.forEach(h => { if (h.completions) this.totalCompletions += h.completions.length })

  const daysSinceOldest = this.calcDaysSinceOldest()
  const possibleCompletions = this.habits.length * Math.max(daysSinceOldest, 1)
  this.overallRate = possibleCompletions > 0
    ? Math.round((this.totalCompletions / possibleCompletions) * 100) : 0
}
```

## Bottom Navigation UI

Pattern per nav bar su schermo piccolo:

```html
<div class="bottom-nav">
  <div class="nav-item {{ view === 'today' ? 'active' : '' }}" onclick="switchView('today')">
    <text class="nav-icon">📋</text>
    <text class="nav-label">Today</text>
  </div>
  <div class="nav-item add-btn" onclick="startAdd()">
    <text class="nav-icon add-icon">+</text>
  </div>
  <div class="nav-item {{ view === 'stats' ? 'active' : '' }}" onclick="switchView('stats')">
    <text class="nav-icon">📊</text>
    <text class="nav-label">Stats</text>
  </div>
</div>
```

```css
.bottom-nav {
  flex-direction: row;
  justify-content: space-around;
  align-items: center;
  padding: 10px 20px 20px;
  background-color: #0f0f1a;
}
.add-btn {
  background-color: #6366f1;
  border-radius: 24px;
  width: 48px;
  height: 48px;
  justify-content: center;
  margin-top: -16px;
}
```

## Form Input Pattern

Per form su schermo piccolo (212×520px):

```html
<text class="add-label">Name</text>
<input class="habit-input" type="text" placeholder="e.g. Exercise 30min"
  onchange="setName($event.value)" value="{{ name }}"/>

<div class="icon-picker">
  <div for="{{ icons }}" class="icon-option {{ selectedIcon === $item ? 'selected' : '' }}"
    onclick="selectIcon($item)">
    <text class="icon-text">{{ $item }}</text>
  </div>
</div>

<div class="color-picker">
  <div for="{{ colors }}" class="color-option {{ selectedColor === $item ? 'selected' : '' }}"
    onclick="selectColor($item)" style="background-color: {{ $item }};">
  </div>
</div>
```

## App References

- **Habit Tracker**: `https://github.com/paraflu/habit-tracker-miband` — Multi-view data tracking con streaks, statistics, icon/color customization
- **Hourly Vibrator**: `https://github.com/paraflu/miband10-hourly-vibrator` — Timer-based app con anti-drift pattern

## Pitfalls

1. **Date comparison**: Usare stringhe ISO, non oggetti Date (fuso orario può rompere i confronti)
2. **Storage limit**: Mantenere max 90 giorni di dati. Pulire automaticamente al load
3. **Array grow**: `completions.push()` può crescere indefinitamente — sempre pair con cleanup
4. **Streak continuity**: Se oggi non è completato e neanche ieri, streak = 0 (non calcolare all'indietro)
5. **ID generazione**: Usare `Date.now().toString()` come ID — semplice e univoco senza UUID library
6. **Navigation state**: Non usare routing multi-pagina per view switch — il data binding è più fluido
