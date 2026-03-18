# Ransomware Toolbox

Incident response decision support tool for ransomware containment, built to reduce errors, accelerate decisions, and structure response.

---

## What This Is

The Ransomware Toolbox generates a context-aware containment and recovery plan based on five triage inputs:

1. **Kill chain stage** — where is the attacker right now?
2. **Infrastructure** — what environment are you defending?
3. **Observed TA activity** — what have you seen so far?
4. **Available security tools** — what can you actually execute with?
5. **Ransomware group** *(optional)* — enables TTP-based prioritisation for known groups

The output is a scored, filtered, and sequenced action plan with dependency warnings, decision gate alerts, and a live incident log — all designed around a single principle: the responder should never have to think about what to do next.

---

## Features

### Triage and Assess

- **Kill chain stage selection** — Initial Access, Lateral Movement, Pre-Encryption, Active Encryption, Post-Encryption
- **"I Don't Know" mode** — surfaces all containment actions matched to the selected infrastructure when the stage is unclear, eliminating decision paralysis
- **Infrastructure context** — Windows/AD, VMware/ESXi, Cloud (AWS/Azure/GCP), SaaS/M365/Entra, Federated Identity, Mixed
- **Observed TA activity** — RDP, RMM Tools, Data Exfiltration, Credential Dumping, DC Access, Backup Access
- **Threat group selection** — 10 groups with TTP-based prioritisation boost

### Containment Action Plan

- **68 containment actions** across 10 categories
- **CES scoring** (Containment Effectiveness Score, 1–4) — balances effectiveness against business impact
- **Priority action pinning** — critical actions are pinned to the top regardless of filters, conditionally triggered by phase and observed TA activity
- **Dependency warnings** — 6 sequencing chains with inline blocking alerts (e.g., isolate PAM vault before any credential rotation)
- **Decision gate alerts** — mutually exclusive strategy detection (e.g., QoS delay tactic vs. active blocking)
- **Redundancy notes** — cross-references between overlapping actions to prevent duplicated effort
- **Kill chain bar** — visual progress indicator across MITRE-mapped stages
- **Category and stage filters** — drill into specific action types without losing context
- **Business impact and effort toggles** — per-action overrides for local context calibration
- **"Not Applicable" flagging** — mark out-of-scope actions with automatic incident log entry
- **Lateral movement observation banners** — highlighted warnings on Host Isolation and Credential Reset when lateral movement is the declared stage

### Incident Log

- **Auto-logging** — every completed action, N/A flag, and phase change is timestamped automatically
- **Manual notes** — free-text entries for observations, decisions, and evidence timestamps
- **Persistent across session** — auto-saves to localStorage

### Recovery Plan

- **Zone-based model** — Red (confirmed infected), Yellow (at risk), Green (clean rebuild base)
- **5 assessment sections** — Data Scope, Evidence Preservation, Insurance and Legal, Backup Status, Recovery Capability
- **12 recovery actions** — from KRBTGT reset to dependency-ordered restore sequencing
- **Insurance guidance** — triggered when cyber insurance is selected

### Summary and Export

- **Live progress tracking** — in-scope actions completed vs. remaining, with N/A count
- **N/A section** — full list of excluded actions with log documentation
- **Plain-text report export** — timestamped incident report including completed, pending, N/A, and manual notes
- **Session save/load** — export and restore session state as JSON
- **Reset button** — full state wipe with confirmation prompt

---

## Threat Groups Covered

| Group | Primary Sectors | Kill Chain Focus |
|---|---|---|
| LockBit 3.0 | Manufacturing, Finance, Healthcare | Lateral Movement to Encryption |
| ALPHV / BlackCat | Healthcare, Finance, Energy | Pre-Encryption, Active Encryption |
| Akira | Education, Finance, Manufacturing | Initial Access to Pre-Encryption |
| Play | Government, Healthcare, Retail | Lateral Movement to Pre-Encryption |
| Cl0p | Finance, Legal, Government | Initial Access, Pre-Encryption |
| Black Basta | Healthcare, Manufacturing, Construction | Lateral Movement to Encryption |
| Medusa | Healthcare, Education, Government | Initial Access to Pre-Encryption |
| Rhysida | Healthcare, Education, Government | Initial Access to Pre-Encryption |
| Royal / BlackSuit | Healthcare, Manufacturing | Lateral Movement, Pre-Encryption |
| Scattered Spider | Hospitality, Finance, Insurance | Initial Access, Lateral Movement |

