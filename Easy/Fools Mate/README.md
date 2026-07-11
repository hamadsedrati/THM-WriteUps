# ♟️ Fools Mate

**Difficulty:** Easy · **Category:** Web Exploitation · **Author:** DrGonz0

> *Can you bypass the engine?*

---

## Table of Contents
- [Scenario](#scenario)
- [Summary](#summary)
- [Recon](#recon)
  - [Port Scan](#port-scan)
  - [SSH Rabbit Hole](#ssh-rabbit-hole)
  - [Web Application](#web-application)
- [Exploitation](#exploitation)
- [Lessons Learned](#lessons-learned)

---

## Scenario

It's mate in one. You know it, the engine knows it, my grandma knows it. The board says checkmate is one click away. The engine says no. **Settle the argument.**

Access the web app from your AttackBox's browser via:
```
http://MACHINE-IP
```

## Summary

The Fools Mate room is a classic example of why you never trust client-side security. It's a web challenge with a chess board where all the game rules are checked right in the browser using javaScript.

Because of that, you can pop open dev tools (or Burp Suite), tweak the client-side logic, trick the game into allowing an illegal checkmate move, and grab the flag. It's the ultimate lesson in why **the server always needs to double-check user input.**

---

## Recon

We start by opening the web app as described in the scenario. A chess board sits in the middle of the page, with a **Reset Position** button to the right.

![Web app interface](https://github.com/hamadsedrati/THM-WriteUps/blob/main/Easy/Fools%20Mate/img/interface.png)

### Port Scan

A quick Nmap scan to check for open ports:

```bash
nmap -sV MACHINE-IP
```

```text
Starting Nmap 7.99 ( https://nmap.org ) at 2026-07-09 14:29 -0400
Nmap scan report for 10.49.146.160
Host is up (0.048s latency).
Not shown: 998 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 9.6p1 Ubuntu 3ubuntu13.16 (Ubuntu Linux; protocol 2.0)
80/tcp open  http    Node.js Express framework
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/.
Nmap done: 1 IP address (1 host up) scanned in 8.27 seconds
```

Two open ports: `22` (SSH) and `80` (HTTP, Node.js/Express).

### SSH Rabbit Hole

> ⚠️ **Note to future self:** this entire section was an UNNECESSARY detour. Keeping it in for the lesson at the bottom :)

The banner showed OpenSSH 9.6p1, which lined up with **CVE-2024-6387** ("regreSSHion"), so I tried exploiting it.

<img width="885" height="156" alt="Failed SSH exploit attempt" src="https://github.com/hamadsedrati/THM-WriteUps/blob/main/Easy/Fools%20Mate/img/sshexploit.png" />

as expected, no luck, the version was patched or just not vulnerable. Time to pivot to the actual intended path: the chess app.

### Web Application

First, I tried moving a piece and watching the Network tab to see what the backend was actually receiving.

<img width="1915" height="321" alt="Network tab showing move request" src="https://github.com/user-attachments/assets/d2c8cf8a-fc1b-4f17-8ecd-84fe7190c8cb" />

Next, I tried an illegal move, sliding the rook to A8 for an immediate "checkmate." The UI didn't take it kindly:

<img width="380" height="157" alt="Client-side validation rejecting the illegal move" src="https://github.com/user-attachments/assets/af0ad03c-74f6-402b-b7ba-70448dd543ce" />

The key observation: **no request was ever sent to the server.** That confirmed the validation was happening entirely client-side. So I dug into `app.js` and found the exact function responsible for validating moves.

<img width="457" height="67" alt="Client-side move validation function in app.js" src="https://github.com/user-attachments/assets/3b9768df-9384-4af1-bf89-7cbe15a639d7" />

---

## Exploitation

With the validation function identified, the fix was simple: patch it in dev tools so it always returns `true`, then reset the board and replay the winning move.

<img width="1170" height="450" alt="image" src="https://github.com/user-attachments/assets/b6a78167-7b9e-4347-bfe2-602698366a65" />


With the move-validation function forced to `return true`, the illegal checkmate was accepted, and the flag was served.

---

## Lessons Learned

- **Never trust the client.** If a security-relevant check can run in the browser, assume an attacker will bypass it. All game/business logic needs to be re-validated server-side.
- **Watch the Network tab before assuming a request failed.** The absence of a request told me more than the presence of one would have, it proved validation never left the browser.
- **Don't chase CVEs on a hunch.** A banner match isn't confirmation of exploitability. The SSH detour cost time that could've gone straight to the web app, which was clearly the intended path .
- **DevTools > custom tooling for quick client-side patches.** Editing `app.js` live in the Sources tab was faster than standing up a proxy for a one-line logic change.
