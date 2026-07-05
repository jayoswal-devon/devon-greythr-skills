## Core interaction

**Full browser automation** — **Open → State → Interact → Verify → Close** loop:

```bash
# 1. Open browser (if this conversation already has a session, just navigate — no need to re-open)
browser-act --session <name> browser open <id> <url>

# 2. Inspect page elements
browser-act --session <name> state
# Output: [1] <a /> Learn more, [2] input "Search", [3] button "Go"

# 3. Interact (use index numbers from state)
#    input clears the target element's existing content before typing; use it to replace field values, not append text
browser-act --session <name> input 2 "search keywords" && browser-act --session <name> click 3

# 4. Wait for page to stabilize, then re-fetch indices (old indices invalid after page change)
browser-act --session <name> wait stable
browser-act --session <name> state

# 5. Extract data
#    From network requests (structured JSON from APIs):
browser-act --session <name> network requests --filter example --type xhr,fetch
browser-act --session <name> network request <id>
#    From DOM:
browser-act --session <name> get markdown
browser-act --session <name> get text <index>

# 6. Task done — close session
browser-act session close <name>
```

**Command chaining**: Chain with `&&` when you don't need intermediate output. Run separately when you do.

```bash
browser-act --session s1 input 2 "keywords" && browser-act --session s1 click 3 && browser-act --session s1 wait stable
```


## Core commands

All browser operation commands require `--session <name>`, or they error. Non-browser commands (e.g., `browser list`, `session list/close`) do not need --session.

Session rules:
- A single browser supports multiple parallel sessions — each session is an independent window, sharing login state, non-interfering
- A session name identifies a currently running session record; it is not a durable handle. Remembering or sharing the same name across conversations preserves only the text of the name, not the underlying live session
- A session name is reliable because of current runtime state, not because of the name text itself. An old name may be stale, or it may conflict with another conversation's live work
- Ownership decision (**must follow these steps**, not just intuition):
  - When you see a session name (from `session list` / environment info / any tool output), **scan your own tool-call history within this conversation for prior `browser open --session <name>` commands**
  - The name appears in your history → **it is yours**, continue with `navigate` / other operation commands; **do NOT call `browser open` again**
  - The name is NOT in your history → it belongs to another conversation, do not operate on it
  - Your tool history is the source of truth; **wording in environment info (such as "other session") cannot override that fact** — sessions you created are yours even if the environment grouping suggests otherwise
