# System Architecture

This document describes the high-level architecture of the SDM Capital AI Lead System.

The system has two independent application paths that connect to the same Supabase project with different privilege levels:

1. The React administration panel connects directly to Supabase using the anon key and an authenticated user session.
2. The Cloudflare Worker acts as the privileged backend for WhatsApp, Claude, Whisper, Resend, catalog queries, lead processing, and secure message delivery.

## Architecture overview

```text
                                      ┌──────────────────────────┐
                                      │     React Admin Panel    │
                                      │                          │
                                      │ - Leads                  │
                                      │ - Visits                 │
                                      │ - Messages               │
                                      │ - Metrics                │
                                      │ - Configuration          │
                                      └────────────┬─────────────┘
                                                   │
                                                   │ Supabase client
                                                   │ anon key + authenticated session
                                                   │ RLS-restricted access
                                                   v
┌──────────────┐      webhook      ┌──────────────────────────┐
│   Customer   │ ────────────────> │   WhatsApp Cloud API     │
└──────────────┘                   └────────────┬─────────────┘
                                               │
                                               │ webhook event
                                               v
                                    ┌──────────────────────────┐
                                    │    Cloudflare Worker     │
                                    │                          │
                                    │ - Webhook validation     │
                                    │ - Conversation routing   │
                                    │ - Lead extraction        │
                                    │ - Lead scoring           │
                                    │ - Property queries       │
                                    │ - Visit scheduling       │
                                    │ - Alerts                 │
                                    │ - Manual message sending │
                                    └───────┬──────┬──────┬────┘
                                            │      │      │
                         ┌──────────────────┘      │      └──────────────────┐
                         │                         │                         │
                         v                         v                         v
              ┌──────────────────┐     ┌──────────────────┐      ┌──────────────────┐
              │ Anthropic Claude │     │  OpenAI Whisper  │      │      Resend      │
              │       API        │     │                  │      │                  │
              └──────────────────┘     └──────────────────┘      └──────────────────┘
                         │
                         │ structured responses and extracted lead data
                         v
              ┌──────────────────────────────────────────────┐
              │             Supabase / PostgreSQL            │
              │                                              │
              │ Shared data store for:                       │
              │ - Leads                                      │
              │ - Conversations and messages                 │
              │ - Visits                                     │
              │ - Property inventory                         │
              │ - Metrics                                    │
              │ - Configuration                              │
              └──────────────────────────────────────────────┘
                         ^
                         │
                         │ service_role access
                         │ privileged backend operations
                         │
              ┌──────────┴──────────┐
              │  Cloudflare Worker  │
              └─────────────────────┘
```

## Architectural principle

The architecture separates browser-safe operations from privileged backend operations.

The administration panel performs authenticated product operations directly against Supabase. Row Level Security limits what each authenticated user can read or modify.

The Cloudflare Worker handles every operation that requires secrets, elevated privileges, third-party APIs, webhook processing, or backend orchestration.

Both paths use the same PostgreSQL database, but they do not have the same permissions.

## Path 1: React administration panel to Supabase

The administration panel connects directly to Supabase from the browser using:

- The Supabase JavaScript client
- The public anon key
- A valid Supabase Auth session
- Row Level Security policies

The panel reads and writes operational data such as:

- Leads
- Visits
- Messages
- Metrics
- Configuration

These requests do not pass through the Cloudflare Worker.

This keeps normal administration workflows simple and reduces unnecessary backend mediation while preserving access control through authentication and RLS.

## Path 2: Cloudflare Worker backend

The Cloudflare Worker is the system's privileged backend and orchestration layer.

It is responsible for:

- Receiving WhatsApp webhook events
- Validating and processing inbound events
- Retrieving conversation context
- Transcribing voice messages with OpenAI Whisper
- Calling Anthropic Claude
- Querying live property inventory
- Extracting structured lead information
- Updating lead state and qualification scores
- Persisting leads, conversations, and messages
- Scheduling visits
- Sending notifications with Resend
- Sending WhatsApp messages
- Supporting secure manual message delivery from the panel

The Worker connects to Supabase using the service_role key.