---

## Action Categories

| Category | Actions | Coverage |
|---|---|---|
| Network Segmentation | 18 | Host isolation, VPN, firewall, lateral movement blocking, DNS sinkhole |
| Access Control | 14 | MFA hardening, SSPR lockdown, conditional access, OAuth app revocation, help desk protocols |
| Virtualization | 9 | ESXi lockdown, VMDK protection, credential hardening, Vpxuser |
| Identities | 6 | AD/Entra sync, service accounts, AWS IAM/federated, credential scanning |
| Privilege Account Management | 6 | PAM vault isolation, privileged account restrictions, credential rotation sequencing |
| Visibility | 5 | EDR deployment, logging, attack surface management, rogue device hunting |
| Application Control | 4 | ASR rules, WDAC, IFEO block, executable blocking |
| Backup | 3 | Key system isolation, DC isolation, backup dependency mapping |
| Content Filtering | 2 | RMM tool URL filtering, SaaS bulk export restriction |
| Evidence and Intelligence | 1 | TA log and tool evidence collection |

---

## Dependency Chains

Actions with sequencing risk surface an inline warning and are blocked from being checked until all prerequisites are complete:

| Action | Must Follow | Risk If Wrong Order |
|---|---|---|
| ID 11 — Service Accounts Rotate | ID 60 — Isolate PAM Vault | New service account credentials written into a compromised vault |
| ID 13 — Rotate All Privileged | ID 60, ID 30, ID 5 | Vault poisoning; credentials replicate back to Entra via live sync |
| ID 29 — Privileged Password Rotations | ID 60, ID 30, ID 5 | Same as above |
| ID 23 — ESXi execInstalledOnly | ID 19 — Unmount LUN/Datastore | Running ransomware retains write access to mounted VMDKs |
| ID 25 — Harden Virt Credentials | ID 30 — Use Local Accounts | AD integration still live negates the credential hardening |
| ID 67 — Revoke OAuth App Registrations | ID 53 — Revoke Refresh Tokens | OAuth app re-issues tokens silently after user session revocation |

---

## SaaS and Vishing Scenario Coverage

The toolbox includes dedicated actions for identity-driven attacks that use no malware, covering ShinyHunters and Scattered Spider style campaigns:

- **ID 33** — Pause MFA device enrollment entirely across Okta, Entra, and Google Workspace
- **ID 53** — Platform-specific session revocation (Okta, Entra ID, Google Workspace)
- **ID 61** — Lock external SSPR portals and harden help desk verification (technical layer)
- **ID 66** — Activate Help Desk Shields-Up Protocol (human/procedural layer) — video ID verification, out-of-band manager approval, vendor impersonation handling
- **ID 67** — Audit and revoke unauthorised OAuth app registrations across GWS, Entra, Okta, and Salesforce
- **ID 68** — Restrict SaaS-native bulk export capabilities: Google Takeout, SharePoint PowerShell, Salesforce Data Loader, Atlassian exports
- **ID 69** — Scan for and remediate exposed credentials in source code, CI/CD pipelines, and config files

---

## File Structure

```
ransomware-ir-toolbox-v6.html   Application — open in browser (requires both JSON files)
containment-actions.json        Containment data: 68 actions, 10 threat groups, kill chain stages
recovery-actions.json           Recovery plan data: sections, assessment questions, 12 recovery actions
README.md                       This file
```

The HTML file loads both JSON files at startup. **All three files must be in the same directory.** The JSON files are the source of truth for all action data — edit them to add, update, or remove actions without touching the application code.

---

## Usage

### Running the tool

Clone or download the repository, then open the HTML file in any modern browser:

```bash
git clone https://github.com/your-username/ransomware-ir-toolbox
cd ransomware-ir-toolbox
open ransomware-ir-toolbox-v6.html
```

