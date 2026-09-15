# AI Skill Productivity

A collection of reusable, agent-agnostic **AgentSkills** that boost productivity. Each skill is a self-contained `SKILL.md` that teaches a coding agent how to perform a specific task — starting with Jira reporting workflows.

Skills follow the standard `SKILL.md` convention (a `name` + `description` frontmatter plus instructions), so they work across agents that support skills: opencode, Claude Code, and others.

## Available Skills

| Skill | Description | Path |
|-------|-------------|------|
| `jira-issue-report` | Generate Jira issue reports from Atlassian MCP tools with parent story and epic enrichment. | [`skills/jira-issue-report/SKILL.md`](skills/jira-issue-report/SKILL.md) |

## Project Structure

```
.
├── README.md
└── skills
    └── jira-issue-report
        └── SKILL.md
```

Convention: each skill lives in `skills/<skill-name>/SKILL.md`. The `SKILL.md` frontmatter contains a `name` and `description` that agents use to discover and trigger the skill.

## Installation

Skills are installed by copying or symlinking the skill folder into your agent's skills directory. Symlinking keeps the skill in sync with this repo.

### opencode

Global (available in all projects):

```sh
ln -s "$PWD/skills/jira-issue-report" ~/.config/opencode/skills/jira-issue-report
```

Per-project:

```sh
ln -s "$PWD/skills/jira-issue-report" .opencode/skills/jira-issue-report
```

### Claude Code

Global (personal):

```sh
ln -s "$PWD/skills/jira-issue-report" ~/.claude/skills/jira-issue-report
```

Per-project:

```sh
ln -s "$PWD/skills/jira-issue-report" .claude/skills/jira-issue-report
```

> Prefer a copy instead of a symlink? Replace `ln -s` with `cp -R`:
> ```sh
> cp -R skills/jira-issue-report ~/.config/opencode/skills/jira-issue-report
> ```

### Other agents

Other coding agents that support skills use the same `SKILL.md` convention but store skills in different directories. Drop the `skills/<skill-name>/` folder into that agent's skills location (see its documentation for the exact path).

## Usage

Once installed, the agent discovers the skill automatically. For example, ask:

> "Generate a Jira issue report for project NQLA updated this week."

The `jira-issue-report` skill then gathers filters, builds a JQL query, enriches issues with parent story and epic details, and outputs both a Markdown table and CSV file.

## Adding a New Skill

1. Create a folder: `skills/<skill-name>/`.
2. Add a `SKILL.md` with frontmatter:

   ```markdown
   ---
   name: my-skill
   description: "One-line description of what the skill does."
   ---
   ```

3. Write the instructions (when to use, requirements, workflow) in the body.
4. Add a row to the [Available Skills](#available-skills) table.

Keep the skill body agent-agnostic: reference generic tool names and avoid agent-specific syntax so it works everywhere.

## License

No license specified yet.
