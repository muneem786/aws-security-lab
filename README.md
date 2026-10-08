# AWS Cloud Security Lab: Account Hardening & Risk Triage

A hands-on lab where I set up a fresh AWS account, scanned it with Prowler, and triaged the findings by risk. For each finding I decided whether to fix it, accept it, or mark it not applicable, then re-scanned to verify the fixes.

## Environment

| Item | Detail |
|---|---|
| Cloud | AWS (Free plan, single account) |
| Region | us-east-2 |
| Workstation | Windows 11 + WSL2 (Ubuntu) |
| Tooling | AWS CLI v2, Prowler 5.44.0 (Python 3.12 venv) |
| Scan command | prowler aws --region us-east-2 --severity critical high --status FAIL |
| Baseline scan | 2026-10-02 (7 high/critical findings) |
| Re-scan | 2026-10-08 (6 high/critical findings) |

## Before vs after

| Service | Baseline | After fixes |
|---|---|---|
| CloudTrail | 1 high | 0 (fixed) |
| S3 | not present | 0 (found after creating the log bucket, then fixed) |
| IAM | 5 (1 critical, 4 high) | 5 (accepted, see below) |
| Config | 1 high | 1 (not applicable) |
| *Total* | *7* | *6* |

Initial full scan (all severities): 87 checks, 39 passed, 48 failed. This lab focuses on critical and high findings only.

## What I did

- [x] Created the account and enabled MFA on the sign-in identity
- [x] Created an IAM user (lab-admin) for CLI access
- [x] Reduced its permissions from AdministratorAccess to SecurityAudit + ViewOnlyAccess (least privilege)
- [x] Configured AWS CLI and verified identity with aws sts get-caller-identity
- [x] Ran a Prowler baseline scan and triaged all findings
- [x] Created a multi-region CloudTrail trail (management events only)
- [x] Turned on S3 Block Public Access at the account level
- [x] Re-scanned to verify both fixes
- [ ] Delete the lab-admin access key when the lab ends
- [ ] Delete the CloudTrail trail and its S3 bucket when the lab ends (to avoid storage cost)

## Findings and decisions

| # | Severity | Check | Decision | Reasoning |
|---|---|---|---|---|
| 1 | Critical | AdministratorAccess policy attached to an identity | Accept | Likely attached to an AWS-managed role created with the account. I did not create it, so I did not modify it |
| 2 | High | AWS Config delegated admin + org aggregator | Not applicable | Meant for AWS Organizations. Single-account lab. Config also incurs cost |
| 3 | High | Root credentials centralized management | Not applicable | Organizations-level feature, not relevant to a single account |
| 4 | High | Role missing confused-deputy prevention | Accept | Role is AWS-managed (role/managed/...), not user-created. Modifying it risks breaking console access |
| 5 | High | IAM user uses long-lived credentials | Accept (temporary) | Needed for CLI in this lab. Mitigated by read-only permissions. Key will be deleted at the end |
| 6 | High | IAM user has no (hardware) MFA | Accept | lab-admin is CLI-only (no console password) and this account offers no MFA option for it. Risk reduced by read-only permissions and planned key deletion |
| 7 | High | No CloudTrail trail logging | *Fixed* | Created a multi-region trail with management events. Verified by re-scan |
| 8 | High | S3 account-level Block Public Access off | *Fixed* | Surfaced only after CloudTrail created its log bucket. Enabled all four Block Public Access settings. Verified by re-scan |

## Key lessons

1. *Least privilege is a risk control, not a formality.* When a credential was accidentally exposed in a screenshot, having already reduced lab-admin to read-only limited the damage to information disclosure.
2. *Not every FAIL should be fixed.* Several findings assume an organization-scale setup. Deciding what to accept, and documenting why, is a core GRC skill.
3. *Fixes can reveal new findings.* Creating a CloudTrail log bucket brought S3 into scope and exposed a missing account-level control. Re-scanning after changes matters.
4. *Know what you did not create.* AWS-managed roles can look alarming in a scan. Check ownership before changing anything.
5. *Cost is a security constraint.* I avoided enabling services like AWS Config without checking pricing, and chose the cheapest CloudTrail setup.
6. *Tooling versions matter.* Prowler failed on Python 3.14; pinning a Python 3.12 virtual environment fixed it.

## Incident note (credential handling)

During the lab, an access key and secret were briefly visible in a photographed terminal. Response: permissions were already reduced to read-only, and the key is scheduled for deletion at lab end. Lesson: run clear before capturing terminals, and treat any exposed key as compromised.

## Next steps

- Build and then detect a deliberately misconfigured S3 bucket
- Review CloudTrail logs
- Map findings to CIS AWS Foundations Benchmark controls
- Prepare for AWS Cloud Practitioner and Security Specialty
-
