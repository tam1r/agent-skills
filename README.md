# Adelante Agent Skills

Customer-facing skills for working with Adelante products from coding agents.

## Adelante Agent Studio Analyst

The Analyst skill teaches Codex, Claude Code, Cursor, and other compatible coding agents how to
inspect and safely remediate an AI support agent through the scoped Adelante Agent Studio MCP.

### Install

```bash
npx skills add tam1r/agent-skills \
  --skill adelante-agent-studio-analyst \
  -g -y
```

The Skills CLI detects supported coding agents automatically. To target one explicitly, append an
agent selector such as `--agent codex`, `--agent claude-code`, or `--agent cursor`.

### Update

```bash
npx skills update -g adelante-agent-studio-analyst -y
```

Installed copies are not updated automatically. Run the update command when Adelante announces a
new version, or add it to the coding agent's startup workflow.

### Connect Agent Studio

The skill contains MCP setup instructions. Adelante provides each customer with a scoped API key.
Never commit that key to a repository.
