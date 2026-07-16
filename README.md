# CloudGoat Labs

**Offensive AWS security writeups** — attack chains against intentionally
vulnerable infrastructure, built and torn down entirely on personal AWS
infrastructure using [CloudGoat](https://github.com/RhinoSecurityLabs/cloudgoat)
by Rhino Security Labs.

Each scenario below documents a full attacker workflow: initial access,
enumeration, exploitation, and impact — not just "ran a tool, got a flag."

---

## Scenarios

| # | Scenario | Attack Path | Difficulty | Status |
|---|---|---|---|---|
| 01 | [`iam_privesc_by_rollback`](./iam_privesc_by_rollback) | IAM policy version rollback → full admin | Easy | ✅ |
| 02 | [`ec2_ssrf`](./ec2_ssrf) | Lambda secrets → SSRF → IMDS credential theft → S3 | Easy | ✅ |
| 03 | [`iam_privesc_by_attachment`](./iam_privesc_by_attachment) | Instance profile reassignment → privilege escalation | Easy | 🔜 |

## What these writeups show

- **Full compromise narratives**, not just command output — every scenario
  states initial access, escalation path, resources reached, and impact
- **Root cause, not just exploitation** — e.g. `ec2_ssrf` explains *why*
  the SSRF was exploitable (unvalidated user-supplied URL) and which
  control (IMDSv2) would have stopped it
- **Attacker mental model over memorized steps** — enumeration-first
  approach (`aws-enumerator`, manual CLI) to establish what's actually
  reachable before attacking, rather than following a fixed script

## Methodology
Deploy → Enumerate → Exploit → Document → Destroy → Verify $0

1. `cloudgoat create <scenario>` — spin up an isolated lab environment
2. Enumerate with AWS CLI, `aws-enumerator`, and Pacu to map what's reachable
3. Exploit the identified attack path, capturing evidence at each pivot
4. Document as a full writeup with compromise narrative and screenshots
5. `cloudgoat destroy <scenario>` — tear down all resources
6. Confirm AWS Billing shows $0 unexpected spend

## Environment

| | |
|---|---|
| Attacker OS | Arch Linux |
| Primary tools | AWS CLI, `aws-enumerator`, Pacu, nmap |
| Cost controls | AWS Budget alert @ $5/mo, CloudWatch billing alarm @ $10 |

## Related work

- [`pwnedlabs-writeups`](https://github.com/B1TR00T/pwnedlabs-writeups) — cloud attack path labs (S3/IAM/DynamoDB, SSRF→IMDS, secrets exposure)
- [`HTB-machines`](https://github.com/B1TR00T/HTB-machines) — HackTheBox machine writeups
