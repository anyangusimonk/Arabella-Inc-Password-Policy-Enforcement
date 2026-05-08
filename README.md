
# Arabella Inc. — Password Policy Enforcement Lab
## Locking Down the Domain — Security Hardening at Arabella Inc.

Central command is part of the efficiency Arabella Inc. gained post-migration, but a domain without enforced password standards is still a liability. Centralization means every user account, 
every workstation, and every department folder now runs through one authentication layer. It is also a single point of failure if credentials are weak, stale, or unprotected. 
Anyone who guesses a password gets in. Anyone who never changes their password stays exposed indefinitely. And when someone gets locked out, or incase of a breach, without a structured resolution 
workflow, the response would be lax hence encouraging the threat actor to encroach further. This lab picks up directly where the migration ended. The domain is built, next, hardening it follows.
I designed and implemented a two-tier password security structure for arabellainc.local, a domain-wide baseline through Group Policy and a stricter fine-grained Password Policy targeting all domain users. 
I then ran a test on the lockout policy by triggering it deliberately, verified the domain controller logged it correctly, and resolved it the way as a standard practice in the real world. Everything here
is documented, configuration before testing, evidence during testing and resolution after.
---
## How This Connects to Lab 1

This lab builds directly on the 
[Arabella AD Migration Lab](https://github.com/anyangusimonk/arabella-ad-migration-lab).

The AD migration established:
- A Windows Server 2025 domain controller — ARABELLA-DC01
- Two domain-joined Windows 10 workstations — ARABELLA-PC001 and ARABELLA-PC002
- Five department user accounts under arabellainc.local
- Centralized identity management via Active Directory
This lab adds the security layer on top of that foundation. The same domain, the same users, the same machines, now with enforced password standards, automatic lockout enforcement, and a documented
resolution workflow.
---
## The Environment

| Component | Details |
|---|---|
| Domain Controller | Windows Server 2025 — ARABELLA-DC01 |
|Workstations | Windows 10 — ARABELLA-PC001, ARABELLA-PC002 |
| Domain | arabellainc.local |
| Policy Tool | Group Policy Management Editor |
| FGPP Tool | Active Directory Administrative Center |
| Test Account | walma (Wanda Aima — Credit Department) |

---
## What I Implemented

### Tier 1 — Default Domain Policy

Configured via Group Policy Management Editor, applying domain-wide to all 28 Arabella Inc. user accounts across all domain-joined machines.

| Setting | Value | Rationale |
|---|---|---|
| Minimum password length | 10 characters | Industry baseline for credential strength |
| Password complexity | Enabled | Requires uppercase, lowercase, digit, symbol |
| Maximum password age | 45 days | Enforces mandatory rotation every 45 days |
| Password history | 24 passwords | Prevents reuse of the last 24 passwords |
| Account lockout threshold | 3 invalid attempts | Locks after 3 consecutive failures |
| Account lockout duration | 7 minutes | Auto-unlocks after 7 minutes |
| Reset lockout counter after | 7 minutes | Clears bad attempt count after 7 minutes |
| Administrator account lockout | Enabled | No privileged account exemptions |

### Tier 2 — Fine-Grained Password Policy

Configured via Active Directory Administrative Center in the Password Settings Container under System. Named "User Account Access Policy" with Precedence 1, the highest priority in the domain, overriding 
the Default Domain Policy for the groups it targets. Applied directly to: Domain Computers, Domain Guests, Domain Users.
This two-tier structure mirrors how enterprise environments actually operate, a baseline for everyone, stricter controls where it matters.
---
## The Lockout Test

I triggered a deliberate lockout on domain user walma (Wanda Aima) to ascertain the enforcement of the policy configuration, by entering incorrect credentials three consecutive times on ARABELLA-PC001.

**On the Workstation:**
The login screen returned "The referenced account is currently locked out and may not be logged on to", confirming the policy enforced at the client level.

**On the Server:**
Event ID 4740 appeared in the Security Event Log on ARABELLA-DC01 within seconds — "A user account was locked out" logged under Task Category: User Account Management, timestamped and attributed to 
the domain controller. The domain had seen the lockout, recorded it, and was ready to be queried.

**Resolution:**
Account unlocked from ARABELLA-DC01 via PowerShell.
---
## What This Lab Demonstrates

This lab proves the ability to harden a live Active Directory domain beyond basic setup — implementing structured password enforcement, configuring granular Fine-Grained Policies for targeted user groups, 
stress-testing the lockout mechanism against real domain accounts, reading Security Event Log entries to detect and diagnose access issues, and resolving account lockouts via PowerShell as it is done in 
production environments.

## Before You Start — Things Worth Knowing

- **Local Security Policy is not the right tool in a domain.** - Settings configured there are overridden by the Default Domain Policy. Always use Group Policy Management for domain-wide enforcement.
- **Minimum password age blocks resets.** - If set above 0 days, password resets via PowerShell will fail with a permissions error until it is temporarily set to 0.
- **Fine-Grained Policies require AD Administrative Center.** - They live in System → Password Settings Container — not in Group Policy Management.
- **Precedence determines which policy wins.** - Lower number = higher priority. Precedence 1 overrides everything else.
- **Event-log activity** - Once joined to a domain, access to the workstations (Client-side) security log is denied, only the Domain controller (Server-side) is accessible to the admins. Incase of a lockout
  after a failed logon event, it is recorded and queried on the Domain controller.
