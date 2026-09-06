List every custom slash command available in this setup, in this format:

| Command | What it does | When to use it |
|---|---|---|
| `/save-progress` | Saves this session's progress, updates memory files, writes today's log | Before closing the terminal |
| `/commands` | Shows this table | When you forget what exists |
| {{`/your-command`}} | {{...}} | {{...}} |

Read the command files in `.claude/commands/` to build the table rather than
answering from recall — the set changes.

> 🔁 **Whenever a new command is added**, update this file *and* the index line
> in `MEMORY.md` in the same session. Deferring it is how the menu drifts out of
> sync with reality.
