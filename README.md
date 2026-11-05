# Reqfleet Skills for AI Agents

This repository provides skills for interacting with the [Reqfleet](https://reqfleet.com) platform and for shared development workflows across Reqfleet repositories.

## Available Skills

| Skill | Purpose |
|-------|---------|
| [`reqfleet-basics`](skills/reqfleet-basics/SKILL.md) | Manage the Reqfleet CLI, load test plans, collections, and results. |
| [`reqfleet-address-pr-comments`](skills/reqfleet-dev/reqfleet-address-pr-comments/SKILL.md) | Triage PR review feedback, implement and verify fixes, and resolve addressed threads on the existing PR. |
| [`reqfleet-commit`](skills/reqfleet-dev/reqfleet-commit/SKILL.md) | Create Conventional Commits with Reqfleet branching, release-please, and attribution conventions. |
| [`reqfleet-issue-to-development`](skills/reqfleet-dev/reqfleet-issue-to-development/SKILL.md) | Fetch an issue, create an issue-numbered branch from the repository's default branch, implement and verify the change, and commit through `reqfleet-commit`. |
| [`reqfleet-go-style-core`](skills/reqfleet-dev/reqfleet-go-style-core/SKILL.md) | Apply core Go style, formatting, readability, nesting, and return-value guidance, with bundled reference material. |
| [`reqfleet-go-performance`](skills/reqfleet-dev/reqfleet-go-performance/SKILL.md) | Review Go hot paths for efficient conversions, container capacity, and value passing; measure results with the target workload. |

Shared engineering skills live under `skills/reqfleet-dev/`, with one directory per skill. Each skill follows the target repository's development and execution rules rather than assuming a particular language or environment.

## Reqfleet Platform Capabilities

- **CLI Management:** Automatically resolves or downloads the Reqfleet CLI (`rqt`).
- **Plan Lifecycle:** Generate load test plans (JMeter/Locust) using natural language prompts, create projects, and upload test files.
- **Collection Management:** Launch, trigger, and monitor test collections with built-in retry logic.
- **Configuration:** Download, update, and upload collection configurations (YAML) for fine-grained control over your tests.
- **Resource Tracking:** Provides direct dashboard URLs for projects, plans, and collections.

## Prerequisites

The `reqfleet-basics` skill requires:

1.  **Reqfleet API Key:** Required for all operations. Get yours from the Reqfleet dashboard.
2.  **Reqfleet API Endpoint:** Defaults to `https://reqfleet.com`.
3.  **Reqfleet CLI (`rqt`):** The agent can download this automatically, or you can provide a path to an existing binary.

## Installation

```bash
npx skills add https://github.com/reqfleet/skills
```

To install only the PR review skill:

```bash
npx skills add reqfleet/skills --skill reqfleet-address-pr-comments
```

The PR review skill requires a local repository clone, authenticated GitHub CLI (`gh`), and the `agynio/gh-pr-review` extension. It does not require a Reqfleet API key or `rqt`. See the [skill instructions](skills/reqfleet-dev/reqfleet-address-pr-comments/SKILL.md) for setup and permissions.

To install only the commit skill:

```bash
npx skills add reqfleet/skills --skill reqfleet-commit
```

The commit skill uses Git and the target repository's required checks. It does not require the PR review extension.

To install the issue workflow and its required commit skill:

```bash
npx skills add reqfleet/skills --skill reqfleet-issue-to-development reqfleet-commit
```

The issue workflow requires a local repository clone, Git, authenticated GitHub CLI (`gh`), and `reqfleet-commit`. It does not require the PR review extension. It asks before switching to the default branch or pulling, and does not push or open a PR unless requested.

To install the Go style and performance skills together:

```bash
npx skills add reqfleet/skills --skill reqfleet-go-style-core reqfleet-go-performance
```

Both skills follow the target repository's Go toolchain, working directory, execution environment, and checks. The style skill includes its principle and formatting references. Performance figures are illustrative rather than guarantees for the target repository.

## Getting Started

When an agent initiates `reqfleet-basics`, it should guide you through an initial setup flow:

1.  **Collection ID (Optional):** If you already have a collection you want to work with.
2.  **CLI Path (Optional):** Provide a path to a local `rqt` binary if available.
3.  **API Endpoint:** Press enter to use the default or provide a custom endpoint.
4.  **API Key (Mandatory):** Provide your API key (it will be handled securely as an environment variable).

## Key Workflows

### 1. Plan Generation & Creation
An agent can help create a new load test plan from scratch:
- Choose between **JMeter** or **Locust**.
- Provide a natural language **prompt** (e.g., "I want to send GET requests to example.com").
- Review the generated plan before saving.
- Select or create a **Project** to house the plan.
- The agent handles the upload and provides a link to the plan in the dashboard.

### 2. Running Collections
If a `collection_id` is provided, the agent can:
- **Launch** the collection engines.
- **Trigger** the test run (with automatic retries for up to 6 minutes while engines warm up).
- Display a **Run Summary** once the test starts.

### 3. Creating & Configuring Collections
If a new test suite is needed:
- **Create** a collection by selecting a project, zone (e.g., `us-east-1`), and provider (e.g., `aws`).
- **Configure** traffic by downloading the collection YAML, adding plans, and specifying runtime parameters like Virtual Users (`vu`), `rampup`, and `engines`.

## Security

- **Secrets Protection:** API keys should be stored in the `REQFLEET_API_KEY` environment variable and never logged or printed.
- **Safe Execution:** Agents should use safe argument passing (argv style) to prevent command injection when handling user-provided prompts.

## Resource Links

Dashboard URLs are typically provided in the following formats:
- **Collection:** `https://reqfleet.com/dashboard/collection/<id>`
- **Plan:** `https://reqfleet.com/dashboard/plan/<id>`
- **Project:** `https://reqfleet.com/dashboard/project/<id>`

---

*Note: This skill is designed as a foundational definition for Reqfleet automation and can be extended for more complex troubleshooting and reporting tasks across different agent platforms.*
