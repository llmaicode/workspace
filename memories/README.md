# AI Agent Memory

## Project

The GitHub account `llmaicode` is a dedicated workspace for an AI agent.

The purpose is to explore what an AI agent can build and operate when given access to development tools and GitHub.

Main areas:

* Web development
* DevOps and cloud engineering
* Analytics
* Cybersecurity
* AI and agent systems
* Automation
* Experiments and prototypes

## Working model

GitHub is the primary source of truth for:

* Code
* Infrastructure as code
* Project state
* Tasks
* Documentation
* Decisions
* Activity history

The preferred workflow is to keep as much of the project as possible inside GitHub.

The agent's main job is to create, modify, inspect, and organize things in GitHub.

The human handles actions that require manual account access, authentication, cloud setup, or clicking through external interfaces. The agent assists with those steps.

## Repositories

Current repositories include:

* `llmaicode/llmaicode` — agent profile and identity
* `llmaicode/llmaicode.github.io` — public website
* `llmaicode/tasks` — task and activity workspace
* `llmaicode/ai-memorys` — persistent project memory
* `llmaicode/virtual-machine` — virtual machine experiments
* `llmaicode/llm-agent-harnes` — agent harness experiments
* `llmaicode/human-notes-about-llmaicode` — human notes
* `llmaicode/test` — private testing repository

## Task management

The preferred task system is GitHub rather than ClickUp.

`llmaicode/tasks` contains:

* `README.md`
* `TASKS.md`
* `CHANGELOG.md`

GitHub Issues and repository files should be used where appropriate.

Avoid unnecessary project-management complexity.

## Website

`llmaicode.github.io` is the public home for the AI agent.

The website should evolve toward:

* Project pages
* Analytics
* Public updates
* Links to GitHub projects
* A simple publishing workflow

## Infrastructure preferences

Prefer:

* Free tiers
* No payment or bank-account requirements
* GitHub-based workflows
* Serverless architectures where appropriate
* Portable code that can run outside a specific platform
* Vercel's free tier for suitable experiments

Avoid unnecessary vendor lock-in.

## Performance

Performance is extremely important.

Tools and workflows should feel fast and direct. Avoid unnecessarily complex systems, slow navigation, or heavyweight tooling when a simpler solution works.

## Communication style

Responses should be:

* Concise
* Essence-first
* Free of filler and repetition
* Focused on the user's actual request

Preserve important information while removing unnecessary words.

Make small logical improvements where useful, but avoid unsolicited ideas, excessive structure, or fluff.

Think:

> Dictation → clean, concise, logically organized version.

Usually make the result shorter than the original input unless additional detail is genuinely necessary.

## Current direction

The overall experiment is to see how much of an AI agent's development workflow can be handled through GitHub alone.

The agent should push GitHub and static-file workflows as far as practical before introducing additional services.

Cloud hosting, databases, deployments, and other external infrastructure can be added when they provide clear value.
