# RTK - Rust Token Killer

**Usage**: Token-optimized CLI proxy (60-90% savings on dev operations)

## Meta Commands (always use rtk directly)

```bash
rtk gain              # Show token savings analytics
rtk gain --history    # Show command usage history with savings
rtk discover          # Analyze Claude Code history for missed opportunities
rtk proxy <cmd>       # Execute raw command without filtering (for debugging)
```

🛑 **`rtk proxy` does not use a shell.** It splits the string on whitespace and execs the
first token, passing everything else — `&&`, later command names, their flags and paths —
as arguments to it. So:

- **Never chain commands in one `rtk proxy` call.** One command per call.
- **Never route `rm` (or any destructive verb) through `rtk proxy`.** Use plain Bash, and
  prefer moving a path to the scratchpad over deleting it.

`rtk proxy "rm -rf A && ls B && wc -l C"` runs as one `rm -rf` over A, B **and** C. With
`-f` it exits 0 and prints nothing, so it looks like it worked.

## Installation Verification

```bash
rtk --version         # Should show: rtk X.Y.Z
rtk gain              # Should work (not "command not found")
which rtk             # Verify correct binary
```

⚠️ **Name collision**: If `rtk gain` fails, you may have reachingforthejack/rtk (Rust Type Kit) installed instead.

## Hook-Based Usage

All other commands are automatically rewritten by the Claude Code hook.
Example: `git status` → `rtk git status` (transparent, 0 tokens overhead)

Refer to CLAUDE.md for full command reference.
