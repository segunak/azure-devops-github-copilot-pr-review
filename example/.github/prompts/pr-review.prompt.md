---
description: Quick-launch a PR review. Provide a PR number or link.
name: prReview
agent: PR Review
tools: ['execute', 'read', 'search', 'ado/*']
---

Review PR ${input:pr:Enter the PR number or paste the PR link} against main in Azure DevOps.

Focus area: ${input:focus:Any specific areas to emphasize in the review? (e.g., security, TypeScript best practices, API design, testing, all)}
