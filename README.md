# Cybersecurity Projects

Hands-on security work spanning vulnerability assessment, detection engineering, and incident response — mostly built against my own home lab.

## Why I built this
I wanted projects that show I can go past "I ran a scanner and here's the output" — actually interpreting findings, prioritizing them, writing detections, and thinking through how a team would respond when something goes wrong.

## Projects

### `vuln-scan-report/`
A vulnerability assessment report from scanning a deliberately-vulnerable lab VM (Metasploitable2), written as a real engagement report: executive summary, findings by severity, remediation priority, not just raw scanner output.

### `detection-rules/`
A handful of Sigma detection rules for common suspicious behaviors (e.g., unusual PowerShell execution patterns, brute-force login attempts), with notes on the reasoning behind each rule and its false-positive tradeoffs.

### `ctf-writeups/`
Writeups from CTF challenges I've done (TryHackMe / picoCTF style), explaining my thought process rather than just the final flag.

### `incident-response-runbook/`
A generic IR runbook covering the phases (Preparation, Identification, Containment, Eradication, Recovery, Lessons Learned) with a worked example for a phishing-led compromise scenario.

## Tools used
Nessus/OpenVAS, Sigma, Security Onion (lab), TryHackMe/picoCTF platforms

## Notes
All scanning and testing was performed against systems I own or lab environments explicitly designed for practice (Metasploitable2, CTF platforms). Nothing here targets third-party systems.
