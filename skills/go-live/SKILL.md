---
name: go-live
description: Use when a user wants to launch their Sharetribe marketplace to production — guides through subscription upgrades, credential setup, production deployment on Render, and establishes a staging-to-production git workflow.
user_invocable: true
---

# Go Live with Your Marketplace

## Overview

Guides the user from a working staging environment to a live production marketplace. The process has two phases: first the user completes a set of manual prerequisites (subscriptions, credentials), then this skill handles the technical setup (git workflow, production config, Render deployment).

## Config File

This skill reads and writes `.sharetribe/config.json` to share state with other Sharetribe skills.

## Process

### Step 0: Read Config

Read `.sharetribe/config.json` if it exists.

If `stagingUrl` or `renderServiceName` are missing, the user has not deployed to staging yet:

> "Before going live, you need a working staging environment. Let's set that up first."

Route them to the `deploy-sharetribe-to-render` skill. Do not proceed.

If `isLive` is already `true`, the user has already gone live:

> "It looks like you've already gone live at `<productionUrl>`. Is everything working, or do you need to reconfigure something?"

If they want to reconfigure, continue through the steps but pre-fill values from existing config. If everything is working, the conversation is done.

### Step 1: Manual Prerequisites

Before any technical setup, the user needs to complete four things in their Sharetribe Console, Stripe, and Render dashboards. Present them as a checklist and do not proceed until the user confirms all four are done.

> "Before I can set up your production environment, you need to do four things. These involve upgrading your accounts and getting your live credentials — I'll walk you through each one.
>
> 1. Upgrade your Sharetribe subscription to the Live plan
>    Go to your Sharetribe Console, then Account, then Subscription. Choose the Live plan.
>
> 2. Create a new Application in your Live environment
>    In your Sharetribe Console, switch to the Live environment (not Dev or Test). Then go to Build, then Advanced, then Applications. Create a new application. Save the Client ID and Client Secret — you will need these later.
>
> 3. Set up Stripe for live payments
>    In your Sharetribe Console under the Live environment, go to Build, then Payments, and connect your live Stripe account. Once connected, get your live publishable key from the Stripe dashboard — it starts with `pk_live_`.
>    Guide: https://www.sharetribe.com/help/en/articles/8413086-how-to-set-up-stripe-for-payments-on-your-marketplace
>
> 4. Upgrade your Render plan
>    Go to your Render Dashboard, then Billing, and make sure you are on a paid plan. The free plan is fine for staging but not for production.
>
> Let me know once you have completed all four steps."

Do not proceed until the user confirms all four are done. If they have questions about any step, help them through it before moving on.

### Step 2: Collect Production Info

Ask the user for their production service name:

> "What would you like to name your production service on Render?"

Suggest `<marketplace-name>-production` using `marketplaceName` from config if available. If the user picks a generic name, append a short random string to ensure uniqueness — the same pattern used in the staging deploy skill. The name determines the initial URL: `https://<service-name>.onrender.com`.

### Step 3: Create Staging Branch

Create a `staging` branch from the current state of `main`. This sets up the workflow where changes go to staging first, then get promoted to production.

First check if the branch already exists (it may if the user is re-running this skill after a partial attempt):

```bash
git branch --list staging
```

If it does not exist, create it:

```bash
git branch staging
git push origin staging
```

If it already exists, skip creation and continue.

Tell the user:

> "I've set up a new workflow for you. From now on, when we make changes to your marketplace, they'll go to your staging site first so you can preview them. When you're happy with a change, I'll help you push it to your live site."

### Step 4: Write `render-production.yaml`

Write a `render-production.yaml` file to the root of the repo. This is the production equivalent of the existing `render.yaml` used for staging.

