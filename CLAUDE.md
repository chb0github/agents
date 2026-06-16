# Global Claude Code Configuration

## Code Navigation

### Prefer LSP Over Grep
When searching for symbols, references, definitions, or type information, prefer LSP tools (go-to-definition, find-references, workspace-symbol) over grep/ripgrep. LSP is semantically aware and avoids false positives from comments, strings, or similarly-named identifiers. Fall back to grep only when LSP is unavailable or for plain-text/non-code searches.

## Behavioral Rules

### CLI Command Formatting
When providing CLI commands to the user:
- NEVER wrap commands in markdown code blocks (``` ```)
- Output commands as plain text only
- Use backslash continuation (\) for multi-line commands
- Commands must be directly copy-pastable

Example (correct):
./tf destroy -auto-approve \
-var=region=us-west-2 \
-var=aws_profile=default

NOT this (wrong):
```bash
./tf destroy -auto-approve \
-var=region=us-west-2
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

### SMS (Twilio)
You can send SMS messages via the Twilio MCP server. Access via MCP tools:

**Send SMS:**
Use `mcp__twilio__send_sms` with:
- `to`: Recipient phone number (E.164 format, e.g., "+17039754376")
- `from`: Twilio number "+14256754882" (your only number)
- `body`: Message text (up to 1600 characters)

**Default recipient:** +17039754376 (user's Google Fi number)

**List recent messages:**
Use `mcp__twilio__list_messages` with optional filters:
- `to`: Filter by recipient
- `from`: Filter by sender  
- `limit`: Number of messages (default: 20, max: 100)

**List phone numbers:**
Use `mcp__twilio__list_phone_numbers` to see available Twilio numbers.

**MCP Server details:**
- Location: `~/.local/share/mcp-servers/twilio-sms/`
- Configured in: `~/.mcp.json`
- Credentials: `TWILIO_ACCOUNT_SID` and `TWILIO_AUTH_TOKEN` env vars
- Authentication: Automatic via environment variables
- Trial account: Can only send to verified numbers (verify at https://www.twilio.com/console/phone-numbers/verified)

**User's phone numbers:**
- Twilio: (425) 675-4882 (+14256754882) - SMS, MMS, Voice enabled (local number, no verification needed)
- Google Fi: (703) 975-4376 (default recipient)
- CallCentric (legacy): (425) 394-2504, (253) 330-8807, (607) 443-1142

## Bootstrap / CLI Tool Prerequisites

If a CLI tool listed in the integrations above is not installed or not authenticated, prompt the user to install/configure it before proceeding. Specifically:

- **gcalcli**: Check with `which gcalcli`. If missing, install with `brew install gcalcli`. If not authenticated (first run fails), prompt the user to run `gcalcli list` interactively to complete OAuth.
- **gh**: Check with `gh auth status`. If not authenticated, prompt the user to run `gh auth login`.
- **~/.netrc entries**: If a curl command fails with 401/auth errors, remind the user which machine entry is needed in ~/.netrc and how to create app passwords if applicable.
