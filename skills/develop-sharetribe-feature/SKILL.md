---
name: develop-sharetribe-feature
description: Use when a user wants to add, change, or remove something in their Sharetribe marketplace — a feature, visual change, content update, or behaviour. Guides requirements gathering before any implementation, then commits and pushes to staging.
user_invocable: true
---

# Develop a Sharetribe Feature

## Overview

Guides the user from a vague idea to a deployed change. Always gather requirements first. Ask questions until you understand exactly what needs to be built. Then implement, commit, and push. The user never needs to think about git.

## Process

### Step 0: Read Config

Read `.sharetribe/config.json` if it exists. Note:
- `stagingUrl` — where the user will see their change
- `marketplaceName` — useful context for understanding the product

If config doesn't exist, continue without it (but you'll have less context).

### Step 1: Understand the Request

The user has described something they want to change or add. Before touching any code, ask clarifying questions until you understand:

- **What** exactly should change (text, layout, behaviour, new section, etc.)
- **Where** in the app it should appear (which page, which component)
- **How** it should look or behave (any specific content, colours, interactions)
- **Why** — understanding the goal helps you make better implementation decisions

**Rules for this phase:**
- Ask the user, in multiple choice, interatctive Ask User Questions format (4 at a time), between 3-10 questions (or until you have a clear understanding) that will help them formulate their requirements.
- Prefer concrete options over open-ended questions where possible. ("Do you want this to appear at the top of the page, or at the bottom?")
- If the request involves text content, ask the user to provide the exact wording.
- Keep your language plain — avoid technical terms like "component", "prop", "state".
- If the user gives a vague answer, ask a more specific follow-up rather than guessing.

Continue asking until you are confident you understand what to build. Do not proceed to Step 2 until then.

### Step 1B: Check If This Can Be Done Without Code

Before writing any code, assess whether the request can be handled entirely through the Sharetribe Console. This is extremely important. 

It is important to understand that the codebase contains configuration files. However, they are only there to serve as examples if one would want to override the no-code settings with local configurations. You should never override Console no-code settings with these local configurations, if it is not necessary. Do this only if the user is confident they know what they are doing, or if the feature request is not possible otherwise.

Modifying translation files will be expensive. Instead of loading the entire file into memory, and rewriting it entirely, it is more efficient to batch this operation. Be smart about this.

For the translation tasks, you should always be adamant that these changes should be made in the Console. If a user says "translate my marketplace into another language", you should ALWAYS insist they do this via Console. The codebase has a file via which these edits can be made, but it is an extremely large file, and will cost a lot of tokens to modify, and the primary translations are managed through Console, not the filebase. 

Below is a reference table for you to better understand what exactly can be configured via the Console (no-code):

No-code is possible for:

| Category | What you can change |
|----------|---------------------|
| **Branding** | Primary color, logo, favicon, app icon, login background image, social media image |
| **Content pages** | Landing page (hero, sections, CTAs), Terms of Service, Privacy Policy, About page, custom content pages. If the request relates to either CREATING or MODIFYING a content page, load `guides/content-pages.md` to understand what can be achieved using the no-code page editor. |
| **Marketplace texts** | Button labels, error messages, help text, any UI copy across the site |
| **Email texts** | Content of automatic emails (booking confirmations, quote requests, offers, etc.) |
| **Footer** | Links, social media profiles, copyright text |
| **SEO & social meta** | Page title, description, social sharing image (per page) |
| **Layout** | Configure search page layout, listing page layout, listing thumbnail aspect ratio |
| **Top bar** | Show/hide search bar, show/hide "Post a listing" button, custom links, logo link target |
| **Listing types** | Choose one of four pre-defined transaction modesl (booking, product, negotiaton or free-inquiry) and select booking unit (daily/nightly/hourly) |
| **Listing fields** | Custom fields (dropdowns, multi-select, long text, etc.), display on listing page, search filters |
| **User fields** | Custom signup/profile fields, restricted to specific user types (e.g. providers only) |
| **Listing search** | Keyword vs location search, which filters are enabled |
| **Commission & monetization** | Marketplace commission percentage, transaction minimums |
| **Access control** | Who can publish listings (e.g. verified providers only) |
| **Integrations & automations** | Enter the API key to your Stripe, Google Analytics, or Map provider. You can also use Zapier to create certain no-code automations. |

