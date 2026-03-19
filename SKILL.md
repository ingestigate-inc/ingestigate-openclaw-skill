---
name: ingestigate
description: Investigative intelligence — document search, entity extraction, and relationship graphing. Analyze document corpuses to find connections between people, organizations, and identifiers.
env:
  INGESTIGATE_TOKEN:
    description: Short-lived access token from the Ingestigate credential generator. Expires in 30 minutes; refresh token valid for 8 hours. User generates at https://app1.ingestigate.com/search/agentic-token
    required: true
  INGESTIGATE_BASE_URL:
    description: API base URL from the credential JSON (e.g., https://app1.ingestigate.com). Provided alongside the token.
    required: true
---

# Ingestigate — Investigative Intelligence for AI Agents

Act as an investigative analyst. Ingestigate provides access to a corpus of documents, entity discovery (people, organizations, emails, phones, crypto addresses, and 25+ other types), relationship path tracing between entities, and retrieval of the specific documents where connections appear. Back every claim with evidence from the API.

## When to Use This Skill

Use Ingestigate when the user asks you to:
- Analyze documents to find connections between people or organizations
- Search a corpus of files (PDFs, emails, spreadsheets, images — 1,000+ formats supported)
- Investigate relationships, follow the money, map a network
- Extract entities from a document set
- Upload and process new files for investigation

## Getting Started

### Step 1: Check if the user has credentials

Say this to the user: "Do you already have an account on Ingestigate? If not, the registration process is quick, and I can guide you through it."

### Step 2a: Existing user — get credentials

Say this to the user: "I need you to log in and provide me with a credentials package so I can collaboratively use Ingestigate with you. Let's see what we can learn about your documents and how different entities are connected, together!

Please open this URL: `https://app1.ingestigate.com/search/agentic-token`

Once you log in, there will be a 'Generate Credentials' button. Click it, then copy the credentials to your clipboard and paste them here in the chat so we can begin."

### Step 2b: New user — guide through registration

Say this to the user: "No problem, registration is straightforward. Please open this URL: `https://app1.ingestigate.com/agentic-registration`

Complete the registration form and check your email for a verification link. After you click the link, you'll set a password and then be presented with a Mobile Authenticator Setup screen. This is for your security — if you haven't used multi-factor authentication before, you'll need an app like Microsoft Authenticator or Google Authenticator on your mobile device. Install it, scan the QR code on the setup screen, and enter the one-time code.

After that, you'll be directed to the page where you can generate agent credentials. Click 'Generate Credentials', copy the credentials to your clipboard, and paste them here in the chat so we can begin."

### Step 3: Parse and store credentials

The user pastes a JSON credential blob. Extract `access_token` and `api_base_url`. Store them as environment variables for the duration of this session only:

```bash
export INGESTIGATE_TOKEN="<access_token from credential JSON>"
export INGESTIGATE_BASE_URL="<api_base_url from credential JSON>"
```

**Credential lifecycle:** The access token expires in 30 minutes. The refresh token is valid for 8 hours. After 8 hours, all credentials are invalid and the user must generate new ones. Credentials are never persisted to disk — they exist only in the current session's environment variables and chat history. This is by design: short-lived tokens limit blast radius if exposed.

When you get a 401, refresh the token using the `token_endpoint` and `client_id` from the original credential JSON:

```
POST <token_endpoint>
Content-Type: application/x-www-form-urlencoded

grant_type=refresh_token&client_id=<client_id>&refresh_token=<refresh_token>
```

If refresh fails, ask the user to generate new credentials.

### Step 4: Load the full guide

After authentication succeeds, fetch the complete Agent Developer Guide for detailed endpoint specs, workflow recipes, and error handling:

```bash
curl --location "${INGESTIGATE_BASE_URL}/api/agent/guide" \
--header "Authorization: Bearer ${INGESTIGATE_TOKEN}" \
--header 'Content-Type: application/json'
```

This guide has everything: endpoint details, request/response formats, pagination, casing rules, and anti-hallucination instructions. Read Sections 1-3 immediately. Reference Section 6 for specific endpoint specs as needed.

## Core Investigation Workflow

**1. See what's available:**
```
GET /api/discover/collections
```

**2. Get the lay of the land — entity dashboard:**
```
POST /api/dashboard/entity-stats
Body: { "limit": 50 }
```
Returns entity counts by type, top entities ranked by document count, and totals. Use this to orient the investigation: "Your corpus contains X documents with Y entity mentions. The most frequently appearing people are..."

**3. Search documents:**
```
POST /api/search-faceted
Body: { "query": "wire transfer", "filters": {}, "page": 1, "pageSize": 10 }
```

**4. Read a specific document:**
```
POST /api/file-details
Body: { "dataSourceName": "elasticsearch", "jobNames": ["<collection>"], "selectedFile": { "docId": "<docId>" }, "format": "clean_text" }
```

**5. Search entities:**
```
POST /api/entities/search
Body: { "query": "john doe", "entity_types": ["Person"], "limit": 50 }
```

**6. Trace relationships between entities:**
```
POST /api/graph/paths
Body: { "entities": [{"type":"Person","value":"john doe"},{"type":"Organization","value":"acme corp"}], "maxBridgeNodes": 20 }
```
Entity values MUST be lowercase. Use `normalized_value` from entity search results.

**7. Get the evidence — source documents for a connection:**
```
GET /api/graph/edge-evidence?entity1Type=Person&entity1Value=john%20doe&entity2Type=Organization&entity2Value=acme%20corp&limit=20
```

## Critical Rules

**API call format — these are mandatory or requests silently fail:**
- Always use `--location` (the API sits behind an authentication reverse proxy that may issue redirects for HTTPS enforcement and path normalization — `--location` ensures these are followed correctly)
- Do NOT use `-s`, `-X`, `-o`, `-w` or other flags
- Use `--data` for POST with body (curl infers POST). Use `--request POST` only for bodyless POSTs.
- Use long-form flags: `--header` not `-H`, `--data` not `-d`
- Always include both headers: `Authorization: Bearer ${INGESTIGATE_TOKEN}` AND `Content-Type: application/json`

**Entity casing:**
- Entity type names are PascalCase: `Person`, `Organization`, `Email`, `CryptoAddress`
- Entity values are always lowercase: `john doe`, `acme corp`
- Search queries are case-insensitive

**Anti-hallucination:**
- If a response includes `processing_status.corpus_ready: false`, results may be incomplete. Tell the user.
- If processing is complete and a query returns zero results, state this definitively. Empty results from a fully processed corpus are authoritative.
- Only make claims based on data returned by the API. Never guess.

**Credential handling and security:**
- **No persistent API keys.** All credentials are short-lived: access token expires in 30 minutes, refresh token in 8 hours. After expiry, credentials are worthless.
- **Session-only storage.** Credentials are stored in environment variables (`INGESTIGATE_TOKEN`, `INGESTIGATE_BASE_URL`) for the current session only. They are never written to disk or persisted beyond the session.
- **User-initiated credential flow.** The user generates credentials in the Ingestigate web UI and pastes them into chat. The paste flow is the intended authentication method — it preserves the user's full identity and permissions through a delegated session. The token appearing in chat history is by design, with limited blast radius due to short expiry.
- **Organization-scoped data isolation.** Every API call executes with the user's exact permissions. No cross-organization data access is possible.
- **MFA required.** All Ingestigate accounts require multi-factor authentication.
