---
name: browser
description: Drive a real browser via the Playwright MCP server to surf the web, explore a site, reproduce UI bugs, fill forms, take screenshots, or manually verify a user flow. Use for live/interactive browser work, not for writing persistent test specs (use the `playwright` skill for that).
---

# Browser

Control a live browser through the Playwright MCP server. Observe, don't codify.

**Tradeoff:** Ephemeral by design. If the flow matters long-term, switch to the `playwright` skill.

## 1. Check the MCP is live

**No MCP, no work. Don't fall back silently.**

If `browser_*` tools aren't available, stop and ask the user to:
- Confirm `.mcp.json` has the `playwright` entry.
- Restart Claude Code and approve the trust prompt.
- Run `/mcp` to verify.

Never substitute by writing `.spec.ts` files. That's a different skill.

## 2. Read before acting

**Snapshot the page; don't guess selectors.**

- Use `browser_snapshot` to see the accessibility tree.
- Act on element refs from the snapshot, not CSS/XPath invented from memory.
- Use `browser_take_screenshot` only when the user wants an image or visual state matters.
- Re-snapshot after any action to verify.

## 3. Stay in scope

**Touch only what was asked.**

- Don't wander to pages or links the user didn't mention.
- Don't submit forms on production unless the user confirms it's safe.
- Don't click destructive buttons (Delete, Pay, Cancel) without confirming.
- Don't invent credentials. If login is needed, ask.
- Close the browser when done.

## 4. Report concretely

**Quote what you saw. Don't summarize vaguely.**

- State observed text, element names, and any console or network errors.
- Separate: what you did, what happened, what's next.
- If a flow looks fragile, flag it and offer to codify as a Playwright spec. Don't switch skills silently.

Weak: "the login worked."
Strong: "clicked 'Sign in', redirected to /dashboard, heading 'Welcome, Thang' visible, no console errors."

## 5. Output Style

**No em-dash, en-dash, or AI-style typography in what you write.**

Applies to every message, report, and any text this skill produces:
- Use comma, colon, or period instead of `—` (em-dash) or `–` (en-dash).
- Use `->` instead of `→`; `<-` instead of `←`.
- Use `...` instead of `…`.
- Use straight quotes `"` and `'`, not curly `" "` or `' '`.
- No emoji. No check or cross symbols.