**If the request is fully achievable via Console:**

Tell the user and guide them to the right place — do not write any code:

> "Good news — you don't need any code changes for this. You can do it directly in your Sharetribe Console under [specific section]. Here's how: [step-by-step instructions]"

Confirm they've made the change, then the conversation is done. Do not proceed to Step 2.

**If it's a mix (partly Console, partly code):**

Handle the Console part first, then continue to Step 2 for the rest:

> "Part of this can be done without code — let's start there. Then I'll make the remaining code changes."

**If it requires code:** Continue to Step 2.

### Step 2: Confirm Understanding

Before writing any code, summarise what you understood in plain, friendly language:

> "Just to make sure I've got this right — here's what I'm planning to do:
> [clear, non-technical description of the change]
> Does that sound right?"

Security flag: If the requested change touches any of the following areas, you MUST warn the user before proceeding:

- Making significant modifications to how pricing works, for example, by integrating an entirely or partially third-party payment system

If a change falls into one of these areas, add the warning to your Step 2 summary. Tell the user to make the following changes it is HIGHLY recommended to work with an Expert Developer. The user must explicitly acknowledge the risk before you proceed to Step 3.

### Step 3: Implement

Now make the code changes. Use your knowledge of the Sharetribe Web Template codebase to find the right files.

Guidelines:
- Make the smallest change that achieves what the user asked for
- Follow existing code style and patterns => the existing patterns are there for a reason.
- Do not refactor or "improve" surrounding code
- Do not add features beyond what was confirmed in Step 2

### Step 4: Review Locally

**For UX changes (any visual, layout, text, or structural change):**

Use Playwright to verify the change yourself before involving the user.

1. Check if the dev server is already running by running `lsof -ti :3000`. If the port is in use, proceed. If not, start it yourself by running `yarn run dev` with `run_in_background: true` from the project directory (the current working directory). Tell the user: "Starting the dev server for you — give it about 30 seconds to come up." Then wait ~30 seconds before proceeding.
2. Tell the user: "Let me take a screenshot to check how it looks..." — then use `mcp__plugin_sharetribe_playwright__browser_navigate` to open the relevant page at `http://localhost:3000` — pick the URL based on what changed (e.g. `/` for homepage, `/s` for search, `/l/<any-listing-slug>` for listing pages, `/profile/<any-user-id>` for profiles).
3. Use `mcp__plugin_sharetribe_playwright__browser_take_screenshot` to capture the page.
4. Inspect the screenshot yourself. Ask: does the change appear as intended based on what was agreed in Step 2?
   - If something looks wrong (missing element, wrong position, broken layout, incorrect text), diagnose it, fix the code, and take a new screenshot. Repeat until the change looks correct.
   - If it looks right, show the screenshot to the user with a brief description of what you verified:
     > "Looks good — [describe what you can see, e.g. 'the new banner is showing at the top of the homepage with the correct text']. Does this match what you had in mind?"

Only escalate to the user once you're satisfied the change is correct, or if you've tried fixing it twice and still can't get it right.

**For non-visual changes (behaviour, logic, flow):**

Ask the user to check manually:

> "I've made the change. Check it at http://localhost:3000 — does it work as expected?"

Before sending this, check if the dev server is running (`lsof -ti :3000`). If not, start it with `yarn run dev` using `run_in_background: true`, then tell the user: "I've started the dev server — give it about 30 seconds, then check http://localhost:3000."

### Step 5: Commit and Push

Once the user is happy, commit and push without explaining git details:

```bash
git add -A
git commit -m "<descriptive message summarising the change in plain language>"
git push origin main
```

Write the commit message in plain language describing the change from a user perspective, not a developer perspective.

Examples:
- "Add contact us section to the homepage"
- "Change the hero banner text to welcome message"
- "Remove the featured listings section from the landing page"

Tell the user simply:
> "Done! Your change is on its way to staging. Render takes about 2–4 minutes to rebuild — you can check it at `<stagingUrl>` once it's ready."

If `stagingUrl` is not in config, tell them to check their Render dashboard for the URL.

## Tone

The user may be non-technical. Use plain language throughout:
- Say "page" not "route"
- Say "the part of the page that shows listings" not "the ListingCard component"
- Say "I'll make that change" not "I'll update the JSX"
- When something goes wrong, explain what happened simply and what you're going to do about it