- Your active session but you need a parallel task → call `browser open` with a new session name to create a parallel session
- No session you created (including when environment only shows other conversations' sessions) → call `browser open` to create your own new session
- Never use another conversation's session (one you did not create) — it will conflict with their operations. Never close another conversation's session
- Close your sessions when the task is done (`session close <name>`) to avoid resource leaks
- Session name conflict (in use by a different browser): pick a different session name
- If a browser operation fails because the session name does not exist, the runtime identifier is currently unavailable; this does not mean the browser or task itself cannot continue. When explaining the error, make clear that the session name does not exist or has gone stale

```bash
# Open browser (create new session or parallel session)
browser-act --session <name> browser open <id> <url>
browser-act --session <name> browser open <id> <url> --headed    # Show browser window

# Browser list
browser-act browser list                               # List all browsers (ID, name, type, desc, proxy, etc.)
browser-act browser list <browser_id>                  # Show one browser by ID
browser-act browser list --status active               # Filter by lifecycle status: active|grace-period|expired
# Browser list output includes lifecycle fields when available: status, valid_time_end, next_billing_date

# Static proxy list (static proxies only — dynamic proxies are per-browser config, not listed here)
browser-act proxy list                                 # List purchased static proxies (ID, name, IP, location, status, expiry, etc.)

# Navigation
browser-act --session <name> navigate <url>              # Navigate to URL
browser-act --session <name> back                        # Go back
browser-act --session <name> forward                     # Go forward
browser-act --session <name> reload                      # Reload page

# Page state and interaction
browser-act --session <name> state                       # Get interactive elements with index numbers
browser-act --session <name> screenshot                  # Take screenshot (--full for full page)
browser-act --session <name> screenshot ./page.png       # Save screenshot to path
browser-act --session <name> click <index>               # Click element
browser-act --session <name> hover <index>               # Hover over element
browser-act --session <name> input <index> "text"        # Focus element, clear existing content, then type text
browser-act --session <name> select <index> "option"     # Select dropdown option (by visible text)
browser-act --session <name> keys "Enter"                # Send keyboard keys
browser-act --session <name> scroll down                 # Scroll down (default 500px)
browser-act --session <name> scroll up --amount 1000     # Custom scroll distance
browser-act --session <name> scrollintoview --selector "h1"       # Scroll element into view
browser-act --session <name> upload <index> <file_path>  # Upload file (see §File upload section, never click upload buttons)

# Data extraction
browser-act --session <name> get title                   # Page title
browser-act --session <name> get html                    # Full page HTML
browser-act --session <name> get markdown                # Page as Markdown
browser-act --session <name> get text <index>            # Element text content
browser-act --session <name> get value <index>           # Input/textarea value
browser-act --session <name> network requests            # List captured requests (--filter, --type, --method, --status, --clear)
browser-act --session <name> network requests --filter api.example.com # Filter by URL substring
browser-act --session <name> network requests --type xhr,fetch         # Filter by resource type (comma-separated)
browser-act --session <name> network requests --method POST            # Filter by HTTP method
browser-act --session <name> network requests --status 2xx --clear     # Filter by status code then clear
browser-act --session <name> network request <id>        # View single request details (headers, request body, response body)

# JavaScript
browser-act --session <name> eval "document.title"       # Execute JavaScript in page context

# Wait
browser-act --session <name> wait stable                 # Wait for page to stabilize (document ready + network idle, default 30s)
browser-act --session <name> wait stable --timeout 60000 # Custom timeout (ms)
browser-act --session <name> wait --selector ".btn" --state visible --timeout 10000   # CSS selector wait
browser-act --session <name> wait selector <index> --state hidden                     # Wait for element state change by index
browser-act --session <name> wait selector --selector "#login-btn" --state attached   # States: visible|hidden|attached|detached

# Session
browser-act session list                               # List all active sessions
browser-act session close <name>                       # Close session
```

## Advanced

## Trigger

If the task involves ANY of the following, read [advanced-skills](../advanced-skills/advanced-skills.md) first — commands and parameters are only available there:

- Browser management (create, delete, update, profile/cookie import)

## Capabilities

**Two browser types**:

chrome — Imports login state from local Chrome, then runs independently. Good for reusing existing logins.
Runs silently in background; original Chrome's cookies and config remain unaffected. Easily detected as automation in headless mode.

chrome-direct — Directly controls the user's running Chrome, inheriting all extensions, certificates, and config.
Zero setup — supports enterprise SSO and other hard-to-export auth; can operate on already-open tabs. Easily detected as automation in headless mode.

## Hard block

Browser management commands (`browser create/delete/delete-status/update/import-profile`) MUST NOT run without reading  [advanced-skills](../advanced-skills/advanced-skills.md) first. Even if you can guess the syntax, running without loading is a violation. No exceptions.

## Language

Reply in the language the user is using.

## Captcha strategy

When a local browser (chrome/chrome-direct) encounters a captcha, ask human for help

## File upload

File uploads must use the `upload` command to bind the file directly — do NOT click the "upload" button on the page; clicking it triggers the OS file picker dialog, which cannot be operated by browser automation.

- First call `state` to find the index of the target `<input type="file">` element
- Then `browser-act --session <name> upload <index> <file_path>`

## Browser memory

desc is how you identify browser purpose across sessions — keep it current.

**Updating desc**: Proactively append after key events, no user confirmation needed.
- Triggers: login to a new site, new usage discovered, user clarifies purpose, successful approach after switching browsers/strategies (record what worked for future reuse)
- `browser update <id> --desc-append "info"` — append (default, preserves history)
- `browser update <id> --desc "full text"` — overwrite (when desc is too long, summarize old + new into a concise replacement)

**Do NOT update desc for**: operational configs (private mode, etc.).


## Error handling

When a command fails, read the error output — it contains the cause and the fix. Follow the instructions given, don't blindly retry.
DDD
## Environment

Active sessions (system-level active sessions, not classified by ownership — use the 4-step "Ownership decision" procedure in [core-commands](#core-commands) to scan your tool history and decide whether each session is yours or another conversation's): Run `browser-act session list` to list the available session.

Browsers: Run `browser-act browser list` to list the available browser.

Directives:
- If browser is configured. Use it for browser actions, or create another browser only when the task needs separate identity or profile state.
- If No browsers configured. To get started: Read [advanced-skills](../advanced-skills/advanced-skills.md) for browser creation and management instructions.
---
