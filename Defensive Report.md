# Blue Team: Summary of Operations

## Table of Contents
- Network Topology
- Description of Targets
- Monitoring the Targets
- Patterns of Traffic & Behavior
- Suggestions for Going Further

### Network Topology

The following machines were identified on the network:

- Kali
      Operating System: Kali Linux
      Purpose: This was the original machine from which we started our attack.
      IP Address: 192.168.1.90
- ELK
      Operating System: Ubuntu 18.04
      Purpose: Elasticsearch, Logstash, and Kabana
      IP Address: 192.168.1.100
- Capstone
      Operating System: Ubuntu 
      Purpose: This system was for testing alerts and otherwise was unused.
      IP Address: 192.168.1.105
- Hyper-V
      Operating System: Windows
      Purpose: This was our VM host system for this network.
      IP Address: 192.168.1.1
- Target 1
      Operating System: Debian/Linux
      Purpose: This system functioned as the vulnerable WordPress server.
      IP Address: 192.168.1.110

### Description of Targets

The target of this attack was: Target 1 - 192.168.1.110.

Target 1 is an Apache web server and has SSH enabled, so ports 80 and 22 are possible ports of entry for attackers. As such, the following alerts have been implemented:

### Monitoring the Targets

Traffic to these services should be carefully monitored. To this end, we have implemented the alerts below:

#### Alert 1
  Excessive HTTP Errors

Alert 1 is implemented as follows:
  - **Metric**: WHEN count() GROUPED OVER top 5 'http.response.status_code' IS ABOVE 400 FOR THE LAST 5 minutes
  - **Threshold**: 400 request codes in the last 5 minutes
  - **Vulnerability Mitigated**: Attacks on the webserver showing a brute force attempt.
  - **Reliability**: Medium

#### Alert 2
  HTTP Request Size Monitor
  
Alert 2 is implemented as follows:
  - **Metric**: WHEN sum() of http.request.bytes OVER all documents IS ABOVE 3500 FOR THE LAST 1 minute
  - **Threshold**: over 3500 for the last minute
  - **Vulnerability Mitigated**: This will monitor for DDoS attacks because it will flag excessive request bytes.
  - **Reliability**: This is relatively reliable because the server shouldnt see massive request amounts during normal activity.

#### Alert 3
  CPU Usage Monitor

Alert 3 is implemented as follows:
  - **Metric**: WHEN max() OF system.process.cpu.total.pct OVER all documents IS ABOVE 0.5 FOR THE LAST 5 minutes
  - **Threshold**: above 0.5 FOR THE LAST 5 minutes
  - **Vulnerability Mitigated**: An attacker accessing the system and making queries or executing files may spike the cpu usage to noticable levels.
  - **Reliability**: This may generate false positives when legitimate work is being conducted on the server and will need to be monitored.


### Suggestions for Going Further (Optional)
_TODO_: 
- Each alert above pertains to a specific vulnerability/exploit. Recall that alerts only detect malicious behavior, but do not stop it. For each vulnerability/exploit identified by the alerts above, suggest a patch. E.g., implementing a blocklist is an effective tactic against brute-force attacks. It is not necessary to explain _how_ to implement each patch.

The logs and alerts generated during the assessment suggest that this network is susceptible to several active threats, identified by the alerts above. In addition to watching for occurrences of such threats, the network should be hardened against them. The Blue Team suggests that IT implement the fixes below to protect the network:
- Vulnerability 1
  - **Patch**: Brute force attacks could be mitigated by implementing some new rules on the network firewall or server to block a specific IP address after 20 error          attempts in one minute. Also the wordpress server could be changed to lock out any user account that has five               failed login attempts in a row.
  - **Why It Works**: Either of these patches would stop someone from brute forcing logins.
- Vulnerability 2
  - **Patch**: The server should implement input validation measures to avoid and DDoS attacks.
  - **Why It Works**: Input validation will help with request size monitoring because it can control request sizes.
- Vulnerability 3
  - **Patch**: Proper authorization and authentication measures should be implemented to better control access. They should also remove any excessive privvileges so that an attacker cant do as much harm on the server.
  - **Why It Works**: Controlling user accounts and privileges will decrease the ability of an attacker to change the system.
