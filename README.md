# gitlab-tier-flip-approval-bypass-vulnerabilty-bug-bounty-on-hackerone
Proof of Concept and automated verification suite for a Broken Access Control vulnerability in GitLab EE (v19.5.1), demonstrating deployment approval bypass via the environment tier API. Submitted to GitLab on HackerOne.


# Ethical Hacking / Security Research for Gitlab Bug Bounty on HackerOne to find vulnerabilities within the company along with Terminal Verification

# Overview
This repository contains the proof of concept (PoC), root cause analysis, and automated verification suite for a **Broken Access Control (CWE-284)** vulnerability discovered in **GitLab Enterprise Edition (EE v19.5.1)**.

The research demonstrates how a user who has **deploy access** to a tier-protected environment (e.g., `production`) but lacks **deployment approval rights** can bypass the required approval gates entirely using a single REST API call. By changing the environment's `tier` attribute (`PUT /api/v4/projects/:id/environments/:environment_id?tier=development`), the environment drops out of group-level protection. This clears the pending approval requirement and allows blocked production code to execute with **zero recorded approvals**.

All testing was conducted strictly on a self-managed GitLab instance running on a local home lab environment (Ultimate Trial license).  

---

### 1. Claude Code Research & Script Generation Output
A terminal capture showing the AI-assisted research phase in Claude Code, where the automated PoC script (poc_tier_flip_approval_bypass.sh) and initial report draft (REPORT-DRAFT.md) were generated, structured, and verified against the local GitLab instance.

<img width="2497" height="1450" alt="image" src="https://github.com/user-attachments/assets/d56e179d-4bc3-44f9-900e-532499a7498c" />

---

### 2. Vulnerability Execution Chain & Root Cause Breakdown
A detailed breakdown of the live verification results and technical root cause analysis (pointing to ee/app/policies/ee/environment_policy.rb), illustrating how a standard environment tier update API call defeats group-level deployment approval gates.

<img width="2048" height="1202" alt="image" src="https://github.com/user-attachments/assets/7bd532c0-6388-4ec8-9fa0-57cf2ec1c1cc" />

---

### 3. Live Video Walkthrough: Control Run & Tier-Flip Approval Bypass
A video recording of the complete attack chain. It demonstrates the initial 400 Bad Request control check when attempting to deploy without approval, followed by the PUT request mutating the environment tier to development, and the subsequent 200 OK execution bypassing deployment gates

https://www.youtube.com/watch?v=TCsrVp7dILc

---

### 4. PowerShell Verification Output — Step-by-Step Attack Chain
Terminal output from verify-all.ps1 showing real-time REST API requests and responses across all 8 steps of the PoC (group protection setup, pipeline blocking, environment tier mutation, and job playback execution).

<img width="1280" height="764" alt="image" src="https://github.com/user-attachments/assets/ae020d02-cdfb-4347-a05a-5ed1080e1337" />

---

### 5. PowerShell Verification Output — Final Results & PASS Assertions
Terminal summary screen showing all automated checks passing ([PASS] 1 through [PASS] 8), confirming instance version 19.5.1-ee and logging the complete transcript to verify-run.txt.

<img width="1280" height="299" alt="image" src="https://github.com/user-attachments/assets/0bcc079f-701d-4a90-aa13-c732e674472f" />


