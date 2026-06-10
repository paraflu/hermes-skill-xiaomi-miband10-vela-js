# Publishing Hermes Skills on GitHub

Reference workflow for publishing Hermes Agent skills as standalone GitHub repositories.

## Complete Publishing Workflow

### 1. Initialize Git Repository

```bash
cd ~/.hermes/skills/<category>/<skill-name>
git init
git branch -m main
git config user.name "Hermes Agent"
git config user.email "hermes@agent.ai"
```

### 2. Create .gitignore

Minimal .gitignore for skill repositories:

```
*.swp
*.swo
*~
.DS_Store
Thumbs.db
```

### 3. Commit Initial Content

```bash
git add .
git commit -m "Initial commit: <Skill Description>

<Multi-line description with features, target use cases, etc.>"
```

### 4. Create GitHub Repository

```bash
gh repo create <repo-name> \
  --public \
  --source=. \
  --description="<One-line description>" \
  --push
```

**Naming convention**: `hermes-skill-<topic>` for clarity (e.g., `hermes-skill-xiaomi-miband10-vela-js`)

### 5. Create README.md (Multi-Agent Support)

Essential sections:
- **Content**: What the skill covers
- **When to use**: Clear trigger scenarios
- **Installation for Hermes Agent**: Clone to `~/.hermes/skills/<category>/`
- **Installation for other agents**:
  - **Claude Code CLI**: Via `.clinerules` or `CLAUDE.md`
  - **Pi Agent (pi.dev)**: Via `AGENTS.md` or direct reference
  - **OpenCode CLI**: Via `--context` flag
  - **Universal approach**: Context files, RAG, or copy/paste
- **Usage**: How the skill loads automatically + explicit loading
- **Structure**: Tree showing SKILL.md + references/ + templates/ + scripts/
- **Resources**: Links to official docs, example projects
- **Contributing**: Fork → branch → PR workflow
- **License**: Link to LICENSE file
- **Author**: Nickname only (privacy)

**Language**: English for maximum reach (README can be bilingual if needed)

### 6. Add LICENSE

MIT License template:

```
MIT License

Copyright (c) <YEAR> @<nickname>

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
```

Commit and push:

```bash
git add LICENSE README.md
git commit -m "Add MIT License and installation documentation"
git push
```

## Privacy Considerations

**DO NOT include**:
- Real names in author sections (use GitHub nickname only)
- Personal contact information
- Private repository references
- Credentials or API keys

**Safe to include**:
- GitHub username/nickname
- Public project links
- Technical specifications
- Generic location (country/city if relevant to skill context)

## Multi-Language Strategy

If skill targets specific language community:
1. Write SKILL.md in target language (it's the working document)
2. Write README.md in English (for discoverability)
3. Optionally add README.<lang>.md for native speakers
4. Link alternate READMEs at top of main README

## Cross-Agent Installation Patterns

### Hermes Agent
```bash
cd ~/.hermes/skills/<category>/
git clone <repo-url> <skill-name>
```

### Claude Code CLI
Create `.clinerules` or `CLAUDE.md` in project root:
```markdown
When working on <topic>, reference the skill documentation at:
<repo-url>/blob/main/SKILL.md
```

### Pi Agent (pi.dev)
Add to `AGENTS.md`:
```markdown
## <Topic> Development
Skill: <repo-url>
```

### OpenCode CLI
```bash
opencode --context <path-to-cloned-skill>/SKILL.md
```

### Universal
Clone locally, reference in project docs, or provide URL to agent for retrieval.

## Maintenance

### Update workflow
```bash
cd ~/.hermes/skills/<category>/<skill-name>
# Make changes via skill_manage(action='patch') or skill_manage(action='write_file')
git add .
git commit -m "<description>"
git push
```

### Version tagging (optional)
```bash
git tag -a v1.0.0 -m "Initial stable release"
git push origin v1.0.0
```

## Common Pitfalls

1. **Forgetting to document structure**: Always include actual project folder structure diagram with all config files (not just minimal)
2. **English-only mindset**: Target language for SKILL.md if skill is region-specific
3. **Over-reliance on Hermes patterns**: Other agents need explicit installation steps
4. **Incomplete toolchain documentation**: Include ALL config files from real projects (.prettierrc, .eslintrc, commitlint, husky, etc.)
5. **Missing copyright year**: Always include current year in LICENSE
6. **Protected skill confusion**: Pinned skills CAN be updated (pin blocks deletion only)

## References

- Hermes Agent docs: https://hermes-agent.nousresearch.com/docs
- Example published skill: https://github.com/paraflu/hermes-skill-xiaomi-miband10-vela-js
- Real project structure: Compare skill docs against actual working projects
