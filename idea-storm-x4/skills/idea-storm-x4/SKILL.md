---
name: idea-storm-x4
description: Generate 50 filtered ideas using 4 parallel agents that debate and converge. Use when the user wants to brainstorm, generate ideas, explore possibilities, or needs creative input for a project, product, feature, or strategy. Also triggers for "give me ideas", "brainstorm this", "what could we do", "idea generation", "creative exploration", or any request for a large volume of ideas on a topic.
---

Run a structured brainstorming process in 3 rounds for the topic or project the user specified. This process uses parallel agents to maximize creative diversity, then progressively filters through debate rounds to surface only the strongest ideas.

## Round 1: Generation (4 agents in parallel)

Launch 4 agents in parallel with model sonnet. Each generates exactly 50 numbered ideas from a different angle:

- **Agent 1 (Growth/Viral)**: Growth ideas, virality, SEO, viral loops, social media, gamification, referral mechanics, network effects
- **Agent 2 (Monetization)**: Revenue models, freemium strategies, B2B plays, affiliate programs, partnerships, digital/physical products, pricing experiments
- **Agent 3 (Community/UX)**: User engagement, UGC, social features, notifications, personalization, retention mechanics, onboarding, habit formation
- **Agent 4 (Content/Data)**: Editorial strategy, storytelling, data-driven features, AI integrations, API integrations, content formats, long-tail SEO, automation

Each agent receives context about the user's project/topic and returns exactly 50 numbered ideas, each with a short title and 1-2 lines of description.

## Round 2: Semifinal Debate (2 agents in parallel)

Launch 2 agents in parallel with model sonnet:

- **Debate A**: Receives the 100 ideas from Agents 1+2. Debates internally and selects the top 50, removing duplicates and least viable ideas.
- **Debate B**: Receives the 100 ideas from Agents 3+4. Same process — select the top 50.

Filtering criteria: technical feasibility, real-world impact, originality, good taste, relevance to the target audience.

## Round 3: Final Debate (1 agent)

Launch 1 agent with model sonnet that receives the 100 semifinalist ideas (50 from each debate). It must:

- Remove duplicates and merge similar ideas
- Apply the same filtering criteria
- Return the FINAL list of exactly 50 numbered ideas

## Presentation

Present the 50 final ideas to the user in a clean, numbered format. Tell them to review the list and let you know which ones they want to add to their TODO list or develop further.

$ARGUMENTS
