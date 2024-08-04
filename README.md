![intro](https://github.com/user-attachments/assets/8edd2168-b8ee-4480-a362-6e08069366d6)

# tryhackme-CTF-writeup-for-Creative

This is a walkthrough for the tryhackme CTF Creative. I will not provide any flags or passwords as this is intended to be used as a guide.

## Scanning/Reconnaissance

First off, let's store the target IP as a variable for easy access.

Command: export ip=xx.xx.xx.xx

Next, let's run an nmap scan on the target IP:
```bash
nmap -A -v $ip -D RND:10 -oN nmap.txt
```

Command break down:

-A: This flag enables aggressive scanning. It combines various scan types (like OS detection, version detection, script scanning, and traceroute) into a single scan.

-v: increases verbosity, providing more detailed output during the scan.

—$ip: provides the target IP we stored as the variable $ip.

-D RND:10 Nmap can send additional packets to confuse network intrusion detection systems (IDS) or hide the true source of the scan by randomly selecting up to 10 decoy IP addresses.

-oN nmap.txt: This option specifies normal output that should be saved to a file named “nmap.txt.

This scan reveals a two open ports:

![nmap](https://github.com/user-attachments/assets/cadfd25b-76ae-45df-8bb0-d3d8eff043d5)

Let's check out he webserver on port 80/tcp.

I was unable to connect the site so I added the site to the my /etc/hosts file. vim /etc/hosts. We can now explore the site.

I then ran gobuster and nikto scans on the target:

![nikto](https://github.com/user-attachments/assets/6c2ceba9-f5c4-43e8-b3d0-cfe569623681)

![gobuster](https://github.com/user-attachments/assets/54590aef-db43-44fa-a332-a6b0a69a1d41)

Let's check out /assets.
