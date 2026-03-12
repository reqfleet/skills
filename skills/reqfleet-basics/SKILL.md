---
name: reqfleet-basics
description: Guides the setup and usage of the Reqfleet CLI (rqt), including authentication, plan generation, and collection execution. Use when starting a new Reqfleet session or performing core load testing tasks.
---
# Reqfleet Basics (Skill v12)

This skill defines the initial flow an agent should follow to start working with Reqfleet.

## Goal

1. Resolve and prepare the Reqfleet CLI (`rqt`).
2. Collect required connection inputs from the user.
3. Prepare for authenticated API usage.

This is the **starting point only** and can be extended later.

## Required Inputs

The agent must ask the user for:

1. **Collection ID** (optional, ask first when loading skills)
	- Prompt text: `Do you already have a collection_id? (optional)`
	- If provided, run `rqt collection get --collection_id=<collection_id>`, extract `project_id`, and store it in future context.
	- If not provided, run `rqt project list --include-collections=true` and show collections for user to choose.
	- After context is set, follow the collection launch/trigger flow in this document.
2. **CLI path** (optional)
	- Prompt text: `If you already have the Reqfleet CLI, please provide the path to it (optional).`
	- If provided: use that binary path and skip download.
3. **API endpoint** (optional)
	- Prompt text: `Please provide your Reqfleet API endpoint (press enter to use default: https://reqfleet.com).`
	- Default if user skips: `https://reqfleet.com`
4. **API key** (mandatory)
	- Prompt text: `Please provide your Reqfleet API key (required).`
	- If missing, stop and ask again.

## Authentication Setup

The CLI discovers credentials from environment variables.

Required before running `rqt` commands:

- Export API key to `REQFLEET_API_KEY`.

Optional but recommended:

- Export endpoint to `REQFLEET_API_ENDPOINT`.

Example:

```bash
export REQFLEET_API_KEY="$USER_API_KEY"
export REQFLEET_API_ENDPOINT="$USER_API_ENDPOINT"
```

## Resource URLs

When a resource is selected or created, show its dashboard URL using the current API endpoint:

- Collection URL: `<api_endpoint>/dashboard/collection/<id>`
- Plan URL: `<api_endpoint>/dashboard/plan/<id>`
- Project URL: `<api_endpoint>/dashboard/project/<id>`

`<api_endpoint>` should come from user input (or default `https://reqfleet.com`).

## CLI Resolution

The agent should use this order:

1. If user provides a CLI path, use that path directly.
2. If not provided, install `rqt` using the official download script below.

If a provided path does not exist or is not executable, ask the user to provide a valid path or proceed with download.

## CLI Download (Fallback)

Use the official installer script:

`curl -fsSL https://blackhole.reqfleet.io/rqt-cli/cli_download.sh | bash`

## Recommended Agent Flow

1. Ask user for optional CLI path.
2. If CLI path is provided, verify it is executable and use it.
3. If CLI path is not provided, run the official download script.
4. Use the downloaded local `./rqt` binary unless the user wants it moved elsewhere.
5. Ask for API endpoint and API key.
6. Export API key as `REQFLEET_API_KEY` for CLI authentication.
7. Export endpoint as `REQFLEET_API_ENDPOINT` (recommended).
8. Store endpoint and key in agent session context (never print API key in logs/output).

## Example Commands

### macOS/Linux shell example

```bash
if [ -n "$USER_RQT_PATH" ]; then
	rqt_bin="$USER_RQT_PATH"
else
	curl -fsSL https://blackhole.reqfleet.io/rqt-cli/cli_download.sh | bash
rqt_bin="./rqt"
fi

"$rqt_bin" --help
```

### Set auth env vars before CLI commands

```bash
export REQFLEET_API_KEY="$USER_API_KEY"
export REQFLEET_API_ENDPOINT="$USER_API_ENDPOINT"
```

## Plan Generation and Creation

After setup, the agent can generate a load test plan using `rqt`.

### Order Requirement

Plan creation must happen **only after**:

1. Plan generation is completed.
2. Generated content is shown to user.
3. User confirms saving the generated plan file.

### Required Questions

The agent must ask the user:

1. `What's the plan kind? (supported: jmeter, locust)`
2. `Do you already have a test file path to use for upload? (optional, e.g. /path/to/test.jmx or /path/to/test.py)`
3. Ask prompt only when needed: `What's the prompt? (example: I want to send GET requests to example.com)`

### Generation Command

Use this argument form:

- Command: `rqt plan generate`
- Args: `--plan_kind`, `kind`, `--prompt`, `prompt`

Prerequisite: `REQFLEET_API_KEY` must be set in the environment so `rqt` can authenticate.

If using a user-provided CLI path, run the same command via that binary path instead of `rqt`.

Where:

- `kind` is the user-selected plan kind (`jmeter` or `locust`)
- `prompt` is the user-provided natural language description

### Command Injection Safety (Required)

Never build shell command strings by concatenating user input.

Required behavior:

1. Execute commands without shell interpretation by passing binary + argument list (argv).
2. Put user-provided values (especially `prompt`) only into argument values, never into a shell string.
3. If a shell is unavoidable, escape/quote user input using a trusted shell-escaping function before execution.

Safe example (argv style):

```text
binary: rqt
args: ["plan", "generate", "--plan_kind", kind, "--prompt", prompt]
```

Unsafe example (do not use):

```text
"rqt plan generate --plan_kind=" + kind + " --prompt=" + prompt
```

### Output Handling Rules

1. Run the generation command and capture the generated plan content.
2. Show the generated content to the user first.
3. Ask for confirmation before saving:
	- `Do you want me to save this generated plan?`
4. Save only after confirmation.
5. File naming by plan kind:
	- `jmeter` -> save as something like `test.jmx`
	- `locust` -> save as something like `test.py`
6. Record the final upload file path (`test_file_path`) to use in plan upload:
	- Prefer user-provided path when generation is skipped.
	- Otherwise use the generated file path.

## Plan Creation (After Generation)

Once the generated file is confirmed and saved, proceed to create the plan record.

### Project Discovery and Selection

Before creating a plan, fetch projects accessible by the user:

`rqt project list`

If `project_id` is already available in context (for example from `rqt collection get --collection_id=<collection_id>`), reuse that `project_id` directly and skip project list/selection prompts.

Then:

1. Show the project list to the user.
2. Ask the user to choose a project.
3. If no listed project matches user requirements, ask for a new project name and create it:

`rqt project create --name=<project_name>`

4. Obtain the selected/created project UID (`project_uid`) for plan creation.
5. Show project URL to user: `<api_endpoint>/dashboard/project/<project_id>`.

When context already has `project_id`, use that as `project_uid` and do not ask the user to select/create project again.

### Plan Create Inputs

Plan creation requires:

- `kind` (`jmeter` or `locust`)
- `name` (plan name chosen by user)
- `project_id` (the selected project UID)

### Plan Create Step

Create the plan with kind, name, and project ID (UID).

The create response returns a `plan_id`.

After plan creation, show plan URL to user:

`<api_endpoint>/dashboard/plan/<plan_id>`

### Upload Generated Test File

After receiving `plan_id`, upload the test file using the CLI upload command that accepts `plan_id` and file path.

Use the recorded `test_file_path` from earlier steps (user-provided path or generated file path).

The agent should ensure:

1. `plan_id` is captured from create response.
2. The uploaded file matches the plan kind.
3. Upload runs only after successful plan creation.

### Suggested Agent Sequence

1. Ask plan kind.
2. Validate plan kind is either `jmeter` or `locust`.
3. Ask whether user already has a test file path; if yes, record it.
4. Confirm whether to generate a new plan file.
5. If generating, ask prompt.
6. If generating, run `rqt plan generate` using argv-style arguments (`plan`, `generate`, `--plan_kind`, `kind`, `--prompt`, `prompt`) without shell string concatenation.
7. If generating, present generated plan content and ask for save confirmation.
8. If confirmed, save with the correct extension and record saved path as `test_file_path`.
9. If generation is skipped, keep user-provided path as `test_file_path`.
10. Run `rqt project list` and show projects to user.
11. If `project_id` is already in context, skip to step 14.
12. Otherwise ask user to select a project or create a new one.
13. If user wants new project, run `rqt project create --name=<project_name>` and get `project_uid`.
14. Ask user for plan name.
15. Create the plan with `kind`, `name`, and `project_id`.
16. Capture returned `plan_id`.
17. Upload test file from `test_file_path` using `plan_id`. Run `rqt plan upload --filepath=<file_path> --plan_id=<plan_id>`.
18. If no file path is available, stop and ask user to provide one.

