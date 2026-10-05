# GitHub Informer for Zoho Cliq — Setup Guide

---

## 1. Purpose of the workflow

The GitHub Informer workflow connects your GitHub repository to a Zoho Cliq channel. Once it is configured, important repository activity is shared in Cliq automatically, so your team does not need to keep checking GitHub manually.

It can do three things:

1. Post GitHub events to a Cliq channel. Pushes, pull requests, issues, releases, deployments, and many other events can be sent to the channel either individually or through a shared default message.
2. Keep pull request updates in one Cliq thread (optional). Instead of creating separate messages for every PR update, the workflow can continue the same conversation thread for that pull request. This requires saving the thread message ID between workflow runs, and a GitHub Project V2 field is used for that.
3. Run an AI review check on pull requests (optional). The action sends the PR diff to OpenAI, Claude, or Gemini and reports the result as a GitHub status check. If this check is marked as required, a failed review can block the merge.

The threading and AI review features are optional and independent. You can use notifications alone, notifications with threading, notifications with AI review, or all three together.

---

## 2. Configuration flow

To configure this workflow correctly, follow these three steps in order:

**Step 1:** Set the environment secrets and repository variables
**Step 2:** Create the workflow and run it
**Step 3:** Set branch rules and required status checks, then validate with a PR

These are the variables we need to set

## GitHub → Settings → Environments → `cliq-production` → Secrets

## Environment secrets 

