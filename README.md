# Ransomware Toolbox

Incident response decision support tool for ransomware containment, built to reduce errors, accelerate decisions, and structure response under pressure.

## What This Is

It is a context-aware containment plan and recovery plan (depending on the circumstance)
1. **Kill chain stage** — where is the attacker right now?
2. **Infrastructure** — what environment are you defending?
3. **Observed TA activity** — what have you seen so far?
4. **Available security tools** — what can you actually execute with?
5. **Ransomware group** *(optional)* — known TTP boost for group-specific prioritisation

The output is a scored, filtered, sequenced action plan with dependency warnings, decision gate alerts, and a live incident log. 

## Why It Exists

Ransomware IR fails in predictable ways:

- Responders execute actions in the wrong order (credential reset before vault isolation → attacker reads new passwords)
- Teams duplicate effort under pressure (two people blocking the same ports via different methods)
- Critical actions get missed because they're not obviously applicable (AD↔Entra sync still running during rotation → dirty hashes replicate to cloud)
- Decisions are made based on generic playbooks that don't account for the specific group, infrastructure, or stage

This tool addresses all four by encoding those dependencies, conflicts, and group-specific intelligence directly into the action cards — surfaced exactly when they're relevant, not buried in a 40-page plan.

## Features

### Triage & Assess
- **Kill chain stage selection** — Initial Access, Lateral Movement, Pre-Encryption, Active Encryption, Post-Encryption
- **"I Don't Know" mode** — surfaces all containment actions matched to selected infrastructure when the stage is unclear; zero paralysis
- **Infrastructure context** — Windows/AD, VMware/ESXi, Cloud (AWS/Azure/GCP), SaaS/M365/Entra, Federated Identity, Mixed
- **Observed TA activity** — RDP, RMM Tools, Data Exfiltration, Credential Dumping, DC Access, Backup Access
- **Threat group selection** — 10 groups with TTP-based prioritisation boost

### Containment Action Plan
- **68 containment actions** across 10 categories
- **CES scoring** (Containment Effectiveness Score, 1–4) — balances effectiveness against business impact
- **Priority action pinning** — critical actions float to the top regardless of filters, phase-conditional and TA-conditional
- **Dependency warnings** — 6 sequencing chains with inline blocking alerts (e.g., isolate PAM vault before any credential rotation)
- **Decision gate alerts** — mutually exclusive strategy detection (e.g., QoS delay tactic vs. active blocking)
- **Redundancy notes** — cross-references between overlapping actions to prevent double effort
- **Kill chain bar** — visual progress indicator across MITRE-mapped stages
- **Category and stage filters** — drill into specific action types without losing context
- **Business impact / effort toggles** — per-action overrides for local context calibration
- **"Not Applicable" flagging** — mark out-of-scope actions with automatic incident log entry
- **Lateral movement observation banners** — highlighted warnings on Host Isolation and Credential Reset when lateral movement is the declared stage

### Incident Log
- **Auto-logging** — every action completed, flagged N/A, or phase change is timestamped automatically
- **Manual notes** — free-text entries for observations, decisions, and evidence timestamps
- **Persistent across session** — auto-saves to localStorage

### Recovery Plan
- **Zone-based model** — Red (confirmed infected), Yellow (at risk), Green (clean rebuild base)
- **5 assessment sections** — Data Scope, Evidence Preservation, Insurance & Legal, Backup Status, Recovery Capability
- **12 recovery actions** — from KRBTGT reset to dependency-ordered restore sequencing
- **Insurance guidance** — triggered by cyber insurance selection

### Summary & Export
- **Live progress tracking** — in-scope actions completed vs. remaining, N/A count
- **N/A section** — full list of excluded actions with log documentation
- **Plain-text report export** — timestamped incident report including completed, pending, N/A, and manual notes
- **Session save/load** — export and restore session state as JSON
- **Reset button** — full state wipe with confirmation prompt

## Threat Groups Covered