## Collection Handling (Existing `collection_id`)

When user provides an existing `collection_id`, use this flow.

### Collection Context Sync

Right after user provides `collection_id`, run:

`rqt collection get --collection_id=<collection_id>`

Then extract `project_id` from the response and set it in future context.

Show resource URLs to user from available IDs:

- `<api_endpoint>/dashboard/collection/<collection_id>`
- `<api_endpoint>/dashboard/project/<project_id>`

After `project_id` is set in context, do not ask the user again for project selection in later plan creation steps.

### Required Confirmation

Ask:

`Do you want me to launch and trigger this collection test now?`

If user does not confirm, do not run collection commands.

### Launch and Trigger Commands

1. Launch collection:

`rqt collection launch --collection_id=<collection_id>`

2. Trigger collection:

`rqt collection trigger --collection_id=<collection_id>`

### Trigger Retry Policy

Trigger may fail if engines are not ready. Keep retrying trigger until success, with a total timeout of 6 minutes.

Suggested behavior:

1. Start retry window after launch. Wait 30 seconds before first trigger attempt to allow engines to start.
2. If trigger fails, wait 30 seconds and retry.
3. Retry `rqt collection trigger --collection_id=<collection_id>` until success or timeout.
4. Stop retries after 6 minutes and report failure if still unsuccessful.

### Run Summary

After trigger succeeds, get results via:

`rqt collection run_summary --collection_id=<collection_id>`

Show the run summary output to the user.

## Collection Creation

Use this flow when user does not have a collection or existing collections do not meet requirements.

### Required Inputs for Creation

1. Ask user to choose a project first (`project_id`).
2. Ask user to choose traffic origin zone and provider.

To list zones/providers:

`rqt region list`

Show the zone list to the user and let them choose both `zone` and `provider`.

### Create Command

Create collection with selected values:

`rqt collection create --name=<collection-name> --project_id=<project_id> --zone=<zone> --provider=<provider>`

After creation, store the returned `collection_id` in context for subsequent launch/trigger/configuration workflows.

Show URLs after creation:

- `<api_endpoint>/dashboard/collection/<collection_id>`
- `<api_endpoint>/dashboard/project/<project_id>`

### Suggested Sequence

1. Confirm collection creation is needed (no collection or no suitable collection).
2. Ask user to select `project_id`.
3. Run `rqt region list`.
4. Show zones/providers and ask user to choose `zone` and `provider`.
5. Ask user for collection name.
6. Run `rqt collection create --name=<collection-name> --project_id=<project_id> --zone=<zone> --provider=<provider>`.
7. Capture created `collection_id` and set it in context.
8. Show collection and project URLs.

## Collection Configuration

Use this flow to inspect and update collection configuration for a selected `collection_id`.

### Recommendation

When users want to configure traffic origin in a collection, recommend doing it in the UI first.


### Download Existing Config

Run:

`rqt collection config_download --collection_id=<collection_id>`

Save the downloaded YAML into a local file (for example `collection-config.yaml`) and show the YAML to the user before making changes.

### List Available Plans in Project

Use the collection context `project_id` and run:

`rqt project get --project_id=<project_id> --include-plans=true`

Show the plan list to the user so they can choose which plan to add into the collection configuration.

### User Inputs for Config Update

Ask user:

1. Which plan to add (from the plan list).
2. Related runtime configuration values (for example: `vu`, `rampup`, `engines`, and other required plan config fields).
3. Traffic origin mapping values: `zone` and `provider`.

### Update Behavior

1. Start from the downloaded YAML configuration.
2. Apply the selected plan and user-provided runtime settings.
3. Ensure provider mapping is set in YAML:

```yaml
providers:
	<zone>: <provider>
```

4. Show the updated YAML content to the user.
5. Ask for confirmation before upload.
6. Upload updated YAML with:

`rqt collection configure --collection_id=<collection_id> --filepath=<yaml-filepath>`

### Example YAML Edits

