# gitlab-tier-flip-approval-bypass-vulnerabilty-bug-bounty-on-hackerone
Proof of Concept and automated verification suite for a Broken Access Control vulnerability in GitLab EE (v19.5.1), demonstrating deployment approval bypass via the environment tier API. Submitted to GitLab on HackerOne.


# Ethical Hacking / Security Research for Gitlab Bug Bounty on HackerOne to find vulnerabilities within the company along with Terminal Verification

# Overview
This repository contains the proof of concept (PoC), root cause analysis, and automated verification suite for a **Broken Access Control (CWE-284)** vulnerability discovered in **GitLab Enterprise Edition (EE v19.5.1)**.

The research demonstrates how a user who has **deploy access** to a tier-protected environment (e.g., `production`) but lacks **deployment approval rights** can bypass the required approval gates entirely using a single REST API call. By changing the environment's `tier` attribute (`PUT /api/v4/projects/:id/environments/:environment_id?tier=development`), the environment drops out of group-level protection. This clears the pending approval requirement and allows blocked production code to execute with **zero recorded approvals**.

All testing was conducted strictly on a self-managed GitLab instance running on a local home lab environment (Ultimate Trial license).  

---

## 1. HackerOne Report #4152812 Submission & Triage Dashboard
A screenshot of the HackerOne bug bounty portal displaying submitted report #4152812 ("Broken Access Control: bypass deployment approvals on tier-protected environments via environment tier API") for the GitLab managed program. It captures the submitted report details, triage status, and timeline.

<img width="1248" height="736" alt="image" src="https://github.com/user-attachments/assets/d9e469d0-af40-4e5d-b1b6-d013c6df8b52" />

---

### 2. Claude Code Research & Script Generation Output
A terminal capture showing the AI-assisted research phase in Claude Code, where the automated PoC script (poc_tier_flip_approval_bypass.sh) and initial report draft (REPORT-DRAFT.md) were generated, structured, and verified against the local GitLab instance.

<img width="2497" height="1450" alt="image" src="https://github.com/user-attachments/assets/d56e179d-4bc3-44f9-900e-532499a7498c" />

---

### 3. Vulnerability Execution Chain & Root Cause Breakdown
A detailed breakdown of the live verification results and technical root cause analysis (pointing to ee/app/policies/ee/environment_policy.rb), illustrating how a standard environment tier update API call defeats group-level deployment approval gates.

<img width="2048" height="1202" alt="image" src="https://github.com/user-attachments/assets/7bd532c0-6388-4ec8-9fa0-57cf2ec1c1cc" />

---

### 4. Live Video Walkthrough: Control Run & Tier-Flip Approval Bypass
A video recording of the complete attack chain. It demonstrates the initial 400 Bad Request control check when attempting to deploy without approval, followed by the PUT request mutating the environment tier to development, and the subsequent 200 OK execution bypassing deployment gates

https://www.youtube.com/watch?v=TCsrVp7dILc

---

### 5. PowerShell Verification Output — Step-by-Step Attack Chain
Terminal output from verify-all.ps1 showing real-time REST API requests and responses across all 8 steps of the PoC (group protection setup, pipeline blocking, environment tier mutation, and job playback execution).

<img width="1280" height="764" alt="image" src="https://github.com/user-attachments/assets/ae020d02-cdfb-4347-a05a-5ed1080e1337" />

---

### 6. PowerShell Verification Output — Final Results & PASS Assertions
Terminal summary screen showing all automated checks passing ([PASS] 1 through [PASS] 8), confirming instance version 19.5.1-ee and logging the complete transcript to verify-run.txt.

<img width="1280" height="299" alt="image" src="https://github.com/user-attachments/assets/0bcc079f-701d-4a90-aa13-c732e674472f" />

---

### 7. verify-all.ps1 (Automated End-to-End Proof of Concept & Verification Suite)
An automated PowerShell script that programmatically executes and validates all 8 steps of the GitLab EE tier-flip deployment approval bypass vulnerability. 

The script establishes a test environment, configures group-level protected environment tiers with approval rules, pushes a CI/CD pipeline as an unauthorized developer, asserts that the control gate successfully blocks execution (returning a 403/400 error), performs the REST API tier-mutation attack (PUT /environments/{id}?tier=development), and asserts that the blocked job successfully replays and achieves full deployment success with zero approvals recorded. All API transactions and test assertions are automatically logged to a verification transcript.

### Repository Files

