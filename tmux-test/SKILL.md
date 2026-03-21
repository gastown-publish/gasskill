---
name: tmux-test
description: Run tests via tmux by sending commands to a pane and capturing output. Use when running test suites, executing test commands, or observing test results in a tmux session. Triggers on requests like "run tests", "run the test suite", "execute tests in tmux", or "run tests in my terminal".
---

# tmux-test

Run test commands inside an existing tmux pane using `send-keys`, then capture the output with `capture-pane`.

## Workflow

### 1. Find the target pane

List available sessions, windows, and panes to identify where to run the tests:

```bash
tmux list-sessions
tmux list-windows -t <session>
tmux list-panes -t <session>:<window>
```

Use the format `<session>:<window>.<pane>` as the target (e.g. `main:0.0`). If the user has not specified a target, pick the most recently active pane or ask.

### 2. Send the test command

```bash
tmux send-keys -t <target-pane> "<test-command>" Enter
```

- Use `-l` for literal strings containing special characters to avoid key-name lookup:
  ```bash
  tmux send-keys -t <target-pane> -l "<test-command>"
  tmux send-keys -t <target-pane> Enter
  ```
- To send `Ctrl+C` to cancel a running process: `tmux send-keys -t <target-pane> C-c`
- To clear the pane first: `tmux send-keys -t <target-pane> "clear" Enter`

### 3. Wait for completion

Poll with `capture-pane` until the shell prompt reappears or output stabilizes:

```bash
sleep 2
```

For slow test suites, poll in a loop checking for the prompt string (e.g. `$`, `%`, `>`).

### 4. Capture the output

```bash
tmux capture-pane -t <target-pane> -p
```

Key flags:
- `-p` — print to stdout instead of a buffer
- `-S <line>` — start line (negative = history lines back; `-S -` = full history)
- `-E <line>` — end line (`-E -` = end of visible pane)
- `-e` — include escape sequences (colors/attributes)
- `-J` — join wrapped lines and preserve trailing spaces (useful for wide output)

To capture full scrollback history:

```bash
tmux capture-pane -t <target-pane> -p -S -
```

### 5. Interpret results

- Look for test framework summaries (PASSED/FAILED/ERROR counts) and shell prompt return.
- To get the exit code of the last command run in the pane:
  ```bash
  tmux send-keys -t <target-pane> "echo $?" Enter
  sleep 0.5
  tmux capture-pane -t <target-pane> -p
  ```

## Detecting the test framework

Check files in the working directory before choosing a command:

| File present | Framework | Command |
|---|---|---|
| `pytest.ini`, `pyproject.toml`, `setup.cfg` | pytest | `pytest` or `pytest -v` |
| `package.json` (jest in deps) | Jest | `npm test` or `npx jest` |
| `Gemfile` | RSpec | `bundle exec rspec` |
| `go.mod` | Go test | `go test ./...` |
| `Cargo.toml` | Cargo | `cargo test` |
| `mix.exs` | ExUnit | `mix test` |
| `pom.xml` | Maven | `mvn test` |
| `build.gradle` | Gradle | `./gradlew test` |

## Notes

- Always confirm the correct pane target before sending keys — sending to the wrong pane can interrupt other work.
- If no tmux session exists, inform the user rather than creating one silently.
- Avoid hardcoding session names; discover them with `tmux list-sessions` first.