Use these as reference patterns when updating the downloaded YAML. Keep the existing YAML schema/keys from the downloaded config and only change/add the relevant plan item fields.

#### Example 1: Add a jmeter plan entry

```yaml
providers:
  us-east-1: aws

plans:
	- plan_id: "pln_jmeter_123"
		kind: "jmeter"
		name: "checkout-smoke"
		config:
			vu: 50
			rampup: "30s"
			engines: 2
```

#### Example 2: Add a locust plan entry

```yaml
providers:
  us-west-2: gcp

plans:
	- plan_id: "pln_locust_456"
		kind: "locust"
		name: "search-load"
		config:
			vu: 200
			rampup: "2m"
			engines: 4
```

If the downloaded YAML already contains `plans`, append/update only the selected plan block rather than replacing unrelated entries.

### Suggested Sequence

1. Ensure `collection_id` and `project_id` are available in context.
2. Run `rqt collection config_download --collection_id=<collection_id>`.
3. Save output as a local yaml file (for example `collection-config.yaml`).
4. Show YAML config to user.
5. Run `rqt project get --project_id=<project_id> --include-plans=true`.
6. Show plan list to user.
7. Ask which plan to add and config values (`vu`, `rampup`, `engines`, etc.), including `zone` and `provider`.
8. Edit the local yaml file and set `providers:<zone>: <provider>`.
9. Show updated YAML and ask for confirmation.
10. On confirmation, run `rqt collection configure --collection_id=<collection_id> --filepath=<yaml-filepath>`.

## Agent Prompt Template

Use the following sequence when starting this skill:

1. `Do you already have a collection_id? (optional)`
2. If `collection_id` is provided, run `rqt collection get --collection_id=<collection_id>`, extract `project_id`, and set it in context.
3. If `collection_id` is not provided, run `rqt project list --include-collections=true`, show collections, and ask user to choose a `collection_id`.
4. If user has no collection or existing collections do not meet requirement, follow Collection Creation flow.
5. `If you already have the Reqfleet CLI, please provide the path to it (optional).`
6. If CLI path is empty: continue with CLI download fallback.
7. `Please provide your Reqfleet API endpoint (press enter to use default: https://reqfleet.com).`
8. If empty input: set endpoint to `https://reqfleet.com`.
9. `Please provide your Reqfleet API key (required).`
10. If API key is empty: `API key is required to continue. Please provide your Reqfleet API key.`
11. Export key: `export REQFLEET_API_KEY="$USER_API_KEY"`.
12. Export endpoint: `export REQFLEET_API_ENDPOINT="$USER_API_ENDPOINT"`.
13. If `collection_id` is available: ask `Do you want me to launch and trigger this collection test now?`
14. If confirmed, run launch -> wait 1 minute -> trigger with retry (timeout 6 minutes) -> run summary.
15. For collection configuration work: first remind user that traffic-origin configuration is recommended in the UI; if they want CLI flow, run `rqt collection config_download --collection_id=<collection_id>`, save/edit yaml file, run `rqt project get --project_id=<project_id> --include-plans=true`, ask for plan + config (`vu`, `rampup`, `engines`, etc.) and `zone`/`provider`, set `providers:<zone>: <provider>`, then upload with `rqt collection configure --collection_id=<collection_id> --filepath=<yaml-filepath>` after confirmation.
16. Whenever a project/plan/collection is selected or created in the above steps, show its dashboard URL.

## Security Notes

- Treat API keys as secrets.
- Put API key in `REQFLEET_API_KEY` so CLI can authenticate.
- Do not echo API keys in command output.
- Do not commit API keys into files.
- Prefer environment variables or secure secret stores.

## Current Scope

This version intentionally covers only:

- CLI resolution (`user path` first, otherwise download `rqt`)
- Endpoint input (with default)
- API key input (required)
- Plan generation, confirmation, project selection/creation, and plan creation/upload flow
- Existing collection launch/trigger flow with retry and run summary
- Context carry-over from `collection_id` via `rqt collection get` (reuse `project_id` without re-asking)
- Collection configuration flow using downloaded YAML + project plan list
- Collection creation flow using project selection + `rqt regions list` + provider/zone
- Resource dashboard URLs for selected/created project, plan, and collection

Future updates can add authentication verification, common `rqt` commands, and troubleshooting.
