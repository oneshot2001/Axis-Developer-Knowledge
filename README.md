# Axis Developer Knowledge Base

> The most comprehensive AI-ready knowledge base for Axis Communications developers, integrators, and channel partners.

Built to eliminate hallucination when AI works with Axis APIs, ACAP development, and partner workflows. Every file is structured to be consumed by AI agents — but written for humans first.

---

## Why This Exists

Generic AI models hallucinate Axis API endpoints. They confuse ARTPEC generations. They don't know how integrators actually phrase questions. They have no idea what LVT cares about versus what a generic reseller cares about.

This repo fixes that. It's a vertical knowledge layer — institutional memory about the Axis ecosystem distilled into files that any AI agent (or human) can load and immediately reason from.

**Powered by [AxisX](https://github.com/yourusername/axisx)** — the voice-first product intelligence platform for Axis channel partners.

---

## What's Here

```
axis-developer-knowledge/
│
├── resources/
│   └── axis-resources.md          Master index of all official + community resources
│
├── api/
│   └── axis-vapix.md              VAPIX API patterns, auth, endpoints, gotchas
│
├── development/
│   └── axis-acap.md               ARTPEC-8/9 development bible, DLPU scheduling rules
│
├── intelligence/
│   ├── axisx-voice-patterns.md    How integrators ask questions + intent decoder
│   ├── partner-personas.md        LVT, Stone, Fred Juhlin, generic integrator profiles
│   └── competitive-context.md     Where Axis wins, where it doesn't, talking points
│
└── CLAUDE.md                      AI agent context and consumption instructions
```

---

## Who Should Use This

- **Developers** building VAPIX or ACAP integrations who want correct API patterns without digging through scattered docs
- **AI agents** working with Axis hardware, APIs, and partner workflows
- **Channel partners** who want a knowledge layer for their internal tools
- **Sales engineers** who need competitive talking points grounded in reality
- **Anyone** tired of AI confidently making up Axis endpoint URLs

---

## How To Use It

### With Claude / Claude Code
Drop `CLAUDE.md` at the root of your project. Claude will load the context automatically and reference the `docs/` files for Axis-specific knowledge.

### With any AI agent
Point your agent at this repo. The files are written in plain Markdown, structured for easy ingestion. Each file has a `PURPOSE` section that tells the agent exactly when and how to use it.

### As a human
Just read the files. They're organized to be genuinely useful as reference documentation — not just AI prompt fodder.

---

## Keeping It Current

This knowledge base tracks:
- **ACAP releases** — currently ACAP 12.8 (February 2026)
- **AXIS OS versions** — AXIS OS 13 is a major breaking change release
- **New Axis products** — via official newsroom and product selector
- **Community repos** — especially [pandosme (Fred Juhlin)](https://github.com/pandosme) for insider ACAP patterns

PRs welcome. If you're an Axis integrator, SE, or developer who knows something that isn't in here — submit it. This gets better with every contribution.

---

## Official Axis Resources

| Resource | URL |
|----------|-----|
| Developer Hub | https://developer.axis.com/ |
| VAPIX Reference | https://developer.axis.com/vapix/ |
| ACAP SDK Docs | https://axiscommunications.github.io/acap-documentation/ |
| Axis GitHub | https://github.com/AxisCommunications |
| For Developers | https://www.axis.com/en-us/for-developers |
| Product Selector | https://www.axis.com/en-us/support/tools/product-selector |
| Developer News | https://www.axis.com/en-us/for-developers/news |

---

## Community Resources

| Resource | URL | Why It Matters |
|----------|-----|----------------|
| Fred Juhlin (pandosme) | https://github.com/pandosme | Axis Sweden team — insider ACAP patterns |

---

## License

MIT. Use it, fork it, build on it. If you make it better, PR back.

---

## Maintained By

Built and maintained by the [AxisX](https://github.com/yourusername/axisx) team.  
Questions? Open an issue.
