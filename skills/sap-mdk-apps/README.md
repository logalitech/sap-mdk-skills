# SAP Mobile Development Kit Apps Skill

Guidance for AI coding agents working on SAP Mobile Development Kit (MDK) apps: the structure of a
project, the metadata syntax, and the rules that prevent common failures on iOS, Android and the web.

## Installation

Install it with the [skills CLI](https://github.com/vercel-labs/skills):

```bash
npx skills add logalitech/sap-mdk-skills --skill sap-mdk-apps
```

## Usage

Start a conversation about an MDK project and the skill triggers on keywords such as:

- "MDK", "Mobile Development Kit", "SAP Mobile Services"
- `.page`, `.action`, `.service`, `Application.app`
- "offline store", "defining request", "upload category"
- "getControl", "redraw", "setValue"

Example prompts that trigger the skill:

- "Add a list page for customers to my MDK app"
- "Why does getControl return undefined on this page?"
- "How do I send one order without uploading the others?"
- "Why does my Styles.less break the bundle?"

## Skill Contents

| Section | Topics |
|---|---|
| Core Principles | metadata, runtime interpretation, runtimes, offline, what the client renders |
| Runtimes | iOS and Android client against the web runtime |
| Quick Reference | project layout, how the pieces connect, metadata syntax, from change to device |
| Implementation Checklist | checks before a change ships |
| Key Guidelines | the ClientAPI, pages and styling, offline and services, tooling and deploy |
## Version

Written against MDK schema version 26.6, September 2026.
