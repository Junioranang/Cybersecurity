# Vulnerability Assessment Report — Lab Target (Metasploitable2)

**Engagement type:** Internal lab assessment (self-directed practice)
**Target:** Metasploitable2 VM, isolated on a lab-only VLAN with no route to the internet or production network
**Tool:** OpenVAS (Greenbone Community Edition)
**Date:** Lab exercise, written up in report format to practice real deliverable writing

## Executive Summary

The scan identified 6 findings rated Critical or High, primarily stemming from outdated, intentionally-vulnerable service versions that Metasploitable2 ships with by design (this VM exists specifically as a training target). While these findings wouldn't be surprising for this specific box, I've written this report the way I would for a real engagement — prioritized by actual exploitability and business impact, not just CVSS score alone.

## Findings

### Critical

**1. VSFTPD 2.3.4 Backdoor (CVE-2011-2523)**
A backdoored version of vsftpd that allows remote command execution via a crafted username during the FTP login sequence.
- **Impact:** Full remote code execution, no authentication required
- **Evidence:** Confirmed via Metasploit's `vsftpd_234_backdoor` module — got a shell in under a minute
- **Remediation:** Uninstall/replace the compromised package version entirely; this isn't a config fix, the binary itself is backdoored

**2. UnrealIRCd Backdoor (CVE-2010-2075)**
Similar to the above — a backdoored IRC daemon that accepts arbitrary commands via a special string in the protocol.
- **Impact:** Remote code execution, no authentication required
- **Remediation:** Replace with a clean build from an official source, verify checksums going forward

### High

**3. Unauthenticated NFS shares**
NFS exports are world-readable/writable with no client restriction.
- **Impact:** Any host on the network can read and modify shared files without credentials
- **Remediation:** Restrict exports to specific host IPs, enable NFSv4 with Kerberos auth if the environment supports it

**4. Samba usermap script RCE (CVE-2007-2447)**
Vulnerable Samba version allows command injection through the `username map script` config option.
- **Impact:** Remote code execution
- **Remediation:** Patch Samba to a version past the fix, disable `username map script` if not explicitly needed

**5. Weak/default credentials on multiple services**
Several services (Tomcat manager, PostgreSQL, MySQL) accept default or trivially weak credentials.
- **Impact:** Credential-based access to application and database layers
- **Remediation:** Enforce credential rotation as part of any base image build process, not just at deployment

### Medium

**6. Outdated OpenSSL exposing known protocol weaknesses**
- **Impact:** Susceptible to several older SSL/TLS attacks; low practical risk on an isolated lab box, but flagged for completeness
- **Remediation:** Upgrade OpenSSL and disable legacy protocol versions in service configs

## Remediation priority

I ranked these by ease of remote exploitation combined with impact, not raw CVSS:

1. VSFTPD backdoor — trivial to exploit, full RCE, fix immediately
2. UnrealIRCd backdoor — same profile as above
3. Samba RCE — requires slightly more setup to exploit but same impact ceiling
4. NFS exposure — no RCE but broad unauthorized data access
5. Weak credentials — depends on what's reachable behind them
6. OpenSSL — lowest urgency here given the isolated lab context

## Notes on methodology
This was run entirely against an intentionally vulnerable training VM on an isolated lab network with no other hosts present. No scanning was performed against any system I don't own or that wasn't explicitly built for this purpose.
