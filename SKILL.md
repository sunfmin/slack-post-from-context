---
name: slack-post-from-context
description: Use when the user asks to draft, share, or post something to Slack about the current conversation — phrasings like "post this to slack", "make a slack message about this", "share this on slack", "slack-format this", "draft a slack update". Composes the message in the *paste-into-composer* dialect of Slack formatting (single-asterisk bold, bare URLs, no headings, `:emoji:`) and pipes it to `pbcopy` so the user pastes it directly. macOS only.
---

# Slack post from current context

When the user wants to share what just happened in this conversation as a
Slack message, draft it in the format the Slack composer accepts on paste
and put it on the clipboard with `pbcopy`. The clipboard is the
deliverable — the user pastes the message into Slack themselves.

## Two dialects: paste vs. API — this skill is *paste*

Slack has two formatting dialects, and the rules differ:

- **API mrkdwn** (used by the Slack API, MCP tools, webhooks): supports
  `<url|text>` links, `<@U123>` mentions, `<#C123|name>` channels.
- **Composer paste** (what `pbcopy` + ⌘V hits): the WYSIWYG composer
  *partially* parses mrkdwn — `*bold*`, `_italic_`, `~strike~`, backticks,
  `:emoji:`, `>` quotes, and bullets all work. But it does **not** parse
  `<url|text>`; it treats the whole thing as one URL and percent-encodes
  the `|` to `%7C`, breaking the link.

This skill targets the *paste* dialect. If the user wants the message
actually sent (not drafted), use the Slack MCP and the API dialect there.

## Formatting cheat-sheet (paste dialect)

| Want | Regular markdown | Paste-into-Slack |
|------|------------------|------------------|
| Bold | `**bold**` | `*bold*` |
| Italic | `*italic*` | `_italic_` |
| Strikethrough | `~~text~~` | `~text~` |
| Inline code | `` `code` `` | `` `code` `` |
| Code block | ``` ```lang ``` | ``` ``` ``` (no language tag) |
| Link | `[text](url)` | bare URL on its own (Slack auto-linkifies; display = URL) |
| Heading | `# Heading` | none — bold the line: `*Heading*` |
| Bullet | `- item` | `• item` (renders nicer than `-`) |
| Numbered | `1. item` | `1. item` |
| Quote | `> text` | `> text` |
| User mention | n/a | `@name` (autocomplete on send turns it into a real ping) |
| Channel | n/a | `#channel` (same — autocomplete) |
| Emoji | n/a | `:rocket:` `:white_check_mark:` `:warning:` |

The three that bite hardest:

- `**bold**` renders as the literal characters `**bold**`. Use `*bold*`.
- `<url|text>` is *API* format — when pasted into the composer, the `|`
  becomes `%7C` and the link breaks. Paste a **bare URL**; Slack
  auto-linkifies it. Custom display text via `[text](url)` only works for
  recipients who have "Format messages with markup" enabled — don't rely
  on it.
- `# Heading` renders as a literal `#`. Bold the line instead.

## Composing the message

Read the conversation. Pick out:

1. **The headline** — what was achieved, decided, broken, or shipped. One
   line, bolded. Lead with it.
2. **Concrete artifacts** — file paths, PR/ticket numbers, commands, URLs,
   numbers. Bullet them.
3. **Anything the audience needs to act on or react to** — questions,
   blockers, asks. Call them out (`:warning:` or `:question:` helps).

Keep it tight. Slack messages over ~10 lines lose readers; condense.
Strip narrative ("First I tried X, then Y…") — Slack wants the result.

## Putting it on the clipboard

Use a heredoc with single-quoted `'EOF'` so `$variables`, backticks, and
backslashes pass through verbatim:

```bash
pbcopy <<'EOF'
*Shipped: KGM-3330 banner mismatch fix*

• Root cause: home carousel banner cached past TTL on app cold start
• PR: https://github.com/org/repo/pull/4711
• Verified on iOS 18.7.2 (iPhone 12 Pro Max) and Android 14 (Pixel 5a)
• cc @felix — please review before mobile release cut on 2026-05-01
EOF
```

Note the bare URL on its own — `<url|PR #4711>` would paste as a broken
URL with `%7C` in it.

After running, reply to the user with **one short line**:

> Copied — paste into Slack.

Do **not** also dump the message body into the chat reply. The user will
read it when they paste; echoing it just makes them read it twice.

## When NOT to use this skill

- The user wants the message *posted* (not drafted) — that's the Slack MCP
  (`slack_send_message` / `slack_send_message_draft`), not `pbcopy`
- They want an email, PR description, commit message, or doc — different
  format
- Non-macOS shell — `pbcopy` is macOS-only. On Linux use `xclip
  -selection clipboard` or `wl-copy`; on Windows, `clip.exe`. If you
  detect a non-mac platform, tell the user and skip the copy step

## Common mistakes

| Mistake | Fix |
|---------|-----|
| `**bold**` | Slack shows literal `**`. Use `*bold*`. |
| `# Heading` | Renders as `#`. Bold the line instead: `*Heading*`. |
| `<url\|text>` in pasted text | API mrkdwn, not composer-paste — the `\|` becomes `%7C` and the link breaks. Paste a bare URL; Slack auto-linkifies. |
| `[text](url)` to get custom display text | Only works for recipients with "Format messages with markup" enabled — unreliable. Use a bare URL. |
| Triple-backtick with language tag | Slack ignores the tag and shows it as code. Drop the language. |
| Echo the message body in the chat after pbcopy | The clipboard is the deliverable; don't make the user re-read it. |
| Use `echo "…" \| pbcopy` for multi-line | Quoting breaks on `*`, backticks, `<>`. Use a `'EOF'`-quoted heredoc. |
| Pipe markdown straight from a tool | Convert it to paste-format first. Don't copy-paste GitHub-style markdown. |
