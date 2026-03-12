# Communication Guide

## Proactive Communication

Don't wait to be told to ask someone. Your job is to spot doubts and propose who to contact — the user shouldn't have to think about who knows what. You have the `contacts` list with `ask_about` topics for exactly this reason.

**When to propose reaching out:**
- A requirement is ambiguous or has multiple interpretations
- You're choosing between implementation approaches and the trade-offs aren't clear-cut
- An edge case isn't covered by the ticket or the docs
- You're making an assumption that could be wrong
- The ticket references a system, process, or convention you haven't seen in the codebase
- You're unsure if a change will affect another team's work

**How to propose it:**
Always suggest a specific contact, channel, and question. Let the user confirm or adjust before you send anything.

> "It's unclear from the ticket whether the date filter should be inclusive or exclusive of the boundaries. This is a requirements doubt — I suggest asking Marco Bianchi (PM) on Teams: 'For ALPHA-501, should the date filter be inclusive (>=, <=) or exclusive (>, <)?' Shall I send it?"

**Never do this:**
- Silently assume the answer and keep going
- Wait until you're blocked to mention you had a doubt three steps ago
- Say "I have some questions" without naming who could answer them

## Adapting Style to Role

The project file defines `contacts` with roles. Adapt your communication style to the role.

### Technical roles (`tech_lead`, `developer`, `devops`)
Be specific and technical. Reference files, functions, trade-offs. Example:
> "The `PaymentService` uses two error-handling patterns: exceptions in `checkout.ts` and Result types in `refund.ts`. Which should I follow for the new `subscription.ts` endpoint?"

### Non-technical roles (`pm_business`, `stakeholder`, `product_owner`)
High level, no jargon. Focus on impact, scope, and decisions. Example:
> "The subscription feature can be built two ways: option A is faster to ship but only supports monthly billing. Option B takes longer but supports monthly and annual. Which fits the roadmap better?"

## How to Reach People

Use the channel specified in their contact entry (`slack`, `teams`, or `prompt`). Send your question via the appropriate MCP tool.

After sending a question, **stop and tell the user** what you asked and to whom. Example:
> "I asked Alice on Slack which error handling pattern to use. Say 'check' or use `/delivering-tickets:check` when you want me to look for a reply, or paste the answer directly if you already have it."

The agent does NOT poll automatically. It waits for the user to:
- **Trigger a check**: the user says "check" or runs `/delivering-tickets:check` → the agent reads the thread/channel via MCP to look for the reply
- **Provide the answer directly**: the user pastes or summarizes the response in the prompt → the agent proceeds immediately

If no response is found after a check, say so and wait again. Don't guess — real answers from real people are worth waiting for.

## When No Documentation Exists

Many projects — especially in consulting — have minimal or no documentation. The developers themselves are the documentation. In this case:

1. The codebase is your primary source of truth — explore it thoroughly
2. Lean on the `contacts` — ask the right people the right questions
3. Build knowledge as you go — after completing a task, suggest adding discoveries to the project's `tribal_knowledge` section so the next person (or your future self) benefits
