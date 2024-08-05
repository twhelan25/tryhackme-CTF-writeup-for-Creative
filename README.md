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
It is forbidden. What I'm going to try next is wfuzz to see if there's any hidden subdomains.
```bash
wfuzz -c -w /usr/share/wordlists/seclists/Discovery/DNS/subdomains-top1million-20000.txt --sc 200 -H "Host: FUZZ.creative.thm" -u http://creative.thm -t 100
```
Command breakdown:

-c: Enables colored output.
-w /usr/share/wordlists/seclists/Discovery/DNS/subdomains-top1million-20000.txt: Specifies the wordlist to use for fuzzing.
--sc 200: Filters results to show only those with a status code of 200.
-H "Host: FUZZ.creative.thm": Sets the Host header, with FUZZ being the placeholder for the subdomains.
-u http://creative.thm: Specifies the target URL.
-t 100: Sets the number of concurrent threads to 100.

![wfuzz](https://github.com/user-attachments/assets/e93bef44-1ac1-4ee0-9633-a9439470c941)

We see a subdomain called beta. Let's add it to the /etc/hosts file and visit it.
Note: In order to sucessfully visit the subdomain it must be entered as http://beta.creative.thm.

![urltester](https://github.com/user-attachments/assets/4a45641d-00b3-46e2-a0e0-d9ea32fdec41)

If we put in http://localhost or http://127.0.0.1 (loop back address) we are taken to a frame work of the site.

![localhost_framework](https://github.com/user-attachments/assets/07d83fcc-f314-4aa4-9695-ff19b46181d0)

Let's create a test file to see if we can get it to display the contents:
```bash
echo "Attention" >> config.html
```
Then python server:
```bash
python3 -m http.server
```
Then in the url tester our IP and file:
```bash
http://<your IP>:8000/config.html
```
It it successfully printed "Attention".