| Group | Primary Sectors | Kill Chain Focus |
|---|---|---|
| LockBit 3.0 | Manufacturing, Finance, Healthcare | Lateral Movement → Encryption |
| ALPHV / BlackCat | Healthcare, Finance, Energy | Pre-Encryption, Active Encryption |
| Akira | Education, Finance, Manufacturing | Initial Access → Pre-Encryption |
| Play | Government, Healthcare, Retail | Lateral Movement → Pre-Encryption |
| Cl0p | Finance, Legal, Government | Initial Access, Pre-Encryption |
| Black Basta | Healthcare, Manufacturing, Construction | Lateral Movement → Encryption |
| Medusa | Healthcare, Education, Government | Initial Access → Pre-Encryption |
| Rhysida | Healthcare, Education, Government | Initial Access → Pre-Encryption |
| Royal / BlackSuit | Healthcare, Manufacturing | Lateral Movement, Pre-Encryption |
| Scattered Spider | Hospitality, Finance, Insurance | Initial Access, Lateral Movement |

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
| Evidence & Intelligence | 1 | TA log and tool evidence collection |

## Dependency Chains

Actions with sequencing risk surface an inline warning and are blocked from being checked until prerequisites are met:

| Action | Must Follow | Risk If Wrong Order |
|---|---|---|
| ID 11 — Service Accounts Rotate | ID 60 — Isolate PAM Vault | New service account credentials written into compromised vault |
| ID 13 — Rotate All Privileged | ID 60, ID 30, ID 5 | Vault poisoning; credentials replicate back to Entra via live sync |
| ID 29 — Privileged Password Rotations | ID 60, ID 30, ID 5 | Same as above |
| ID 23 — ESXi execInstalledOnly | ID 19 — Unmount LUN/Datastore | Running ransomware retains write access to mounted VMDKs |
| ID 25 — Harden Virt Credentials | ID 30 — Use Local Accounts | AD integration still live negates the credential hardening |
| ID 67 — Revoke OAuth App Registrations | ID 53 — Revoke Refresh Tokens | OAuth app re-issues tokens silently after user session revocation |

## SaaS / Vishing Scenario Coverage (ShinyHunters / Scattered Spider Profile)

The toolbox includes dedicated actions for identity-driven attacks that use no malware:

- **ID 33** — Pause MFA device enrollment entirely across Okta / Entra / Google Workspace
- **ID 61** — Lock external SSPR portals and harden help desk verification (technical layer)
- **ID 66** — Activate Help Desk Shields-Up Protocol (human/procedural layer) — video ID verification, out-of-band manager approval, vendor impersonation handling
- **ID 67** — Audit and revoke unauthorised OAuth app registrations across GWS, Entra, Okta, Salesforce
- **ID 68** — Restrict SaaS-native bulk export capabilities: Google Takeout, SharePoint PowerShell, Salesforce Data Loader, Atlassian exports
- **ID 69** — Scan for and remediate exposed credentials in source code, CI/CD pipelines, and config files
- **ID 53** — Platform-specific session revocation (Okta, Entra ID, Google Workspace)

## File Structure

ransomware-ir-toolbox-v6.html ← Application (open in browser — requires the two JSON files)
containment-actions.json ← Containment data (68 actions, 10 threat groups, kill chain stages)
recovery-actions.json ← Recovery plan data (sections, assessment questions, 12 recovery actions)
README.md ← This file

The HTML file loads both JSON files at startup. **All three files must be in the same directory.** The JSON files are the source of truth for all action data — edit them to add, update, or remove actions without touching the application code.

## Usage

### Running the tool

Clone or download the repository so all three files are in the same directory, then open the HTML file in any modern browser:

```bash
git clone https://github.com/your-username/ransomware-ir-toolbox
cd ransomware-ir-toolbox
open ransomware-ir-toolbox-v6.html
# or
firefox ransomware-ir-toolbox-v6.html
# or double-click the file in your file manager
```