* **Automated Verification Script:** [`verify-all.ps1`](verify-all.ps1)
* **Shell PoC Script:** [`poc_tier_flip_approval_bypass.sh`](poc_tier_flip_approval_bypass.sh)
* **HackerOne Final Report:** [`REPORT-FINAL.md`](REPORT-FINAL.md)
* **Verification Transcript:** [`verify-run.txt`](evidence/verify-run.txt)
* **Proof of Concept Video:** [`Gitlabs POC Recording.mp4`](Gitlabs%20POC%20Recording.mp4)

---

### 8. HackerOne Security Report — Full Technical Submission
The full bug bounty report submitted to the GitLab Security Team on HackerOne. It documents the vulnerability summary, preconditions, step-by-step reproduction commands, technical root cause in ee/app/policies/ee/environment_policy.rb, control vs. bypass API comparison, expected vs. actual behavior, and prior-art duplicate checking.

> NOTE! Thanks for submitting a report! Please note that initial triage is handled by HackerOne staff. They are identified with a `HackerOne triage` badge and will escalate to the GitLab team any valid security issue or any questions that they need answered to properly handle your report.

### Summary

In GitLab EE (Premium+), a user who has **deploy** access to a tier-protected environment but is **not** covered by its deployment approval rule can bypass the approval requirement entirely by changing the environment's **tier** through the standard environment-update API (`PUT /projects/:id/environments/:environment_id?tier=<other tier>`).

Group-level protected environments protect environments *by tier*. The EE environment policy only blocks `update_environment` when the user is "protected from" the environment — i.e. when the user **lacks deploy access** to at least one protecting protected-environment. A user who *does* have deploy access therefore passes the policy check, and nothing distinguishes `tier` — the exact attribute the protection keys on — from innocuous attributes like `description` or `external_url`.

Once the tier no longer matches, the environment stops being protected: the pending-approval count of any in-flight deployment drops to 0, the approval-blocked job becomes playable, and the deployment runs having received **zero approvals**. This research was AI-assisted (Claude Code); every request below was executed and verified by me on my own self-managed instance, with a control run and full screen recording.

### Steps to reproduce

Preconditions (all on my own self-managed instance):

1. GitLab EE with an active license (deployment approvals + protected environments available).
2. A **group-level protected environment** on tier `production` with:
   - Deploy access level: `Developers + Maintainers (30)`
   - Approval rule: `Maintainers (40)` requiring **1 approval**
3. Two users:
   - `attacker` — Developer (30) in the group and project. Covered by the deploy access level, **not** covered by the approval rule (cannot approve).
   - a Maintainer who would be the intended approver.

Reproduction:

1. As `attacker`, push a branch and a CI job deploying to a new environment on the protected tier:

   ```yaml
   deploy-payments:
     stage: deploy
     script: echo deploying payments
     environment:
       name: payments
       deployment_tier: production
     when: manual
   ```

2. Start the pipeline: `POST /api/v4/projects/<id>/pipeline?ref=<branch>`.
   A deployment for `payments` is created with status **`blocked`**, waiting for the Maintainer approval.

3. **Control** — `attacker` tries to run the job while approval is pending:

   ```
   POST /api/v4/projects/<id>/jobs/<job_id>/play
   → 400 {"message":"400 Bad request - Unplayable Job"}
   ```

   The approval gate correctly blocks execution.

4. **Bypass** — `attacker` changes the environment's tier with the ordinary environment-update API:

   ```
   PUT /api/v4/projects/<id>/environments/<environment_id>?tier=development
   → 200 OK
   ```

   No approval has been cast and `attacker` holds no approval rights, but the request is accepted.

5. `attacker` plays the same job again:

   ```
   POST /api/v4/projects/<id>/jobs/<job_id>/play
   → 200 OK
   ```

   The job runs end-to-end and the deployment transitions to **`success` with zero approvals recorded** in `deployment_approvals`.

### Impact

Any user who is allowed to **deploy** to a tier-protected environment — the exact population deployment approvals are designed to gate — can silently remove that environment from protection and execute their deployment without the required approvals. This breaks segregation of duties on protected tiers (e.g. `production`), allowing unauthorized code deployment to approval-gated environments with a single standard API call, with no CI-file changes and no Maintainer involvement. Approval-bypass issues in this feature area have previously been treated as security issues and assigned CVEs (e.g. CVE-2023-4700, CVE-2026-86340/86341).

### Examples

Reproduced on a local, licensed self-managed instance. Not reproducible on GitLab.com without violating the `Rules of Engagement` (it requires a licensed self-managed instance with group-level protected environments), so no GitLab.com project path is provided.

Tested on GitLab EE **19.5.1** (self-managed). A script that performs the full sequence (`poc_tier_flip_approval_bypass.sh`) and a PowerShell verification run with PASS/FAIL checks (`verify-all.ps1`) are included as attachments.

