# claude-plugins

Claude Code plugins by Leandro Zubrezki.

## Plugins

### spec-interview

A multi-round interview skill that refines a rough `SPEC.md` into a complete, implementation-ready specification, inspired from Thariq's tweet:

<img width="1360" height="592" alt="image" src="https://github.com/user-attachments/assets/ea111427-7f60-419a-9a3d-fec1536ddebf" />

**What it does:**

1. Reads your `SPEC.md` file and explores the relevant codebase
2. Interviews you across multiple rounds, covering edge cases, data modeling, security, performance, UX, and tradeoffs
3. Writes a refined, unambiguous specification
4. Creates an ordered task list with dependencies for implementation
5. After implementation, runs verification, code review, and simplification passes

![2026-02-25 15 30 18](https://github.com/user-attachments/assets/db21fd2f-4237-4cfb-97f0-c9887fa57335)

### idea-storm-x4

A structured brainstorming skill that generates 50 high-quality ideas using 4 parallel agents that debate and converge across 3 rounds.

**What it does:**

1. Launches 4 parallel agents, each generating 50 ideas from a different angle (Growth, Monetization, Community/UX, Content/Data)
2. Runs 2 semifinal debate agents that filter 200 ideas down to 100
3. Runs a final debate agent that merges duplicates and selects the top 50
4. Presents the curated list for you to pick winners

### session-stats

A diagnostic skill that analyzes your Claude Code session data to surface usage patterns and help debug context limit issues.

**What it does:**

1. Parses all session files from `~/.claude/projects/`
2. Reports total tokens, messages, tool uses, and duration across all sessions
3. Ranks projects by token consumption
4. Flags heaviest sessions most likely to hit context limits
5. Lists longest sessions and recent activity
6. Suggests optimizations (when to `/compact`, use subagents, or break sessions)

```
/session-stats
```

## Installation

**Step 1** — Add the marketplace:

```
/plugin marketplace add leandroz/claude-plugins
```

**Step 2** — Install a plugin:

```
/plugin install spec-interview@leandroz
/plugin install idea-storm-x4@leandroz
/plugin install session-stats@leandroz
```

### Local development

```bash
claude --plugin-dir ./claude-plugins
```

## Usage

1. Create a `SPEC.md` file in your project root with your rough idea or draft spec
2. Run the skill:

```
/spec-interview:spec-interview
```

Or just describe what you want to build and Claude will invoke it automatically when it detects a `SPEC.md` in your project.

## Requirements

- Claude Code 1.0.33 or later

## License

MIT