> **Important:** The HTML file fetches `containment-actions.json` and `recovery-actions.json` at load time. Both JSON files must be present in the same directory as the HTML file, or the tool will not load any actions.
>
> If you see an empty action list, check that both JSON files are alongside the HTML and that your browser is not blocking local file access. In Chrome, you may need to use a simple local server: `python3 -m http.server 8080` then navigate to `http://localhost:8080`.

No build step, no package manager, no server required beyond a basic file server for local use.

### Customising actions

The containment and recovery data live in `containment-actions.json` and `recovery-actions.json`. Edit those files directly — the application reads them fresh on each page load. Each action follows this schema:


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

Field reference:

| Field | Values | Notes |
|---|---|---|
| `effectiveness` | `High`, `Medium-High`, `Medium`, `Low-Medium`, `Low` | Drives CES score if `ces` not set |
| `business_impact` | `High`, `Medium-High`, `Medium`, `Low-Medium`, `Low` | Combined with effectiveness for CES |
| `phases` | `initial_access`, `lateral_movement`, `pre_encryption`, `active_encryption`, `post_encryption` | When the action is relevant |
| `infra` | `windows_ad`, `vmware`, `cloud`, `saas`, `federated`, `mixed` | Empty array = appears for all infra |
| `ta` | `rdp`, `rmm`, `exfil`, `cred_dump`, `dc_access`, `backup_access`, `unknown` | Empty array = appears for all TA activity |
| `killchain` | `recon`, `initial_access`, `execution`, `persistence`, `privilege_esc`, `lateral`, `collection`, `impact` | MITRE kill chain stage |
| `ces` | `1`–`4` | Override CES score directly. 4=Optimal, 1=Low |
| `depends_on` | `[id, id, ...]` | IDs that must be completed first |
| `decision_gate` | `true` | Marks mutually exclusive strategy actions |

### Adding a ransomware group


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


`priority_actions` lists the action IDs that receive a sort boost when this group is selected — put your highest-confidence TTP-matched actions here.

## Scoring and Prioritisation

The order of actions in the containment plan is not arbitrary. Every action is ranked using a multi-layer prioritisation system that combines a static score, context bonuses, and hard overrides. Understanding how it works helps when adding new actions or interpreting why a particular action appears at the top.

### Containment Effectiveness Score (CES)

CES is the primary sort key for every action. It is a 1–4 integer that balances how effective an action is against how disruptive it is to the business.

| Score | Label | Meaning |

| 4 | Optimal | High effectiveness, low business impact — execute these first |
| 3 | Good | High effectiveness with moderate impact, or essential emergency actions |
| 2 | Moderate | Medium effectiveness or high-impact action that needs deliberate judgement |
| 1 | Low | Low effectiveness or low-value activity in the current context |

How CES is calculated:

Each action has two fields: `effectiveness` and `business_impact`. Both are five-tier scales (`High`, `Medium-High`, `Medium`, `Low-Medium`, `Low`). The formula rewards high effectiveness and penalises high business impact, so an action that is highly effective but also highly disruptive scores lower than one that is equally effective but low-impact.

If you set the `ces` field directly on an action in the JSON, that value is used as-is and the formula is bypassed. This allows calibration of individual actions where the formula does not produce the right result for your environment.

The CES score is visible on every action card as a labelled pill (Optimal / Good / Moderate / Low) with a dot indicator. Users can also override the business impact per-action in the UI using the impact toggle, which recalculates the effective CES for that session without editing the JSON.

Full Sort Order

Actions are ranked by a composite score computed as follows, in descending priority:

1. Priority action pinning — actions flagged as priority for the current phase and context float to the very top, regardless of CES or any other factor. These are not sortable — they are always first.

2. CES (1–4) — the primary sort key for all non-pinned actions. A CES=4 action always outranks a CES=3 action, and so on.

3. TTP match bonus — if the selected ransomware group has TA tags (e.g., `cred_dump`, `exfil`) and an action's `ta` array contains one or more of those tags, a small bonus is added per matching tag. This causes group-relevant actions to surface above generic ones with the same CES.