> **Important:** The HTML file fetches `containment-actions.json` and `recovery-actions.json` at load time. Both JSON files must be in the same directory as the HTML file, otherwise the tool will load with no actions.
>
> If you see an empty action list, confirm both JSON files are present alongside the HTML. In Chrome, local file fetching may be blocked — run a simple local server instead:
> ```
> python3 -m http.server 8080
> ```
> Then navigate to `http://localhost:8080`.

No build step, no package manager, and no server required beyond a basic file server for local use.

### Customising actions

Edit `containment-actions.json` or `recovery-actions.json` directly. The application reads both files on every page load. Each action follows this schema:

```json
{
  "id": 70,
  "activity": "Your Action Title",
  "effectiveness": "High",
  "category": "Access Control",
  "business_impact": "Low",
  "notes": "Step-by-step guidance for the responder...",
  "phases": ["initial_access", "lateral_movement"],
  "infra": ["saas", "cloud"],
  "ta": ["cred_dump", "unknown"],
  "killchain": "initial_access",
  "ces": 4,
  "depends_on": [53]
}
```

**Field reference:**

| Field | Values | Notes |
|---|---|---|
| `effectiveness` | `High`, `Medium-High`, `Medium`, `Low-Medium`, `Low` | Drives CES score if `ces` is not set |
| `business_impact` | `High`, `Medium-High`, `Medium`, `Low-Medium`, `Low` | Combined with effectiveness for CES calculation |
| `phases` | `initial_access`, `lateral_movement`, `pre_encryption`, `active_encryption`, `post_encryption` | Defines when the action is relevant |
| `infra` | `windows_ad`, `vmware`, `cloud`, `saas`, `federated`, `mixed` | Empty array means the action appears for all infrastructure types |
| `ta` | `rdp`, `rmm`, `exfil`, `cred_dump`, `dc_access`, `backup_access`, `unknown` | Empty array means the action appears for all observed TA activity |
| `killchain` | `recon`, `initial_access`, `execution`, `persistence`, `privilege_esc`, `lateral`, `collection`, `impact` | MITRE kill chain stage mapping |
| `ces` | `1`–`4` | Directly overrides the calculated CES score. 4 = Optimal, 1 = Low |
| `depends_on` | `[id, id, ...]` | IDs of actions that must be completed first |
| `decision_gate` | `true` | Marks mutually exclusive strategy actions |

### Adding a ransomware group

```json
{
  "id": "my_group",
  "name": "GroupName",
  "sector": "Healthcare, Finance",
  "desc": "Brief description of the group's TTPs and characteristics.",
  "ttps": ["TTP 1", "TTP 2", "TTP 3"],
  "ta_tags": ["cred_dump", "exfil"],
  "kill_chain_focus": ["lateral_movement", "pre_encryption"],
  "infra_targets": ["windows_ad", "vmware"],
  "priority_actions": [4, 7, 13, 29, 43, 60]
}
```

`priority_actions` lists the action IDs that receive a sort boost when this group is selected. Add the highest-confidence TTP-matched actions here.

---

## Scoring and Prioritisation

The order of actions in the containment plan is not arbitrary. Every action is ranked using a multi-layer system that combines a static score, context bonuses, and hard overrides.

### Containment Effectiveness Score (CES)

CES is the primary sort key for every action — a 1–4 integer that balances containment effectiveness against business disruption.

| Score | Label | Meaning |
|---|---|---|
| 4 | Optimal | High effectiveness, low business impact — execute these first |
| 3 | Good | High effectiveness with moderate impact, or essential emergency actions |
| 2 | Moderate | Medium effectiveness, or high-impact actions that require deliberate judgement |
| 1 | Low | Low effectiveness or low-value activity in the current context |

Each action has two fields — `effectiveness` and `business_impact` — both on a five-tier scale. The formula rewards high effectiveness and penalises high business impact. Setting the `ces` field directly on an action bypasses the formula entirely, which is useful for calibrating actions where the calculated score does not reflect local conditions.

The CES score is shown on every action card as a labelled pill with a dot indicator. Responders can also override the business impact per action using the UI toggles, which recalculates the effective CES for that session without modifying the JSON.

### Full Sort Order

Actions are ranked by a composite score in descending priority:

1. **Priority action pinning** — pinned actions always appear first, regardless of CES or any other factor.
2. **CES (1–4)** — the primary sort key for all non-pinned actions.
3. **TTP match bonus** — actions whose `ta` array matches one or more of the selected group's TA tags are ranked above generic actions at the same CES level.
4. **Group priority boost** — actions listed in the selected group's `priority_actions` array receive an additional boost, surfacing group-specific intelligence at the top of the plan.
5. **Phase match bonus** — actions directly applicable to the declared kill chain stage rank above cross-phase actions at the same CES level.

### Priority Action Pinning

Certain actions are pinned unconditionally based on phase and context. They appear before all CES-scored actions and cannot be filtered out:

| Trigger condition | Pinned actions |
|---|---|
| Phase: active_encryption or post_encryption (any infra) | Isolate Host, Rotate All Privileged, Block ALL Traffic |
| Phase: pre_encryption (any infra) | Isolate Host, Rotate All Privileged |
| Phase: pre_encryption + SaaS or Cloud infra | SaaS Compromise — Revoke Refresh Tokens |
| Phase: pre_encryption, lateral_movement, or initial_access + federated infra | Block Compromised AWS Federated User |
| Phase: lateral_movement + Windows AD infra + DC access observed | Isolate One Domain Controller |

Pinned actions are displayed with a "Priority Action" badge and tracked separately in the progress counter.

### Dependency Blocking

Actions with a `depends_on` relationship display a sequencing warning and their checkbox is disabled until all prerequisite actions are marked complete. This is a hard UI block, not advisory guidance. The dependency system prevents ordering mistakes where the wrong sequence produces a worse outcome than no action at all — for example, rotating credentials into a compromised vault before isolating it hands the new passwords directly to the attacker.

### Decision Gates

The Firewall QoS Delay action (ID 51) is marked as a decision gate. It represents a mutually exclusive strategy: choosing to delay and investigate is incompatible with choosing to block and evict. When this action appears in the plan, the card carries a clear warning explaining the tradeoff and the specific actions it conflicts with.

---

## Design Principles

**1. Context over comprehensiveness.** Every filter — phase, infra, TA activity, group — narrows the action list to what is actually applicable. A responder handling Cl0p on SaaS gets around 14 targeted actions, not 68 generic ones.

**2. Errors are caught before they happen.** Dependency chains block incorrect sequencing at the UI level. Conflict notes prevent duplicated effort. Redundancy flags prevent wasted time. The tool fails safe.

**3. Speed is a safety property.** Ransomware IR is time-critical. Every second spent searching a playbook is a second the attacker is still running. The CES scoring, priority pinning, and "I Don't Know" mode are designed to surface an immediately actionable top five within 60 seconds of opening the tool.

**4. The log is the audit trail.** Every meaningful action — phase selection, action completed, action flagged N/A — is auto-logged with a timestamp. Sessions can be exported and loaded by a shift handoff. The exported report is a complete incident timeline.

**5. JSON-driven.** The data layer is fully separated from the application layer. All actions, groups, and recovery sections live in editable JSON files — no code changes are required to add, update, or remove content.

---

## Contributing

Contributions to the action data, group profiles, and recovery guidance are welcome. The most useful contributions are:

- **New threat group profiles** — particularly groups with distinct TTP signatures not covered by existing entries
- **New containment actions** — especially for cloud-native (GCP, Azure), OT/ICS, or SaaS-specific scenarios
- **Notes improvements** — more precise step-by-step guidance, platform-specific commands, and updated references
- **Dependency additions** — sequencing relationships between actions where the wrong order creates real risk
- **Conflict documentation** — notes on redundancy or mutual exclusion between actions

Keep action notes tactical and operational. The audience is a responder under pressure, not a policy author.

---

## License

MIT License — see [LICENSE](LICENSE) for details.

---

## Disclaimer

**This tool is designed to support incident response activities and is not a substitute for professional incident response expertise. Action notes and guidance reflect general industry practice and the author's operational experience. They do not constitute legal, compliance, or professional services advice. Always validate actions against your specific environment before execution. Some actions carry significant business impact — review carefully before applying in production**.

