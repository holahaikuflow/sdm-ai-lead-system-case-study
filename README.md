# SDM Capital AI Lead System

Technical case study of a production AI-powered lead acquisition and qualification system integrated with WhatsApp Cloud API.

> The production source code is private because it contains commercial logic, credentials, infrastructure configuration, and internal operational details. This repository documents the system architecture, product decisions, workflows, security model, and engineering lessons without exposing sensitive information.

## Overview

SDM Capital needed a more effective way to respond to inbound real-estate leads, qualify prospects, access live property information, and alert the sales team when a lead required human attention.

I designed and deployed a production system that combines conversational AI, real-time inventory queries, structured data extraction, lead scoring, voice transcription, operational alerts, and human takeover.

The system operates through WhatsApp and is connected to a web administration panel used by the sales team.

## My role

I worked across:

- Product definition
- Conversation design
- System architecture
- API integrations
- Database design
- Lead qualification logic
- Frontend administration workflows
- Security decisions
- Testing and production verification
- Deployment and operational iteration

## Core capabilities

The system:

- Handles customer conversations through WhatsApp
- Responds automatically outside normal business hours
- Queries live property inventory
- Transcribes voice messages with OpenAI Whisper
- Extracts structured lead information from conversations
- Classifies leads by level of commercial interest
- Schedules visits
- Sends operational email alerts
- Persists conversation and lead data
- Allows the sales team to take over conversations manually
- Provides metrics and lead-management workflows through a React administration panel

## High-level architecture

```text
Customer
   |
   v
WhatsApp Cloud API
   |
   v
Cloudflare Worker
   |
   +--> Anthropic Claude API
   |
   +--> OpenAI Whisper
   |
   +--> Supabase / PostgreSQL
   |
   +--> Resend
   |
   v
React administration panel
```

## Main workflow

1. A customer sends a message through WhatsApp.
2. WhatsApp Cloud API sends the event to a Cloudflare Worker.
3. The Worker validates and processes the webhook.
4. The system retrieves relevant conversation and property context.
5. Voice messages are transcribed with OpenAI Whisper when necessary.
6. Claude generates a response and extracts structured lead information.
7. Lead data and conversation state are stored in PostgreSQL through Supabase.
8. The system updates the prospect score and determines whether human intervention is required.
9. Operational alerts are sent when a lead becomes commercially relevant.
10. The sales team can continue the conversation through the administration workflow.

## Lead qualification

The system extracts and evaluates information such as:

- Property type
- Preferred location
- Budget
- Financing status
- Purchase timeframe
- Level of urgency
- Contact intent
- Visit interest

This information is converted into structured lead data and used to classify prospects into operational priority levels.

The purpose of the scoring system is not to replace commercial judgment. It helps the sales team focus attention on leads that show stronger purchase intent while preserving the conversation history and extracted context.

## Human takeover

The system includes a human-in-the-loop workflow.

When a salesperson takes control:

- Automated responses can be paused
- Conversation history remains available
- Extracted lead information remains visible
- The salesperson can continue from the existing context
- Automation can be resumed when appropriate

This prevents the AI layer from becoming a closed system and keeps the sales team responsible for sensitive or commercially important interactions.

## Technology stack

- Cloudflare Workers
- Anthropic Claude API
- OpenAI Whisper
- WhatsApp Cloud API
- Supabase
- PostgreSQL
- React
- TypeScript
- Resend
- Cloudflare Pages
- Git and GitHub

## Security and privacy

The system was designed with the following controls:

- Secrets stored in Cloudflare environment configuration
- Service-role credentials restricted to backend execution
- Row Level Security for database access
- Authenticated administration workflows
- Signed access patterns where required
- Separation between public webhooks and internal operations
- HTML escaping and input validation
- No credentials or production secrets stored in the public repository

## Testing and production verification

The system was tested through:

- End-to-end WhatsApp conversations
- Voice-message transcription scenarios
- Property-search queries
- Incomplete and ambiguous customer input
- Lead-scoring validation
- Human takeover workflows
- Webhook retries and duplicated events
- Administration-panel verification
- Production monitoring and iterative debugging

Testing focused not only on ideal conversations, but also on adversarial, incomplete, repetitive, and operationally realistic user behavior.

## Engineering challenges

### Maintaining structured state across conversations

Customer conversations are not linear. Users frequently change requirements, provide partial information, or return after interruptions.

The system needed to preserve structured lead information while allowing new messages to update or correct previous assumptions.

### Combining conversational and operational behavior

The assistant needed to sound natural while also performing deterministic tasks such as inventory queries, lead extraction, scoring, scheduling, and alerts.

This required separating conversational generation from structured backend operations.

### Preventing premature automation

Not every message should trigger a sales alert or classify a prospect as qualified.

The system uses accumulated context rather than relying on a single phrase or isolated message.

### Supporting human intervention

The system needed to create operational leverage without blocking the sales team from taking control.

Human takeover was therefore treated as a core product requirement rather than an exception.

## Product impact

The system transformed WhatsApp from a manual contact channel into an operational lead-management workflow.

It reduced repetitive response work, created structured lead records from conversations, improved visibility into prospect quality, and allowed the sales team to focus on interactions with stronger commercial intent.

## What this case study demonstrates

This project demonstrates experience in:

- Applied AI product engineering
- Conversational systems
- LLM API integration
- Voice transcription
- Serverless backend architecture
- Structured data extraction
- Lead-scoring workflows
- PostgreSQL data modeling
- Human-in-the-loop system design
- Production deployment and debugging
- Translating business operations into software

## Repository scope

This repository contains documentation only.

It does not expose:

- Production source code
- API credentials
- Phone numbers
- Customer information
- Internal property data
- Private prompts
- Commercial scoring thresholds
- Infrastructure identifiers
