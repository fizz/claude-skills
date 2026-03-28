# claude-skills

Claude Code skills for platform engineering. Each skill encodes a workflow — not a prompt template, but a complete methodology for how to approach a recurring task.

These grew out of a year of using Claude Code across consulting engagements, personal projects, and production infrastructure. The problem they solve is session amnesia: every new conversation starts from zero unless you encode what you've learned into something the next session can execute.

## Skills

### morning-paper

A briefing that answers: what happened while you were away? Pulls from Slack, Jira, and meeting transcripts, then presents it as one readable summary. Like a newspaper: headlines first, details on demand.

Three editions scaled to the time gap:
- **Morning** — overnight activity, full context
- **Afternoon** — last few hours, just the delta
- **Saturday** — multi-day digest, weighted by recency

Same pipeline, different time window. The newspaper industry figured this out before electricity.

[Blog post: "Newspapers aren't dead. You read one every morning."](https://ferkakta.dev/blog/newspapers-arent-dead/)

### session-start

Full orientation at the start of every session. Checks git state across repos, reads all recent handoffs, queries Jira across multiple projects, checks PRs on both GitHub and Bitbucket, detects new or sloppy GHA workflows, flags infra drift from live patches that weren't baked into terraform, and reports stale local branches from merged PRs.

Eight steps: git state → handoffs → work items → PRs → workflow hygiene → pipeline state → summarize → load briefing skills.

The skill that prevents "loaded wrong plan," "wrong branch," and "missed overnight activity" errors.

## Tool-agnostic by design

The skills describe *what to check and why*, not which binary to use. My setup uses CLI tools (`slackcli`, `jira`, `kubectl`) but yours might use MCP servers for Slack, Atlassian, or Google Calendar. The methodology is the same — the skill tells your agent which channels to scan, which Jira queries to run, and how to prioritize the results. How it reaches the API is your problem.

## Installing

Drop the skill directory into your Claude Code skills path:

```bash
# Copy a single skill
cp -r morning-paper/ ~/.claude/skills/morning-paper/

# Or clone the whole repo
git clone https://github.com/fizz/claude-skills.git ~/.claude/skills-public/
```

Then reference it in your Claude Code configuration or invoke it with `/morning-paper`.

## Design philosophy

These skills follow a pattern I call Ferkakta Protagonist Support — building systems that serve the person doing the work instead of the system doing the tracking. A Jira board is a database. A morning paper is a briefing. The same data, shaped for a different reader.

Every skill starts from the same question: what does the protagonist need to know right now, in what order, at what depth?

## More skills coming

More skills coming. Fork the repo and send PRs — I want to see how other people encode their workflows.

## License

MIT
