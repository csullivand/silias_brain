# Session Start

When a new session begins (first user message), IMMEDIATELY run the startup sequence below. Print a status message before and after each step so the user can see what's happening.

1. Print: `[obsidian] Reading vault "silias_brain" CLAUDE.md...`
2. Run: `obsidian read vault="silias_brain" file="CLAUDE"`
3. Print: `[obsidian] Reading Active Context...`
4. Run: `obsidian read vault="silias_brain" file="Active Context"`
5. Print: `[obsidian] Session context loaded.`
6. Summarize what you found and ask the user what they're working on.

```bash
obsidian read vault="silias_brain" file="CLAUDE"
obsidian read vault="silias_brain" file="Active Context"
```

---

# New Session

When starting work on a new ticket or topic (not continuing an existing one), create a new session note. Print status messages so the user can see what's happening.

1. Print: `[obsidian] Reading Session Template...`
2. Run: `obsidian read vault="silias_brain" file="Session Template"`
3. Print: `[obsidian] Creating session note for {topic}...`
4. Create the session note at `Claude Sessions/{project}/{topic}/{date}.md` using the template, filling in Context fields
5. Print: `[obsidian] Updating Active Context...`
6. Overwrite `Active Context` with the new session pointer
7. Print: `[obsidian] Updating Index...`
8. Add the new session to `Claude Sessions/Index`
9. Print: `[obsidian] New session ready.`

```bash
obsidian read vault="silias_brain" file="Session Template"
obsidian create vault="silias_brain" path="Claude Sessions/{project}/{topic}/{date}.md" silent content="..."
obsidian create vault="silias_brain" path="Claude Sessions/Active Context.md" overwrite silent content="..."
obsidian create vault="silias_brain" path="Claude Sessions/Index.md" overwrite silent content="..."
```

---

# Auto-Save (every 10 minutes)

After completing Session Start, set up a `/loop 10m` auto-save. The loop should:

1. Check if anything changed since the last save (new work, files modified, decisions made)
2. If something changed:
   - Append a checkpoint to the session note
   - Overwrite Active Context with the latest state
   - Print: `[obsidian] Auto-save: checkpoint saved.`
3. If nothing changed:
   - Stay completely silent — no output, no writes

Start the loop with:
```
/loop 10m Auto-save: check if work was done since last save. If yes, append checkpoint to session note and update Active Context. If nothing changed, stay silent.
```

---

# Session End

When the user ends the session or says goodbye, IMMEDIATELY run the shutdown sequence below. Print status messages so the user can see what's happening.

1. Print: `[obsidian] Saving session note...`
2. Append or create the session note at `Claude Sessions/{project}/{topic}/{date}.md` with a detailed checkpoint
3. Print: `[obsidian] Updating Active Context...`
4. Overwrite `Active Context` with the latest checkpoint: what was done, current state, and how to continue
5. Print: `[obsidian] Updating Index...`
6. Update `Claude Sessions/Index` if a new session was created or a topic's status changed
7. Print: `[obsidian] Session saved.`

```bash
obsidian create vault="silias_brain" path="Claude Sessions/{project}/{topic}/{date}.md" overwrite silent content="..."
obsidian create vault="silias_brain" path="Claude Sessions/Active Context.md" overwrite silent content="..."
obsidian create vault="silias_brain" path="Claude Sessions/Index.md" overwrite silent content="..."
```

---

# Knowledge Capture

During a session, when you encounter knowledge worth preserving beyond the session note, proactively save it to the appropriate vault folder without being asked. Print `[obsidian] Saving note: {name}...` so the user sees it.

What to capture:
- Architecture decisions or patterns → `/projects`
- Concepts learned (e.g., how a service works, what a term means) → `/concepts`
- External URLs, dashboards, docs, APIs → `/references`
- Quick daily observations or learnings → `/journal`

Rules:
- ALWAYS check if a note on that topic already exists first — update instead of duplicating
- Follow vault conventions: Title Case filename, YAML frontmatter with tags, wikilinks to related notes
- Keep notes atomic — one concept per note
- Don't save things that are only relevant to the current session (those go in the session note)

---

# Template Sync

When any section above (Session Start, New Session, Auto-Save, Session End, Knowledge Capture) is modified, also update the vault template so new projects stay in sync. Print `[obsidian] Syncing CLAUDE.local Template...` before updating.

```bash
obsidian create vault="silias_brain" path="Templates/CLAUDE.local Template.md" overwrite silent content="..."
```

Note: Only sync the shared sections. Project-specific sections (like Code Style Preferences below) should NOT be copied to the template.

---

# Code Style Preferences

> Add project-specific preferences below this line.