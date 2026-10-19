# FortiAnalyzer 7.6 Analyst — Weak Areas Study Notes

Source: questions missed across Practice Tests II, III, IV, V, VI.
Grouped by topic so you can drill the areas you keep getting wrong.

---

## 1. Reports, Datasets, Charts, Macros

### Key rule: CHART exports carry their DATASETS. Templates don't.
When you export:
- **Chart → dataset comes with it** ✅ (this is the correct answer you missed)
- **Template → just the layout, no charts, no datasets**
- **Datasets** are not directly exportable as standalone items

**Why:** a chart is *useless without its data*, so export bundles the dataset. A template is just a skeleton.

### Datasets
- A **dataset is an SQL SELECT query** — that's it.
- It *retrieves* data from the database. The result populates the chart.
- Datasets do NOT define chart types, do NOT provide report layouts, do NOT set template content.

Memory hook: **Dataset = the question you ask the DB. Chart = how you draw the answer.**

### Macros
| True | False |
|---|---|
| Macros are **abbreviated dataset queries** | Macros are predefined templates ❌ |
| Macros **do NOT need to be tied to a chart** | Macros only work on FortiGate ADOM ❌ |
| Macros are **ADOM-specific** — each ADOM has its own | Macros can't be customized ❌ |

Memory hook: **Macro = shortcut SELECT, scoped to one ADOM.**

### Chart Builder / Export features
- **Log View** → **Chart Builder** builds dataset+chart from filter results
- **FortiView** → **Export to Report Chart** does the same thing
- Both take your current filter and auto-build the chart

