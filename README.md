# CloudGoat Labs — B1TR00T

> Offensive AWS security lab writeups based on [CloudGoat](https://github.com/RhinoSecurityLabs/cloudgoat) by Rhino Security Labs.
> Each scenario is deployed on personal AWS infrastructure, attacked, documented, and destroyed.

---

## Progress

| # | Scenario | Category | Difficulty | Status | Date |
|---|----------|----------|------------|--------|------|
| 01 | [iam_privesc_by_rollback](./iam_privesc_by_rollback/) | IAM / Privilege Escalation | Easy | ✅ Done | 2026-06-12 |
| 02 | vulnerable_lambda | Lambda / IAM | Easy | 🔜 Next | — |
| 03 | cloud_breach_s3 | SSRF / S3 | Easy | 🔜 | — |
| 04 | ecs_takeover | Container / IAM | Medium | 🔜 | — |
| 05 | rce_web_app | RCE / Credential Theft | Medium | 🔜 | — |
| 06 | codebuild_secrets | CI/CD | Medium | 🔜 | — |

---

## Methodology

Every scenario follows the same workflow:

```
Deploy → Enumerate → Exploit → Document → Destroy → Verify $0
```

1. `cloudgoat create <scenario>` — spin up isolated lab
2. Attack with `aws cli`, `aws-enumerator`, `pacu` and manual techniques
3. Document in Obsidian, push writeup to this repo
4. `cloudgoat destroy <scenario>` — tear down all resources
5. Verify AWS billing shows no unexpected charges

---

## Environment

- Attacker OS: Arch Linux
- Cost controls: AWS Budget alert at $5/month + CloudWatch billing alarm at $10

---

## Related Repos

- [pwnedlabs-writeups](https://github.com/B1TR00T/pwnedlabs-writeups) — PwnedLabs cloud attack path writeups
