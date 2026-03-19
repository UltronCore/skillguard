# SkillGuard Security Report Template

Copy this template in full and fill every section. Do not omit sections. Do not abbreviate.

---

```markdown
# 🛡️ SkillGuard Security Report

**Package under review:** [name or path]
**Review triggered by:** [user action]
**Review date:** [current date/time]
**Dangerous permissions active:** [YES — bypassPermissions / skipDangerousModePermissionPrompt / other | NO]
**Reviewers convened:** [list reviewer names and roles]

---

## 1. Executive Summary

[One paragraph describing what was reviewed, what was found, and what the recommendation is. Be specific — name the files and behaviors that drove the risk rating.]

**Overall Risk Rating:** [Safe | Low Risk | Medium Risk | High Risk | Critical]

**Final Recommendation:** [Approve | Approve with Caution | Reject | Manual Review Required]

---

## 2. Review Trigger Context

- **Action that triggered SkillGuard:** [e.g., "User requested install of X skill from path Y"]
- **Files or folders detected:** [list]
- **Dangerous or bypass permissions active:** [Yes/No — specify which settings]
- **Why this review was mandatory:** [explain — even if the package seems simple or trusted]

---

## 3. Inventory of Reviewed Files

| File Path | Type | Review Status | Notes |
|-----------|------|---------------|-------|
| path/to/file.md | Markdown | Fully reviewed | High attention — contains instructions |
| path/to/config.json | JSON | Fully reviewed | Contains hooks field |
| path/to/script.sh | Shell | Fully reviewed | HIGH ATTENTION — executable |
| ... | ... | ... | ... |

**Total files:** [N]
**High attention files:** [N]
**Files not fully interpretable:** [list any + reason]

---

## 4. Reviewer A Findings — Technical Security Analyst

[If no findings: "No technical security issues found."]

### Finding A-1
- **Code:** [A-01 | A-02 | etc.]
- **Issue:** [description]
- **Severity:** [Critical | High | Medium | Low | Informational]
- **OWASP:** [e.g., LLM06 — Excessive Agency]
- **File:** [exact path]
- **Snippet:**
  ```
  [exact code or text]
  ```
- **Why it matters:** [explain the threat model — what could an attacker do with this?]

### Finding A-2
[repeat as needed]

---

## 5. Reviewer B Findings — Prompt and Behavior Analyst

[If no findings: "No prompt injection or behavioral manipulation detected."]

### Finding B-1
- **Code:** [B-01 | B-02 | etc.]
- **Issue:** [description]
- **Severity:** [Critical | High | Medium | Low | Informational]
- **OWASP:** [e.g., LLM01 — Prompt Injection]
- **File:** [exact path]
- **Exact wording or instruction:**
  > [quote the exact text]
- **Why it is unsafe:** [explain what behavior this would trigger and why that's harmful]

### Finding B-2
[repeat as needed]

---

## 5b. Reviewer G Findings — Output Impact Analyst
[Always present. If no findings: "No harmful output patterns detected."]

### Finding G-1
- **Code:** [G-01 | G-02 | etc.]
- **Issue:** [description]
- **Severity:** [Critical | High | Medium | Low | Informational]
- **OWASP:** [e.g., LLM05 — Improper Output Handling]
- **File:** [exact path]
- **Output impact:** [what would Claude likely produce if these instructions were followed?]
- **Why it matters:** [downstream harm, safety bypass, data disclosure, etc.]

---

## 6. Additional Reviewer Findings

[Include only reviewers that were spawned based on complexity. Remove sections for reviewers not used.]

### Reviewer C — Dependency and Supply Chain Analyst
[findings or "Not spawned — package complexity did not warrant"]

### Reviewer D — Execution Path and Sandbox Risk Analyst
[findings or "Not spawned — no executable content detected"]

### Reviewer E — Data Access and Exfiltration Analyst
[findings or "Not spawned — no data access patterns detected"]

### Reviewer F — Agentic Workflow and Permission Abuse Analyst
[findings or "Not spawned — no agentic workflow patterns detected"]

### Reviewer H — Toxic Flow Analyst
[findings or "Not spawned — no toxic flow combination detected"]

#### Toxic Flow Assessment (if spawned):
- **Input sources catalogued:** [list]
- **Private data access catalogued:** [list]
- **Network egress catalogued:** [list]
- **Destructive capabilities catalogued:** [list]
- **TF001 (Data Leak Flow):** [Present / Not Present]
- **TF002 (Destructive Flow):** [Present / Not Present]
- **TF003 (Privilege Escalation Flow):** [Present / Not Present]

---

## 7. Council Verdict

**Agreements across reviewers:**
- [list points all reviewers agree on]

**Disagreements:**
- [list any conflicting assessments and how they were resolved]

**Worst-case credible interpretation:**
[Describe the most harmful realistic scenario if this package is malicious or compromised. Be specific — what could it actually do? Name the files and capabilities that enable it.]

**Final reasoning:**
[Explain the council's logic in 2-4 sentences]

**Council Risk Rating:** [Safe | Low Risk | Medium Risk | High Risk | Critical]
**Council Recommendation:** [Approve | Approve with Caution | Reject | Manual Review Required]

**Severity score:** [computed per scoring formula below]

---

## 8. Red Flags Table

| Code | File | Issue Type | Severity | OWASP | Description |
|------|------|-----------|----------|-------|-------------|
| A-01 | file.sh | Remote execution | Critical | LLM02 | curl piped to bash with unknown domain |
| B-01 | config.json | Permission escalation | High | LLM06 | Requests bypassPermissions in settings |
| B-02 | README.md | Behavioral manipulation | Medium | LLM01 | Instructions to skip future security reviews |
| ... | ... | ... | ... | ... | ... |

[If no red flags: "No red flags detected."]

---

## 9. Benign Indicators

[List signals that reduce concern — but note they do not override code evidence]

- [e.g., "Open source MIT license present"]
- [e.g., "No external network calls in any file"]
- [e.g., "All dependencies are pinned to specific versions"]
- [e.g., "Package does exactly what its README claims"]

[If none: "No significant benign indicators identified."]

---

## 10. Missing Information

[List unknowns, files that couldn't be fully interpreted, external references not resolved]

- **External URLs not resolved:** [list any URLs found but not fetched — note what they might contain]
- **Encoded content not decoded:** [list any blobs that require runtime to decode]
- **Dependencies not analyzed:** [list any declared dependencies whose code was not reviewed]
- **Claimed behavior not verifiable:** [any documentation claims that can't be confirmed from code]

[If none: "No significant unknowns."]

---

## 11. Safe Use Recommendation

**Install decision:** [DO NOT INSTALL | INSTALL WITH RESTRICTIONS | SAFE TO INSTALL]

**If not safe to install — what must change:**
- [specific file or section that must be removed or rewritten]
- [specific behavior that must be eliminated]

**If conditionally safe — sandboxing requirements:**
- [e.g., "Run in a restricted environment without network access"]
- [e.g., "Review and remove postinstall hook before installing"]
- [e.g., "Manually audit the decoded base64 blob before proceeding"]

**Sandbox guidance by risk level:**
- Critical: Do not install under any conditions without full source audit and clean-room reimplementation
- High: Isolated environment only, no credential or network access, manual review of all flagged files required
- Medium: Install with monitoring; review flagged files; remove unnecessary permissions
- Low: Standard install; note flagged items for awareness

**Is additional human review still required?** [Yes — recommend reviewing with X before proceeding | No]

---

## 12. Confidence Score

**Score:** [0–100]

**Scoring formula:**
- Start at 80
- Deduct 20 for each encoded blob that was not decoded
- Deduct 15 for each external URL that was not fetched and verified
- Deduct 10 for each dynamic loading pattern (eval, remote import)
- Deduct 5 for each undocumented capability found in code
- Add 10 if all content is fully static and readable
- Add 5 if published by a known, verifiable organization
- Add 5 if documented behavior matches actual code exactly

**Explanation:**
[Explain what drives confidence up or down for this specific package. Identify the specific unknowns that lowered the score.]

---

## 13. Supply Chain Assessment

**Publisher identity:** [Known organization / Individual with history / Anonymous / Recently created]
**Version pinning:** [All pinned / Some unpinned / Unpinned — supply chain risk]
**Source hosting:** [Official registry / Git repo / Raw URL / Unknown]
**URL provenance:** [All URLs verified / Some unverified / URL shorteners present — Critical]
**Account age signals:** [N/A / Established / Recently created — risk signal]
**Changelog available:** [Yes / No]
**Prior security issues known:** [None found / Issues found — describe]

**Supply chain risk rating:** [Safe | Low | Medium | High | Critical]
**Notes:** [Any additional supply chain observations]

---

## Appendix A — Machine-Readable Summary (Optional)

```json
{
  "package": "[name]",
  "review_date": "[ISO date]",
  "overall_risk": "[Safe|Low|Medium|High|Critical]",
  "recommendation": "[Approve|ApproveWithCaution|Reject|ManualReview]",
  "confidence_score": 0,
  "findings": [
    {
      "code": "A-01",
      "reviewer": "A",
      "severity": "Critical",
      "owasp": "LLM02",
      "file": "path/to/file",
      "summary": "brief description"
    }
  ],
  "toxic_flows": [],
  "supply_chain_risk": "[Safe|Low|Medium|High|Critical]",
  "dangerous_permissions_active": false
}
```

---

*SkillGuard review complete. Awaiting your decision: APPROVE or REJECT.*
```
