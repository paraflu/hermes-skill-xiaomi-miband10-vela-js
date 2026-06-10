# Xiaomi Mi Band 10 Vela JS Development Skill

Complete skill for developing Vela JS Quick Apps on **Xiaomi Mi Band 10** using AI agents.

## 📋 Contents

- **Complete workflow**: AIoT-IDE setup, project structure, build, deploy
- **Band↔Android communication**: `system.interconnect` + XMS Wearable SDK
- **Sensors**: accelerometer, barometer, GPS, heart rate
- **UI Components**: layout, swiper, list, chart, canvas
- **Storage & File System**: persistent storage, file operations
- **Real examples**: patterns from `open-vela/packages_apps`
- **Pitfalls & Troubleshooting**: common issues and solutions

## 🎯 When to use this skill

- Developing Quick Apps for Xiaomi Mi Band 10
- Integrating wearable sensors
- Band-to-Android smartphone communication
- UI design for 212×520px AMOLED display
- RAM/CPU optimization on embedded devices

## 📦 Installation

### For Hermes Agent

Clone this skill into your Hermes skills directory:

```bash
# Navigate to skills directory
cd ~/.hermes/skills/software-development/

# Clone the repository
git clone https://github.com/paraflu/hermes-skill-xiaomi-miband10-vela-js.git xiaomi-miband10-vela-js

# Verify installation
ls -la xiaomi-miband10-vela-js/
```

#### Multiple profiles

If you use multiple Hermes profiles (e.g., `default`, `work`):

```bash
# For "work" profile
cd ~/.hermes/profiles/work/skills/software-development/
git clone https://github.com/paraflu/hermes-skill-xiaomi-miband10-vela-js.git xiaomi-miband10-vela-js
```

#### Updates

To update the skill to the latest version:

```bash
cd ~/.hermes/skills/software-development/xiaomi-miband10-vela-js
git pull origin main
```

### For Claude Code CLI

Claude Code doesn't have native skill management, but you can reference this documentation:

```bash
# Clone to a shared documentation directory
mkdir -p ~/ai-skills/
cd ~/ai-skills/
git clone https://github.com/paraflu/hermes-skill-xiaomi-miband10-vela-js.git xiaomi-miband10-vela-js

# Then reference it in your prompt:
codex "Read ~/ai-skills/xiaomi-miband10-vela-js/SKILL.md and help me create a Mi Band 10 app"
```

**Tip**: Add the SKILL.md content to your project's `.clinerules` or `CLAUDE.md` file to make Claude Code automatically aware of Vela JS patterns when working in that directory.

### For Pi Agent (pi.dev)

Pi coding agent can load external documentation via context files:

```bash
# Clone to your preferred location
cd ~/documents/dev-guides/
git clone https://github.com/paraflu/hermes-skill-xiaomi-miband10-vela-js.git xiaomi-miband10-vela-js

# Reference in your Pi session:
pi "Using the guide at ~/documents/dev-guides/xiaomi-miband10-vela-js/SKILL.md, help me build a Quick App for Mi Band 10"
```

**Alternative**: Copy key sections from `SKILL.md` into your project's `AGENTS.md` or similar context file that Pi reads automatically.

### For OpenCode CLI

OpenCode can reference local documentation files:

```bash
# Clone to a shared location
mkdir -p ~/.opencode/guides/
cd ~/.opencode/guides/
git clone https://github.com/paraflu/hermes-skill-xiaomi-miband10-vela-js.git xiaomi-miband10-vela-js

# Use with OpenCode:
opencode --context ~/.opencode/guides/xiaomi-miband10-vela-js/SKILL.md "Create a Vela JS app for Mi Band 10"
```

### For Other AI Agents

For any AI agent that supports:
- **Context files**: Clone the repository and reference `SKILL.md` in your prompts
- **Project conventions**: Copy relevant sections to `.cursorrules`, `AGENTS.md`, `CLAUDE.md`, or similar files
- **RAG/embeddings**: Index the `SKILL.md` and `references/` directory in your knowledge base

**Universal approach**:
```bash
# Clone anywhere convenient
git clone https://github.com/paraflu/hermes-skill-xiaomi-miband10-vela-js.git

# Then copy/paste relevant sections or reference the file path when prompting your AI agent
```

## 🚀 Usage

### With Hermes Agent

Once installed, Hermes will automatically load the skill when you:
- Ask to develop apps for Mi Band 10
- Mention "Vela JS" or "Quick App"
- Work on Xiaomi wearable projects

You can also load it explicitly:

```
"Load the xiaomi-miband10-vela-js skill and help me create a step counter app"
```

### With Other Agents

Reference the documentation file in your prompt:

```
"Following the guide in ~/path/to/xiaomi-miband10-vela-js/SKILL.md, help me create a Vela JS app that vibrates hourly"
```

## 📁 Structure

```
xiaomi-miband10-vela-js/
├── SKILL.md                          # Main documentation
├── references/
│   └── timer-scheduler-pattern.md    # Timer/scheduler patterns
├── README.md                          # This file
└── .gitignore
```

## 🔗 Related Resources

- **Original skill location**: `~/.hermes/skills/software-development/xiaomi-miband10-vela-js/`
- **Vela JS Documentation**: https://iot.mi.com/vela/quickapp/en/guide/
- **OpenVela Repository**: https://github.com/open-vela/packages_apps
- **Complete example app**: https://github.com/paraflu/miband10-hourly-vibrator

## 🤝 Contributing

Discovered new patterns, APIs, or pitfalls? Contributions are welcome:

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/new-pattern`)
3. Commit your changes (`git commit -m 'Add: new pattern for...'`)
4. Push the branch (`git push origin feature/new-pattern`)
5. Open a Pull Request

## 📄 License

This skill is part of the Hermes Agent ecosystem and is released under the same terms.

## 👤 Author

Created by **Andrea Forlin** (@paraflu) - developed with Hermes Agent

---

**Target**: Xiaomi Mi Band 10 (HyperOS 2, OpenVela RTOS)  
**Runtime**: Vela JS (ES6+, .ux template syntax)  
**Display**: 212×520px AMOLED (480px logical)
