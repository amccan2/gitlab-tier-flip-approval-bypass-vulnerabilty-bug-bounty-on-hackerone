# gitlab-tier-flip-approval-bypass-vulnerabilty-bug-bounty-on-hackerone
Proof of Concept and automated verification suite for a Broken Access Control vulnerability in GitLab EE (v19.5.1), demonstrating deployment approval bypass via the environment tier API. Submitted to GitLab on HackerOne.


# Ethical Hacking / Security Research for Gitlab Bug Bounty on HackerOne to find vulnerabilities within the company along with Terminal Verification

# Overview

---

### 1. Claude Code Research & Script Generation Output
A terminal capture showing the AI-assisted research phase in Claude Code, where the automated PoC script (poc_tier_flip_approval_bypass.sh) and initial report draft (REPORT-DRAFT.md) were generated, structured, and verified against the local GitLab instance.

<img width="2497" height="1450" alt="image" src="https://github.com/user-attachments/assets/d56e179d-4bc3-44f9-900e-532499a7498c" />



### 2. Vulnerability Execution Chain & Root Cause Breakdown
A detailed breakdown of the live verification results and technical root cause analysis (pointing to ee/app/policies/ee/environment_policy.rb), illustrating how a standard environment tier update API call defeats group-level deployment approval gates.

<img width="2048" height="1202" alt="image" src="https://github.com/user-attachments/assets/7bd532c0-6388-4ec8-9fa0-57cf2ec1c1cc" />

