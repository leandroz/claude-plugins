# Spec Interview Skill

Inspired from Thariq's tweet

https://x.com/trq212/status/2005315275026260309?s=20

<img width="1360" height="592" alt="image" src="https://github.com/user-attachments/assets/ea111427-7f60-419a-9a3d-fec1536ddebf" />

A Claude Code plugin that conducts a multi-round interview to refine a rough `SPEC.md` into a complete, implementation-ready specification.

## What it does

1. Reads your `SPEC.md` file and explores the relevant codebase
2. Interviews you across multiple rounds, covering edge cases, data modeling, security, performance, UX, and tradeoffs
3. Writes a refined, unambiguous specification
4. Creates an ordered task list with dependencies for implementation
5. After implementation, runs verification, code review, and simplification passes

## Installation

Install from GitHub:

```
/plugin install https://github.com/leandrozubrezki/spec-interview
```

Or load locally for development:

```bash
claude --plugin-dir ./spec-interview
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
- A `SPEC.md` file in your working directory

## License

MIT
