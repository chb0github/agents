# Global Claude Code Configuration

## Behavioral Rules

### Context Display
At the END of each response (the very last line), show the time, current working directory (relative to ~), and git branch with dirty indicator — mimicking the oh-my-zsh candy theme:
```
[HH:MM:SS] 📂 ~/relative/path (branch-name *) ← * if there are uncommitted changes
[HH:MM:SS] 📂 ~/relative/path (branch-name)   ← clean
```

### Bash History
Every command you run via the Bash tool must be appended to `~/.zsh_history` so the user can find it later. After running a command, append it:
```bash
print -s 'the command you just ran'
```
Or if that doesn't work in non-interactive mode:
```bash
echo ": $(date +%s):0;the command" >> ~/.zsh_history
```

## Available Integrations

### Jira
You have direct access to Jira Cloud via REST API. Credentials are stored in ~/.netrc.

```bash
curl -snL "https://fandango.atlassian.net/rest/api/3/issue/TICKET-KEY" | jq .
```

- Do NOT use the `-f` curl flag (it silently swallows errors)
- Use this to read tickets, add comments, update status, or query projects on the user's behalf

### Confluence
You have access to Confluence via the Atlassian MCP server (OAuth-authenticated). Use `mcp__atlassian__` tools to:
- Search pages/content (`searchConfluenceUsingCql`, `search`)
- Read full page content (`getConfluencePage`)
- List spaces (`getConfluenceSpaces`)
- Browse page trees (`getConfluencePageDescendants`)
- Read/write comments (`getConfluencePageFooterComments`, `createConfluenceFooterComment`)

Also accessible via curl with ~/.netrc:
```bash
curl -snL "https://fandango.atlassian.net/wiki/rest/api/content?spaceKey=SPACE&title=Page+Title" | jq .
```

### Jira (MCP)
In addition to curl access, the Atlassian MCP server provides native Jira tools (`mcp__atlassian__` prefix):
- Search issues (`searchJiraIssuesUsingJql`, `search`)
- Read/create/update issues (`getJiraIssue`, `createJiraIssue`, `editJiraIssue`)
- Transition status (`transitionJiraIssue`)
- Add comments and worklogs

### GitHub
You have access to GitHub via the `gh` CLI and the GitHub MCP server. Two accounts are logged in:
- **christianBongiorno-fd** (work) — default active account
- **chb0github** (personal)

Switch with `gh auth switch --user <name>`. Always switch back to the previous account when done.

Use these to:
- Read/create/update PRs and issues
- View check status, review comments, diffs
- Search code and repositories

### Global CLAUDE.md Repo
The global `~/.claude/CLAUDE.md` is tracked in `chb0github/agents` (local: `~/dev/mine/agents`). After any change to the global CLAUDE.md, copy it to the agents repo and push:
```bash
cp ~/.claude/CLAUDE.md ~/dev/mine/agents/CLAUDE.md
cd ~/dev/mine/agents && gh auth switch --user chb0github && git add -A && git commit -m "..." && git push && gh auth switch --user christianBongiorno-fd
```

### Jenkins
You have access to Jenkins CI at https://ci-jenkins.fandango.com/ via REST API. Credentials (user: cbongiorno) are stored in ~/.netrc.

```bash
curl -sfnL "https://ci-jenkins.fandango.com/api/json" | jq .
```

- Use flags `-sfnL` for Jenkins API calls
- Use this to check build status, trigger builds, view console output, list jobs, and download artifacts
- Append `api/json` to any Jenkins URL to get its API response
- Use `?tree=` parameter to limit response fields
- Key jobs are in the `HOME-ENT-SERVICES` folder: `zoltar-client-utils`, `zoltar-modeltraining`, `zoltar-kc`
- Job URL pattern: `https://ci-jenkins.fandango.com/job/HOME-ENT-SERVICES/job/JOB_NAME/job/BRANCH/...`
- Don't use `?tree=` parameter — it returns empty results on this server

### Nexus
You have read access to Nexus 3 at https://nexus3.mgo.com/ — no auth required.

```bash
curl -sfL "https://nexus3.mgo.com/service/rest/v1/repositories" | jq '.[].name'
```