This gives the backend elevated access that bypasses Row Level Security. For that reason, the service_role key exists only in the Worker environment and is never exposed to the browser.

## Manual message exception

Manual message delivery is the only administration workflow that requires the React panel to call the Cloudflare Worker.

The browser cannot communicate directly with WhatsApp Cloud API because the WhatsApp access token is secret.

The flow is:

```text
Authenticated salesperson
        |
        v
React administration panel
        |
        | POST /send-manual
        | Supabase Auth session token
        v
Cloudflare Worker
        |
        | Validate Supabase session
        | Validate request and permissions
        v
WhatsApp Cloud API
        |
        v
Customer
```

Before sending the message, the Worker validates the Supabase Auth session included in the request.

This preserves the direct panel-to-Supabase model for normal administration while keeping secret-dependent operations behind the backend boundary.

## Privilege model

The system uses two distinct database access levels.

### Administration panel

```text
Credential: Supabase anon key
Identity: Authenticated Supabase user
Authorization: Row Level Security
Environment: Browser
Access level: Limited
```

The anon key is public by design. Security depends on the authenticated session and the database policies applied through RLS.

### Cloudflare Worker

```text
Credential: Supabase service_role key
Identity: Trusted backend
Authorization: Full privileged access
Environment: Cloudflare Worker
Access level: Elevated
```

The service_role key bypasses Row Level Security and must remain secret.

It is used only for backend operations that require broader access, including webhook processing, inventory queries, lead persistence, and system orchestration.

## Data consistency

The React administration panel and Cloudflare Worker write to the same Supabase/PostgreSQL database.

This creates a shared operational state across:

- Automated conversations
- Human-managed conversations
- Lead qualification
- Visit scheduling
- Metrics
- Configuration
- Property inventory

The database acts as the system of record.

Because both paths operate on the same data model, updates created by the Worker become visible to authenticated users in the panel, and authorized panel changes become available to backend workflows.

## External integrations

### WhatsApp Cloud API

Used for:

- Receiving inbound messages
- Sending automated replies
- Sending manually authored messages
- Processing conversation events

Inbound events enter through a public Worker webhook.

Outbound messages are sent only by the Worker because the API token must remain secret.

### Anthropic Claude API

Used for:

- Conversational response generation
- Context-aware interaction
- Structured lead-data extraction
- Interpretation of customer intent

Claude is part of the reasoning and extraction layer, not the system of record.

### OpenAI Whisper

Used for:

- Transcribing customer voice messages
- Converting audio input into text before conversational processing

The resulting transcript is passed into the same downstream workflow used for text messages.

### Resend

Used for:

- Operational email notifications
- Alerts when leads require attention
- Commercially relevant workflow events

## Security boundaries

The main security boundaries are:

1. Browser versus trusted backend
2. Public anon key versus private service_role key
3. RLS-restricted access versus privileged backend access
4. Authenticated administration requests versus public webhook traffic
5. Public application code versus secret third-party credentials

The architecture follows these rules:

- The service_role key is never exposed to the React application
- WhatsApp, Claude, Whisper, and Resend secrets remain in the Worker environment
- Browser database access is restricted through Supabase Auth and RLS
- Manual message requests require session validation
- Public webhook handling is separated from authenticated administration workflows
- Sensitive operational details are not included in this public repository

## Why this architecture was chosen

The architecture balances simplicity, security, and operational flexibility.

Direct React-to-Supabase access avoids building unnecessary backend endpoints for every administration workflow.

The Cloudflare Worker centralizes the operations that genuinely require backend control:

- Secret management
- Webhook processing
- Third-party API orchestration
- Elevated database access
- Secure outbound communication
- AI processing
- Operational automation

This separation reduces infrastructure complexity without weakening the security boundary around privileged operations.

## What this architecture demonstrates

This architecture demonstrates:

- Serverless backend design
- Direct browser-to-database application architecture
- Supabase Auth and Row Level Security
- Privileged service-role access
- Webhook processing
- Third-party API orchestration
- Human-in-the-loop workflows
- Shared operational state
- Security-boundary design
- Production-oriented AI system architecture