### What is the current *bug* behavior?

`PUT /projects/:id/environments/:environment_id?tier=...` is accepted for a user who has deploy access but no approval rights, even when the environment is currently protected *by that very tier* and a deployment is blocked awaiting approvals. After the change, `environment#protected?` returns false, `deployment#pending_approval_count` returns 0, `Ci::Build#playable?` (EE: `super && !waiting_for_deployment_approval?`) flips to true, and the blocked deployment executes with no approvals.

Relevant code:

- `ee/app/policies/ee/environment_policy.rb` — `prevent :update_environment` only under `protected_environment` = `protected_from?(user)`
- `app/models/ee/environment.rb#protected_from?` — false whenever the user has deploy access to all protecting protected-environments
- `app/services/environments/update_service.rb` — `tier` is in `ALLOWED_ATTRIBUTES`
- `app/models/ee/environment.rb` / `ee/app/models/ee/deployment.rb` — `pending_approval_count` returns 0 when the environment is no longer protected

Actual results (control vs. bypass, same user, same job, one API call apart):

```text
POST /jobs/<id>/play   → 400 Bad request - Unplayable Job   (before tier change)
PUT  /environments/<id>?tier=development → 200 OK
POST /jobs/<id>/play   → 200 OK                             (after tier change)
deployment status      → success   (deployment_approvals: none)
```

### What is the expected *correct* behavior?

Changing the tier (or any attribute that protected-environment matching keys on) of an environment matched by a protected environment should require `admin_protected_environments` (Maintainer/Owner configuration rights) regardless of the requester's deploy access — or the update should be rejected while the environment is matched by any protected environment. A deploy-accessible user without approval rights must not be able to detach an approval-gated environment from its protection, and a blocked deployment must not become playable as a side effect of a metadata update.

### Relevant logs and/or screenshots

Screen recording attached (`Gitlabs POC Recording.mp4`) showing the full control-then-bypass flow on my own instance. Full API transcript attached (`verify-run.txt`), including the PASS/FAIL verification run:

```text
[PASS] 1  Group PE has deploy 30 + approval 40x1                  (200)
[PASS] 2  Attacker pushed CI job                                  (201)
[PASS] 3  Deployment created and BLOCKED awaiting approval        (blocked)
[PASS] 4  Play BLOCKED while approval pending                     (400)
[PASS] 5  Tier changed via plain environment API                  (200)
[PASS] 6  Play SUCCEEDS after tier flip                           (200)
[PASS] 6b Job executed end-to-end                                 (success)
[PASS] 7  Deployment SUCCESS with zero approvals                  (success)
[PASS] 8  Version captured 19.5.1-ee
```

### Output of checks

This bug does **not** happen on GitLab.com — it was reproduced on a self-managed instance. `gitlab:check` reports a healthy instance.

#### Results of GitLab environment info

```text
System information
System:
Proxy:		no
Current User:	git
Ruby Version:	3.3.12
Gem Version:	3.7.2
Bundler Version:4.0.18
Rake Version:	13.4.2
Redis Version:	7.2.16
Sidekiq Version:7.3.9

GitLab information
Version:	19.5.1-ee
Revision:	48423440144
DB Adapter:	PostgreSQL
DB Version:	17.11
URL:		http://gitlab.lab.local
Elasticsearch:	no
Geo:		no
Using LDAP:	no
Using Omniauth:	yes
```

License: active Ultimate trial (deployment approvals and protected environments enforced during testing).

### AI disclosure

**This security research was AI-assisted (Claude Code, terminal agent). All conclusions were verified by me:** I executed every request myself on my own self-managed GitLab instance, confirmed the control run (blocked) and the bypass run (success) on screen, and recorded the entire reproduction on video. No automated scanning was used and nothing was tested against GitLab.com or any instance I do not own.

#### Prior-art / duplicates checked

- gitlab-org/gitlab#628468 — group-tier protection bypass via **CI-config** `deployment_tier` changes (deploy access + environment-scoped variables). This report is a **different vector**: the environment-update API, exercised by a user who already holds deploy access, defeating deployment **approvals** with no CI/code change.
- gitlab-org/gitlab#443683 / #444256 — per-rule approval counting semantics (intended behavior), unrelated.
- Known fixed CVEs in this area — CVE-2023-4700 (approval rules bypass), CVE-2026-86340/86341 (approver deletion), CVE-2026-1752 (developer modifies protected-environment settings), CVE-2026-0934 (custom roles) — all distinct from this vector.