| Link | Secret name | Required | Allowed value | 
| --- | --- | --- | --- |
| [3.1](https://github.com/Integrations-dev/RepositoryTemplate/blob/main/README.md#31-endpoint---the-channel-endpoint-url) | `ENDPOINT` | Yes | `https://cliq.zoho.in/api/v2/channelsbyname/githubreponotification/message?zapikey=1001.xxxxxxxx` |
| [3.2](https://github.com/Integrations-dev/RepositoryTemplate/blob/main/README.md#32-creating-the-classic-pat-for-project_token) | `PROJECT_TOKEN` | Only if `CLIQ_THREAD_STORAGE_MODE=project` | `XXX_S7Up0fXXXXXXXX` | 
| [3.3](https://github.com/Integrations-dev/RepositoryTemplate/blob/main/README.md#33-ai_review_token-for-pr-ai-review-gate) | `AI_REVIEW_TOKEN` | Only if `AI_REVIEW_ENABLED=true` | Provider API key for the selected AI service. | 

## Repository Settings → Secrets and variables → Actions → Variables.

## Environment variables 

| Link | Variable name | Required | Allowed values | 
| --- | --- | --- | --- |
| [4.1](https://github.com/Integrations-dev/RepositoryTemplate/blob/main/README.md#41-decide-post-as-a-user-or-as-a-bot) |`CLIQ_NOTIFICATION_MODE` | Yes | `user` or `bot` |
| [4.2](https://github.com/Integrations-dev/RepositoryTemplate/blob/main/README.md#42-if-posting-as-a-user) | `CLIQ_USER_MODE_BOT_DISPLAY_NAME` | Optional in user mode | Any string, for example `GitHub Updates` |
| [4.2](https://github.com/Integrations-dev/RepositoryTemplate/blob/main/README.md#42-if-posting-as-a-user) | `CLIQ_USER_MODE_BOT_IMAGE_URL` | Optional in user mode | Valid image URL |
| [4.3](https://github.com/Integrations-dev/RepositoryTemplate/blob/main/README.md#43-if-posting-as-a-bot--getting-the-bot-unique-name) |`CLIQ_BOT_UNIQUE_NAME` | Only if mode is `bot` | `githubnotificationbot`. Lower-case, no spaces. |
| [4.4](https://github.com/Integrations-dev/RepositoryTemplate/blob/main/README.md#44-if-posting-pr-updates-into-a-thread) |`CLIQ_THREAD_STORAGE_MODE` | Yes | `project` for per-PR threads |
| [4.4.a](https://github.com/Integrations-dev/RepositoryTemplate/blob/main/README.md#44a-steps-to-create-the-project-and-custom-field) |`PROJECT_NUMBER` | Only if thread mode is `project` | Integer value from the project URL, for example `7`. |
| [4.4.a](https://github.com/Integrations-dev/RepositoryTemplate/blob/main/README.md#44a-steps-to-create-the-project-and-custom-field) |`PROJECT_THREAD_FIELD_ID` | Only if thread mode is `project` | Numeric field ID from the field settings URL `401236883` |
| [4.5](https://github.com/Integrations-dev/RepositoryTemplate/blob/main/README.md#45-ai-provider-settings) |`AI_REVIEW_ENABLED` | Yes | `true` or `false` |
| [4.5](https://github.com/Integrations-dev/RepositoryTemplate/blob/main/README.md#45-ai-provider-settings) |`AI_REVIEW_SERVICE` | Yes | `openai`, `claude`, or `gemini` |
| [4.5](https://github.com/Integrations-dev/RepositoryTemplate/blob/main/README.md#45-ai-provider-settings) |`AI_REVIEW_MODEL` | Yes | A model ID that belongs to the chosen service. See the table in section [4.5.a](https://github.com/Integrations-dev/RepositoryTemplate/blob/main/README.md#45a-if-ai_review_enabledtrue) |


## 3. How to configure environment secrets

Go to your repository or organization and configure the required values.

If no environment exists, create a new environment named `cliq-production` first. Then open:

#### GitHub → Settings → Environments → `cliq-production` → Secrets

Add the required secrets under this environment.

### 3.1 ENDPOINT - The channel endpoint URL

All notifications are sent to a single URL: the channel endpoint. Its format is:

`<region-base>/api/v2/channelsbyname/<CHANNEL_UNIQUE_NAME>/message?zapikey=<WEBHOOK_TOKEN>`

Three parts need to be filled in:

#### 3.1.a. Region base

This must match the data centre where your Cliq organisation is hosted. The wrong domain fails quietly and no message is posted.

| Region | Base URL |
| --- | --- |
| US | `https://cliq.zoho.com` |
| IN | `https://cliq.zoho.in` |
| EU | `https://cliq.zoho.eu` |
| AU | `https://cliq.zoho.com.au` |
| JP | `https://cliq.zoho.jp` |

If you are unsure, check the address bar in your browser while using Cliq.

#### 3.1.b. Channel unique name

This is not the display name. In Cliq, open the channel and go to **Channel Actions** or **Info**. The unique name is shown there.

Look for the value labeled **Unique Name** in the channel details panel. This is the exact value used in the URL after `/channelsbyname/`.

`Example Unique Name: githubreponotification`

**So the endpoint becomes:**

`https://cliq.zoho.com/api/v2/channelsbyname/githubreponotification/message?zapikey=1001.xxxxxxxx`

The part after `/channelsbyname/` must match the channel's unique name exactly.

#### 3.1.c. Webhook token

This is generated from your Zoho Cliq profile and must be created before you assemble the channel endpoint URL.

**Follow these steps:**

1. Click on your **profile picture** located in the top-right corner of the screen.
2. Select **Bots & Tools** from the dropdown menu.
3. Look at the left sidebar menu under the *Integrations* section and click on **Webhook Tokens**.
4. Complete the two-factor authentication (2FA) identity verification if prompted by the platform.
5. Click the **Generate New Token** button.
6. Give your token a recognizable name, then click create to view your unique hex string token.

After the token is created, copy the generated webhook token and combine it with the region base and channel unique name to build the final endpoint.

**Assembled example:**

`https://cliq.zoho.in/api/v2/channelsbyname/githubreponotification/message?zapikey=1001.xxxxxxxx`

This complete string, including the `?zapikey=` part, is what you store as the `ENDPOINT` secret.



### Required value for this step

| Variable Type | Name | Allowed Value |
| --- | --- | --- |
| Environment Secrets | `ENDPOINT` | `https://cliq.zoho.in/api/v2/channelsbyname/githubreponotification/message?zapikey=1001.xxxxxxxx` |


> Treat this value as a credential. Anyone who has it can post to the channel.


### 3.2 Creating the classic PAT for `PROJECT_TOKEN`

1. Go to **GitHub → Settings → Developer settings → Personal access tokens → Tokens (classic)**. [Link](https://github.com/settings/tokens)
2. Click **Generate new token (classic)**.
3. Set an **expiry date**. Choose a short duration and plan to rotate it before expiry. Do not create a non-expiring token.
4. Select the scopes: **`repo`** and **`project`**.
5. Copy the token and save it as the `PROJECT_TOKEN` environment secret.


### Required value for this step

| Variable Type | Name | Allowed Value |
| --- | --- | --- |
| Environment Secrets | `PROJECT_TOKEN` | `XXX_S7Up0fXXXXXXXX` |


> Treat this value as a credential. Anyone who has it can access your project.


### 3.3 AI_REVIEW_TOKEN for PR AI Review Gate

The workflow uses this token to authenticate the AI API request. Without it, the action cannot fetch the model output, cannot create the review result, and cannot post the status or PR comment.

#### 3.3.a Generate the API token

| Service | Token generation URL |
| --- | --- |
| OpenAI | https://platform.openai.com/api-keys |
| Claude | https://console.anthropic.com/settings/keys |
| Gemini | https://aistudio.google.com/app/apikey |


### Required value for this step

| Variable Type | Name | Allowed Value |
| --- | --- | --- |
| Environment Secrets | `AI_REVIEW_TOKEN` | `sk- / sk-ant / AIza` |


> Treat this value as a credential. Anyone who has it can access your AI service.


## 4. How to configure environment variables

### **Where these go:** Repository **Settings → Secrets and variables → Actions → Variables**.

These are repository variables, not environment secrets, and they are not environment-scoped.

### 4.1 Decide: post as a user or as a bot?

Choose the mode based on who should appear as the sender of the notification:

| Mode | Who the message appears to come from | Extra setup |
| --- | --- | --- |
| User (webhook) mode | The person who created the webhook | No bot setup needed |
| Bot mode | A dedicated Cliq bot | Create the bot and add it to the channel |

Use this setting in the GitHub repository variables:

### Required value for this step

| Variable Type | Variable Name | Allowed Values |
| --- | --- | --- |
| Repository Variables | `CLIQ_NOTIFICATION_MODE` | `user` or `bot` |
| Repository Variables | `CLIQ_USER_MODE_BOT_DISPLAY_NAME` | Optional string override for user mode | 


Use user mode when you want the fastest setup and do not mind the message appearing to come from the webhook creator. Use bot mode when you want a stable, shared sender for ongoing notifications, especially if the person creating the webhook leaves the team or the channel.

Both modes still use the same `ENDPOINT` secret. The only extra value needed in bot mode is `CLIQ_BOT_UNIQUE_NAME`.

### 4.2 If posting as a user

In user mode, there is no extra configuration beyond the channel endpoint itself.

The Cliq channel message is posted as the authenticated user, and the default display name and image are used unless you override them with repository variables.

Optional overrides for user mode:

| Variable Type | Variable Name | Allowed Values | Notes |
| --- | --- | --- | --- |
| Repository Variables | `CLIQ_USER_MODE_BOT_DISPLAY_NAME` | Any string | Optional override for the display name in user mode |
| Repository Variables | `CLIQ_USER_MODE_BOT_IMAGE_URL` | Valid image URL | Optional override for the thumbnail in user mode |

Fallback order in user mode:

1. If both values are provided, use the user-provided display name and image URL.
2. If only the display name is provided, use it and fall back to the default image URL.
3. If only the image URL is provided, use the default display name and the provided image URL.
4. If both are omitted, use the default display name and default image URL.

Default user-mode values:

- Display name: `GitHub Informer for Zoho Cliq`
- Image URL: the built-in default WorkDrive image URL used by the workflow

### 4.3 If posting as a bot — getting the bot unique name

1. Open Zoho Cliq.
2. Click on your profile picture in the top-right corner.
3. Select **Bots & Tools**.
4. In the left sidebar, open **Integrations** and click **Bots**.
5. Select the bot you created or want to use.
6. While creating the bot, **enable the required channel permissions** so the bot can post messages in the target channel.
7. Open the bot details panel and look for the **API Endpoint** value.
8. The value after `/bots/` is the bot unique name.

**Example:**

`https://cliq.zoho.com/api/v2/bots/githubnotificationbot/message`

**The bot unique name is:**

`githubnotificationbot`

This is not necessarily the display name. It is the unique identifier that must be added to `CLIQ_BOT_UNIQUE_NAME`.

9. Add the bot to the target channel. This is the most common bot-mode failure.
10. Set the variable:

### Required value for this step

| Variable Type | Variable Name | Allowed Values |
| --- | --- | --- |
| Repository Variables | `CLIQ_BOT_UNIQUE_NAME` | `githubnotificationbot` |


### 4.4 If posting PR updates into a thread

Skip this section if you are fine with each pull request event being posted as a separate message in the channel.

If you want all PR updates to continue in the same Cliq thread, set:

`CLIQ_THREAD_STORAGE_MODE=project`

Why this is needed:

Each workflow run starts fresh and does not remember the previous message automatically. To continue replying in the same Cliq thread, the action must store the message ID from an earlier run and reuse it later. This is done by saving the thread ID in a custom text field on a GitHub Project V2 item.

#### 4.4.a Steps to create the project and custom field

1. Create or select a GitHub Project V2. Note the project number.
   - Example: `https://github.com/orgs/<org-name>/projects/1` → `1` is the `PROJECT_NUMBER`.
2. Make sure the project owner and the workflow owner are the same.
3. Add a custom field of type `Text` to the project, for example `Cliq Thread ID`.
4. Get the field identifier. Either of these values is accepted:
   - the numeric ID visible in the field settings URL.
     - Example: `https://github.com/orgs/<org-name>/projects/1/settings/fields/401236883` → `401236883` is the `PROJECT_THREAD_FIELD_ID`.
   - the GraphQL node ID starting with `PVTF_`.
5. Ensure pull requests are added to the project.

### Required values for this step

| Variable Type | Variable Name | Allowed Values |
| --- | --- | --- |
| Repository Variables | `CLIQ_THREAD_STORAGE_MODE` | `project` |
| Repository Variables | `PROJECT_NUMBER` | `1` |
| Repository Variables | `PROJECT_THREAD_FIELD_ID` | `401236883` |

> When `CLIQ_THREAD_STORAGE_MODE=project`, also set `PROJECT_NUMBER` and `PROJECT_THREAD_FIELD_ID`.


## 4.5 AI Provider settings

This section defines which AI provider the workflow should use when the AI review feature is enabled. The provider must match the token you created and the model you select; otherwise the review request will fail.

#### 4.5.a If `AI_REVIEW_ENABLED=true`

When this variable is set to `true`, the workflow triggers the AI review check for the pull request. You must also configure both `AI_REVIEW_SERVICE` and `AI_REVIEW_MODEL`.

| Provider | `AI_REVIEW_SERVICE` | `AI_REVIEW_MODEL` | `Reference` |
| --- | --- | --- | --- |
| OpenAI | `openai` | `gpt-4.1-mini / gpt-4.1 / gpt-4.1-nano` | [OpenAI](https://developers.openai.com/api/docs/models) |
| Claude | `claude` | `claude-sonnet-5 / claude-opus-4-1 / claude-haiku-4-5` | [Claude](https://platform.claude.com/docs/en/models/overview) |
| Gemini | `gemini` | `gemini-2.5-flash / gemini-2.5-pro / gemini-2.5-flash-lite` | [Gemini](https://ai.google.dev/gemini-api/docs/models) |

> Current Claude model IDs are dateless, such as `claude-opus-5`, `claude-sonnet-5`, and `claude-haiku-4-5`. Legacy aliases like `claude-3-5-sonnet-latest` should not be used for new setups.

#### 4.5.b If `AI_REVIEW_ENABLED=false`

When this variable is set to `false`, the workflow exits the AI review path before any diff fetch, AI API call, GitHub check creation, or PR comment is attempted. In practice, this means:

- no diff is fetched
- no AI API request is sent
- no AI review GitHub status check is created
- no AI review PR comment is posted
- no AI review gate blocks the merge

This is the intended "feature off" mode. If you want to disable AI review completely, set the variable to `false` and remove the AI Review Gate from the required status checks in GitHub branch protection or rulesets. Otherwise, GitHub can still block the merge even though the workflow is not running the AI review logic.

### Required values for this step

| Variable Type | Variable Name | Allowed Values |
| --- | --- | --- |
| Repository Variables | `AI_REVIEW_ENABLED` | `true / false` |
| Repository Variables | `AI_REVIEW_SERVICE` | `openai / claude / gemini` |
| Repository Variables | `AI_REVIEW_MODEL` | `gpt-4.1-mini / claude-sonnet-5 / gemini-2.5-flash` |


## 5. Create the workflow file

Use the GitHub Actions UI to create the workflow instead of creating files manually in the repository.

1. Open the target repository on GitHub.
2. Go to **Actions**.
3. Click **New workflow**.
4. Choose **Set up a workflow yourself**.
5. Name the workflow file as `CliqConnector.yml`.
6. Copy the workflow content from the template [repository](https://github.com/Integrations-dev/RepositoryTemplate)
7. Click **Commit changes...** to save the workflow.
8. Commit the changes to the branch you are using. Once the workflow is committed, GitHub Actions will automatically trigger the workflow run.

Template repository: [https://github.com/Integrations-dev/RepositoryTemplate](https://github.com/Integrations-dev/RepositoryTemplate)


## 6. Now branch protection

Order matters here. GitHub cannot require a status check that has never been reported, so the workflow must run once before the branch rule can reference it.

1. Open or update a pull request so the workflow fires.
2. Check the **Actions** tab for the run result and confirm the message arrived in the expected Cliq channel.
3. Only after that, go to **Settings → Rules** (or **Branches → branch protection rule**).
4. Enable **Require status checks to pass**.
5. Add the check name exactly as set in `ai-review-check-name` — default is **`AI Review Gate`**.

Notes:

- Name matching is literal. Any mismatch leaves pull requests stuck on **Waiting for status to be reported**. After renaming the check, push a commit or re-run checks once so the new context is registered.
- Protecting all branches does not by itself make AI review block anything. The check only blocks merges where it is listed as a required status check.
- Recommended approach: protect only the release branches (`main`, `master`, or `release`) and keep AI review as PR-level validation elsewhere.
- In practice, add the pre-live branches that should gate deployments before code moves forward, such as `main`, `master`, and `release`, to the branch protection rule so the workflow runs and enforces the check before merge.



## Conclusion

This workflow is ready to use once the required secrets and repository variables are configured, the workflow is created from the GitHub Actions UI, and the branch protection rule is enabled for the AI review check. After that, your repository can automatically send notifications to Cliq, keep PR updates in a single thread when enabled, and enforce AI-based review checks before merge when configured.