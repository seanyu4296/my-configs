on write md docs/specs, ask me if u will write in tmp/ path except if i say later otherwise. don't always write a doc except if i told you so

when TS, use strict standards and fucntional programming style more/

When reporting information to me, be extremely concise and sacrifice grammar for sake of concision.

## File Writing (Claude Opus 4.5 / Sonnet 4.6)

Large writes cause stream aborts. Chunk file writes:
- `fs_write` for initial ~50 lines
- `fs_append` for subsequent chunks (~50-80 lines each)
- Never write 200+ lines in one call

For temp files: use workspace `tmp/` folder (system $TMPDIR is restricted)
