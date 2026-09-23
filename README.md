# Break Things: DevOps Failure Labs

25 deliberate break-and-fix exercises across Linux, AWS, CI/CD, Terraform and Ansible.
Each lab: break it, predict the symptom, diagnose from logs and tools only, fix it, write it up.

Part of **Building in DevOps**.

## Ground rules

- Throwaway VM or sandbox AWS account only. Snapshot/AMI first.
- AWS: budget alert on, tag everything (`lab=break-things`) so cleanup is easy.
- Write your **prediction before** running any diagnostic command.
- Diagnose using logs and tools only. Fix without simply undoing the break command.
- No secrets, account IDs or IPs in commits. Redact before pushing.

## Repo structure

```
break-things/
├── README.md
├── linux/01-disk-and-inodes/incident.md
├── aws/01-security-group-rule/incident.md
├── cicd/01-missing-secret/incident.md
├── terraform/01-drift/incident.md
└── ansible/01-ssh-auth/incident.md
```

One folder per lab, each with an `incident.md` (template at the bottom) and optional `evidence/` for redacted logs or screenshots.

## Progress tracker

Status: ⬜ not started · 🟨 in progress · ✅ done and written up

### Linux

| # | Lab | Status | Root cause (one line) | Write-up |
|---|-----|--------|-----------------------|----------|
| 1 | Fill the disk and inodes | ⬜ | | |
| 2 | Permission and ownership breakage | ⬜ | | |
| 3 | Break a systemd service | ⬜ | | |
| 4 | Firewall and DNS breakage | ⬜ | | |
| 5 | Memory pressure and the OOM killer | ⬜ | | |

### AWS

| # | Lab | Status | Root cause (one line) | Write-up |
|---|-----|--------|-----------------------|----------|
| 1 | Remove SG ingress rule | ⬜ | | |
| 2 | Delete the internet route | ⬜ | | |
| 3 | Detach IAM role from instance | ⬜ | | |
| 4 | NACL stateless trap | ⬜ | | |
| 5 | Unhealthy ALB targets | ⬜ | | |

### CI/CD

| # | Lab | Status | Root cause (one line) | Write-up |
|---|-----|--------|-----------------------|----------|
| 1 | Delete a required secret | ⬜ | | |
| 2 | Failing test, prove the gate holds | ⬜ | | |
| 3 | Break the Docker build | ⬜ | | |
| 4 | Bad release and rollback | ⬜ | | |
| 5 | Break the runner / Jenkins | ⬜ | | |

### Terraform

| # | Lab | Status | Root cause (one line) | Write-up |
|---|-----|--------|-----------------------|----------|
| 1 | Create drift | ⬜ | | |
| 2 | Lose the state file | ⬜ | | |
| 3 | Delete a resource out-of-band | ⬜ | | |
| 4 | Trigger a state lock | ⬜ | | |
| 5 | Dependency cycle / provider conflict | ⬜ | | |

### Ansible

| # | Lab | Status | Root cause (one line) | Write-up |
|---|-----|--------|-----------------------|----------|
| 1 | Break SSH auth | ⬜ | | |
| 2 | Break YAML syntax | ⬜ | | |
| 3 | Remove a required variable | ⬜ | | |
| 4 | Break idempotency | ⬜ | | |
| 5 | Bad config plus restart handler | ⬜ | | |

## Incident template (copy into each lab's `incident.md`)

```markdown
# <Lab name>

**Category:** Linux | AWS | CI/CD | Terraform | Ansible
**Date:** YYYY-MM-DD
**Environment:** (e.g. Ubuntu 24.04 EC2 t3.micro, sandbox account)

## 1. What I broke
The exact command(s) I ran.

## 2. Prediction (written BEFORE diagnosing)
What I expected to see, and why.

## 3. Actual symptom
What actually happened. Error messages, HTTP codes, timeouts (redacted).

## 4. Diagnosis trail
Commands I ran, in order, and what each told me.
Dead ends included. They are the most useful part.

## 5. Root cause
One or two sentences.

## 6. Fix
What I changed and why it is the right fix (not just an undo).

## 7. Time to detect / time to fix
- Detect:
- Fix:

## 8. Prevention
Monitoring, validation, guardrails or process that would have caught this earlier.

## 9. What I got wrong / learned
Where my prediction was off.
```
