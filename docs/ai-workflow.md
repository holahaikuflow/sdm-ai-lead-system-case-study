# AI-Assisted Engineering Workflow

This document describes how AI coding tools were used during the development and operation of the SDM Capital AI Lead System.

The project was built with an AI-assisted engineering workflow using Claude Code, Cursor, and ChatGPT. These tools supported investigation, implementation, debugging, documentation, and review, while product, architecture, security, validation, and release decisions remained under human responsibility.

## Purpose of the workflow

The purpose of using AI tools was to accelerate engineering work without delegating technical accountability.

The tools were used to:

- Investigate unfamiliar APIs and integration requirements
- Compare implementation alternatives
- Generate and refine code changes
- Trace bugs across multiple system components
- Review architecture and security implications
- Prepare test scenarios
- Document system behavior and decisions
- Reduce repetitive implementation work

They were not treated as autonomous decision-makers.

## Tools used

### Claude Code

Claude Code was used primarily for repository-level engineering work.

Typical uses included:

- Inspecting the existing codebase
- Tracing data flow across files
- Proposing implementation plans
- Editing multiple related files
- Investigating bugs
- Reviewing integration logic
- Preparing changes for testing
- Documenting technical behavior

Because Claude Code could operate across the repository, its proposals were reviewed as engineering suggestions rather than accepted automatically.

### Cursor

Cursor was used for local code navigation, focused edits, inline assistance, and repository review.

Typical uses included:

- Understanding individual functions and modules
- Reviewing implementation details
- Applying small changes
- Comparing code alternatives
- Inspecting types and data structures
- Supporting debugging during active development

### ChatGPT

ChatGPT was used as a reasoning and review partner outside the production repository.

Typical uses included:

- Reviewing architecture before implementation
- Challenging proposed technical solutions
- Comparing trade-offs
- Identifying risks
- Improving prompts for coding agents
- Structuring documentation
- Preparing public case studies
- Translating technical work into clear professional evidence

## Responsibility model

AI tools accelerated execution, but responsibility remained with the engineer.

Human responsibility included:

- Defining the product problem
- Deciding system scope
- Choosing the architecture
- Approving database and API designs
- Evaluating security boundaries
- Reviewing generated code
- Testing system behavior
- Verifying production changes
- Approving deployments
- Handling operational incidents

The central rule was:

> AI could propose or implement changes, but it could not approve them.

## Engineering workflow

The workflow followed a repeated sequence.

### 1. Define the problem

Before using an AI tool, the problem was described in operational terms.

Examples included:

- A webhook was not persisting the expected lead state
- A conversation was losing structured context
- A voice message was not entering the normal processing flow
- A manual message needed to be sent securely from the administration panel
- A lead was being classified too early
- A database operation required a different privilege level

The objective was to define the failure or requirement before requesting implementation.

### 2. Investigate the existing system

The relevant code, data flow, database structure, and external integrations were inspected before changes were made.

The investigation focused on:

- Where the behavior originated
- Which components were involved
- What data was entering and leaving each step
- Whether the issue was architectural, logical, integration-related, or operational
- Which secrets or privilege boundaries were involved

AI tools were used to accelerate this analysis, but conclusions were checked against the actual code and observed behavior.

### 3. Compare alternatives

When multiple solutions were possible, alternatives were compared before implementation.

Evaluation criteria included:

- Security
- Simplicity
- Maintainability
- Operational reliability
- Cost
- Compatibility with the current architecture
- Risk of regressions
- Ease of production verification

For example, direct browser access to WhatsApp Cloud API was rejected because it would expose a secret token. The secure alternative was to route manual outbound messages through the Cloudflare Worker and validate the Supabase Auth session.

### 4. Implement in controlled scope

Changes were kept as narrow as possible.

The implementation process aimed to:

- Modify only the required components
- Preserve existing behavior outside the target scope
- Avoid unnecessary refactoring during incident resolution
- Keep secrets out of client-side code
- Maintain the separation between browser-safe and privileged operations

AI-generated code was reviewed before it became part of the production system.

### 5. Validate locally and functionally

Changes were tested against the expected workflow.

Validation included:

