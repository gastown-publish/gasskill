---
name: tmux-test
description: Run tests via tmux by sending commands to a pane and capturing output. Use when running test suites, executing test commands, or observing test results in a tmux session. Triggers on requests like "run tests", "run the test suite", "execute tests in tmux", or "run tests in my terminal".
---

# tmux-test

Run test commands inside an existing tmux pane using `send-keys`, then capture output with `capture-pane`.

## Key facts (verified by testing)

- Pane indices start at **1**, not 0 (e.g. `skill-test:1.1`, not `skill-test:0.0`)
- `capture-pane -p` pads output to the full terminal height with blank lines — always strip them before processing
- `$?` in a `send-keys` argument **expands in the calling shell**, not tmux — use single quotes to pass it literally

## Workflow

### 1. Discover the target pane

```bash
tmux list-sessions
tmux list-panes -t <session> -F "#{session_name}:#{window_index}.#{pane_index} #{pane_current_command}"
```

Target format: `<session>:<window>.<pane>` — e.g. `main:1.1`

If the user has not specified a target, pick the most recently active pane or ask.

### 2. Send the test command

```bash
tmux send-keys -t <target> "<test-command>" Enter
```

For commands containing special characters, send in two steps using `-l` (literal, disables key-name lookup):

```bash
tmux send-keys -t <target> -l "<test-command>"
tmux send-keys -t <target> Enter
```

Other useful key sends:
- Cancel a running process: `tmux send-keys -t <target> C-c`
- Clear the pane: `tmux send-keys -t <target> "clear" Enter`

### 3. Wait for completion — poll for the shell prompt

`capture-pane` pads output with blank lines to fill the terminal height. Strip them, then check the last real line for a shell prompt:

```bash
for i in $(seq 1 30); do
  sleep 0.5
  last=$(tmux capture-pane -t <target> -p | sed '/^[[:space:]]*$/d' | tail -1)
  if echo "$last" | grep -qE '%\s*$|\$\s*$|#\s*$|>\s*$'; then
    break
  fi
done
```

For fast tests a single `sleep 1` is sufficient. For slow suites use the polling loop.

### 4. Capture the output

```bash
tmux capture-pane -t <target> -p | sed '/^[[:space:]]*$/d'
```

Key flags:
- `-p` — print to stdout
- `-S -` — include full scrollback history (e.g. `tmux capture-pane -t <target> -p -S -`)
- `-E -` — capture to end of visible pane
- `-e` — include ANSI escape sequences (color)
- `-J` — join wrapped lines, preserve trailing spaces

### 5. Get the exit code

**Always use single quotes** so `$?` is not expanded by the calling shell:

```bash
tmux send-keys -t <target> 'echo EXIT:$?' Enter
sleep 0.3
tmux capture-pane -t <target> -p | sed '/^[[:space:]]*$/d' | grep 'EXIT:' | tail -1
```

## Detect the test framework

Check files in the working directory before choosing a command:

| File present | Framework | Command |
|---|---|---|
| `pytest.ini`, `pyproject.toml`, `setup.cfg` | pytest | `pytest -v` |
| `package.json` (jest in deps) | Jest | `npm test` |
| `Gemfile` | RSpec | `bundle exec rspec` |
| `go.mod` | Go test | `go test ./...` |
| `Cargo.toml` | Cargo | `cargo test` |
| `mix.exs` | ExUnit | `mix test` |
| `pom.xml` | Maven | `mvn test` |
| `build.gradle` | Gradle | `./gradlew test` |

## Notes

- Always confirm the correct pane target before sending keys — wrong pane can interrupt other work.
- If no tmux session exists, inform the user; do not silently create one.
- Avoid hardcoding session names; always discover with `tmux list-sessions` first.
