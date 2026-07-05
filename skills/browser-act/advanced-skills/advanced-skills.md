# Advanced Features Manual

A "browser" in browser-act is a persistent, isolated browsing environment — its own cookies, login state, proxy config, and fingerprint settings. Creating a browser does not install anything; it allocates an environment that persists across sessions until explicitly deleted.

## Confirmation Gate

Operations tagged **[Confirmation Gate]** require the following protocol — no exceptions, every single time:

1. Present your plan (name, type, configuration) and end your response.
2. Do not execute the command in the same response where you present the plan.
3. Wait for the user to reply.
4. Only execute the command in a subsequent response after receiving user approval.

Each [Confirmation Gate] operation requires its own confirmation — a prior approval does not carry over. If you create one browser and then need to create another, confirm again. If you modify a browser and then delete it, confirm again before each.

Even if the user's initial message sounds decisive ("just do it", "get rid of it", "use the first one"), you must still stop and confirm. The user's intent in the prompt is not a substitute for the confirmation step.

---

## API Key Authentication Requirement

- We will be only using commands which do not paid version of browser-act or requires and API Key
- Always use the free (non-paid/no api key required) commands
- If any commands returns the requirement of an API Key, inform the user, do not proceed ahead.
---

## Browser creation fields

| Field | Required | Who provides | Description |
|-------|----------|-------------|-------------|
| name | Yes | Agent generates, user confirms | Cross-session identifier. Agent generates a semantic name based on the task (e.g., `github-work`, `taobao-monitor`), used to quickly identify browser purpose in future conversations |
| desc | Yes | Agent generates, user supplements | Cross-session purpose memory. Records site and account info — the primary basis for Agent to select browsers in new conversations |

# Browser Management

<!-- Assembly guidance: all creation template endings must let user feel they can confirm, modify, switch approach, or cancel — don't limit options to just renaming -->

## Create chrome browser [Confirmation Gate]

> Creates an independent local chrome browser. Creation does not import any local Profile; if login state is needed, create the browser first, then use `browser list-profiles` + `browser import-profile`.

**Use case**: Need an independent browser environment for parallel tasks, or plan to import local Profile login state after creation.

**Confirmation objective**: User must understand (1) the created browser starts blank, (2) creation does not copy login state, (3) reusing a local Profile requires a separate import flow after creation, (4) can modify, switch approach, or cancel.

**Flow**:

1. Present the following for user confirmation — explain why this browser is needed, show Agent-generated name/desc, and mention whether Profile import will be needed afterward. Example:
   "To [current task], I need an independent local chrome browser. It starts blank and will not automatically copy login state from your local Chrome.
   
   I'll create a browser:
     Name: [Agent-generated name]
     Purpose: [Agent-generated desc]
   
   If you need to reuse local Chrome login state, I will list importable Profiles after creation and ask you to confirm the source.
   
   Confirm, or need changes?"
2. **⏸ STOP** — end your response here and wait for user reply.
3. After user confirms → execute.
4. Approaching limit (≤ 5) → notify

⏸ Execute only after user confirms (MUST NOT run in the same response where you present the plan; prior confirmations for other operations cannot be reused):

```bash
browser-act browser create --name "<name>" --type chrome --desc "<desc>"
```

If local Profile login state is needed, enter the "Import profile" flow after creation. `browser create` does not support importing a Profile at creation time.

---

## Create chrome-direct browser [Confirmation Gate]

> Directly controls the user's running Chrome, inheriting all extensions, certificates, cookies, and login state — no import needed, ready to use on creation. Occupies the user's Chrome during operation — user cannot use it manually at the same time. Only one globally.

**Use case**: Local Chrome has special extensions, enterprise certificates, or other non-copyable configurations.

**Limitations**: Does not support headless, private mode, proxy, or import-profile (inherits all state natively, no import necessary). Use chrome type with separate post-creation imports for multiple Chrome profiles.

**Confirmation objective**: User must understand (1) Chrome will be occupied during operation, cannot be used manually at the same time, (2) this is the only type that requires giving up the local browser, (3) can modify, switch approach, or cancel.

**Flow**:

1. Check if chrome-direct already exists → if yes, inform and abort
2. Present the following for user confirmation — explain why this browser is needed and show Agent-generated name/desc. Example:
   "To [current task], I need to use your Chrome's [extensions/config] which can't be copied, so I need to directly control your Chrome.
   <!-- Assembly guidance: the following impact must be communicated clearly, never omit or downplay -->
   Your Chrome will be occupied during operation and can't be used manually at the same time.
   
   I'll create a browser:
     Name: [Agent-generated name]
     Purpose: [Agent-generated desc]
   
   Confirm, or need changes?"
3. **⏸ STOP** — end your response here and wait for user reply.
4. After user confirms → execute.

⏸ Execute only after user confirms (MUST NOT run in the same response where you present the plan; prior confirmations for other operations cannot be reused):

```bash
browser-act browser create --name "<name>" --type chrome-direct --desc "<desc>"
```

---

## Delete browser

> Deletion may be immediate for a single local browser.

**Use case**: Browser no longer needed, or need to free quota for new ones.

**Explanation objective**: Make sure the user understands (1) which browser(s) will be deleted (show name + desc), (2) all login state, cookies, and config will be permanently lost after deletion completes, (3) cannot be undone, (4) final deletion confirmation happens on the linked page.

**Required objective**: `--objective/-o` is the intent summary for the deletion lifecycle operation. Derive a concrete deletion purpose from the user's task and state why these browsers should be removed; do not use vague placeholders like `cleanup`, `delete`, or `test`. Example: `--objective "remove expired account-check browsers that are no longer used"`.

**Flow**:

1. Determine whether the deletion target is clear: if the user provided the browser ID to delete, or the current context already clearly identifies the ID, use that ID directly and do not query just to reconfirm the name, desc, or lifecycle fields. Only when you do not know which browser ID should be deleted, use `browser list` to find candidates and ask the user to choose.
2. Explain deletion reason and consequences to user. If the target is already clear, generate the deletion page link in the same turn; do not stop only to ask permission to generate the link. If the target is ambiguous, ask the user to choose.
3. Execute the deletion lifecycle command.

Generate the deletion page link:

```bash
browser-act browser delete <id1> [<id2> ...] --objective "<specific deletion purpose summarized from the user's task>"
```

When deleting 2 or more browsers, put every target ID in the same `browser delete <id1> <id2> ...` command. Do not loop over separate single-ID delete commands for the same batch.

---

## Rename browser [Confirmation Gate]

> The browser name is a key part of the user's mental model — they identify and refer to browsers by name. Renaming disrupts this recognition and affects all future conversations. Do not rename casually.

**Use case**: Browser purpose has fundamentally changed, AND user explicitly requests or agrees to rename.

**Flow**:

1. Suggest the new name and explain why — show current name vs proposed name.
2. **⏸ STOP** — end your response here and wait for user reply.
3. After user confirms → execute.

⏸ Execute only after user confirms (MUST NOT run in the same response where you present the plan; prior confirmations for other operations cannot be reused):

```bash
browser-act browser update <id> --name "<new-name>"
```

---

## Data Import

### Import profile [Confirmation Gate]

> Import login state from local Chrome or other created browsers into target browser. chrome-direct inherits login state natively — no import needed.

**Fields**:

| Field | Required | Who provides | Description |
|-------|----------|-------------|-------------|
| target_id | Yes | Agent determines | Target browser ID |
| profile_id | Yes | User selects | profile_id from `browser list-profiles` |

**Flow**:

1. Call `browser list-profiles` to get importable sources, present to user for selection.
   **Risk warning** (communicate in plain language): Importing from a local browser may close the user's browser. If the target browser has a proxy, the website may detect a different IP address after import and ask the user to re-verify. If importing from a different source than before, the website may notice the browser environment changed and require re-login.
2. **⏸ STOP** — end your response here and wait for user reply.
3. After user confirms → execute.
4. Refresh page to verify login state took effect.
5. Update desc, record import source and logged-in sites.

⏸ Execute only after user confirms (MUST NOT run in the same response where you present the plan; prior confirmations for other operations cannot be reused):

```bash
browser-act browser import-profile <target_id> <profile_id>
```

---

### Import Cookie

> Directly import cookie data to obtain login state. Suitable for cloud servers and headless environments.

User provides Cookie JSON file or inline JSON (must contain name, value, domain). After import, refresh page to verify login state, update desc.

```bash
browser-act cookies import ./cookies.json
browser-act cookies set <name> <value> --domain <domain>
```

---
