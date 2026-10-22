---
name: deploy-sharetribe-to-render
description: Use when a user wants to deploy the Sharetribe Web Template to Render as a staging environment via a Blueprint deployment, including writing render.yaml and walking through the Render dashboard setup.
user_invocable: true
---

# Deploy Sharetribe Web Template to Render

## Overview

Interactive guide to deploy the Sharetribe Web Template to Render as a staging environment. The skill writes `render.yaml` to the repo (pre-filling all non-secret values), then walks the user through each dashboard step with checkpoints.

## Config File

This skill reads and writes `.sharetribe/config.json` to share state with other Sharetribe skills.

## Process

### Step 0: Check Existing Config

Read `.sharetribe/config.json` if it exists.

If it contains `renderServiceName` and `stagingUrl`:
> "It looks like you've already set up a Render deployment: `<stagingUrl>`. Do you want to configure a new deployment, or update the existing one?"

If the user wants to continue anyway, pre-fill the service name and marketplace name from config where available and ask the user to confirm or change each value.

If no config or no `renderServiceName`, continue with Step 1.

### Step 1: Collect Setup Info

Ask the user two questions before touching any files:

**1a. Service name**
> "What do you want to name your Render service?"

If `renderServiceName` was found in config, suggest it as the default. Note that the service name should be unique (if not unique, render appends a random string to the name. This will cause issues with deployment, since we need to match assign the deployment URL as an environment variable). If a user types in a generic name, append a short random string to the name to ensure uniqueness.

Also ask for:
- **Marketplace name** (e.g. "My Marketplace") — pre-fill from config `marketplaceName` if present

**1b. Basic auth**
> "Do you want to password-protect your staging environment? This is recommended — it keeps the site private while you're still developing and prevents search engine indexing."

If yes: ask for a username and password they want to use.

Note the `REACT_APP_ENV` value depends on this:
- Basic auth enabled → `REACT_APP_ENV=production`
- Basic auth disabled → `REACT_APP_ENV=development`

### Step 2: Write render.yaml

Write the following file to the root of the repo, substituting `<service-name>` and `<REACT_APP_ENV_VALUE>` from Step 1:

```yaml
services:
  - type: web
    name: <service-name>
    runtime: node
    buildCommand: yarn install --production=false; yarn build
    startCommand: yarn start
    plan: free
    envVars:
      - key: NODE_VERSION
        value: 22.22.0
      - key: NODE_ENV
        value: production
      - key: REACT_APP_ENV
        value: <REACT_APP_ENV_VALUE>
      - key: REACT_APP_MARKETPLACE_ROOT_URL
        value: https://<service-name>.onrender.com
      - key: REACT_APP_SHARETRIBE_USING_SSL
        value: false
      - key: SERVER_SHARETRIBE_TRUST_PROXY
        value: true
      - key: REACT_APP_CSP
        value: block
      - key: REACT_APP_AVAILABILITY_ENABLED
        value: true
      - key: REACT_APP_DEFAULT_SEARCHES_ENABLED
        value: true
      # Identifies this marketplace as built using sharetribe/skills.
      - key: REACT_APP_BUILT_WITH
        value: sharetribe/skills
      - key: REACT_APP_SHARETRIBE_SDK_CLIENT_ID
        sync: false
      - key: SHARETRIBE_SDK_CLIENT_SECRET
        sync: false
      - key: REACT_APP_STRIPE_PUBLISHABLE_KEY
        sync: false
      - key: REACT_APP_MARKETPLACE_NAME
        sync: false
```

If basic auth was requested, append to `envVars`:
```yaml
      - key: BASIC_AUTH_USERNAME
        sync: false
      - key: BASIC_AUTH_PASSWORD
        sync: false
```

Tell the user the file has been written and explain that `sync: false` means Render will prompt them to fill those values in the dashboard.

### Step 3: Test SSR Locally

Before deploying, test server-side rendering locally, on behalf of the user:

```bash
yarn run dev-server
```

> "Does the app load without errors at http://localhost:4000?"

If they hit errors mentioning `window` or browser globals — explain these are SSR errors caused by code that references browser APIs. They need to fix these before deploying or the production server will crash.

Only proceed once SSR is confirmed working.

### Step 4: Commit and Push

Commit and push `render.yaml` on behalf of the user:

```bash
git add render.yaml
git commit -m "Add Render deployment config"
git push
```

### Step 5: Render Dashboard Walkthrough

Walk through each step, asking "Done?" before moving to the next.

**5a. Create account**
> "Do you have a Render account? If not, sign up at https://dashboard.render.com/register"

**5b. Create new Blueprint**
> "In the Render dashboard, navigate to **Blueprints**, and click on **New Blueprint Instance**. Click on **Configure Git Provider**, and configure the new instance using your GitHub account.
> "Render will detect the `render.yaml` file and pre-fill the service name, build command, and all non-secret settings automatically."

**5c. Add secret environment variables**
Tell the user that Render will show a form for all the `sync: false` variables from `render.yaml`. For each one, tell them where to find the value:

| Variable | Value / Where to find it |
|----------|--------------------------|
| `REACT_APP_SHARETRIBE_SDK_CLIENT_ID` | Sharetribe Console → Build → Advanced → Applications. Use your **Dev environment** client ID for a dev staging app. |
| `SHARETRIBE_SDK_CLIENT_SECRET` | Same location as Client ID |
| `REACT_APP_STRIPE_PUBLISHABLE_KEY` | Stripe dashboard — use the test key starting with `pk_test` |
| `REACT_APP_MARKETPLACE_NAME` | Whatever you want your marketplace to be called |

If basic auth was enabled, also add:
| `BASIC_AUTH_USERNAME` | The username chosen in Step 1 |
| `BASIC_AUTH_PASSWORD` | The password chosen in Step 1 |

**5d. Apply Blueprint**
> "Click 'Apply'. The first build takes a few minutes — you can watch the logs in the Render dashboard."

### Step 6: Write Config

Write (or merge into) `.sharetribe/config.json`:

```json
{
  "renderServiceName": "<service-name>",
  "stagingUrl": "https://<service-name>.onrender.com",
  "basicAuthEnabled": true,
  "marketplaceName": "<marketplace-name>",
}
```

Set `basicAuthEnabled` to `false` if the user chose not to use basic auth.
If the file already exists, merge these fields — do not overwrite other fields (e.g. `githubRepo`).

### Step 7: Verify

Once the build completes:

> "Visit https://<service-name>.onrender.com — does your marketplace load?"

If it does not load, the issue is likely in the service name. If a non-unique service name is provided, render will append a random string to the name. This means that REACT_APP_MARKETPLACE_ROOT_URL must match the unique identifier. You must guide the user to find the deployment url via the Render Dashboard, and update the .yaml file accordingly, and push it to git.

### Common issues

If the user is having issues with the deployment, ask them to copy and paste the deploy logs into the chat.
