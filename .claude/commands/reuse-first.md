---
description: Run the REUSE-FIRST DISCOVERY GATE before building anything new
argument-hint: [optional: what you're about to build]
---

Before any code is written, complete the REUSE-FIRST DISCOVERY GATE (see CLAUDE.md → "REUSE-FIRST DISCOVERY GATE" for the full rule).

Produce these five fields, in this order, then STOP and wait for user approval:

1. **Interpreted goal** — one sentence.
2. **Contained build boundary** — IN scope / OUT of scope.
3. **Best existing options** — 2–4 maintained candidates, each with: name, link, last-updated signal, license, what it gives, what it lacks.
4. **Build-vs-borrow recommendation** — use / wrap / compose / build-from-scratch, with reason. From-scratch requires explicit justification.
5. **One concrete next step** — the smallest single action, not a plan.

Search scope (minimum): GitHub, Hugging Face, npm / PyPI / crates.io, Docker Hub, n8n template library, MCP server registries (modelcontextprotocol.io + awesome-mcp), Claude / ClawHub resources if available, official vendor docs, awesome-\* lists, starter kits.

If a search dimension is unavailable in this environment, say so explicitly. Do not produce vague summaries — name specific projects with links, or justify a from-scratch build out loud.

Topic / target: $ARGUMENTS