### SQL SELECT queries — true statement
**SELECT queries are NOT used in macros.**
Also true:
- SELECT cannot PURGE logs (that's DELETE/TRUNCATE)
- WHERE clause is optional
- SELECT does NOT display schema (that's DESCRIBE/SHOW)

### Auto-cache (for reports)
- **Reduces report generation time** for reports with long dataset assembly
- **Automatically enabled on scheduled reports** (don't need to toggle it)
- It does **NOT** shrink report size or cache in memory — caches to disk

### Why a report might be missing data
1. **Time frame wrong** — most common cause
2. **Logs overwritten by data retention policy**
Not because of: auto-cache, indexing delays (usually).

---

## 2. Event Statuses (you keep getting this wrong — memorize it)

Four statuses on FortiAnalyzer events:

| Status | Meaning |
|---|---|
| **Unhandled** | Risk **not** mitigated or contained — event is **OPEN** |
| **Contained** | Risk **source isolated** |
| **Mitigated** | Risk **blocked or dropped** ← most common correct answer |
| **(Blank)** | Other/non-security cases |

Memory hook: **U-C-M-B** → Unhandled-Contained-Mitigated-Blank.
- If question says "Mitigated" → answer is "blocked or dropped"
- If question says "action=pass, Unhandled" → answer is "still open"

---

## 3. Compromised Hosts — two things you always need

To see Compromised Hosts on FortiAnalyzer:

1. **Enable Web Filter in FortiGate firewall policies** and send logs to FAZ
2. **Subscribe FAZ to FortiGuard** (keeps threat DB current)

NOT needed:
- Device detection on FortiGate interfaces ❌
- All endpoints reachable by FAZ ❌ (FAZ talks to FortiGate, not endpoints)

Memory hook: **Web filter logs + FortiGuard feed = compromised host detection.**

---

## 4. Playbooks

**Two components** (memorize — these are the only valid ones):
- **Triggers** — conditions that start the workflow
- **Actions** — tasks the workflow performs

NOT playbook components:
- Workflow templates ❌
- Fabric connectors ❌ (those are a separate Fabric feature)

### "Attach Data" action in a playbook
When you see an analyst configuring "Attach Data" with a `report_uuid` placeholder:
- They're creating an **OUTPUT variable** — the playbook will populate the UUID dynamically and pass it forward
- NOT a trigger variable (triggers are in the Playbook Starter)
- NOT creating a report itself — just attaching one

---

## 5. Device Management

### Adding a FortiGate to FortiAnalyzer — two valid methods
1. **Device Manager on FAZ** — manually add FortiGate (IP + credentials)
2. **Configure remote logging on FortiGate** — FortiGate pushes logs to FAZ

NOT valid:
- "Accept All Devices" — security risk ❌
- Heartbeat port + auto-discovery — not how FAZ works ❌

### Connection status "Unauthorized"
Means: **Device tried to register but an admin hasn't accepted it yet.**
- NOT about ADOM assignment (that's later)
- NOT about pre-shared keys
- NOT about unsupported device

Fix: Go to Device Manager → accept the registration.

### Moving a FortiGate to a new ADOM → rebuild the ADOM database
Purpose: **To run reports on the device's analytics logs in the new ADOM.**
Without the rebuild, the new ADOM doesn't have the indexed data to report on.

---

## 6. ADOMs

### Modes
- **Normal** = default mode
- **Advanced** = lets you assign **individual VDOMs from one FortiGate to multiple ADOMs**
- You can change mode via **CLI or GUI** (not CLI-only)
- Disk quota is configurable in **both** modes (not fixed in normal)

### Restricting an admin to specific ADOMs
**Assign the ADOMs to the administrator's account.**

NOT:
- Trusted hosts (those restrict source IP, not ADOM scope) ❌
- Super_User profile (unrestricted access) ❌
- ADOM mode (that enables ADOMs, doesn't restrict admins) ❌

---

## 7. High Availability (HA)

### Two true statements
1. **HA syncs logs + some system/config settings**
2. **All cluster members must run the same operation mode** (all analyzer OR all collector, not mixed)

### Firmware upgrade on HA cluster
**Upgrade the SECONDARY first, then the PRIMARY.** Not both at once.
No "uninterruptible-upgrade" feature in FAZ HA.

### VRRP
FAZ HA needs **VRRP** to be permitted on the network — so some public clouds that block VRRP can't run FAZ HA.

---

## 8. Log Fetching

### Two true statements
1. **Both devices must run the same firmware version**
2. **Used to retrieve historical/archived logs** from one FAZ to another for queries/reports

### False statements
- "For redundancy" ❌ — it's **not** a redundancy/sync mechanism
- "A FAZ can be both fetch client and server at once with the same peer" ❌ — one role at a time per peer pair

Memory hook: **Fetch = pull archived logs for analysis, not sync.**

---

## 9. FortiView resolve-ip

Command:
```
config system fortiview settings
  set resolve-ip enable
end
```

**What it does: resolves BOTH source AND destination IPs to hostnames in FortiView.** (Not destination only.)

But the real recommended fix to avoid FAZ performance impact:
**Resolve IPs on the FortiGate, not on FortiAnalyzer.**

---

## 10. Log Checksum

Command:
```
set log-checksum md5
```
→ **Records MD5 hash of the log file** for integrity verification (tamper detection).

Not about encryption, not about tagging source, not about authentication code.

Variant:
- `md5-auth` = MD5 hash **+ authentication code**

---

## 11. Daemons (process responsibilities)

| Daemon | Responsibility |
|---|---|
| **logfiled** | Enforces **log file size** + disk quota |
| **sqlplugind** | Enforces **SQL database size** |
| **oftpd** | Enforces **archive file size** |
| **miglogd** | Log migration |

Memory hook: **logfiled = log files & quota. sqlplugind = database size. oftpd = archives.**

---

## 12. Raw vs Formatted Logs

- **Raw logs** = logs exactly as they appear in the log file (less human-readable)
- **Formatted logs** = structured, organized, easier to read
- Both can be viewed in GUI, both exportable to CSV
- Raw logs are **NOT CLI-only**

---

## 13. Log Forwarding

### Two true statements
1. **Client retains a local copy** after forwarding
2. **Aggregation mode** works only FAZ → FAZ (typically)

### False
- Forwarded logs CAN be filtered by criteria
- Logs are NOT real-time only — **aggregation mode batches them**

---

## 14. IPsec between FAZ and FortiGate

### Two true statements
1. Need **tunnel ID + pre-shared key** at both ends
2. **IPsec on FAZ is CLI-only** (no GUI toggle)

### False
- Must configure BOTH ends, not just FAZ
- IPsec and SSL can coexist

---

## 15. Restricting Admin by Location

**Configure trusted hosts for that administrator.**
Trusted hosts = list of allowed source IPs/hostnames for login.

NOT:
- Geo-location on interface (not granular enough)
- 2FA (adds factor, doesn't restrict location)
- ADOM per location (wrong tool for the job)

---

## 16. Filters — web login syntax

Template for matching "logins to the web interface from Laptop1, not by admin user":
```
operation-login & performed_on=="GUI(<LAPTOP-IP>)" & user!=admin
```

Key notes:
- Use `performed_on=="GUI(IP)"` — NOT `srcip==`, NOT `dstip==`
- The IP in `performed_on` is the **source (laptop) IP**
- `user!=admin` negates match (everyone EXCEPT admin)

---

## 17. System Backup Contents

Backup contains:
- ✅ **Device list**
- ✅ **System information** (config)

Does NOT contain:
- ❌ Generated reports
- ❌ Authorized device logs

Memory hook: **Backup = config only. Reports and logs are separate.**

---

## 18. FortiAnalyzer Centralizes (3 things)

1. **Graphical reporting**
2. **Content archiving / data mining**
3. **Security log analysis / forensics**

NOT:
- Network analysis (it's secondary, not centralized)
- Vulnerability assessment (different product category)

---

## 19. Incident Response & SOC

### Primary feature for managing/investigating incidents
**SOC (Security Operations Center)** — not Reports, not Logging, not System Settings.

### What FAZ's incident response actually does
**Tracks and manages incidents for SOC teams to resolve.**
- NOT apply patches
- NOT send logs back to FortiGate
- NOT shut down devices automatically

### Automated report scheduling
**Report Manager** — not Event Handler, not Log Viewer, not Playbook.

---

## 20. Outbreak Detection Service

Automatically downloads:
1. **Report Template**
2. **Event Handler**

Does NOT download:
- Incident template
- Customized playbook

---

## Exam-Day Quick-Recall Cheat Sheet

| Topic | Answer |
|---|---|
| Chart export carries... | its dataset |
| Dataset is a... | SQL SELECT query |
| Macros are... | abbreviated dataset queries, ADOM-specific |
| Event "Mitigated" means... | blocked or dropped |
| Event "Unhandled" means... | open, not mitigated |
| Event "Contained" means... | source isolated |
| Compromised hosts need... | Web Filter logs + FortiGuard subscription |
| Playbook components are... | Triggers + Actions (only) |
| Unauthorized device = ... | registration not yet accepted |
| Restrict admin to some ADOMs... | assign ADOMs to admin account |
| Restrict admin by location... | trusted hosts |
| HA upgrade order... | secondary first, then primary |
| Log fetching requires... | same firmware both sides |
| resolve-ip enables... | source+destination hostname resolution in FortiView |
| log-checksum md5... | integrity hash of log file |
| logfiled enforces... | log file size & disk quota |
| Auto-cache on scheduled reports... | already enabled |
| Missing report data... | time frame OR retention overwrote |
| Backup contains... | device list + system config (no logs, no reports) |
| SOC feature used for... | incident management & investigation |
| Report scheduling... | Report Manager |
| Outbreak Detection downloads... | Report Template + Event Handler |
| FortiAnalyzer centralizes... | reporting + archiving + log forensics |

---

## Study Method Suggestion

1. Cover the "Answer" column and test yourself on the cheat sheet
2. Re-read the Event Status table (section 2) — you missed it twice
3. Memorize the 3 daemon responsibilities — they test this with similar-looking answer choices
4. Write out the `performed_on=="GUI(IP)"` filter syntax from memory
5. List the 4 event statuses + their meanings without looking

Do a pass through this doc, then retake Practice Test II to see which topics are still shaky.
