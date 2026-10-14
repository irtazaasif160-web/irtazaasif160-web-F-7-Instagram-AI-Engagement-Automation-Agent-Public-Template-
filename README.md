# InstaEngage AI — Instagram DM, Comments & Human Handoff Automation

**An AI-powered Instagram customer engagement system built using n8n, DeepSeek AI, Instagram Graph API, Google Sheets, and Gmail.**

InstaEngage AI automatically responds to Instagram direct messages, manages conversation memory, replies to comments, and transfers conversations to a human when necessary.

This repository contains reusable n8n workflows, Google Sheets templates, configuration instructions, and documentation.

> **Important:** This is a workflow template, not a ready-to-use hosted application. You must configure your own API credentials, Instagram account, Google Sheets, and Meta webhook settings.

## 1. Features

- **AI Instagram DM Replies:** Automatically respond to incoming text messages using DeepSeek.
- **Conversation Memory:** Maintain customer-specific history in Google Sheets, using up to 10 recent messages as AI context.
- **Knowledge-Based Answers:** Read approved business projects from Google Sheets to generate relevant responses.
- **Instagram Comment Automation:** Automatically send public replies and, where configured and permitted, private replies.
- **Duplicate Prevention:** Avoid processing the same message or comment repeatedly.
- **Human Handoff:** Pause AI responses for customers who need human assistance.
- **Developer Commands:** Accept, decline, and close handoff requests directly through Instagram.
- **Google Sheets Logging:** Store messages, comments, handoff requests, and failed operations.
- **Error Monitoring:** Log workflow failures and send administrator email alerts.

## 2. Technologies

| Technology | Purpose |
|---|---|
| n8n | Automation engine |
| DeepSeek AI | AI-generated responses |
| Instagram API | Receive and send Instagram messages and comment replies |
| Google Sheets | Database and conversation memory |
| Gmail | Error notifications |
| GitHub Pages | Public privacy policy hosting |

## 3. Workflow Architecture

```text
Instagram Webhook
       |
       v
Verify Webhook Signature
       |
       v
Identify Event Type
       |
       +-- Incoming DM
       |      |
       |      +-- Check Duplicates
       |      +-- Check Human Handoff
       |      +-- Load Customer Memory
       |      +-- Load Approved Projects
       |      +-- DeepSeek AI
       |      +-- Send Instagram Reply
       |      +-- Save Logs and Memory
       |
       +-- Instagram Comment
       |      |
       |      +-- Check Comment Rules
       |      +-- Check Duplicates
       |      +-- Send Public Reply
       |      +-- Optional Private Reply
       |      +-- Save Comment Log
       |
       +-- Developer Command
              |
              +-- View Handoff Requests
              +-- Accept Request
              +-- Decline Request
              +-- Close Human Chat

Separate Error Workflow
       |
       +-- Error Trigger
       +-- Process Error
       +-- Google Sheets Error Log
       +-- Gmail Alert
```

## 4. Requirements

Before installation, prepare:

1. A running n8n instance with a public HTTPS address.
2. An Instagram Professional account (Business or Creator).
3. A Meta Developer account.
4. A Meta application configured for Instagram API access.
5. A DeepSeek API key.
6. A Google account with access to Google Sheets.
7. A Gmail account for optional error notifications.
8. A public privacy policy URL.

## 5. Google Sheets Database Setup

Download the clean spreadsheet templates from the `google-sheets` directory.

Upload them to Google Drive, open them using Google Sheets, and connect them to n8n.

The sample data below is fictional. Do not publish actual customer conversations.

### Sheet 1 — DM_LOGS

Stores incoming Instagram messages and reply outcomes.

| sender_id | message_id | message_text | received_at | status | ai_reply | replied_at |
|---|---|---|---|---|---|---|
| DEMO_USER_001 | DEMO_MSG_001 | Do you provide automation services? | 14-10-2026 10:00 | Replied | Yes, we provide automation solutions. | 14-10-2026 10:01 |

### Sheet 2 — DM_Memory

Stores customer-specific conversation history.

| sender_id | role | message | created_at |
|---|---|---|---|
| DEMO_USER_001 | user | Tell me about your services | 14-10-2026 10:00 |
| DEMO_USER_001 | assistant | We provide AI automation solutions. | 14-10-2026 10:01 |

The AI uses recent conversation history to maintain context.

### Sheet 3 — Failed_DMs

Records unsuccessful Instagram message deliveries.

| message_id | sender_id | ai_reply | error_message | failed_at | status |
|---|---|---|---|---|---|
| DEMO_MSG_002 | DEMO_USER_002 | Example response | API request failed | 14-10-2026 10:05 | Failed |

