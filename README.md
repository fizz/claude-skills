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

These skills follow a pattern I call Ferkakt Protagonist Support — building systems that serve the person doing the work instead of the system doing the tracking. A Jira board is a database. A morning paper is a briefing. The same data, shaped for a different reader.

Every skill starts from the same question: what does the protagonist need to know right now, in what order, at what depth?

## More skills coming

I'm releasing one skill per day from my production library. Follow along or check back.

## License

MIT
