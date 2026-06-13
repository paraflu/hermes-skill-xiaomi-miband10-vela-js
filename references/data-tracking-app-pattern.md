# Data-Tracking App Pattern (Multi-View)

Pattern for Vela JS apps that collect and display structured data (habit tracker, mood tracker, daily spending, study timer). Validated with the Habit Tracker app (com.forlin.habittracker).

## When to use this pattern

- Apps with 2+ navigable views (today, add, stats)
- Persistent data with complex structures (arrays of dates, nested objects)
- Statistics computed from stored data
- UI with interactive list + input form + dashboard

## Architecture

```
src/pages/index/index.ux
├── <template>
│   ├── Header (title + date)
│   ├── Content (view switcher)
│   │   ├── View: today      — interactive list with checkbox
│   │   ├── View: add        — input form + picker
│   │   └── View: stats      — dashboard with cards + ranking
│   └── Bottom Nav (3 items: list, +, stats)
└── <script>
    ├── data: { view, items[], form fields, computed stats }
    ├── Storage: load/save with JSON.stringify/parse
    ├── Business logic: CRUD, streak calc, cleanup
    └── Navigation: switchView()
```

## Navigation Pattern

Use `data.view` to switch between views (not multi-page routing):

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

**Why not routing?** On a 212×520px screen, instant navigation without transitions is smoother. Multi-page routing is better for apps with independent pages.

## Storage Pattern (Array of dates)

For apps that track daily completions:

```javascript
// Data structure
habits = [
  {
    id: '1718000000000',     // Date.now() as string
    name: 'Exercise',
    icon: '💪',
    color: '#22c55e',
    completions: ['2026-06-10', '2026-06-09', '2026-06-08'],
    streak: 3,
    createdAt: '2026-06-01'
  }
]

// Save
storage.set({ key: 'habits_data', value: JSON.stringify(this.habits) })

// Load
storage.get({
  key: 'habits_data',
  success: (data) => { if (data) this.habits = JSON.parse(data) }
})
```

**Pitfall:** `storage.set` only accepts strings. Always use `JSON.stringify/parse`.

## Streak Calculation

Algorithm to calculate consecutive days:

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

**Note:** Use ISO strings (`YYYY-MM-DD`) for comparisons, not Date objects.

## Cleanup Old Data

Avoid uncontrolled storage growth:

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

Pattern for computing statistics from stored data:

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

Pattern for nav bar on small screen:

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

For forms on small screen (212×520px):

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

- **Habit Tracker**: `https://github.com/paraflu/habit-tracker-miband` — Multi-view data tracking with streaks, statistics, icon/color customization
- **Hourly Vibrator**: `https://github.com/paraflu/miband10-hourly-vibrator` — Timer-based app with anti-drift pattern

## Advanced: Auto-save on onHide

Automatically saves when the app goes to background:

```javascript
onShow() {
  this.loadData()
  this.updateStats()
},

onHide() {
  // Auto-save when the app goes to background
  this.saveData()
  console.log('Data saved on hide')
},

onDestroy() {
  this.saveData()
  global.runGC()
}
```

## Advanced: Data Export

Exports data as JSON file for backup or transfer:

```javascript
import file from '@system.file'

exportData() {
  const exportData = {
    version: '1.0',
    exportedAt: new Date().toISOString(),
    habits: this.habits
  }

  file.writeText({
    uri: 'internal://cache/habits_export.json',
    text: JSON.stringify(exportData, null, 2),
    success: () => {
      prompt.showToast({ message: 'Data exported' })
    },
    fail: (err) => {
      console.error('Export failed:', err)
    }
  })
}
```

## Advanced: global.runGC()

```javascript
refreshAll() {
  this.loadData()
  this.updateStats()
  this.cleanOldCompletions()
  global.runGC() // Free memory after refresh
}
```

## Pitfalls

1. **Date comparison**: Use ISO strings, not Date objects (timezone can break comparisons)
2. **Storage limit**: Keep max 90 days of data. Clean automatically on load
3. **Array grow**: `completions.push()` can grow indefinitely — always pair with cleanup
4. **Streak continuity**: If today is not completed and neither is yesterday, streak = 0 (don't calculate backwards)
5. **ID generation**: Use `Date.now().toString()` as ID — simple and unique without UUID library
6. **Navigation state**: Don't use multi-page routing for view switching — data binding is smoother
7. **Always save in `onHide()`**: The app can be killed in background without going through `onDestroy()`
8. **Use `global.runGC()`** after heavy data refresh to keep memory free