### Sheet 4 — Comment_logs

Stores keyword rules and media mappings.

| keyword | post_url | media_id | public_reply | private_dm | status |
|---|---|---|---|---|---|
| details | https://www.instagram.com/p/EXAMPLE/ | DEMO_MEDIA_ID | Thanks for your interest! | Here are the details. | Active |

### Sheet 5 — Comment_Logs

Stores comment-processing results.

| comment_id | commentor_id | username | comment_text | media_id | created_at | public_reply_status | private_dm_status |
|---|---|---|---|---|---|---|---|
| DEMO_COMMENT_001 | DEMO_USER_003 | sample_user | Details please | DEMO_MEDIA_ID | 14-10-2026 10:10 | Replied | Pending |

**Important:** `Comment_logs` and `Comment_Logs` are different tabs in the exported workflow. Map each node to its intended tab.

### Sheet 6 — Human_Handoff

Stores human-support requests.

| request_id | sender_id | mode | handoff_status | reason(last_message) | conversation_memory | requested_at | updated_at |
|---|---|---|---|---|---|---|---|
| HR-001 | DEMO_USER_004 | human | pending | Wants human assistance | Asked about services | 14-10-2026 10:15 | 14-10-2026 10:15 |

When human mode is active, AI replies must stop for that sender.

### Separate Projects Spreadsheet — Sheet1

Stores approved projects the AI may discuss.

| project_id | project_name | category | description | technologies_used | price | instagram_video | status(For F-7) |
|---|---|---|---|---|---|---|---|
| P-001 | AI Support Assistant | Customer support | Automatically responds to customer questions | n8n, DeepSeek, Google Sheets | Contact for quote | https://www.instagram.com/ | Approved |

Only projects marked `Approved` should be treated as approved portfolio entries.

### Separate Error Spreadsheet — Log Error

| workflow_name | workflow_id | execution_id | execution_url | failed_node | node_type | error_message | error_type | error_category | status_code | timestamp |
|---|---|---|---|---|---|---|---|---|---|---|---|
| Demo Workflow | WF-001 | EX-001 | https://example.com/execution | HTTP Request | HTTP Request | Unauthorized | HttpError | AUTH_ERROR | 401 | 14-10-2026 10:20 |

**Important:** Keep the exact column headers expected by the n8n nodes. After importing the workflow, select the correct spreadsheet and tab in every Google Sheets node and refresh the column mappings.

## 6. Create Instagram API Credentials

### Step 1 — Prepare Instagram

1. Open Instagram.
2. Switch to a Professional account if necessary.
3. Confirm that your account is eligible for Instagram API integration.

### Step 2 — Create a Meta Developer App

1. Visit https://developers.facebook.com/apps/
2. Click **Create App**.
3. Select the appropriate Instagram API use case.
4. Configure **Instagram API with Instagram Login**, where available.
5. Connect and authorize your Instagram Professional account.

The exported workflow uses `graph.instagram.com` and is designed around Instagram Login. Do not mix its credentials with the separate Facebook Login integration.

### Step 3 — Configure Permissions

Request the permissions required by your workflow, generally:

- `instagram_business_basic`
- `instagram_business_manage_messages`
- `instagram_business_manage_comments`

Additional approval may be required to serve Instagram accounts outside your app's authorized roles or testers.

Switching the app to Live mode does not automatically grant every permission.

### Step 4 — Generate an Instagram Access Token

Use the token-generation or authorization flow provided in your Meta Instagram API configuration.

Generate an authorized Instagram user access token and keep it private.

### Step 5 — Create the n8n Credential

Open an Instagram HTTP Request node in n8n.

Select:

**Authentication → Generic Credential Type → Header Auth**

Enter:

```text
Name:
Authorization

Value:
Bearer YOUR_INSTAGRAM_ACCESS_TOKEN
```

Replace the placeholder with your real token inside the private n8n credential.

Use the credential in all relevant Instagram API HTTP Request nodes.

Never put the real token in your public GitHub repository.

## 7. Configure Instagram App Secret

Open your Meta app settings and securely obtain the **App Secret**.

In n8n, configure the Crypto credential used by the `Hashing Instagram App Secret` node.

The App Secret is used to validate the webhook's HMAC-SHA256 signature.

It is different from both the Instagram access token and the webhook verify token.

For POST requests, verify the `X-Hub-Signature-256` header against the exact raw request body before processing customer data.

## 8. Create DeepSeek API Credentials

1. Visit https://platform.deepseek.com/
2. Create an account or sign in.
3. Add API credit if required.
4. Generate an API key.
5. Open the **DeepSeek Chat Model** node in n8n.
6. Create a DeepSeek API credential.
7. Paste your API key and save.

