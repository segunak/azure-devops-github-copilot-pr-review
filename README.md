# GitHub Copilot PR Review Agent for Azure DevOps

> **This project is a workaround.** GitHub Copilot already has a built-in [Code Review agent](https://docs.github.com/en/copilot/how-tos/use-copilot-agents/request-a-code-review/use-code-review) that reviews pull requests automatically, but it only works with GitHub-hosted repositories. Microsoft will likely extend that capability to Azure DevOps Git repos at some point. Until they do, this project fills the gap. **If your repo is on GitHub, skip all of this and use the built-in Code Review agent instead.**

Review Azure DevOps pull requests with GitHub Copilot from inside VS Code. Drop one prompt file in your repo, run it, and get a fully tailored PR Review Agent in minutes. Zero infrastructure, zero code, just Markdown.

## The Problem

GitHub Copilot's built-in Code Review agent only works with GitHub-hosted repositories. If your repo is on Azure DevOps, you cannot use it, and there is no native alternative.

This project fills that gap. It gives you a GitHub Copilot [custom agent](https://code.visualstudio.com/docs/copilot/customization/custom-agents) that reviews Azure DevOps PRs from inside VS Code, posts inline comments with individual approval, and enforces your team's coding standards. The setup is a few Markdown files and one JSON config. Any team can have it working the same day. When Microsoft ships native Copilot PR review support for Azure DevOps, this project becomes unnecessary.

## How It Works

An engineer opens VS Code, switches to the "PR Review" agent in the chat dropdown, provides a PR number or link, and the agent:

1. Fetches the PR details and linked work items from Azure DevOps via the [Azure DevOps MCP Server](https://github.com/microsoft/azure-devops-mcp)
2. Diffs the remote PR branch against the remote default branch using git (the user's local branch is irrelevant)
3. Explores the broader repo structure on the default branch to understand established patterns around the changed files
4. Reviews every changed file against the team's review standards and industry best practices, actively hunting for bugs, logical errors, security issues, and architectural concerns
5. Presents a structured summary in chat, grouped by severity (Critical Issues, Suggestions, Good Practices)
6. Walks the engineer through each finding one at a time, showing the exact comment text and asking for individual approval before posting each one
7. Posts approved comments as inline threads on the PR in Azure DevOps, replies to existing threads instead of duplicating

The agent never auto-posts comments. There is no bulk "approve all" option. The engineer reads and approves (or skips, or edits) every single comment individually.

## Prerequisites

- [VS Code](https://code.visualstudio.com/) with [GitHub Copilot](https://docs.github.com/en/copilot/managing-copilot/managing-copilot-as-an-individual-subscriber/about-github-copilot-free) (free tier works)
- [Node.js 20+](https://nodejs.org/) (the [Azure DevOps MCP Server](https://github.com/microsoft/azure-devops-mcp) runs via npx)
- Access to your Azure DevOps organization (the MCP server uses your local ADO credentials)

## Getting Started

### The Fast Way - Use the Scaffolding Prompt

There is a [prompt file](https://code.visualstudio.com/docs/copilot/customization/prompt-files) that does all of the setup for you. It analyzes your codebase, detects your tech stack and conventions, and generates every file the PR Review Agent needs, tailored to your repository. Its design follows the same patterns as GitHub Copilot's built-in [`/init` command](https://github.com/microsoft/vscode-copilot-chat/blob/main/assets/prompts/init.prompt.md), which VS Code uses to generate workspace instructions by analyzing the codebase.

1. Download [`create-pr-review-agent.prompt.md`](create-pr-review-agent.prompt.md) from this repository and place it in your repo at `.github/prompts/create-pr-review-agent.prompt.md`. The file name and folder path must be exact.
2. Open VS Code, open the GitHub Copilot chat panel, and type `/createPrReviewAgent`.
3. It will ask for your ADO organization, project, ADO base URL, and default branch. Fill those in and let it run. The repository name is auto-detected from the git remote and presented for confirmation before proceeding.
4. Review the generated files, especially `.github/instructions/code-review.instructions.md` (the AI's best guess at your team's coding standards). Edit anything that does not match your expectations.
5. Commit and push. Your team can start using the agent immediately.

The prompt generates these five core files (or merges into them if they already exist):

| File | Purpose |
|------|---------|
| `.github/agents/pr-review.agent.md` | The custom agent - persona, 7-phase workflow, guardrails |
| `.github/instructions/code-review.instructions.md` | Review standards the agent reviews against, generated from your codebase |
| `.github/prompts/pr-review.prompt.md` | A `/prReview` slash command for quick-launching reviews |
| `.vscode/mcp.json` | Azure DevOps MCP server configuration |
| `.github/copilot-instructions.md` | Global Copilot context (created or updated with ADO integration and agent description) |

### The Manual Way - Set Up Each File Yourself

If you prefer to set things up by hand, see the [`example/`](example/) folder for a complete working reference (Node.js/TypeScript). To adapt it for your team:

1. **Copy the file structure** from `example/` into your repo:
   - `.github/agents/pr-review.agent.md`
   - `.github/instructions/code-review.instructions.md`
   - `.github/prompts/pr-review.prompt.md` (optional, adds a `/prReview` slash command)
   - `.vscode/mcp.json`
   - `.github/copilot-instructions.md` (optional but recommended)

2. **Update the agent file.** There is a Team Configuration comment block at the top listing values to find-and-replace: ADO organization, ADO project, repository name, default branch, and ADO base URL. Do the find-and-replace, then update the Phase 4 review categories to match your team's tech stack.

3. **Update the MCP config.** In `.vscode/mcp.json`, change the organization name argument to your ADO org.

4. **Write your review standards.** Edit `.github/instructions/code-review.instructions.md` to describe your team's coding standards, architectural rules, and domain-specific patterns. You can add more instruction files to `.github/instructions/` for additional standards - the agent reads every file in that folder automatically.

5. **Commit and push.** Your team can now use the agent immediately.

## Using the Agent

Once set up, there are two ways to start a review:

**Option A: Agent dropdown.** In VS Code, open GitHub Copilot chat, click the agent dropdown (or type `@`), select "PR Review", and type a PR number (e.g., `12345`) or paste a PR link.

**Option B: Slash command.** Type `/prReview` in the chat panel (requires the prompt file from step 1). It will ask for the PR number and an optional focus area.

The agent handles the rest: fetching the diff, reading context, reviewing against your standards, presenting findings, and posting approved comments to Azure DevOps.

## What Gets Generated

The scaffolding prompt creates five interconnected files. Here is what each one does and why it exists.

### Custom Agent (`.github/agents/pr-review.agent.md`)

A [custom agent](https://code.visualstudio.com/docs/copilot/customization/custom-agents) is a Markdown file with YAML frontmatter that defines a persona, tools, and workflow instructions for GitHub Copilot. This agent defines a 7-phase workflow:

1. **Identify the PR** - Fetch metadata from ADO, display summary
2. **Get the Diff** - Remote-only diffs via git (never local branch)
3. **Gather Broader Context** - Explore the repo on the default branch to understand patterns
4. **Review Against Standards** - Analyze every changed file against instruction files and best practices
5. **Chat Summary** - Structured report grouped by severity
6. **Post Comments** - One-at-a-time approval flow, duplicate-aware posting
7. **Post-Review Summary** - Final tally with PR link

The `tools` frontmatter tells GitHub Copilot which tools the agent can use. `ado/*` gives it access to all Azure DevOps MCP tools. `execute`, `read`, and `search` give it access to terminals (for git commands), file reading, and code search.

### Instruction Files (`.github/instructions/`)

[Custom instructions](https://code.visualstudio.com/docs/copilot/customization/custom-instructions) are Markdown files that define rules GitHub Copilot should follow. The agent reads all files in `.github/instructions/` to understand your team's review criteria. You can have one file or many - the agent picks them all up.

### Prompt File (`.github/prompts/pr-review.prompt.md`)

A [prompt file](https://code.visualstudio.com/docs/copilot/customization/prompt-files) provides a `/prReview` slash command as a convenience. It references the custom agent and defines two input variables: the PR number and an optional focus area. This file is optional. Most engineers will just switch to the agent via the dropdown.

### MCP Server Configuration (`.vscode/mcp.json`)

[MCP servers](https://code.visualstudio.com/docs/copilot/customization/mcp-servers) let GitHub Copilot interact with external systems via the [Model Context Protocol](https://modelcontextprotocol.io/). This config sets up the [Azure DevOps MCP Server](https://github.com/microsoft/azure-devops-mcp), giving the agent access to ADO tools for fetching PR details, reading comment threads, and posting inline comments.

### Global Copilot Instructions (`.github/copilot-instructions.md`)

The workspace-level [copilot-instructions.md](https://code.visualstudio.com/docs/copilot/customization/instructions) provides GitHub Copilot with general context about the repository, the ADO integration, and a description of the PR Review Agent. This helps with discoverability and ensures all Copilot interactions (not just PR reviews) have appropriate context about the project.

## Design Principles

- **No noise.** The agent does not flag cosmetic issues, minor style preferences, or trivial formatting. If a finding would not meaningfully improve the code, it is not surfaced. Noise destroys trust in the review.
- **No fabrication.** If the code is clean, the agent says so. A clean PR is a good outcome. The agent never invents findings to justify the review.
- **Bug hunting.** The agent actively looks for bugs, logical errors, off-by-one mistakes, race conditions, null/empty guards, and places where changes in one file could break assumptions in another.
- **Remote-only diffs.** All diffs and file reads come from the remote refs (`origin/{defaultBranch}` and `origin/{sourceBranch}`). The user's locally checked-out branch is never used.
- **Broader context awareness.** Before reviewing, the agent explores the repo structure on the default branch to understand sibling files, shared modules, and dependency relationships around the changed files.
- **Flexible categories.** The agent organizes findings into categories that fit the PR, not a rigid checklist. It adapts categories based on what the changes actually touch.
- **Individual comment approval.** Every comment is shown to the user with exact text before posting. The user can approve, skip, or edit each one. No bulk approvals.
- **Duplicate-aware posting.** When re-running the agent on a PR that already has review threads, it replies to existing threads instead of creating duplicates.

## Customization

### Adding Review Standards

Create additional instruction files in `.github/instructions/` for domain-specific standards. The agent reads every file in that folder. For example:

- `pipeline-yaml.instructions.md` - Rules for Azure Pipelines YAML
- `database.instructions.md` - SQL and migration standards
- `infrastructure.instructions.md` - Bicep or ARM template rules

### Changing Phase 4 Categories

The agent file has example review categories in Phase 4. These are suggestions, not a fixed checklist. The agent adapts them per-PR. But if you want the default suggestions to better match your stack, edit that section in the agent file.

### Using a Specific Model

Add a `model` field to the agent's YAML frontmatter to pin a specific model:

```yaml
---
description: Review an Azure DevOps PR against main. Provide a PR number or link.
name: PR Review
tools: ['execute', 'read', 'search', 'ado/*']
model: ['Claude Opus 4.6']
---
```

The scaffolding prompt does not set a model by default, so it uses whatever model you have selected in VS Code.

## Limitations

- **Local only.** This runs in each engineer's VS Code instance. There is no server-side automation, no webhook trigger, and no way to run it automatically when a PR is created. The engineer must explicitly invoke it.
- **Requires Node.js 20+.** The Azure DevOps MCP server runs via `npx` and needs Node.js installed on the machine.
- **ADO authentication.** The MCP server uses your local Azure DevOps credentials. If you are not authenticated to ADO, the server will not work. See the [Azure DevOps MCP Server docs](https://github.com/microsoft/azure-devops-mcp) for authentication details.
- **Not a replacement for human review.** This is a tool to assist reviewers, not replace them. It catches patterns, enforces standards, and surfaces things humans might miss. The engineer still makes the final call.
- **VS Code only.** Custom agents are a VS Code feature. This does not work in Visual Studio, JetBrains, or other editors.

## Alternatives and Prior Art

This is not the only way to get AI-powered PR reviews on Azure DevOps. Here is what else exists and why this approach was chosen.

### GitHub Copilot Code Review (GitHub-Hosted Repos Only)

GitHub has a [built-in Code Review agent](https://docs.github.com/en/copilot/how-tos/use-copilot-agents/request-a-code-review/use-code-review) that can review PRs automatically. It is excellent, but only works with GitHub-hosted repositories. If your repo is on Azure DevOps, you cannot use it. This project exists specifically to fill that gap.

### Pipeline-Based Automation

Extensions like [ado-copilot-code-review](https://github.com/little-fort/ado-copilot-code-review) run as Azure DevOps pipeline tasks. They use the Copilot CLI to review diffs and post general comments on PRs automatically. The trade-off: they require a PAT (personal access token) for authentication, post only general comments (not inline on specific lines), have no human-in-the-loop approval, and do not understand the broader codebase context. They are fully automated, which is a strength or a weakness depending on your needs.

### GitHub Copilot SDK

The [GitHub Copilot SDK](https://github.com/github/copilot-sdk) (Technical Preview) lets you build programmable agents with full API access. You could build a server-side solution that triggers on PR webhooks, reviews code, and posts comments without any human intervention. The engineering effort would be significant (weeks, not hours), and the auth story for Azure DevOps integration is still evolving. If you want full automation and are willing to build and maintain the infrastructure, the SDK is the right foundation.

### Community Discussion

There is an [active community discussion](https://github.com/orgs/community/discussions/151205) (280+ upvotes) requesting native Copilot support for Azure DevOps PR reviews. Microsoft will likely ship something eventually. Until then, this project provides a zero-friction alternative that works today.

### Why This Approach

This project optimizes for time-to-value. One prompt file, five minutes of setup, no infrastructure to maintain. The trade-off is that it runs locally in each engineer's VS Code, which means there is no automation - someone has to invoke the agent. For teams that want a reviewer assistant rather than a reviewer replacement, that is the right trade-off.

## Repository Structure

```txt
create-pr-review-agent.prompt.md    # The scaffolding prompt (copy to your repo)
example/                            # Complete working reference (Node.js/TypeScript)
  .github/
    agents/pr-review.agent.md       # Example custom agent
    instructions/                   # Example review standards
      code-review.instructions.md
    prompts/pr-review.prompt.md     # Example slash command
    copilot-instructions.md         # Example workspace instructions
  .vscode/
    mcp.json                        # Example MCP server config
LICENSE                             # MIT
README.md                          # This file
```

## License

[MIT](LICENSE)
