<a href="https://opendeploy.dev/"><img src="https://oss.opendeploy.dev/static/og-image.png" alt="OpenDeploy — the agent-first deployment platform" /></a>

# OpenDeploy Cursor Plugin

This repository contains the Cursor plugin marketplace entry and plugin source for OpenDeploy.

## What's OpenDeploy?

[**OpenDeploy**](https://opendeploy.dev) is the agent-first deployment platform — *help your agent deploy, host, and scale your app*.

One command from any AI coding agent (Claude Code, Codex, Cursor, OpenClaw, …) takes a project from local source → live URL with **no account required for the first deploy**. We support every framework and language: Next.js, Vite, Astro, Nuxt, SvelteKit, Remix, Express, Fastify, Hono, Django, Flask, FastAPI, Rails, Phoenix, Laravel, Spring, .NET, Go, Rust, Bun, Deno, static sites — anything with a build + run.

What the skill does, end-to-end:

1. **Detects the framework** locally (no source upload yet, no telemetry)
2. **Provisions any database** the app needs — postgres / mysql / mongo / redis
3. **Builds and deploys** to `*.opendeploy.run`
4. **Returns a live URL** plus a one-time **claim URL** the user can sign in to later (via SSO) to adopt the project under their account

The split is intentional: **agents deploy, humans observe**. The agent registers an anonymous token and does the work. The human holds the safety escape hatches (delete, revoke token, set budget cap), and watches the deploy in the dashboard.

---

## Install

During local development, link the plugin into Cursor's local plugins directory:

```sh
mkdir -p ~/.cursor/plugins/local
ln -s /Users/ziyanhe/Desktop/workspace/opendeploy-cursor-plugin/opendeploy ~/.cursor/plugins/local/opendeploy
```

Restart Cursor, then open Agent chat and type:

```text
/opendeploy deploy this project.
```

You can also ask naturally:

```text
Deploy this project with OpenDeploy.
```

After this repository is pushed, install it from Cursor with `/add-plugin` and the repository URL:

```text
https://github.com/opendeploy-dev/opendeploy-cursor-plugin
```

## Structure

- `.cursor-plugin/marketplace.json` - Cursor marketplace manifest.
- `opendeploy/` - OpenDeploy plugin source.
- `opendeploy/.cursor-plugin/plugin.json` - plugin manifest.
- `opendeploy/skills/` - Cursor skills synced from the canonical OpenDeploy skill source.
- `opendeploy/skills/opendeploy/SKILL.md` - canonical OpenDeploy autoplan skill.
- `opendeploy/skills/deploy/SKILL.md` - short alias skill.

## Usage

In Cursor Agent chat, use:

```text
/opendeploy deploy this project.
```

If Cursor lists skills without a leading slash in your build, search for `opendeploy` in the slash command menu.

## Update

Pull the latest repo changes, then restart Cursor or reinstall the plugin from the plugin settings.

## License

MIT