The exported workflow references `deepseek-flash`. Check whether that model is available through your current n8n integration. If necessary, select a supported DeepSeek model and test the structured output.

The AI is expected to produce fields such as:

```json
{
  "mode": "AI",
  "reply": "Hello! How can I help you?",
  "summary": "Customer requested information."
}
```

## 9. Create Google Sheets Credentials

### Step 1 — Google Cloud

1. Visit https://console.cloud.google.com/
2. Create or select a Google Cloud project.
3. Enable the Google Sheets API and other APIs required by your credential configuration.
4. Configure the OAuth consent screen.
5. Create an OAuth client.

### Step 2 — n8n

1. Open **Credentials** in n8n.
2. Create a **Google Sheets OAuth2** credential.
3. Copy the OAuth Redirect URL displayed by n8n.
4. Add that URL to the authorized redirect URIs in Google Cloud.
5. Enter the Client ID and Client Secret in n8n.
6. Sign in with your Google account.
7. Grant the required permissions.

### Step 3 — Connect Spreadsheets

Open each Google Sheets node.

Select the correct credential, spreadsheet, tab, and column mappings.

The workflow requires mappings for the main Instagram database, the separate projects spreadsheet, and the separate error spreadsheet.

## 10. Create Gmail Credentials

Gmail is used for optional administrator error alerts.

1. Enable the Gmail API in your Google Cloud project if required.
2. Create a Gmail OAuth2 credential in n8n.
3. Complete the Google authorization process.
4. Open the `Error Alert` node in the error workflow.
5. Select the Gmail credential.
6. Enter your own administrator email address.

Test the notification using a controlled workflow failure.

## 11. Import the n8n Workflows

Open n8n and import the JSON files from the `workflows` folder.

Import these as two separate workflows:

```text
F-7 Instagram AI Engagement & Automation Agent (Public Template).json

Universal Error Handling (Public template).json
```

After importing:

1. Reconnect Instagram credentials.
2. Reconnect DeepSeek credentials.
3. Reconnect Google Sheets credentials.
4. Reconnect Gmail credentials.
5. Replace placeholder spreadsheet IDs and tab mappings.
6. Configure your Instagram Professional account ID.
7. Configure your developer Instagram-scoped ID.
8. Configure the Meta App Secret.
9. Correct the error workflow's timestamp and status-code expressions.
10. Select the error workflow in the main workflow's error-handling settings.

Do not activate the workflow until configuration and testing are complete.

## 12. Configure Instagram Webhooks

Meta uses a callback verification challenge and subsequent webhook event delivery.

### GET Verification

When registering the callback, Meta sends a GET request containing values such as:

```text
hub.mode
hub.verify_token
hub.challenge
```

Your verification endpoint must compare the supplied verify token against the token you configured.

If valid, return the challenge as plain text with HTTP 200.

If invalid, reject the request.

The included main workflow uses a POST webhook. A separate GET verification handler, or an equivalent supported gateway configuration, is required for the initial verification challenge.

### POST Events

For incoming messages and comments:

1. Receive the webhook POST request.
2. Validate the Meta signature using your App Secret.
3. Reject invalid signatures.
4. Identify the event type.
5. Process the relevant DM or comment branch.
6. Prevent duplicate processing.
7. Return an appropriate HTTP response.

### Meta Dashboard Configuration

In your Instagram webhook settings:

1. Enter the public HTTPS callback URL.
2. Enter your webhook verify token.
3. Complete callback verification.
4. Subscribe to the relevant messaging and comment event fields.
5. Test using an authorized Instagram account.

Your production webhook URL must be reachable over HTTPS.

## 13. Create an Instagram Privacy Policy

A privacy policy explains what customer information your automation processes and how that information is handled.

It should cover:

- Who operates the automation.
- What Instagram information is collected.
- How messages and comments are used.
- How DeepSeek processes conversation context.
- How Google Sheets stores information.
- Who can access the information.
- How long information is retained.
- How customers can request deletion.
- How customers can contact the operator.

### Host the Privacy Policy Using GitHub Pages

1. Create a privacy policy page in your GitHub repository.
2. Replace all sample names, emails, and retention details with accurate information.
3. Open **Repository Settings → Pages**.
4. Select **Deploy from a branch**.
5. Select `main` and the appropriate folder.
6. Save and wait for deployment.
7. Open the public HTTPS URL and verify that it works without signing in.

Example URL:

```text
https://YOUR_USERNAME.github.io/YOUR_REPOSITORY/privacy-policy.html
```

Then open:

**Meta for Developers → App Settings → Basic**

