# Incident Response Runbook: Phishing-Led Compromise

Generic runbook, worked through with a fictional scenario for illustration.

## Scenario
An employee clicks a phishing link, enters their credentials on a fake login page. The attacker uses the stolen credentials to log into the employee's email from an unfamiliar location.

## 1. Preparation (already in place before this happens)
- MFA enforced on all accounts
- Email gateway logging enabled and retained 90 days
- Defined escalation contacts (security lead, IT, legal if needed)

## 2. Identification
- Alert triggers: impossible-travel login alert from IdP (e.g., login from two countries within an hour)
- Confirm: pull sign-in logs, check source IP/geolocation, check for MFA prompt acceptance vs. denial
- Determine scope: has the attacker sent emails, set up mail forwarding rules, or accessed other systems with the same credentials?

## 3. Containment
- Force password reset and revoke active sessions/tokens immediately
- Disable any mail forwarding/auto-reply rules the attacker may have created (common persistence technique)
- If lateral movement suspected, isolate affected endpoint from network

## 4. Eradication
- Confirm no persistence mechanisms remain (forwarding rules, OAuth app grants, new MFA devices registered by attacker)
- Patch/address the root cause if a technical vulnerability was also involved (not just user click)

## 5. Recovery
- Restore normal access once account is confirmed clean
- Monitor the account more closely for a defined period (e.g., 30 days) post-incident

## 6. Lessons Learned
- Was the phishing email similar to previously reported ones? Update email filtering rules if so.
- Does this employee/team need targeted awareness training?
- Update this runbook if any step was unclear or missing during the actual response.

## Key takeaway
The fastest actual containment step in most phishing-credential incidents is killing active sessions and forcing re-auth — not lengthy investigation first. Speed matters more than certainty in the first 15 minutes.
