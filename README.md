# Breakout-VulnHub-CTF-Walkthrough
**Machine:** Breakout  
**Date:** 2026-07-14  
**Difficulty:** Easy/Medium  
**Target IP:** 192.168.1.59

---

## Summary

Breakout is a Linux VM that requires thorough enumeration. Initial access was gained via a Brainfuck-encoded password found in the webpage source. The password allowed login as user `cyber`. Privilege escalation was achieved through a SUID `tar` binary, which was used to read a backup of `/etc/shadow`, revealing the root password.

---

## Tools Used

- `nmap` – port scanning
- `gobuster` – directory brute‑forcing
- `enum4linux` – SMB enumeration
- `smbclient` – SMB access (failed)
- `brainfuck` online interpreter – password decoding
- `tar` – SUID exploitation

---

## Reconnaissance

First, I discovered the machine IP using `netdiscover`.

```bash
nmap -p- -sV -sC 192.168.1.59
(  ports:
  80/tcp – Apache HTTP
  139/tcp – SMB
  445/tcp – SMB
)
```
SMB enumeration with enum4linux revealed some usernames. Actual username = Cyber.

##Web Enumeration

The default Apache page at http://192.168.1.59 contained a hidden comment in the HTML source:

`<!---
don't worry no one will get here, it's safe to share with you my access. Its encrypted :)
+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
...`

This part of the webpage was coded using "Brainfuck". Result after decoding:

`.2uqPEfj3D<P'a-3`
