# Windows Patch for claude_agent_sdk

## Problem
On Windows, `atexit` cleanup may not fire when the parent Python process is killed/crashes,
leaving orphaned `claude.exe` subprocesses.

## Fix
Add `signal.SIGBREAK` and `signal.SIGINT` handlers to `_kill_active_children()` on Windows.

## File
`claude_agent_sdk/_internal/transport/subprocess_cli.py`

## Patch

Replace:

```python
atexit.register(_kill_active_children)
```

With:

```python
atexit.register(_kill_active_children)

# Windows: atexit may not fire on crash/kill. Also register SIGBREAK (Ctrl+Break)
# and SIGINT so child processes are cleaned up on more exit paths.
if platform.system() == "Windows":
    for _sig in (signal.SIGBREAK, signal.SIGINT):
        try:
            signal.signal(_sig, lambda s, f: (_kill_active_children(), signal.signal(s, signal.SIG_DFL)))
        except (OSError, ValueError):
            pass
```

## Upstream
Submit PR to `anthropics/claude-code` (the `claude_agent_sdk` package).