- Reviewing the code diff
- Checking types and build output
- Running realistic conversation scenarios
- Testing both text and voice inputs when relevant
- Verifying database persistence
- Checking lead extraction and scoring behavior
- Confirming alert delivery
- Testing human takeover
- Confirming that authenticated and privileged paths remained separated

### 6. Test non-ideal behavior

The system was not tested only with ideal inputs.

Additional scenarios included:

- Incomplete customer information
- Ambiguous intent
- Repeated messages
- Requirement changes during a conversation
- Voice messages
- Duplicate webhook delivery
- Delayed responses
- Incorrect assumptions in extracted data
- Premature lead qualification
- Manual intervention during an active automated conversation

This was necessary because production conversational systems receive inconsistent and unpredictable input.

### 7. Verify in production

A successful local implementation was not considered sufficient.

Production verification included:

- Confirming webhook delivery
- Reviewing stored messages and lead records
- Testing WhatsApp responses
- Checking voice transcription
- Confirming property-query results
- Verifying notifications
- Reviewing administration-panel state
- Ensuring manual takeover worked as expected

A change was considered complete only after the operational workflow was verified.

### 8. Document the result

Important changes and architectural decisions were documented.

Documentation focused on:

- What problem was solved
- Why the chosen approach was used
- Which components were affected
- What security constraints applied
- How the change was tested
- What operational risks remained

This reduced dependence on conversational history with AI tools and preserved engineering context for future work.

## Prompting principles

Prompts to coding agents were most effective when they included:

- The exact problem
- Relevant system context
- The expected behavior
- Known constraints
- Security boundaries
- Files or components to inspect
- What should not be changed
- Required verification steps

A useful request was structured around investigation first, implementation second.

For example:

```text
Investigate why manual WhatsApp messages cannot be sent directly from the React panel.

Context:
- The panel uses Supabase Auth and direct browser access to Supabase.
- The WhatsApp token must remain secret.
- The Cloudflare Worker already handles outbound WhatsApp requests.

Before changing code:
1. Trace the current flow.
2. Identify the security boundary.
3. Propose the smallest safe solution.
4. List the files that would change.
5. Explain how the Supabase session should be validated.
6. Do not implement until the plan is reviewed.
```

This approach reduced the risk of broad or poorly understood changes.

## Review criteria for AI-generated changes

AI-generated changes were reviewed against the following questions:

- Does the change solve the actual problem?
- Does it preserve the existing architecture?
- Does it expose any secret?
- Does it weaken Row Level Security or authentication?
- Does it introduce unnecessary complexity?
- Does it change unrelated behavior?
- Are failure cases handled?
- Can the behavior be verified in production?
- Is the implementation understandable enough to maintain later?

If these questions could not be answered clearly, the change was not accepted.

## Examples of rejected approaches

Some categories of proposals were rejected or revised when they conflicted with the system architecture.

Examples include:

- Exposing privileged API tokens to the browser
- Using the Supabase service_role key in frontend code
- Bypassing authentication for manual message endpoints
- Triggering lead alerts from isolated phrases without accumulated context
- Allowing AI-generated text to perform privileged operations directly
- Mixing conversational generation with database authorization logic
- Refactoring broad areas of the codebase during a narrow production fix

The rejection criterion was not whether the code appeared functional. It was whether the approach was safe, maintainable, and consistent with the product.

## Limitations of the AI-assisted workflow

The workflow had important limitations.

AI tools could:

- Misunderstand the current codebase
- Invent APIs or configuration details
- Propose insecure shortcuts
- Produce code that compiled but failed operationally
- Miss edge cases
- Over-refactor
- Confidently describe behavior that had not been verified

For that reason, AI output was treated as untrusted until reviewed and tested.

The workflow depended on:

- Clear problem definition
- Access to the real code and logs
- Human understanding of the architecture
- Manual verification
- Production observation
- Written documentation

## What this workflow demonstrates

This workflow demonstrates:

- Practical use of AI coding agents
- Repository-level investigation
- Architecture review
- Security-aware implementation
- Human oversight of generated code
- Production debugging
- Structured validation
- Documentation discipline
- Responsible use of AI in software engineering

The value of the workflow was not that AI wrote code faster.

The value was that AI increased the speed of investigation and implementation while engineering responsibility remained explicit.
