<p align="center">
  <a href="https://www.getadelante.com/">
    <img src="assets/adelante-logo.png" alt="Adelante CX" width="300">
  </a>
</p>

<h1 align="center">Agent Studio skills for your coding agent</h1>

<p align="center">
  Investigate real support conversations, improve your AI agent, and verify production fixes<br>
  from Codex, Claude Code, Cursor, or any compatible coding agent.
</p>

<p align="center">
  <a href="https://www.getadelante.com/agent-platform"><strong>Explore Agent Studio</strong></a>
  ·
  <a href="https://www.getadelante.com/quiz"><strong>Get your AI agent</strong></a>
  ·
  <a href="https://github.com/tam1r/agent-skills/releases"><strong>Releases</strong></a>
</p>

<p align="center">
  <a href="https://skills.sh/tam1r/agent-skills"><img alt="skills.sh" src="https://img.shields.io/badge/skills.sh-Agent%20Skills-6C63D9"></a>
  <a href="https://github.com/tam1r/agent-skills/releases"><img alt="Latest release" src="https://img.shields.io/github/v/release/tam1r/agent-skills?color=6C63D9"></a>
  <a href="LICENSE"><img alt="MIT License" src="https://img.shields.io/badge/license-MIT-111827"></a>
</p>

---

## Adelante Agent Studio Analyst

The Analyst skill gives your coding agent a safe operating process for Adelante Agent Studio. It
can reconstruct a customer case, inspect the live prompt, knowledge, assigned-tool contracts, and
feedback, then apply the smallest authorized fix and verify the live result.

It is built for production support work—not generic chatbot prompting.

<p align="center">
  <img src="assets/analyst-workflow.svg" alt="Investigate, find the cause, apply safely, and verify live" width="100%">
</p>

### What you can do

| Use case | What your coding agent does |
|---|---|
| Investigate a ticket | Reconstructs the conversation, tool calls, retrieved knowledge, prompt rules, and feedback |
| Triage feedback | Groups recurring issues, separates root causes, and identifies the smallest safe fix |
| Improve knowledge | Updates a scoped internal snippet or directs you to the authoritative URL source |
| Improve behavior | Applies a guarded prompt change only when a broad behavioral rule is necessary |
| Audit your agent | Finds contradictions across prompt, knowledge, and available assigned-tool contracts |
| Report outcomes | Analyzes support volume, handovers, resolution patterns, and attributed revenue |

### Safety is part of the workflow

- Access stays limited to the production agents assigned to your API key.
- Shared tool descriptions, webhook URLs, credentials, and execution settings remain hidden.
- Prompt and direct knowledge edits use guarded writes to avoid overwriting newer changes.
- URL and Google Doc content is edited at its source, then reindexed by Agent Studio.
- Every production change requires live read-back before it is reported as verified.
- Admin and MCP feedback approval use the same standard feedback pipeline.

## Install

Install the skill globally and let the Skills CLI detect your coding agents:

```bash
npx skills add tam1r/agent-skills \
  --skill adelante-agent-studio-analyst \
  -g -y
```

Or target one coding agent explicitly:

```bash
npx skills add tam1r/agent-skills \
  --skill adelante-agent-studio-analyst \
  --agent codex \
  -g -y
```

Replace `codex` with `claude-code`, `cursor`, or another agent supported by the
[Skills CLI](https://github.com/vercel-labs/skills).

## Connect Agent Studio

The skill contains the complete MCP setup instructions. Adelante provides each customer with a
scoped Agent Studio API key; keep it out of git and load it from an environment variable.

Once connected, start with:

> Show me the production agents I can access and summarize their pending feedback. Do not change
> anything yet.

Then ask naturally—for example:

> Investigate ticket 96728. Reconstruct what happened, inspect the prompt, retrieved knowledge,
> and tool results, then identify the root cause. Do not make changes.

## Stay up to date

```bash
npx skills update -g adelante-agent-studio-analyst -y
```

Installed copies are not updated automatically. Run this when Adelante announces a new version or
add it to your coding-agent startup workflow.

## Want an AI support agent for your business?

[Adelante](https://www.getadelante.com/) builds, launches, and continuously improves AI support
agents that answer customers, use approved tools, complete support work, and hand off with context.

**[Get your AI agent →](https://www.getadelante.com/quiz)**

---

<p align="center">
  <a href="https://www.getadelante.com/">Website</a>
  ·
  <a href="https://www.getadelante.com/agent-platform">Agent Studio</a>
  ·
  <a href="https://github.com/tam1r/agent-skills/issues">Support</a>
</p>
