# Agent Skills

A collection of skills for AI coding agents. Skills are packaged instructions and scripts that extend agent capabilities.

Skills follow the [Agent Skills](https://agentskills.io/) format.

See [the RFC](https://github.com/emberjs/rfcs/pull/1165) for todos for things that should/could go in this repo

## Available Skills

### ember-best-practices

Ember.js best practices from the Ember.js community, prioritized by impact. Agents load the rules a task needs from `references/`.

**Use when:**

- Writing new Ember components or routes
- Implementing data fetching with WarpDrive
- Reviewing code for performance issues
- Optimizing bundle size or load times
- Implementing accessibility features

**Categories covered:**

- Route Loading and Data Fetching (Critical)
- Build and Bundle Optimization (Critical)
- Component and Reactivity (High)
- Accessibility Best Practices (High)
- Service and State Management (Medium-High)
- Template Optimization (Medium)
- Performance Optimization (Medium)
- Testing Best Practices (Medium)
- Tooling and Configuration (Medium)
- Advanced Patterns (Medium-High)

## Installation

Install the skills with the [skills CLI](https://github.com/vercel-labs/skills):

```bash
pnpm dlx skills add ember-tooling/agent-skills
```

## Usage

Skills are automatically available once installed. The agent will use them when relevant tasks are detected.

**Examples:**

```
Review this Ember component for performance issues
```

```
Help me optimize this Ember.js route
```

## Skill Structure

Each skill contains:

- `SKILL.md` - Instructions for the agent
- `scripts/` - Helper scripts for automation (optional)
- `references/` - Supporting documentation (optional)

## License

MIT
