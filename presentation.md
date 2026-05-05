# OAuth — A Gentle Primer

**The mental model — before the protocol details**

*A friendly walk-through of what OAuth is, what it solves, and the words you'll meet in Part 1 (the protocol) and Part 2 (MCP servers and the provider landscape).*

```
valet key -> time-bound -> scoped -> revocable
```

Picture  |  Vocabulary  |  Roadmap

---

## Table of Contents

1. [Topics](#slide-01--topics)
2. [The Valet Key — One-Picture Mental Model](#slide-02--the-valet-key--one-picture-mental-model)
3. [The Most-Confused Pair — AuthN vs AuthZ](#slide-03--the-most-confused-pair--authn-vs-authz)
4. [Why OAuth Was Invented — The Password Problem](#slide-04--why-oauth-was-invented--the-password-problem)
5. ["Sign in with Google" — What's Actually Happening](#slide-05--sign-in-with-google--whats-actually-happening)
6. [The Five Universal Steps of Any OAuth Flow](#slide-06--the-five-universal-steps-of-any-oauth-flow)
7. [Tokens — What They Are, Why They Expire](#slide-07--tokens--what-they-are-why-they-expire)
8. [Scopes — Fine-Grained Permissions](#slide-08--scopes--fine-grained-permissions)
9. [One-Page Glossary](#slide-09--one-page-glossary)
10. [Three Things OAuth Is *Not*](#slide-10--three-things-oauth-is-not)
11. [Where You've Already Met OAuth](#slide-11--where-youve-already-met-oauth)
12. [The MCP Angle — Why This Is Suddenly Topical Again](#slide-12--the-mcp-angle--why-this-is-suddenly-topical-again)
13. [Self-Check — Three Quick Questions](#slide-13--self-check--three-quick-questions)
14. [Reading Roadmap](#slide-14--reading-roadmap)
15. [Summary — Three Sentences You Can Screenshot](#slide-15--summary--three-sentences-you-can-screenshot)

---

## Slide 01 — Topics

### The picture

- The valet-key analogy — three actors, one delegation
- Authentication vs Authorisation — the most-confused pair
- The password-sharing problem OAuth was invented to kill
- "Sign in with Google" — what actually happens

### The vocabulary

- Tokens — what they are and why they expire
- Scopes — fine-grained permissions
- The five universal steps of any OAuth flow
- One-page glossary of every word you'll meet

### Reality check

- Three things OAuth is *not*
- Where you've already met OAuth without noticing
- Why AI agents and MCP servers care about it now

### Where to next

- Self-check: three quick questions
- One-page summary you can screenshot
- Reading roadmap — Part 1 (the protocol) and Part 2 (MCP & providers)

---

## Slide 02 — The Valet Key — One-Picture Mental Model

Imagine you drive to a hotel. You hand the valet a **special key** that *only* opens the doors and starts the engine — it doesn't open the boot, the glove box, or your house. You can take it back at any time. **That is OAuth.**

```
You ──issues a limited key──▶ The Valet ──drives your car──▶ Your Car
(user)                        (an app, on your behalf)        (your data / account)
```

The valet never sees your *master* key (your password). They get a *limited* key (a **token**) which:

- opens only what's needed (**scope**)
- expires soon (**lifetime**)
- can be taken back any time (**revocation**)

---

## Slide 03 — The Most-Confused Pair — AuthN vs AuthZ

### Authentication (AuthN) — "*who are you?*"

- Proving identity. Username + password. Passkey. Fingerprint. The login screen.
- Output: a claim like *"this is alice@example.com"*.
- Examples: logging into Gmail, unlocking your phone.

### Authorisation (AuthZ) — "*what may you do?*"

- Granting permission. Once we know who you are, what can you touch?
- Output: a permission like *"may read photos, may not delete"*.
- Examples: a shared Google Doc set to "comment-only"; an ATM letting you withdraw but not transfer.

### Where OAuth sits

- **OAuth** is fundamentally an **authorisation** framework — the valet key.
- It assumes the user has already authenticated *somewhere*.
- That "somewhere" is usually a separate trusted service — Google, Microsoft, your company's IdP.
- OAuth then issues a token that *delegates* permission from the user to a third-party app.

### "But I see 'Sign in with Google' all the time"

That button uses **OpenID Connect** — a small identity layer built *on top of* OAuth. It bundles "who you are" with "what you may do" into one round trip. We'll see it later.

### Memorable rule

"AuthN is the bouncer at the door. AuthZ is the wristband that says which rooms you can enter."

---

## Slide 04 — Why OAuth Was Invented — The Password Problem

Roll back to ~2006. You upload photos to Flickr. A new printing site, "PhotoPrint", offers to turn them into a calendar — *"just give us your Flickr password"*. People did. **It was a disaster.**

### What was wrong with sharing the password

- PhotoPrint now had **total access** — read, delete, change settings.
- Access was **forever** — it lasted as long as the password did.
- Revoking the app meant **changing your Flickr password**, which broke every other integration too.
- It trained users to **type their password into anything that asked** — a phisher's dream.
- If PhotoPrint got hacked, **everybody's Flickr passwords leaked**.

### What OAuth gives you instead

- **No password sharing** — PhotoPrint never sees your Flickr password.
- **Scoped** access — "read photos", not "do anything".
- **Expires** — typically minutes to an hour; renewable in the background.
- **Per-app revocation** — disconnect PhotoPrint without rotating your password.
- **Auditable** — Flickr knows it was PhotoPrint who read those photos at 14:07.

### In one sentence

"*App A wants to do thing R on user U's behalf at service S — without ever seeing U's password.*"

Every part of OAuth exists to make that one sentence safe.

---

## Slide 05 — "Sign in with Google" — What's Actually Happening

You've done it a hundred times. Click the button on a website, end up on Google, click "Allow", land back on the site already logged in. Behind the scenes:

```
You (browser)         "Cool New App"         Google
    │                       │                   │
1.  ├─click "Sign in"──────▶│                   │
2.  ◀──"go talk to Google"──┤                   │
3.  ├──browser opens accounts.google.com───────▶│
4.  ◀────"log in (if not already)"──────────────┤
5.  ◀────"Cool New App wants your name+email — Allow?"──
6.                          ◀──"here's a token + an ID about Alice"──┤
7.  ◀──"welcome, Alice"─────┤                   │
```

"Cool New App" never saw your Google password. It got an ID assertion (who) and a token (what it can do).

Step 5 — the consent screen — is the moment of delegation. Without it, OAuth would just be password-sharing with extra steps.

---

## Slide 06 — The Five Universal Steps of Any OAuth Flow

```
1. Ask  →  2. Authenticate  →  3. Consent  →  4. Token  →  5. Use
```

### 1. Ask

The app sends you to the auth server with a request: "*I'm app X, I want these permissions, send me back here when done.*"

### 2. Authenticate

The auth server makes you log in (if you aren't already). Password, passkey, MFA — whatever the org's policy says. **The app never sees this.**

### 3. Consent

The auth server shows you what the app is asking for in plain language. You click *Allow* (or *Deny*).

### 4. Token

The auth server hands the app a short-lived **access token** — the valet key. (And usually a longer-lived *refresh token* to mint new access tokens later.)

### 5. Use

The app calls the resource's API, attaching the token. The resource verifies the token and serves the request.

### Every OAuth flow

Same five beats. The differences between flows are *where* each step happens — in a browser, on a server, on a TV — and how the token gets back to the app.

---

## Slide 07 — Tokens — What They Are, Why They Expire

A **token** is just a string of characters that means "*the bearer of this is allowed to do X for the next Y minutes*". It's a digital wristband.

### Three kinds of token, three jobs

- **Access token** — the wristband. Sent on every API call. *Short-lived* (often 5–60 min).
- **Refresh token** — the spare wristband. Used *only* to ask the auth server for a new access token. Lives longer; never sent to the resource server.
- **ID token** — a signed slip of paper saying *"this is Alice"*. Used for login, not for API calls.

### Why they expire

- If a wristband leaks, the damage window is bounded by its lifetime.
- Long-lived secrets get logged, screenshotted, pushed to GitHub. Short-lived ones do too — but they expire before the attacker can use them.
- The refresh token is the only long-lived item, and it's protected by being usable only by the legitimate app talking to the auth server.

### Bearer tokens

Most tokens are *bearer* tokens — like cash. Whoever holds it can spend it. Modern OAuth has stronger options (DPoP, mTLS) that bind the token to the legitimate holder, but the default mental model is "treat it like cash".

### Rule

Never log a token. Never put one in a URL. Never email one. Treat them like passwords.

---

## Slide 08 — Scopes — Fine-Grained Permissions

A **scope** is just a label that says "this token may do *this specific thing*". The auth server stamps the scopes onto the token; the resource server checks them on every call.

### Examples you've seen on consent screens

- `email` · `profile` — your name and email
- `read photos` · `upload photos`
- `read your repositories` · `create issues`
- `send messages on your behalf`
- `see your calendar` · `add events to your calendar`

### Why this matters

Granular scopes let you say *"yes to read my photos, no to delete them"*. They also let the app be honest about what it actually needs — and let users walk away if it's asking for too much.

### "Why does this app need *those* permissions?"

- If a flashlight app asks for your contacts — that's an OAuth-scope conversation.
- The same instinct applies to any login-with-Google button or MCP server install screen.

### Audience — a sister concept

Scope says *what* the token can do; **audience** says *which* service it can do it to. A token issued for "Slack" shouldn't be usable at "GitHub", even if both trust the same auth server. Audience is how OAuth keeps tokens from being laundered between APIs.

---

## Slide 09 — One-Page Glossary

| Word | Plain meaning | The valet analogy |
|---|---|---|
| **Resource Owner** | The user (you). | The car owner. |
| **Client** | The third-party app asking for access. | The valet. |
| **Authorization Server (AS)** | The trusted service that authenticates you and issues tokens. | The hotel front desk that prints valet keys. |
| **Resource Server (RS)** | The API or service holding your data. | The car. |
| **Scope** | A specific permission attached to a token. | "May start engine; may not open boot." |
| **Audience** | Which service the token is meant for. | "Key works at this hotel only." |
| **Access token** | The credential the client sends to the resource server. | The valet key. |
| **Refresh token** | A longer-lived credential used only to mint new access tokens. | The card you exchange at the desk for a fresh key. |
| **ID token** | A signed statement saying "this is user U" (OpenID Connect). | The receipt with your name on it. |
| **Consent screen** | The "App X wants to: …" dialog the AS shows you. | "Sir, the valet would like to start your engine — OK?" |
| **Grant / Flow** | A specific recipe for getting a token (different recipes for browser, mobile, TV, server-to-server). | Different ways to hand over the key (in person, posted, on-screen code). |
| **PKCE** | An extra secret only the legitimate app knows, sent at the end to prove it's the same app that started the flow. Pronounced "pixie". | A wristband stamp the valet must show to claim the key. |
| **OpenID Connect (OIDC)** | A small identity layer on top of OAuth — adds the ID token. | OAuth + the receipt with your name on it. |
| **Bearer token** | A token where "whoever holds it can use it" — like cash. | A valet key anyone could pick up and use. |
| **DPoP / mTLS** | Ways to bind a token to the legitimate holder, so a stolen one is useless. | A key that only works if you also present your fingerprint. |

---

## Slide 10 — Three Things OAuth Is *Not*

### 1. Not authentication, on its own

An access token says *"this app may call this API"*, not *"this is Alice"*. If you build a "log in" feature using just an OAuth access token, you've built a bug — the user has no proven identity inside your system. **OpenID Connect** is what adds identity (an ID token) on top.

### 2. Not encryption

OAuth doesn't encrypt your data or your tokens — TLS does. OAuth assumes the channel is already secure. Run everything over HTTPS, end of story. Tokens travel in clear over an encrypted pipe.

### 3. Not a single protocol

OAuth is a *framework* with several flows (browser, mobile, TV, server-to-server) and many optional extensions. There is no single "OAuth library" you bolt on; pick the right flow for your situation. Part 1 walks you through that choice.

### A useful summary

OAuth is the **plumbing for delegation**: handing an app a limited, expiring, revocable, audit-able permission to act on your behalf. Anything beyond that — login, encryption, governance, MFA, single sign-on — is a different problem that often *uses* OAuth as one ingredient.

---

## Slide 11 — Where You've Already Met OAuth

OAuth runs more of your day than you realise.

### Login buttons

- "Sign in with Google / Microsoft / Apple / GitHub" — every one is OAuth + OpenID Connect.
- Your work SSO ("Sign in with Okta", "Continue with Microsoft 365") — same.

### "Connect your account" experiences

- Connecting your bank to a budgeting app — that consent screen is OAuth.
- A scheduling tool reading your Google Calendar.
- A photo-printing site pulling your Instagram or iCloud photos.

### CLIs and terminals

- `gh auth login`, `gcloud auth login`, `az login` — open a browser, you click Allow, the CLI gets a token.
- Smart-TV logins ("go to acme.com/tv and enter ABCD") use the same family of flows.

### Mobile apps

- Every modern mobile app that talks to a SaaS uses OAuth — it is what gets the token onto your phone without anyone typing a corporate password into a third-party app.

### APIs you call from code

- Stripe webhooks, Slack bots, GitHub Actions reaching out to other services — most use OAuth tokens (issued either to a user or to the application itself).

### And — newly — AI agents

- Remote MCP servers (the standard way LLM apps talk to outside tools) authenticate with OAuth.
- That's why we have Part 2 of this series.

---

## Slide 12 — The MCP Angle — Why This Is Suddenly Topical Again

### What's MCP, in one paragraph

The **Model Context Protocol** (Anthropic, 2024) is an open standard that lets LLM apps — Claude Desktop, Cursor, claude.ai, Goose, custom agents — call *tools* and read *resources* in a uniform way. Think "USB-C for AI integrations".

### Why OAuth shows up

- Many useful MCP servers don't run on your laptop — they live on the public internet (a Slack server, a Jira server, a GitHub server).
- Letting an AI agent take any action on your behalf at one of those services *is exactly the OAuth problem*: scoped, expiring, revocable delegation.
- The MCP standard adopts OAuth as its mandated way to do this.

### The three things to know going in

1. Every remote MCP server speaks OAuth, the same way every "Sign in with Google" button does.
2. Tools like the **Docker MCP Gateway** run that OAuth dance once, locally, on your behalf — so you don't have to log in inside every chat app.
3. You'll see real provider names — Auth0, Microsoft Entra, Keycloak, ZITADEL — just like you'd see "Visa / Mastercard" if we were talking about payments.

### If you remember one line

Remote MCP didn't invent any new authorisation idea — it picks the modern OAuth defaults and pins them. *If you understand OAuth, you understand MCP auth.*

---

## Slide 13 — Self-Check — Three Quick Questions

### Q1.

*You give "PhotoPrint" your Flickr password. Why is that **not** what OAuth does?*

> **Because OAuth never shares the password.** The app is sent to Flickr's auth server, you log in there, you click Allow, and the app gets a *token* instead. The password stays with you and Flickr.

### Q2.

*An access token says "this client can call this API". Can you use it as proof of **who the user is** in your app?*

> **No.** An access token is permission, not identity. For identity, use the *ID token* from OpenID Connect. Conflating them is the most common OAuth bug in the wild.

### Q3.

*Why are access tokens short-lived (5–60 min) when refresh tokens last for days or months?*

> **Damage limitation.** The access token travels widely (every API call) so it's the most likely to leak — keep its lifetime short. The refresh token only ever talks to the auth server, over a strictly-controlled channel, so it can be longer-lived. Together you get convenience (no constant re-login) *and* a small breach window.

### If you got all three…

You have the right mental model. Part 1 will turn it into the actual protocol; Part 2 will apply it to MCP servers and the provider landscape.

---

## Slide 14 — Reading Roadmap

### Part 0 — You are here

- The mental model
- The vocabulary
- What OAuth is and isn't
- Why MCP cares about it

Aimed at: anyone new to delegated auth, including non-developers.

### Part 1 — Introduction to OAuth

- The full history (OAuth 1.0a → 2.0 → 2.1)
- Every grant type, with diagrams
- Authorization Code + PKCE in depth
- OpenID Connect, JWTs, DPoP
- Common attacks & defences
- Choosing a flow & a library

Aimed at: developers building or integrating an OAuth client / server.

### Part 2 — OAuth for MCP & Providers

- The MCP authorisation profile
- Resource Indicators & audience binding
- Discovery, Dynamic Client Registration
- The Docker MCP Gateway
- Commercial providers: Auth0, Entra, Cognito, Stytch, WorkOS, Clerk, Descope, FusionAuth
- Open-source providers: Keycloak, ZITADEL, Authentik, Ory Hydra, Authelia, Logto, SuperTokens
- Production checklist

Aimed at: anyone shipping or operating remote MCP servers.

### Recommended order

Part 0 (this deck, ~15 min) → Part 1 (full protocol, ~45 min) → Part 2 (MCP & providers, ~45 min). If you already write OAuth code, skip straight to Part 2 and use Part 1 as reference.

---

## Slide 15 — Summary — Three Sentences You Can Screenshot

### If you remember three things

1. **OAuth is delegation, not login.** It hands an app a limited, expiring, revocable permission to act on your behalf at another service.
2. **A token is a digital wristband.** Short-lived, scope-limited, audience-bound. Treat it like cash.
3. **"Sign in with X" = OAuth + a thin identity layer (OpenID Connect).** Same machinery; one extra slip of paper that says who you are.

### The five-step rhythm

Ask → Authenticate → Consent → Token → Use. Every flow you'll ever meet — browser, mobile, TV, MCP server — is a variation on those five beats.

### What you just learned the words for

Resource owner · client · authorization server · resource server · scope · audience · access token · refresh token · ID token · consent · grant / flow · bearer token · PKCE · OpenID Connect · DPoP / mTLS.

Every one of those will reappear in Part 1 with the full technical treatment.

### Where to go next

**Part 1 — Introduction to OAuth** for the protocol; **Part 2 — OAuth for MCP Servers & the Provider Landscape** for the AI-agent and provider angle.

### One-line takeaway

OAuth is the polite way to lend your account to someone else's app — for a little while, for a specific purpose, with the right to take it back.