```yaml
services:
  - type: web
    name: <service-name>
    runtime: node
    buildCommand: yarn install --production=false; yarn build
    startCommand: yarn start
    plan: starter
    envVars:
      - key: NODE_VERSION
        value: 22.22.0
      - key: NODE_ENV
        value: production
      - key: REACT_APP_ENV
        value: production
      - key: REACT_APP_MARKETPLACE_ROOT_URL
        value: https://<service-name>.onrender.com
      - key: REACT_APP_SHARETRIBE_USING_SSL
        value: true
      - key: SERVER_SHARETRIBE_TRUST_PROXY
        value: true
      # Identifies this marketplace as built using sharetribe/skills.
      - key: REACT_APP_BUILT_WITH
        value: sharetribe/skills
      - key: REACT_APP_CSP
        value: block
      - key: REACT_APP_AVAILABILITY_ENABLED
        value: true
      - key: REACT_APP_DEFAULT_SEARCHES_ENABLED
        value: true
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

Substitute `<service-name>` with the name from Step 2.

Tell the user the file has been written and explain that `sync: false` means Render will prompt them to fill those values in the dashboard.

### Step 5: Test SSR Locally

Before deploying, test server-side rendering locally:

```bash
yarn run dev-server
```

> "Does the app load without errors at http://localhost:4000?"

If they hit errors mentioning `window` or browser globals, explain these are SSR errors caused by code that references browser APIs. These need to be fixed before deploying or the production server will crash. Proceed with addressing those problems before continuing with going live.

Only proceed once SSR is confirmed working.

### Step 6: Commit and Push

Commit the production config to `main`, then merge into `staging` so both branches start in sync:

```bash
git add render-production.yaml
git commit -m "Add production deployment config"
git push origin main
git checkout staging
git merge main
git push origin staging
git checkout main
```

### Step 7: Render Dashboard Walkthrough

Walk through two tasks. Ask "Done?" before moving on from each.

#### 7a. Update staging service to track the staging branch

> "Go to your Render Dashboard and find your staging service (`<renderServiceName>`). Go to Settings, then Build, and change the branch from `main` to `staging`. Save the change."

#### 7b. Create production deployment

> "Now we need to create a new deployment for production. In the Render Dashboard, click on "New", then select "Blueprint". Connect it to your GitHub repo. When Render asks for a Blueprint file, look for an option to specify the file path and enter `render-production.yaml`. Set the branch to `main`."

The user still needs to fill in the secret environment variables:

> "You will need to manually enter the following environment variables:
>
> REACT_APP_SHARETRIBE_SDK_CLIENT_ID — your Live Client ID from step 1
> SHARETRIBE_SDK_CLIENT_SECRET — your Live Client Secret from step 1
> REACT_APP_STRIPE_PUBLISHABLE_KEY — your live Stripe key starting with pk_live_ from step 1
> REACT_APP_MARKETPLACE_NAME — the name of your marketplace
>
> Once you have filled these in, click Apply. The first build takes a few minutes — you can watch the logs in the Render dashboard."

### Step 8: Verify

Once the build completes:

> "Visit https://<service-name>.onrender.com — does your marketplace load?"

If it loads, congratulate the user:

> "Your marketplace is live! Congratulations."

If it does not load, the issue is likely the service name. If a non-unique service name was provided, Render will have appended a random string to the name. This means `REACT_APP_MARKETPLACE_ROOT_URL` will not match the actual deployment URL. Guide the user to find the actual URL in the Render Dashboard, update `render-production.yaml` accordingly, and push to git.

### Step 9: Write Config

Write (or merge into) `.sharetribe/config.json`:

```json
{
  "productionServiceName": "<service-name>",
  "productionUrl": "https://<service-name>.onrender.com",
  "isLive": true
}
```

If the file already exists, merge these fields — do not overwrite other fields.

### Step 10: What's Next

> "Here are a couple of things you might want to do next:
>
> Set up a custom domain through the Sharetribe Console — right now your marketplace is at `<productionUrl>`. When you do, make sure to also update the REACT_APP_MARKETPLACE_ROOT_URL environment variable in your Render production service to match your new domain.
>
> From now on, when we work on changes together, they will go to your staging site first. Once you are happy with a change, I will help you push it to your live site."

## Common Issues

If the user is having issues with the deployment, ask them to copy and paste the deploy logs into the chat.

If the production URL does not match because Render appended a random string to the service name, guide the user to find the actual URL in the Render Dashboard, update `render-production.yaml`, and push the change.

## Tone

The user may be non-technical. Use plain language throughout:
- Say "your live site" not "production deployment"
- Say "staging site" not "staging environment"
- Say "push it to your live site" not "merge to main"
- When something goes wrong, explain what happened simply and what you are going to do about it
