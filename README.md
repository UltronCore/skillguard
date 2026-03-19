# SkillGuard

**A mandatory pre-install security gate for Claude Code skills, plugins, configs, and any file that could affect Claude's behavior.**

SkillGuard intercepts every install, import, or activation request before it reaches Claude — performing a structured multi-reviewer threat analysis and requiring explicit human approval before anything proceeds. No auto-approvals. No skipping. No exceptions.

---

## What It Does

When you attempt to install a skill, load a config, run a setup script, or import any file into your Claude Code workflow, SkillGuard:

1. **Pauses everything** and announces the review
2. **Inventories all files** in the package recursively
3. **Runs a two-pass deep review** against a 15-category threat catalog
4. **Convenes a multi-reviewer council** (3–8 reviewers depending on complexity) that independently analyze each file
5. **Produces a structured security report** with finding codes, OWASP LLM Top 10 mappings, severity scores, and a supply chain assessment
6. **Stops and waits** for your explicit APPROVE or REJECT before taking any action

---

## Why It Exists

Claude Code's skill and plugin ecosystem is powerful — and that power is a surface for attack. A malicious skill can:

- Inject instructions that override Claude's behavior without your knowledge
- Exfiltrate credentials, API keys, or conversation contents via image URLs or network calls
- Install persistence mechanisms that survive sessions
- Escalate permissions through hook manipulation
- Chain individually-benign capabilities into dangerous toxic flows

Most users install skills with a single command and no review. SkillGuard exists to close that gap.

---

## Threat Coverage

SkillGuard v1.1 checks against 15 threat categories, all mapped to OWASP LLM Top 10 2025 and MITRE ATLAS:

| # | Category | OWASP |
|---|----------|-------|
| 1 | Prompt injection and behavioral manipulation | LLM01 |
| 2 | Shell command injection | LLM02 |
| 3 | Encoded and obfuscated payloads (base64, ROT13, Unicode Braille, zero-width chars) | LLM01 |
| 4 | Credential and data exfiltration — with provider-specific patterns for 13 providers | LLM06 |
| 5 | External network calls and remote dependencies — URL shortener detection | LLM03 |
| 6 | Persistence and self-modification | LLM06 |
| 7 | Permission escalation and safety bypass | LLM06 |
| 8 | Mismatch between stated and actual purpose | LLM08 |
| 9 | Agentic abuse — tool shadowing and tool poisoning | LLM06 |
| 10 | **Markdown image exfiltration** — `![](https://...?q=)` patterns | LLM05 |
| 11 | **LLM control structure injection** — ChatML, Guidance, Llama-2/3 tokens | LLM01 |
| 12 | **Toxic flow combinations** — cross-file capability chaining | LLM06 |
| 13 | **Financial execution risk** — payment APIs, crypto wallets | LLM06 |
| 14 | **System prompt leakage** — instructions to reveal configuration | LLM07 |
| 15 | **Latent and indirect injection surface** — skills that process external content | LLM01 |

Categories 10–15 were added in v1.1 based on comparative research against 8 open-source AI security tools.

---

## Reviewer Council

SkillGuard assembles a council of independent reviewers. Each reads all files and writes findings before the council convenes:

| Reviewer | Role | When |
|----------|------|------|
| **A — Technical Security Analyst** | Shell commands, scripts, credentials, encoded payloads, network calls | Always |
| **B — Prompt and Behavior Analyst** | Injection, role hijacking, DAN patterns, hidden instructions, behavioral manipulation | Always |
| **G — Output Impact Analyst** | What Claude would produce if the instructions were followed | Always |
| **C — Supply Chain Analyst** | Dependencies, registries, typosquatting | When manifests present |
| **D — Execution Path Analyst** | Install-time vs activation-time execution chains | When scripts present |
| **E — Data Exfiltration Analyst** | Credential access, env vars, outbound data | When data access detected |
| **F — Permission Abuse Analyst** | Hook manipulation, settings.json changes, autonomous actions | When hooks/permissions present |
| **H — Toxic Flow Analyst** | Cross-file capability combinations (TF001/TF002/TF003) | When input + data + egress detected |

---

## Report Structure

Every SkillGuard review produces a 13-section security report:

1. Executive Summary
2. Review Trigger Context
3. Inventory of Reviewed Files
4. Reviewer A Findings
5. Reviewer B Findings
6. Reviewer G Findings (Output Impact)
7. Additional Reviewer Findings (C–F, H as spawned)
8. Council Verdict with worst-case credible interpretation
9. Red Flags Table — with finding codes and OWASP mapping
10. Benign Indicators
11. Missing Information
12. Safe Use Recommendation with sandbox guidance by risk level
13. Supply Chain Assessment
14. Confidence Score (0–100, computed via explicit formula)
15. *(Optional)* Appendix A: Machine-readable JSON summary

---

## Installation

