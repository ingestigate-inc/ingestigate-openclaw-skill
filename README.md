# Ingestigate — OpenClaw Skill

Investigative intelligence for AI agents. This skill gives OpenClaw agents the ability to search document corpuses, extract entities, trace relationship paths, and retrieve evidence — capabilities that are impossible for an agent working with raw files alone.

## Why This Exists

An AI agent can't OCR a scanned passport, parse a blockchain CSV, extract crypto addresses from a PDF, and map how those connect to shell companies in a DOCX — all in one session. The files are too varied, the formats too complex, and the cross-referencing too large for any context window.

Ingestigate handles all of that before the agent ever sees the data. It ingests documents in 1,000+ formats, automatically extracts 30+ entity types (people, organizations, emails, phones, crypto addresses, bank accounts, and more), and builds a relationship graph mapping connections across the entire corpus. The agent gets structured, evidence-backed results through a simple API — not raw files it can't process.

## What the Agent Can Do

- **Search** across all documents with full-text search and faceted filtering
- **Extract entities** — people, organizations, emails, phones, crypto addresses, dates, and 25+ more types
- **Trace relationships** — find connection paths between any two entities across the corpus
- **Retrieve evidence** — get the specific documents where two entities co-occur
- **Upload and process** new document sets through an automated ETL pipeline
- **Monitor processing** — real-time status of document ingestion, entity extraction, and graph building

## Self-Teaching

The agent fetches the complete developer guide from the API after authenticating. The platform teaches the agent how to use it — no external documentation required.

## Setup

1. **Create an account** at [app1.ingestigate.com/agentic-registration](https://app1.ingestigate.com/agentic-registration) or log in if you already have one.
2. **Generate credentials** at [app1.ingestigate.com/search/agentic-token](https://app1.ingestigate.com/search/agentic-token).
3. **Set the environment variable** with your access token:
   ```bash
   export INGESTIGATE_TOKEN="<your access token>"
   ```

The agent will guide you through this process if you haven't set it up yet.

## Plans

| Plan | Agentic API Calls/Day | Price |
|------|----------------------|-------|
| Trial | 50 | Free for 14 days |
| Starter | 300 | $49/month |
| Professional | Unlimited | $1,999/month |
| Enterprise | Unlimited | Custom |

## Security

- **No persistent API keys.** Short-lived delegated sessions only (30-minute access tokens, 8-hour refresh). When the session ends, the credentials are worthless.
- **Organization-scoped data isolation.** Every agent action is scoped to the user's exact permissions. No cross-organization data leakage.
- **Full audit trail.** Every action the agent takes is traceable to a specific authenticated user.
- **MFA required.** All accounts use multi-factor authentication.
- **Processing-aware responses.** API responses include corpus readiness signals so agents never report conclusions from incomplete data.
- **Air-gapped deployment available** for government, defense, and regulated industries.

## Links

- [Ingestigate](https://ingestigate.com)
- [Agent Developer Guide](https://app1.ingestigate.com/api/agent/guide) (requires authentication)

## License

This skill wrapper is licensed under [MIT-0](https://opensource.org/license/mit-0) (MIT No Attribution). The Ingestigate platform and API are proprietary.
