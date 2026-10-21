---
name: setup-sharetribe-web-template
description: Use when a user wants to install and configure the Sharetribe Web Template for the first time in a local development environment, including cloning the repo, gathering credentials, and running the dev server.
user_invocable: true
---

# Setup Sharetribe Web Template

## Overview

Interactive guide to clone, configure, and start the Sharetribe Web Template locally. Walk the user through each stage in order, asking for credentials at the right moment.

## Process

### Step 0: Check Existing Config

Read `.sharetribe/config.json` if it exists.

If it exists and contains `localSetupComplete: true`:
> "It looks like you've already set up the Sharetribe Web Template locally. Do you want to run setup again, or are you looking to do something else (like set up a GitHub repo or deploy to Render)?"

If the user wants to re-run, continue. Otherwise stop and point them to the relevant skill.

If the file doesn't exist, continue with Step 1.

### Step 1: Prerequisites

Check for the required tools automatically by running these commands on behalf of the user (do not ask the user to run them):

```bash
git --version
node --version
yarn --version
```

For any missing tools, attempt to install them on the user's behalf. Detect the user's OS and use the appropriate method:

- **macOS**: Use Homebrew (`brew install <package>`). If Homebrew is not installed, install it first via `/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"`.
- **Linux**: Use the system package manager (`apt`, `yum`, `dnf`, etc.).
- **Windows**: Use `winget` if available. If not, fall back to telling the user to download installers from the links below.

Install missing tools in this order:

1. If **Git** is missing → install it. Fallback: [git-scm.com/downloads](https://git-scm.com/downloads)
2. If **Node.js** is missing or below version 22 → install Node.js 22.x. Fallback: [nodejs.org](https://nodejs.org/)
3. If **Yarn** is missing → run `npm install -g yarn`

After each installation, re-run the version check to confirm it succeeded. If an automated install fails, link the user to the fallback URL and wait for them to confirm they've installed it manually.

Once all three tools are confirmed present, check whether git is configured with a name and email:

```bash
git config --global user.name
git config --global user.email
```

If either returns blank, ask the user for their name and email, then run:

```bash
git config --global user.name "Their Name"
git config --global user.email "their@email.com"
```

Explain: "Git needs this to label your commits. Without it, saving your changes later will fail."

### Step 2: Clone and Install

Once prerequisites are confirmed, run:

```bash
git clone https://github.com/sharetribe/web-template.git
cd web-template/
yarn install
```

Tell the user this installs all dependencies and may take a minute.

### Step 3: Find Your Credentials

Tell the user:

> "Before we run the setup script, you'll need to find three credentials. You'll enter them directly into the terminal."

Walk the user through finding each one. Don't ask them to share the values with you.

**1. Sharetribe Client ID and Client Secret**
> Go to [console.sharetribe.com](https://console.sharetribe.com) (sign up free if you don't have an account).
> In the top-left corner, make sure you're in the **Dev environment** — not Test or Live.
> Navigate to **Build → Advanced → Applications**.
> Copy the **Client ID** and **Client Secret**. Keep them handy (e.g. open Notepad or a text file).
>
> ⚠️ Using the wrong environment's credentials is the most common reason setup doesn't work. Double-check the top-left switcher says **Dev**.

**2. Stripe Publishable Key (Sandbox)**
> First, connect Stripe to your Sharetribe marketplace by following this guide: [Set up Stripe for a custom marketplace](https://www.sharetribe.com/help/en/articles/8413086-how-to-set-up-stripe-for-payments-on-your-marketplace).
> Once connected, your Sandbox publishable key will be shown in the Console.

Ask: "Do you have all three credentials? Let me know when you're ready and I'll walk you through the next step."

### Step 4: Run Config Script

Once the user confirms they have their credentials ready, tell them to type the following into their terminal (still in the `web-template/` folder). Tell the user how to open a terminal window, if necessary, depending on what operating system they are on.

```bash
yarn run config
```

Explain what will happen:

> "The terminal will ask you three questions, one at a time. Paste each value when prompted and press Enter."
>
> 1. `REACT_APP_SHARETRIBE_SDK_CLIENT_ID` → paste your Sharetribe Client ID
> 2. `SHARETRIBE_SDK_CLIENT_SECRET` → paste your Sharetribe Client Secret
> 3. `REACT_APP_STRIPE_PUBLISHABLE_KEY` → paste your Stripe Sandbox Publishable Key

> "This creates a `.env` file in the project. The app won't start without it."

Ask: "Did the script finish without errors? You should see a message saying the config was saved."

### Step 5: Start the Server

Run the following command on behalf of the user.

```bash
yarn run dev
```

This starts the dev server with hot module replacement. The app opens automatically at `http://localhost:3000`.

### Step 6: Write Config

Create the `.sharetribe/` directory if it doesn't exist, then write (or merge into) `.sharetribe/config.json`:

```json
{
  "localSetupComplete": true
}
```

If the file already exists, merge this value in — do not overwrite other fields.

## Quick Reference

| Command | Purpose |
|---------|---------|
| `yarn install` | Install dependencies |
| `yarn run config` | Set up `.env` with required credentials |
| `yarn run dev` | Start local dev server |

## Summary Commands

```bash
git clone https://github.com/sharetribe/web-template.git
cd web-template/
yarn install
yarn run config
yarn run dev
```