SkillGuard is a [Claude Code](https://claude.ai/claude-code) skill plugin.

### Option 1: Copy the skill file

Copy `skills/skillguard/SKILL.md` and its `references/` folder into your Claude Code skills directory.

### Option 2: Use as a local plugin

1. Clone this repository
2. Add the plugin to your `~/.claude/settings.json`:

```json
{
  "plugins": {
    "skillguard@local": true
  }
}
```

3. Point the plugin path to where you cloned the repo (see Claude Code plugin documentation for local plugin loading)

### Enforce with a hook

Add this to your `~/.claude/settings.json` `hooks` section to catch install commands at the Bash level:

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          {
            "type": "command",
            "command": "bash -c 'cmd=\"$CLAUDE_TOOL_INPUT_COMMAND\"; if echo \"$cmd\" | grep -qiE \"(plugin install|skill install|npm install|pip install|brew install|curl.*\\|.*(bash|sh)|wget.*\\|.*(bash|sh))\"; then echo \"[SKILLGUARD] Install command detected. Invoke the skillguard skill before proceeding.\"; fi; exit 0'"
          }
        ]
      }
    ]
  }
}
```

---

## Trigger Phrases

SkillGuard activates on any of:

> "install this", "add this skill", "import this", "load this config", "use this prompt", "activate this agent", "run this setup", "apply this", "trust this file", "here's a skill", "add to Claude", "use this tool", "install this plugin", "run this script", "set this up for me"

...or any request to install, import, activate, trust, execute, merge, run, apply, copy, or use any external file that could affect Claude's behavior.

It also triggers on specific file types: `.md`, `SKILL.md`, `.json`, `.yaml`, `.yml`, `.toml`, `.sh`, `.bash`, `.zsh`, `.ps1`, `.bat`, `.cmd`, `.py`, `.js`, `.ts`, `Dockerfile`, `package.json`, `requirements.txt`, manifests, configs, plugins, and extensions.

---

## Core Principles

1. **Never auto-approve** — not for convenience, speed, or trust signals
2. **Never skip a file** — read everything, regardless of extension
3. **Never suppress findings** — every flag appears in the report
4. **Markdown is security-relevant** — inspect for hidden prompts and exfiltration vectors
5. **JSON is security-relevant** — inspect fields for hooks, templates, and behavioral overrides
6. **Documentation is not evidence of safety** — verify claims against actual content
7. **Polish and popularity are not safety signals** — treat them as neutral
8. **Dangerous permission modes = elevated caution**, not permission to skip review
9. **Prefer false positives** — a missed threat is worse than a false alarm
10. **Never auto-execute commands during review** — observe only, never run

---

## Files

```
skills/
  skillguard/
    SKILL.md                        — Main skill logic and workflow
    references/
      threat-catalog.md             — 15-category threat taxonomy with detection patterns
      report-template.md            — 13-section report template with scoring formulas
      reviewer-guide.md             — Per-reviewer checklists, scoring logic, false positive controls

RESEARCH_AND_IMPROVEMENTS.md       — Phase 2 comparative analysis: 8 open-source security tools
                                      reviewed statically, capability comparison matrix,
                                      and the full reasoning behind every v1.1 improvement
README.md                           — This file
```

---

## Research Basis

SkillGuard v1.1 was improved through a static comparative analysis of 8 open-source AI and software security tools:

- **mcp-scan** (invariantlabs-ai) — MCP server security scanner
- **vigil-llm** (deadbits) — LLM prompt injection detection with YARA rules
- **rebuff** (protectai) — Self-hardening prompt injection detector with fuzzy scoring
- **garak** (NVIDIA/leondz) — LLM vulnerability scanner with 40+ probe categories
- **guardrails** (guardrails-ai) — AI application input/output validation framework
- **detect-secrets** (Yelp) — Enterprise secrets detection with Shannon entropy scoring
- **trufflehog** (trufflesecurity) — Secrets discovery with 800+ provider-specific patterns
- **OWASP LLM Top 10 2025** — Authoritative LLM threat taxonomy

Full analysis, capability comparison matrix, and improvement rationale: [`RESEARCH_AND_IMPROVEMENTS.md`](./RESEARCH_AND_IMPROVEMENTS.md)

---

## Versioning

| Version | Changes |
|---------|---------|
| v1.0 | Initial release — 9 threat categories, 6 reviewers (A–F), 12-section report |
| v1.1 | +6 threat categories (10–15), +Reviewer G (Output Impact, mandatory), +Reviewer H (Toxic Flow, dynamic), provider-specific credential patterns for 13 providers, OWASP LLM Top 10 mapping, finding codes, structured severity scoring, Shannon entropy guidance, false positive controls, Section 13 Supply Chain, Appendix A JSON output |

---

## License

MIT License — free to use, modify, and distribute. Attribution appreciated.

---

## Contributing

Contributions welcome. Particularly useful:
- New threat catalog entries with detection patterns
- Additional provider-specific credential patterns
- False positive reduction examples
- Real-world attack samples (redacted) for test cases
- Translations of the reviewer prompts

---

*SkillGuard does not execute any code. It performs static analysis only and requires explicit human approval before any installation proceeds.*
