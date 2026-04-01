---
name: session-stats
description: Analyze Claude Code session data to surface usage patterns, heavy sessions, and context limit issues. Run /session-stats to get a full report of your token usage across all projects.
---

Analyze Claude Code session data from `~/.claude/projects/` to give the user a clear picture of their usage patterns.

Run this Python script via Bash to extract session stats. The script dynamically detects the user's home directory and project path prefixes — it works on any machine.

```bash
python3 << 'PYEOF'
import json, os, glob, sys, re
from datetime import datetime, timezone
from collections import defaultdict
from pathlib import Path

home = str(Path.home())
projects_dir = os.path.join(home, ".claude", "projects")

if not os.path.isdir(projects_dir):
    print("No Claude Code session data found at ~/.claude/projects/")
    sys.exit(0)

# Build a regex to strip the home dir prefix from project names
# e.g. "-Users-leandrozubrezki-Desktop-Projects-" -> ""
home_parts = home.strip("/").split("/")
home_prefix = "-" + "-".join(home_parts) + "-"
# Common project subdirs to also strip
project_dirs_pattern = re.compile(
    r"^" + re.escape(home_prefix) + r"(Desktop-Projects-|Projects-|repos-|code-|src-)?",
    re.IGNORECASE
)

sessions = []

for jsonl_path in glob.glob(os.path.join(projects_dir, "**", "*.jsonl"), recursive=True):
    project_dir = os.path.basename(os.path.dirname(jsonl_path))
    # Clean up project name
    project_name = project_dirs_pattern.sub("", project_dir)
    if not project_name or project_name == home_prefix.rstrip("-"):
        project_name = "~"
    session_id = os.path.splitext(os.path.basename(jsonl_path))[0]

    input_tok = 0
    output_tok = 0
    cache_create = 0
    cache_read = 0
    msg_count = 0
    user_msgs = 0
    tool_uses = 0
    models = set()
    first_ts = None
    last_ts = None
    version = None

    try:
        with open(jsonl_path, "r") as f:
            for line in f:
                try:
                    d = json.loads(line)
                    ts = d.get("timestamp")
                    if ts:
                        if isinstance(ts, str):
                            t = datetime.fromisoformat(ts.replace("Z", "+00:00"))
                        elif isinstance(ts, (int, float)):
                            t = datetime.fromtimestamp(
                                ts / 1000 if ts > 1e12 else ts, tz=timezone.utc
                            )
                        else:
                            t = None
                        if t:
                            if not first_ts or t < first_ts:
                                first_ts = t
                            if not last_ts or t > last_ts:
                                last_ts = t

                    if d.get("type") == "assistant":
                        msg = d.get("message", {})
                        u = msg.get("usage", {})
                        if u:
                            input_tok += u.get("input_tokens", 0)
                            output_tok += u.get("output_tokens", 0)
                            cache_create += u.get("cache_creation_input_tokens", 0)
                            cache_read += u.get("cache_read_input_tokens", 0)
                            msg_count += 1
                        model = msg.get("model")
                        if model:
                            models.add(model)
                        v = d.get("version")
                        if v:
                            version = v
                        content = msg.get("content", [])
                        for block in content:
                            if isinstance(block, dict) and block.get("type") == "tool_use":
                                tool_uses += 1

                    if d.get("type") == "user":
                        user_msgs += 1

                except (json.JSONDecodeError, ValueError):
                    continue
    except Exception:
        continue

    if msg_count == 0:
        continue

    duration = (last_ts - first_ts).total_seconds() if first_ts and last_ts else 0
    total_tokens = input_tok + output_tok + cache_create + cache_read

    sessions.append({
        "project": project_name,
        "session_id": session_id[:8],
        "messages": msg_count,
        "user_msgs": user_msgs,
        "tool_uses": tool_uses,
        "input_tokens": input_tok,
        "output_tokens": output_tok,
        "cache_create": cache_create,
        "cache_read": cache_read,
        "total_tokens": total_tokens,
        "models": models,
        "duration_min": duration / 60,
        "first_ts": first_ts,
        "last_ts": last_ts,
        "version": version,
        "file_size": os.path.getsize(jsonl_path),
    })

if not sessions:
    print("No sessions with usage data found.")
    sys.exit(0)

# Sort by most recent
sessions.sort(
    key=lambda s: s["last_ts"] or datetime.min.replace(tzinfo=timezone.utc), reverse=True
)

# Overall stats
total_sessions = len(sessions)
total_all_tokens = sum(s["total_tokens"] for s in sessions)
total_output = sum(s["output_tokens"] for s in sessions)
total_user_msgs = sum(s["user_msgs"] for s in sessions)
total_tool_uses = sum(s["tool_uses"] for s in sessions)
avg_tokens = total_all_tokens / total_sessions
avg_duration = sum(s["duration_min"] for s in sessions) / total_sessions

# Per-project aggregation
by_project = defaultdict(lambda: {"sessions": 0, "tokens": 0, "output": 0, "msgs": 0})
for s in sessions:
    p = by_project[s["project"]]
    p["sessions"] += 1
    p["tokens"] += s["total_tokens"]
    p["output"] += s["output_tokens"]
    p["msgs"] += s["user_msgs"]

print("=" * 60)
print("CLAUDE CODE SESSION REPORT")
print("=" * 60)
print(f"Total sessions:      {total_sessions}")
print(f"Total tokens:        {total_all_tokens:,.0f}")
print(f"Total output tokens: {total_output:,.0f}")
print(f"Total user messages: {total_user_msgs:,.0f}")
print(f"Total tool uses:     {total_tool_uses:,.0f}")
print(f"Avg tokens/session:  {avg_tokens:,.0f}")
print(f"Avg duration:        {avg_duration:.1f} min")
print()

print("-" * 60)
print("TOP PROJECTS BY TOKEN USAGE")
print("-" * 60)
sorted_projects = sorted(by_project.items(), key=lambda x: x[1]["tokens"], reverse=True)
for name, data in sorted_projects[:10]:
    print(
        f"  {name:<40} {data['sessions']:>4} sessions  {data['tokens']:>14,.0f} tokens"
    )
print()

print("-" * 60)
print("HEAVIEST SESSIONS (most likely to hit context limits)")
print("-" * 60)
heavy = sorted(sessions, key=lambda s: s["total_tokens"], reverse=True)[:10]
for s in heavy:
    date_str = s["last_ts"].strftime("%Y-%m-%d") if s["last_ts"] else "unknown"
    print(
        f"  {s['project']:<30} {date_str}  {s['total_tokens']:>14,.0f} tok  {s['messages']:>4} msgs  {s['duration_min']:>6.1f} min"
    )
print()

print("-" * 60)
print("LONGEST SESSIONS BY DURATION")
print("-" * 60)
longest = sorted(sessions, key=lambda s: s["duration_min"], reverse=True)[:10]
for s in longest:
    date_str = s["last_ts"].strftime("%Y-%m-%d") if s["last_ts"] else "unknown"
    print(
        f"  {s['project']:<30} {date_str}  {s['duration_min']:>6.1f} min  {s['messages']:>4} msgs  {s['total_tokens']:>14,.0f} tok"
    )
print()

print("-" * 60)
print("LAST 10 SESSIONS")
print("-" * 60)
for s in sessions[:10]:
    date_str = s["last_ts"].strftime("%Y-%m-%d %H:%M") if s["last_ts"] else "unknown"
    model_str = ", ".join(m for m in s["models"] if not m.startswith("<")) if s["models"] else "?"
    print(
        f"  {s['project']:<30} {date_str}  {s['total_tokens']:>12,.0f} tok  {s['duration_min']:>5.1f} min  {model_str}"
    )

print()
print("=" * 60)
PYEOF
```

After running the script, present the output to the user and provide analysis:

1. **Flag sessions that likely hit context limits** — sessions with very high total tokens (over 100M) or very long durations (over 500 minutes with many messages)
2. **Identify projects that consume the most context** — these may need CLAUDE.md optimization or skill extraction
3. **Note patterns** — are sessions getting heavier over time? Is one project disproportionately expensive?
4. **Suggest optimizations** based on what you see:
   - Heavy sessions (100M+ tokens): suggest breaking work into smaller sessions, using `/clear` at natural boundaries
   - Long sessions (500+ min): suggest using `/compact` proactively before auto-compaction kicks in
   - High cache creation tokens: suggests large CLAUDE.md files or many file reads — recommend keeping CLAUDE.md under 200 lines and using path-scoped rules
   - Many tool uses per session: suggest delegating verbose operations to subagents
   - Sessions over 200 messages: almost always less effective than multiple focused sessions