- No `-n` flag needed (no auth)
- Search for artifacts: `/service/rest/v1/search?repository=REPO&name=ARTIFACT`
- Key repos: `maven-central`, `maven-public`, `maven-snapshots`, `production`, `staging`

### Stack Overflow / Stack Exchange
When the user says "check my stackoverflow" or "search my stackoverflow" for a topic, use the `sof` CLI at `~/.local/bin/sof` to search their posts (user ID: 889053).

```bash
sof <query>              # keyword search across posts
sof -t <tag> [query]    # search by tag (+ optional keyword)
sof -s <site> <query>   # search a specific SE site (default: stackoverflow)
sof -a [tag]            # list answers (optionally by tag)
sof -q [tag]            # list questions (optionally by tag)
sof -n <num> ...        # number of results (default: 10)
```

Examples:
- `sof jq` — search posts mentioning jq
- `sof -t java streams` — search [java] tagged posts for "streams"
- `sof -s unix -t bash` — search unix.stackexchange for [bash] posts

### Email (IMAP)
You have access to the user's personal email via curl+IMAP. Credentials are stored in ~/.netrc.
Use this to read/search emails when asked.

### Google Calendar
You can create, list, and manage calendar events via `gcalcli` (installed via brew).
- List calendars: `gcalcli list`
- List upcoming events: `gcalcli agenda`
- Create event: `gcalcli add --title "..." --when "..." --duration N --where "..." --description "..."`
- OAuth tokens are stored locally after first interactive login.

### SMS (CallCentric)
You can send SMS messages from the user's CallCentric number (425) 394-2504 via `~/.local/bin/sms`.

```bash
sms <phone_number> "message text"
```

- Sends from (425) 394-2504 via CallCentric's web AJAX API
- Session cookies stored in `/tmp/cc_cookies.txt`
- Credentials in ~/.netrc as `machine www.callcentric.com`
- **Must use Firefox User-Agent** to bypass Cloudflare
- If session expires, the script re-authenticates automatically
- If email verification is required (new IP/session), fetch the code from Gmail via IMAP (search All Mail for FROM callcentric)

**Direct API call (if script unavailable):**
```bash
curl -s -b /tmp/cc_cookies.txt \
  -A "Mozilla/5.0 (Macintosh; Intel Mac OS X 10.15; rv:128.0) Gecko/20100101 Firefox/128.0" \
  -H "Content-Type: application/x-www-form-urlencoded; charset=UTF-8" \
  -H "X-Requested-With: XMLHttpRequest" \
  -H "Referer: https://my.callcentric.com/sms_conversation.php?rd=<TO>&ad=14253942504" \
  -X POST "https://my.callcentric.com/sms.ajax.php" \
  -d "action=send&rd=<TO>&ad=14253942504&message=<URL_ENCODED_MSG>"
```

**Login flow (if session expired):**
1. POST `https://www.callcentric.com/login/` with `go=login&backref=...&l_login=...&l_passwd=...`
2. If redirected to `email_verify.php`: fetch code from Gmail IMAP (`imaps://imap.gmail.com/%5BGmail%5D/All%20Mail` search FROM callcentric), then POST `email_verify.php` with `go=1&auth_code=<code>`

**User's phone numbers:**
- Google Fi: (703) 975-4376
- CallCentric texting: (425) 394-2504 (primary, on Pixel 9 Pro)
- CallCentric secondary: (253) 330-8807
- CallCentric inactive: (607) 443-1142

## Bootstrap / CLI Tool Prerequisites

If a CLI tool listed in the integrations above is not installed or not authenticated, prompt the user to install/configure it before proceeding. Specifically:

- **gcalcli**: Check with `which gcalcli`. If missing, install with `brew install gcalcli`. If not authenticated (first run fails), prompt the user to run `gcalcli list` interactively to complete OAuth.
- **gh**: Check with `gh auth status`. If not authenticated, prompt the user to run `gh auth login`.
- **~/.netrc entries**: If a curl command fails with 401/auth errors, remind the user which machine entry is needed in ~/.netrc and how to create app passwords if applicable.
