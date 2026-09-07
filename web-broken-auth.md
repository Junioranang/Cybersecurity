# CTF Writeup: Web Exploitation — "Broken Auth" Style Challenge

Platform: TryHackMe-style room, beginner/intermediate web track. Writeup focuses on thought process, not just the final flag.

## Challenge summary
A login portal for a fictional internal tool. Goal: gain admin access without valid credentials.

## Recon
Started by just poking at the login form manually before running any tools — checking how the app responded to a wrong username vs. a wrong password. The error messages were different ("user not found" vs. "incorrect password"), which is a classic username enumeration issue. That told me I could confirm valid usernames before even touching the password field.

Ran a quick directory brute-force with `gobuster` in parallel while manually testing the login form, since there's no reason to sit idle waiting on one approach:

```
gobuster dir -u http://target/ -w /usr/share/wordlists/dirb/common.txt
```

Found a `/admin` path returning a 403 (exists, but blocked) and a `/api/` path that wasn't in the visible site navigation.

## Finding the actual bug
The `/api/login` endpoint accepted a POST with `username` and `password`. I tried a basic SQL injection probe in the username field (`' OR '1'='1`) mostly to rule it out — it didn't work, the app was using parameterized queries there, which was actually reassuring to confirm rather than assume.

What did work: the API returned a JWT on login, and decoding it (just base64, no cracking needed) showed the algorithm field was set to `alg: none`. That meant the server would accept a token with an empty signature as long as the payload claimed to be valid. I forged a token with `"role": "admin"` and an empty signature, submitted it as the auth header, and got into the admin panel.

## Root cause
Two separate weaknesses compounded: verbose login error messages let me confirm a valid username, and the JWT implementation didn't reject the `none` algorithm — a known JWT library pitfall (CVE-adjacent pattern, several real libraries have had this exact issue).

## What I'd tell a dev team
The username enumeration alone isn't usually critical on its own, but here it fed directly into the more serious JWT issue. I'd flag both, but the "alg: none" acceptance is the one that actually needs a same-day fix — it means authentication is effectively decorative.

## Lesson for myself
I went for the SQLi angle first because it's the "classic" move, and it was a dead end. The actual bug was in a completely different layer of the auth flow. Worth remembering: don't get anchored on the first vulnerability class you think of, check what the app is actually doing at each step of the request.
