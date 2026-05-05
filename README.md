# 🔐 OAuth — A Gentle Primer

An interactive Reveal.js presentation: a friendly, no-RFC, no-code primer on **OAuth** — the mental model and vocabulary you'll meet in Part 1 (the protocol) and Part 2 (MCP servers and the provider landscape).

## ▶ [Open the Presentation](https://brendanjameslynskey.github.io/OAuth_Primer/)

## 📄 [Markdown Version](presentation.md)

## 📚 [Part 1 — Introduction to OAuth](https://brendanjameslynskey.github.io/Introduction_to_OAuth/) · [Part 2 — OAuth for MCP Servers](https://brendanjameslynskey.github.io/OAuth_for_MCP/)

---

## Contents

| # | Topic | Description |
|---|-------|-------------|
| 01 | Title | Picture · Vocabulary · Roadmap |
| 02 | Topics | What this primer covers |
| 03 | The Valet Key | One-picture mental model — three actors, one delegation |
| 04 | AuthN vs AuthZ | The most-confused pair, defined and separated |
| 05 | The Password Problem | The 2006 Flickr / PhotoPrint story OAuth was invented to fix |
| 06 | "Sign in with Google" | What's actually happening, step by step |
| 07 | The Five Universal Steps | Ask → Authenticate → Consent → Token → Use |
| 08 | Tokens | What they are, why they expire, the bearer rule |
| 09 | Scopes | Fine-grained permissions and the audience sister-concept |
| 10 | Glossary | One-page reference for every term that follows |
| 11 | What OAuth Is *Not* | Three common misunderstandings |
| 12 | Where You've Met OAuth | Login buttons, "connect your account", CLIs, mobile apps, MCP |
| 13 | The MCP Angle | Why this is suddenly topical again |
| 14 | Self-Check | Three quick questions with answers |
| 15 | Reading Roadmap | Part 0 → Part 1 → Part 2 |
| 16 | Summary | Three sentences you can screenshot |

---

## Who this primer is for

- **Anyone new to OAuth** — non-developers welcome.
- Engineers who keep nodding through OAuth conversations and want a mental scaffolding before reading the spec.
- Anyone planning to read Part 1 (Introduction to OAuth) or Part 2 (OAuth for MCP) and wants the vocabulary in their head first.

It deliberately contains no RFC numbers, no code, and no JSON in the slide body — those start in Part 1.

## Slide Controls

| Action | Key |
|--------|-----|
| Next / Previous | `→` `←` or swipe |
| Overview | `Esc` |
| Fullscreen | `F` |
| Export to PDF | Append `?print-pdf` to URL, then print |

## Technology

[Reveal.js 4.6](https://revealjs.com) · [highlight.js](https://highlightjs.org) · Playfair Display + DM Sans + JetBrains Mono · inline SVG diagrams.

Single self-contained `index.html` — no build step, no npm, no dependencies to install.

## See also

- [Introduction to OAuth](https://github.com/BrendanJamesLynskey/Introduction_to_OAuth) — Part 1, the full protocol.
- [OAuth for MCP Servers](https://github.com/BrendanJamesLynskey/OAuth_for_MCP) — Part 2, MCP authorisation and the identity-provider landscape.
- [Introduction to OpenID Connect](https://github.com/BrendanJamesLynskey/Introduction_to_OpenID_Connect) — the identity layer used wherever an app needs to know *who* the user is.

## License

Educational use. Code examples provided as-is.
