---
tracker:
  kind: linear
  api_key: $LINEAR_API_KEY
  project_slug: "nodes-a99695dd4f09"
  active_states:
    - Todo
    - In Progress
  terminal_states:
    - Closed
    - Cancelled
    - Canceled
    - Duplicate
    - Done
polling:
  interval_ms: 30000
workspace:
  root: .symphony/workspaces
hooks:
  after_create: |
    git clone https://github.com/erikgj63/swift-markdown-engine.git .
    git remote add upstream https://github.com/nodes-app/swift-markdown-engine.git
agent:
  max_concurrent_agents: 2
  max_turns: 20
codex:
  command: codex app-server
  approval_policy: never
  thread_sandbox: workspace-write
  turn_sandbox_policy:
    type: workspaceWrite
    networkAccess: true
---
You are working on Linear issue `{{ issue.identifier }}`.

{% if attempt %}
Continuation context:
- This is retry or continuation attempt #{{ attempt }}.
- Resume from the current workspace state. Do not restart investigation that is already captured in commits, notes, or test output unless the issue changed.
{% endif %}

Issue context:
- Identifier: {{ issue.identifier }}
- Title: {{ issue.title }}
- Current status: {{ issue.state }}
- URL: {{ issue.url }}
- Labels: {{ issue.labels }}

Description:
{% if issue.description %}
{{ issue.description }}
{% else %}
No description provided.
{% endif %}

Workflow:
1. Work only inside the provided repository workspace.
2. Inspect `AGENTS.md`, `README.md`, `CONTRIBUTING.md`, `Package.swift`, and the relevant source/tests before changing code.
3. Start by checking `git status --short --branch` and `git remote -v`.
4. For code changes, create or reuse a branch named from the Linear issue identifier, make small commits, and keep `main` clean.
5. Reproduce or verify the issue signal before changing behavior whenever practical.
6. Run the relevant verification before claiming completion. For this Swift package, default to `swift build -v` and `swift test --parallel`; for demo app work, also build `Demo/MarkdownEngineDemo.xcodeproj`.
7. If GitHub tooling and credentials are available, push the branch to `origin` and open a pull request back to `erikgj63/swift-markdown-engine`.
8. Use the available Linear MCP or Symphony `linear_graphql` tool to leave a concise progress/result comment. If Linear writes are unavailable, report that as a blocker in the final message.
9. Stop only for true blockers such as missing credentials, unavailable required tools, failing upstream baseline, or ambiguous product requirements that cannot be resolved from the issue.

Final response requirements:
- Summarize completed actions.
- List verification commands and their results.
- List blockers only if any remain.