4. Group priority boost — if the selected ransomware group lists an action in its `priority_actions` array, that action receives an additional boost. This is the mechanism by which group-specific intelligence influences ranking: an IR team handling LockBit sees DC isolation and backup actions boosted; one handling Scattered Spider sees OAuth revocation and help desk protocols boosted.

5. Phase match bonus — actions whose `phases` array includes the declared kill chain stage receive a small additional bonus over cross-phase actions that appear only because of mandatory pinning.

Priority Action Pinning

Certain actions are pinned to the top of the plan unconditionally — they appear before any CES-scored actions and cannot be filtered out. Pinning is rule-based and context-conditional:

| Trigger condition | Pinned actions |
|---|---|
| Phase: active_encryption or post_encryption (any infra) | Isolate Host, Rotate All Privileged, Block ALL Traffic |
| Phase: pre_encryption (any infra) | Isolate Host, Rotate All Privileged |
| Phase: pre_encryption + saas or cloud infra | SaaS Compromise — Revoke Refresh Tokens |
| Phase: pre/lateral/initial + federated infra | Block Compromised AWS Federated User |
| Phase: lateral_movement + windows_ad infra + dc_access observed | Isolate One Domain Controller |

These rules encode the most time-critical actions for each scenario — actions where a wrong decision or omission has an outsized consequence. They are always shown with a "Priority Action" badge and are checked separately from the standard progress counter.

Dependency Blocking

Actions with a `depends_on` relationship display a sequencing warning on the card and the checkbox is disabled until all prerequisite actions are marked complete. This is not advisory — it is a hard block in the UI. The dependency system encodes ordering constraints where the wrong sequence creates a worse outcome than not acting at all (for example, rotating credentials into a compromised vault before isolating it).

Decision Gates

One action — Firewall QoS Delay (ID 51) — is marked as a decision gate. When it appears in the plan, the card carries a prominent warning that this is a mutually exclusive strategy: choosing to delay and investigate (this action) is incompatible with choosing to block and evict (DNS Sinkhole, Egress Filtering, Block ALL Traffic, Block TOR Ranges). The notes make the tradeoff explicit and prevent a responder from running both strategies simultaneously.

Design Principles

1. Context over comprehensiveness. Every filter — phase, infra, TA activity, group — narrows the action list to what is actually applicable. A responder dealing with Cl0p on SaaS gets ~14 targeted actions, not 68 generic ones.

2. Errors are caught before they happen. Dependency chains block the wrong order of operations at the UI level. Conflict notes prevent double effort. Redundancy flags prevent wasted time. The tool fails safe.

3. Speed is a safety property. Ransomware IR is a time-critical discipline. Every second spent searching a playbook for the right action is a second the attacker is still running. The CES scoring, priority pinning, and "I Don't Know" mode are all designed to produce an immediately actionable top-5 within 60 seconds of opening the tool.

4. The log is the audit trail. Every meaningful action — phase selection, action completed, action flagged N/A — is auto-logged with a timestamp. The session can be exported and loaded by a shift handoff. The exported report is a complete incident timeline.

5. JSON-driven. The data layer is fully separated from the application layer. All actions, groups, and recovery sections live in editable JSON — no code changes required to add, remove, or update any content.

Contributing

Contributions to the action data, group profiles, and detection logic are welcome. The most useful contributions are:

- **New threat group profiles** — especially groups with distinct TTP signatures not covered by existing entries
- **New containment actions** — particularly for cloud-native (GCP, Azure), OT/ICS, or SaaS-specific scenarios
- **Notes improvements** — more precise step-by-step guidance, platform-specific commands, updated CVE references
- **Dependency additions** — sequencing relationships between actions that have real-world ordering risk
- **Conflict documentation** — notes on redundancy or mutual exclusion between actions

License

MIT License — see [LICENSE](LICENSE) for details.

Disclaimer

**This tool is designed to support incident response activities and is not a substitute for professional incident response expertise. Action notes and guidance reflect general industry practice and the author's operational experience. They do not constitute legal, compliance, or professional services advice. Always validate actions against your specific environment before execution. Some actions carry significant business impact — review carefully before applying in production.**
