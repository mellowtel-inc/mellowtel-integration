# mellowtel-integration

An open agent skill that integrates Mellowtel monetization into any supported project.
Follows the [agentskills.io](https://agentskills.io) open standard - works with Claude Code,
Cursor, Gemini CLI, GitHub Copilot, Windsurf, OpenCode, and any other compatible agent.

---

## How it works for developers

```bash
# 1. Install the skill (works with any compatible agent)
npx skills add https://github.com/mellowtel-inc/mellowtel-integration

# 2. Open their agent in their project and invoke the skill
/mellowtel-integration YOUR_API_KEY
```

The agent detects the project type, reads the matching guide locally, and executes
the full integration without any manual steps.

---

## Repo structure

```
mellowtel-integration/
  mellowtel-integration/        <- skill directory (directory name = slash command)
    SKILL.md                    <- entry point: detection logic + dispatch
    references/
      extensions-guide.md       <- vanilla/generic browser extension
      plasmo.md                 <- Plasmo framework
      wxt.md                    <- WXT framework
      electron.md               <- Electron apps
      windows-native.md         <- Windows .NET apps
  README.md
```

After `npx skills add`, the `mellowtel-integration/` skill directory lands at the standard
location for the agent being used (e.g. `.claude/skills/` for Claude Code, `.cursor/skills/`
for Cursor, etc.). Guides are local from this point. No network calls at runtime.

---

## SKILL.md design

### Frontmatter (agentskills.io standard fields only)

```yaml
---
name: mellowtel-integration
description: Integrates Mellowtel consensual bandwidth monetization into browser extensions
  (Plasmo, WXT, vanilla), Electron apps, and Windows .NET apps. Use when the user
  wants to add Mellowtel or provides a Mellowtel API key for integration.
compatibility: Requires file read/write access to the project directory.
license: MIT
---
```

No agent-specific extensions. Any agent that follows the open standard will load this correctly.

### Body: what the skill instructs the agent to do

**Step 1 - Resolve API key**
The key is passed as an argument after the command name. If not provided, ask the
user for their Mellowtel API key before proceeding.

**Step 2 - Detect project type** by reading local files in this order:

| Check | Result |
|---|---|
| `package.json` contains `"plasmo"` in dependencies | Plasmo extension |
| `wxt.config.ts` or `wxt.config.js` exists | WXT extension |
| Any `.csproj` file exists in the project | Windows .NET app |
| `package.json` contains `"electron"` in dependencies | Electron app |
| `manifest.json` exists in root or `src/` | Vanilla browser extension |

**Step 3 - Load the matching guide** using relative paths from the skill directory:

| Project type | Guide to read |
|---|---|
| Plasmo | `references/plasmo.md` |
| WXT | `references/wxt.md` |
| Electron | `references/electron.md` |
| Windows .NET | `references/windows-native.md` |
| Vanilla / other | `references/extensions-guide.md` |

Relative paths work across all agents since the entire skill directory is local
after install.

**Step 4 - Execute the integration**
Follow the guide exactly. Replace every placeholder (`TEST_CONFIG_KEY`,
`YOUR_API_KEY`, `YOUR_INTEGRATION_ID`) with the actual key from step 1.

**Step 5 - Report**
List every file created or modified so the developer knows what changed.

---

## References directory

The five files in `references/` are direct copies of the existing guides from
`integration-agent/agent/guides/`. No content changes - just copied into the
standard skill directory layout.

| Skill reference file | Source in this repo |
|---|---|
| `references/extensions-guide.md` | `guides/extensions/guide.md` |
| `references/plasmo.md` | `guides/extensions/plasmo.md` |
| `references/wxt.md` | `guides/extensions/wxt.md` |
| `references/electron.md` | `guides/electron/guide.md` |
| `references/windows-native.md` | `guides/windows-native/guide.md` |

---

## Build plan

### Phase 1 - Scaffold
- [ ] Create `mellowtel-integration/SKILL.md` with standard frontmatter and detection logic
- [ ] Copy all 5 guide files into `mellowtel-integration/references/`
- [ ] Validate with `npx skills-ref validate ./mellowtel-integration`

### Phase 2 - Test locally
- [ ] Run `npx skills add ./mellowtel-integration` in a test project
- [ ] Invoke `/mellowtel-integration TEST_KEY` and verify detection works for each project type
- [ ] Confirm guide loads and integration runs end to end

### Phase 3 - Publish
- [ ] Push repo to `github.com/mellowtel-inc/mellowtel-integration`
- [ ] Verify `npx skills add https://github.com/mellowtel-inc/mellowtel-integration` works
- [ ] Add the one-liner install command to mellowtel.com docs

---

## What is NOT in scope

- No npm package publishing
- No Vercel hosting (GitHub is the source, `npx skills add` pulls directly from it)
- No changes to existing guide content
- No runtime network fetches (all guides are local after install)
- No agent-specific frontmatter fields (pure agentskills.io standard)
