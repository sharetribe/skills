---
name: setup-remote-repository
description: Use when a user wants to connect their local Sharetribe Web Template to a new GitHub repository, set up remote tracking, and push their code for the first time.
user_invocable: true
---

# Setup Sharetribe Git Remote

## Overview

Guides the user through creating a GitHub repository and connecting it to their local Sharetribe Web Template clone. Keeps the original Sharetribe repo as `upstream` so the user can pull future updates.

## Config File

This skill reads and writes `.sharetribe/config.json` to share state with other Sharetribe skills.

## Process

### Step 0: Check Existing Config

Read `.sharetribe/config.json` if it exists.

If it contains `githubRepo`:
> "It looks like you've already connected a GitHub repository: `<githubRepo>`. Do you want to set up a new one, or is this already correct?"

If correct, stop. If updating, continue with Step 1.

If no config or no `githubRepo`, continue with Step 1.

### Step 1: Create a GitHub Repository

Ask: "Have you created a new GitHub repository yet?"

If not, direct them to: [Create a GitHub repository](https://help.github.com/en/github/getting-started-with-github/create-a-repo)

**Important:** Tell the user — **do not initialize the repo with anything** (no README, .gitignore, or license). They are importing an existing repository.

Once created, ask the user for:
- Their **GitHub username**
- The **repository name** they chose

### Step 2: Check Current Remotes

Run to verify the starting state:

```bash
git remote -v
```

Expected output (origin pointing to Sharetribe's repo):

```
origin  https://github.com/sharetribe/web-template.git (fetch)
origin  https://github.com/sharetribe/web-template.git (push)
```

If the output doesn't match, tell the user and ask them to confirm they're in the `web-template` directory before continuing.

### Step 3: Rename origin to upstream

Rename the Sharetribe remote from `origin` to `upstream` so you can pull future updates from it:

```bash
git remote rename origin upstream
```

Verify:

```bash
git remote -v
```

Expected output:

```
upstream  https://github.com/sharetribe/web-template.git (fetch)
upstream  https://github.com/sharetribe/web-template.git (push)
```

### Step 4: Add Your Repository as origin

Using the GitHub username and repo name collected in Step 1:

```bash
git remote add origin https://github.com/<username>/<repo-name>.git
```

Verify:

```bash
git remote -v
```

Expected output:

```
origin    https://github.com/<username>/<repo-name>.git (fetch)
origin    https://github.com/<username>/<repo-name>.git (push)
upstream  https://github.com/sharetribe/web-template.git (fetch)
upstream  https://github.com/sharetribe/web-template.git (push)
```

### Step 5: Push to Remote

Push the code to the new repository:

```bash
git push -u origin main
```

The `-u` flag sets `origin/main` as the default upstream branch for future pushes.

Confirm with the user that the push succeeded and their repo is now live on GitHub.

### Step 6: Write Config

Write (or merge into) `.sharetribe/config.json` with the values collected in this skill:

```json
{
  "githubUsername": "<username>",
  "githubRepo": "<username>/<repo-name>",
  "githubRepoUrl": "https://github.com/<username>/<repo-name>.git"
}
```

If the file already exists, merge these fields — do not overwrite other fields.
