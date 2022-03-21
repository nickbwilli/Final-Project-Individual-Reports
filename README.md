# Bootcamp-Final-Project
These are screenshots from my engagement. Also see above the Offensive and Defensive Reports.


Red Team: Summary of Operations
Table of Contents
Exposed Services
Critical Vulnerabilities
Exploitation
Exposed Services
Nmap scan results for each machine reveal the below services and OS details:

$ root@Kali:~# nmap 192.168.1.0/24
Starting Nmap 7.80 ( https://nmap.org ) at 2022-03-16 18:47 PDT
Nmap scan report for 192.168.1.1
Host is up (0.00066s latency).
Not shown: 995 filtered ports
PORT     STATE SERVICE
135/tcp  open  msrpc
139/tcp  open  netbios-ssn
445/tcp  open  microsoft-ds
2179/tcp open  vmrdp
3389/tcp open  ms-wbt-server
MAC Address: 00:15:5D:00:04:0D (Microsoft)

Nmap scan report for 192.168.1.100
Host is up (0.00066s latency).
Not shown: 998 closed ports
PORT     STATE SERVICE
22/tcp   open  ssh
9200/tcp open  wap-wsp
MAC Address: 4C:EB:42:D2:D5:D7 (Intel Corporate)

Nmap scan report for 192.168.1.105
Host is up (0.00041s latency).
Not shown: 998 closed ports
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http
MAC Address: 00:15:5D:00:04:0F (Microsoft)

Nmap scan report for 192.168.1.110
Host is up (0.00059s latency).
Not shown: 995 closed ports
PORT    STATE SERVICE
22/tcp  open  ssh
80/tcp  open  http
111/tcp open  rpcbind
139/tcp open  netbios-ssn
445/tcp open  microsoft-ds
MAC Address: 00:15:5D:00:04:10 (Microsoft)

Nmap scan report for 192.168.1.115
Host is up (0.00082s latency).
Not shown: 995 closed ports
PORT    STATE SERVICE
22/tcp  open  ssh
80/tcp  open  http
111/tcp open  rpcbind
139/tcp open  netbios-ssn
445/tcp open  microsoft-ds
MAC Address: 00:15:5D:00:04:11 (Microsoft)

Nmap scan report for 192.168.1.90
Host is up (0.0000070s latency).
Not shown: 999 closed ports
PORT   STATE SERVICE
22/tcp open  ssh

Nmap done: 256 IP addresses (6 hosts up) scanned in 6.69 seconds
  # 
This scan identifies the services below as potential points of entry:

Target 1
root@Kali:~# nmap -sV 192.168.1.110
Starting Nmap 7.80 ( https://nmap.org ) at 2022-03-16 18:54 PDT
Nmap scan report for 192.168.1.110
Host is up (0.00081s latency).
Not shown: 995 closed ports
PORT    STATE SERVICE     VERSION
22/tcp  open  ssh         OpenSSH 6.7p1 Debian 5+deb8u4 (protocol 2.0)
80/tcp  open  http        Apache httpd 2.4.10 ((Debian))
111/tcp open  rpcbind     2-4 (RPC #100000)
139/tcp open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
445/tcp open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
MAC Address: 00:15:5D:00:04:10 (Microsoft)
Service Info: Host: TARGET1; OS: Linux; CPE: cpe:/o:linux:linux_kernel
The following vulnerabilities were identified on each target:

Target 1
Vulnerability 1: Guessed weak password for Michael. CVE - 521: Weak Password Requirements The user was able to use their name as a password which is too simple to crack with a number of brute forcing tools.
Vulnerability 2: SQL server password exposed in wp_config.php file in plain text. CWE - 200: Exposure of Sensitive Information to an Unauthorized Actor The wordpress server kept the password in plaintext so that unauthorized users could find it and exploit it.
Vulnerability 3: Python code execution allowed privilege escalation. CWE - 269: Improper Privilege Management The user Steven had python executiion privileges allowing us to exploit a tty shell that gave us root privileges.
Exploitation
TODO: Fill out the details below. Include screenshots where possible.

The Red Team was able to penetrate Target 1 and retrieve the following confidential data:

Target 1

flag1.txt: {b9bbcb33e11b80be759c4e844862482d}
Flag1close up.JPG
image

Exploit Used This flag was discovered by first using SSH to access Michael’s account due to guessing his weak password, then navigating into the /var/www/html directory, then running a grep command to search for flag1.

Commands:

ssh michael@192.168.1.110
cd /var/www/html
grep -R flag1
flag2.txt: {fc3fd58dcdad9ab23faca6e9a36e581c}

flag2.jpg

image

Exploit Used
This flag was discovered by SSH into the server as Michael. Then navigating to the /var/www directory. Then using the cat command to show the flag file.
Commands:
ssh michael@192.168.1.110
cd /var/www
ls - cat flag2.txt








Blue Team: Summary of Operations
Table of Contents
Network Topology
Description of Targets
Monitoring the Targets
Patterns of Traffic & Behavior
Suggestions for Going Further
Network Topology
The following machines were identified on the network:

Kali Operating System: Kali Linux Purpose: This was the original machine from which we started our attack. IP Address: 192.168.1.90
ELK Operating System: Ubuntu 18.04 Purpose: Elasticsearch, Logstash, and Kabana IP Address: 192.168.1.100
Capstone Operating System: Ubuntu Purpose: This system was for testing alerts and otherwise was unused. IP Address: 192.168.1.105
Hyper-V Operating System: Windows Purpose: This was our VM host system for this network. IP Address: 192.168.1.1
Target 1 Operating System: Debian/Linux Purpose: This system functioned as the vulnerable WordPress server. IP Address: 192.168.1.110
Description of Targets
The target of this attack was: Target 1 - 192.168.1.110.

Target 1 is an Apache web server and has SSH enabled, so ports 80 and 22 are possible ports of entry for attackers. As such, the following alerts have been implemented:

Monitoring the Targets
Traffic to these services should be carefully monitored. To this end, we have implemented the alerts below:

Alert 1
Excessive HTTP Errors

Alert 1 is implemented as follows:

Metric: WHEN count() GROUPED OVER top 5 'http.response.status_code' IS ABOVE 400 FOR THE LAST 5 minutes
Threshold: 400 request codes in the last 5 minutes
Vulnerability Mitigated: Attacks on the webserver showing a brute force attempt.
Reliability: Medium
Alert 2
HTTP Request Size Monitor

Alert 2 is implemented as follows:

Metric: WHEN sum() of http.request.bytes OVER all documents IS ABOVE 3500 FOR THE LAST 1 minute
Threshold: over 3500 for the last minute
Vulnerability Mitigated: This will monitor for DDoS attacks because it will flag excessive request bytes.
Reliability: This is relatively reliable because the server shouldnt see massive request amounts during normal activity.
Alert 3
CPU Usage Monitor

Alert 3 is implemented as follows:

Metric: WHEN max() OF system.process.cpu.total.pct OVER all documents IS ABOVE 0.5 FOR THE LAST 5 minutes
Threshold: above 0.5 FOR THE LAST 5 minutes
Vulnerability Mitigated: An attacker accessing the system and making queries or executing files may spike the cpu usage to noticable levels.
Reliability: This may generate false positives when legitimate work is being conducted on the server and will need to be monitored.
Suggestions for Going Further (Optional)
TODO:

Each alert above pertains to a specific vulnerability/exploit. Recall that alerts only detect malicious behavior, but do not stop it. For each vulnerability/exploit identified by the alerts above, suggest a patch. E.g., implementing a blocklist is an effective tactic against brute-force attacks. It is not necessary to explain how to implement each patch.
The logs and alerts generated during the assessment suggest that this network is susceptible to several active threats, identified by the alerts above. In addition to watching for occurrences of such threats, the network should be hardened against them. The Blue Team suggests that IT implement the fixes below to protect the network:

Vulnerability 1
Patch: Brute force attacks could be mitigated by implementing some new rules on the network firewall or server to block a specific IP address after 20 error attempts in one minute. Also the wordpress server could be changed to lock out any user account that has five failed login attempts in a row.
Why It Works: Either of these patches would stop someone from brute forcing logins.
Vulnerability 2
Patch: The server should implement input validation measures to avoid and DDoS attacks.
Why It Works: Input validation will help with request size monitoring because it can control request sizes.
Vulnerability 3
Patch: Proper authorization and authentication measures should be implemented to better control access. They should also remove any excessive privvileges so that an attacker cant do as much harm on the server.
Why It Works: Controlling user accounts and privileges will decrease the ability of an attacker to change the system.
