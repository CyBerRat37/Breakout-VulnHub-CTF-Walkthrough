# Breakout-VulnHub-CTF-Walkthrough
**Machine:** Breakout  
**Date:** 2021-10-21  
**Difficulty:** Easy  
**Target IP:** 192.168.56.102

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

Using `enum4linux -a 192.168.56.102`, we identify a suitable user. Logging into the system requires the previously discovered password `.2uqPEfj3D<P'a-3`.

And here we are in the system. Now run `ls` and see `user.txt`. Here we find first flag:
`bmp!r3{You_Manage_To_Break_To_My_Secure_Access}`

Half is Done. Now we get the root flag.

## Privilege Escalation

Inside the home directory, I found a suspicious tar binary owned by root:

`ls -la /home/cyber/
-rwxr-xr-x 1 root root 531928 Oct 19 2021 tar`

It had no SUID bit set, but further investigation showed that it could be used to read files that were otherwise inaccessible. After checking `/` directory, I find suspect file `.old_pass.back` on way `/var/backups`.

This file cannot be read simply using `cat`. Therefore, we will use the compromised `tar` file.

`/home/cyber/tar -cvf pass.tar /var/backups/.old_pass.bak`

Inside shadow.bak was the root password hash (or sometimes the plaintext password). In this case, the password was `T&4&YurgtRX(=~h`.
Then I switched to root:

`su -
Password: T&4&YurgtRX(=~h`

Next, we locate the file `r00t.txt` and read it. We find the final flag there as well.

## Flags

User flag (/home/cyber/user.txt):

`bmp!r3{You_Manage_To_Break_To_My_Secure_Access}`

Root flag (/root/root.txt):

3mp!r3|You_Manage_To_BreakOut_From_My_System_Congratulation|

# Lessons Learned

    Always check HTML source comments for hidden clues.

    Brainfuck or other esoteric languages are often used to hide passwords.

    SUID binaries (even without the s bit, if they are owned by root and can be executed) can be dangerous if they allow reading protected files.

    Don't forget to check for Usermin/Webmin on port 10000.

# References

    VulnHub – Breakout

    Brainfuck interpreter
