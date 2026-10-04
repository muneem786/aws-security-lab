# aws-security-lab
Hands-on AWS security lab: IAM least privilege, Prowler scan &amp; risk triage with documented decisions
# AWS Cloud Security Lab: Account Hardening & Risk Triage

A hands-on lab where I set up a fresh AWS account, scanned it with Prowler, and triaged the findings by risk. For each finding I decided whether to fix it, accept it, or mark it not applicable, and documented why.

## Environment

| Item | Detail |
|---|---|
| Cloud | AWS (Free plan, single account) |
| Region | us-east-2 |
| Workstation | Windows 11 + WSL2 (Ubuntu) |
| Tooling | AWS CLI v2, Prowler 5.44.0 (Python 3.12 venv) |
| Scan date | 2026-10-02 |
| Scan scope | `prowler aws --region us-east-2 --severity critical high --status FAIL` |

## What I did

- [x] Created the account and enabled MFA on the sign-in identity
- [x] Created an IAM user (`lab-admin`) for CLI access
- [x] Reduced its permissions from `AdministratorAccess` to `SecurityAudit` + `ViewOnlyAccess` (least privilege)
- [x] Configured AWS CLI and verified identity with `aws sts get-caller-identity`
- [x] Ran a Prowler scan (7 high/critical findings)
- [x] Triaged all 7 findings (table below)
- [ ] Enable MFA on `lab-admin`
- [ ] Create a multi-region CloudTrail trail
- [ ] Delete the `lab-admin` access key when the lab ends

## Findings and decisions

| # | Severity | Check | Decision | Reasoning |
|---|---|---|---|---|
| 1 | Critical | AdministratorAccess policy attached to an identity | Accept / verify | Likely attached to an AWS-managed role created with the account. I did not create it, so I did not modify it. To verify via IAM > Policies > Entities attached |
| 2 | High | AWS Config delegated admin + org aggregator | Not applicable | Meant for AWS Organizations. Single-account lab. Config also incurs cost |
| 3 | High | Root credentials centralized management | Not applicable | Organizations-level feature, not relevant to a single account |
| 4 | High | Role missing confused-deputy prevention | Accept | Role is AWS-managed (`role/managed/...`), not user-created. Modifying it risks breaking console access |
| 5 | High | IAM user uses long-lived credentials | Accept (temporary) | Needed for CLI in this lab. Mitigated by read-only permissions. Key will be deleted at the end |
| 6 | High | IAM user has no (hardware) MFA | Remediate | Add MFA to `lab-admin`. Note: this specific check asks for *hardware* MFA, so a virtual authenticator may still be flagged |
| 7 | High | No CloudTrail trail logging | Remediate | Without CloudTrail there is no audit trail of API activity |

## Key lessons

1. **Least privilege is a risk control, not a formality.** When a credential was accidentally exposed in a screenshot, having already reduced `lab-admin` to read-only limited the damage to information disclosure.
2. **Not every FAIL should be fixed.** Several findings assume an organization-scale setup. Deciding what to accept, and documenting why, is a core GRC skill.
3. **Know what you did not create.** AWS-managed roles can look alarming in a scan. Check ownership before changing anything.
4. **Cost is a security constraint.** I avoided enabling services like AWS Config in a free-plan account without checking pricing first.
5. **Tooling versions matter.** Prowler failed on Python 3.14; pinning a Python 3.12 virtual environment fixed it.

## Incident note (credential handling)

During the lab, an access key and secret were briefly visible in a photographed terminal. Response: permissions were already reduced to read-only, the key is scheduled for deletion at lab end, and the screenshots were removed from the device. Lesson: run `clear` before capturing terminals, and treat any exposed key as compromised.

## Next steps

- Build and then detect a deliberately misconfigured S3 bucket
- Enable and review CloudTrail logs
- Map findings to CIS AWS Foundations Benchmark controls
- Prepare for AWS Cloud Practitioner and Security Specialty
