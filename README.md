# Slack Post From Context

A Claude Code skill that drafts a Slack-formatted message about whatever you just did in the conversation and copies it to the clipboard with `pbcopy`. You paste it into Slack yourself — the skill never sends.

## What problem this solves

Slack does **not** render standard markdown. `**bold**`, `[text](url)`, and `# Heading` all render as literal characters. Every agent that hasn't been told this ships ugly Slack messages on the first try.

This skill bakes in the mrkdwn syntax (single-asterisk bold, `<url|text>` links, `:emoji:`, no headings) and the `pbcopy` heredoc incantation that survives multi-line content with `*`, backticks, and angle brackets.

## What's in the skill

| File | Purpose |
|------|---------|
| `SKILL.md` | mrkdwn cheat-sheet, composition rules, and the `pbcopy` heredoc pattern |

No helper scripts — the skill body is the deliverable. Claude composes the message in mrkdwn and runs `pbcopy <<'EOF' … EOF` directly.

## Requirements

- macOS (uses `pbcopy`)
- Claude Code (or any agent that reads `SKILL.md` files)

## Install

Use the [`skills`](https://www.npmjs.com/package/skills) CLI from vercel-labs:

```bash
# user-level (available in every project)
npx skills add sunfmin/slack-post-from-context -g

# project-level (drops into ./.claude/skills/)
npx skills add sunfmin/slack-post-from-context
```

## Updating / publishing changes

This repo is the upstream that `skills add` reads from — there's no separate registry. To publish a fix:

```bash
git commit -am "…"
git push origin main
```

Existing installs pick up the change with:

```bash
npx skills update slack-post-from-context
```

## Usage

Once installed, just ask Claude Code naturally at the end of a working session:

```
> post this to slack
> draft a slack update about what we just shipped
> slack-format this for the team
```

Claude composes the message in Slack mrkdwn, pipes it to `pbcopy`, and replies "Copied — paste into Slack." Switch to Slack, paste, send.

## License

MIT
