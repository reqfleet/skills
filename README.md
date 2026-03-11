# Reqfleet Skill for AI Agents

The Reqfleet skill empowers AI agents to interact with the [Reqfleet](https://reqfleet.com) platform directly from a terminal or command-line environment. It streamlines load testing workflows, from plan generation to test execution and result analysis.

## Core Capabilities

- **CLI Management:** Automatically resolves or downloads the Reqfleet CLI (`rqt`).
- **Plan Lifecycle:** Generate load test plans (JMeter/Locust) using natural language prompts, create projects, and upload test files.
- **Collection Management:** Launch, trigger, and monitor test collections with built-in retry logic.
- **Configuration:** Download, update, and upload collection configurations (YAML) for fine-grained control over your tests.
- **Resource Tracking:** Provides direct dashboard URLs for projects, plans, and collections.

## Prerequisites

To use this skill effectively, the environment should have:

1.  **Reqfleet API Key:** Required for all operations. Get yours from the Reqfleet dashboard.
2.  **Reqfleet API Endpoint:** Defaults to `https://reqfleet.com`.
3.  **Reqfleet CLI (`rqt`):** The agent can download this automatically, or you can provide a path to an existing binary.

## Installation

```bash
npx skills add https://github.com/reqfleet/skills
```



## Getting Started

When an agent initiates the Reqfleet skill, it should guide you through an initial setup flow:

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