Paste the public URL into the **Privacy Policy URL** field.

Also complete any required data-deletion instructions or callback configuration.

A privacy policy template must be customized to your actual data practices and applicable legal requirements.

## 14. Human Handoff System

When a customer requests human assistance, the workflow can create a handoff record and pause automated AI replies for that sender.

The developer can manage requests through Instagram messages from the authorized developer account.

### Developer Commands

| Command | Purpose |
|---|---|
| `SEND` | Retrieve handoff requests |
| `HR-001: ACCEPTED` | Accept a human handoff |
| `HR-001: DECLINED` | Decline a handoff |
| `HR-001: CHAT CLOSED` | Close the human chat and return to AI mode |

Replace `HR-001` with the actual request ID.

Only the configured developer account should be authorized to execute these commands.

Human handoff controls the automation's reply behavior; it does not automatically create a native Instagram support-agent assignment.

## 15. Testing Checklist

Before using the automation with real customers, verify:

- [ ] Instagram Professional account is connected.
- [ ] Meta API permissions are configured.
- [ ] Instagram access token works.
- [ ] Webhook GET verification succeeds.
- [ ] Webhook POST signatures are validated.
- [ ] Invalid webhook signatures are rejected.
- [ ] Incoming text DMs trigger the AI.
- [ ] Customers receive one reply per message.
- [ ] Conversation memory remains separated by sender.
- [ ] Approved projects load correctly.
- [ ] Duplicate messages are ignored.
- [ ] Comment rules work correctly.
- [ ] Duplicate and self-comments are ignored.
- [ ] Human handoff pauses AI responses.
- [ ] Developer commands work only for the authorized account.
- [ ] Google Sheets logs update correctly.
- [ ] Error logging works.
- [ ] Gmail error alerts are delivered.
- [ ] Privacy policy URL is publicly accessible.

## 16. Troubleshooting

| Problem | Possible cause |
|---|---|
| Webhook verification fails | Incorrect verify token or missing GET verification handler |
| Invalid webhook signature | Incorrect App Secret or changed request body |
| Instagram API returns 401 | Expired or incorrect access token |
| Instagram API returns 403 | Missing permissions or access restrictions |
| AI does not reply | Incorrect event routing, handoff state, DeepSeek credential, or spreadsheet mapping |
| Duplicate replies | Missing deduplication or repeated webhook events |
| Google Sheets error | Incorrect spreadsheet ID, tab, OAuth credential, or column mapping |
| Comments do not trigger | Missing comment subscription, rules, or permissions |
| Developer commands fail | Incorrect developer ID, request ID, or command format |
| Error workflow fails | Incorrect timestamp expression, field mapping, or credentials |

## 17. Security and Privacy

**Never publish:**

- Instagram access tokens
- Meta App Secrets
- DeepSeek API keys
- Google OAuth client secrets
- Private customer conversations
- Real customer identifiers
- Sensitive spreadsheet contents
- n8n credential exports

Store API credentials securely in n8n.

Use HTTPS, restrict Google Sheets access, and apply appropriate retention and deletion practices.

Treat incoming Instagram messages as untrusted data. Customer messages must not be allowed to override developer authorization or security rules.

The included sanitized workflows must be reviewed and tested before production deployment.

## 18. Repository Structure

```text
InstaEngage-AI-Instagram-Automation/
|
|-- README.md
|-- LICENSE
|
|-- workflows/
|   |-- InstaEngage_AI_GitHub_Sanitized.json
|   |-- Universal_Error_Handling_GitHub_Sanitized.json
|
|-- google-sheets/
|   |-- Google Sheets Templates
|
|-- docs/
|   |-- PRIVACY_POLICY_TEMPLATE.md
|
|-- media/
|   |-- Demo and Screenshots
```

## 19. Demo

**Instagram Demo:** https://www.instagram.com/reel/Dec_JLxCx2l/?utm_source=ig_web_copy_link&obrf=MzRlODBiNWFlZA==&srtk=MzRlODBiNWFlZA==.

## 20. Official Documentation

- [Meta for Developers](https://developers.facebook.com/)
- [Meta Instagram API — Official Postman Collection](https://www.postman.com/meta/instagram/folder/6raa77c/instagram-api-with-instagram-login)
- [n8n Documentation](https://docs.n8n.io/)
- [DeepSeek Platform](https://platform.deepseek.com/)
- [Google Cloud Console](https://console.cloud.google.com/)
- [GitHub Pages Documentation](https://docs.github.com/en/pages)

## 21. License

This project is distributed under the MIT License. See the `LICENSE` file for details.

---

**InstaEngage AI — Automating Instagram conversations while keeping human support available when needed.**
