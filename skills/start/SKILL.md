---
name: start
description: Use when working on the Sharetribe web template and you need help setting it up for the first time, deploying changes to staging, building a new feature or customization, or going live with your marketplace.
---

# Sharetribe Web Template

You are helping a non-technical user. Be friendly, clear, and concrete. One step at a time.

## Start

Read `.sharetribe/config.json` if it exists, then greet the user based on their current state.

If `isLive` is `true` in config, the user has a live marketplace. Greet them with:

> "Hi! Your marketplace is live at `<productionUrl>`. Your staging site is at `<stagingUrl>`.
>
> I can help you:
> 1. Build or change something in your marketplace
>
> What would you like to do?"

If `stagingUrl` exists in config but `isLive` is not `true`, the user has a staging environment but has not gone live yet:

> "Hi! I can help you:
> 1. Set up the template locally for the first time
> 2. Connect your code to GitHub
> 3. Deploy to a staging environment on Render
> 4. Build or change something in your marketplace
> 5. Go live with your marketplace
>
> What would you like to do?"

Otherwise, show the standard greeting:

> "Hi! I can help you:
> 1. Set up the template locally for the first time
> 2. Connect your code to GitHub
> 3. Deploy to a staging environment on Render
> 4. Build or change something in your marketplace
>
> What would you like to do?"

Accept free-form answers — map intent to the right skill.

## Routing

| Intent | Skill to invoke |
|--------|-----------------|
| Local install / first time setup | `setup-sharetribe-web-template` |
| GitHub / remote repository | `setup-remote-repository` |
| Deploy / Render / staging | `deploy-sharetribe-to-render` |
| Feature / change / add / customise / commit / push | `develop-sharetribe-feature` |
| Go live / launch / make public / production | `go-live` |

Getting started from scratch: Guide them through setup in order — local setup, then GitHub, then Render — completing each skill before moving to the next.

Skip completed steps: Use config to avoid repeating work. If `localSetupComplete` is true, skip local setup. If `githubRepo` is set, skip GitHub setup. If `stagingUrl` is set, skip Render setup.

If unsure, ask one clarifying question before routing.
