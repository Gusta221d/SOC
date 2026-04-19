# SOC Lab Report: Web Server Compromise & Exfiltration

**Platform:** TryHackMe | **Scenario:** Splunk Basics - Did you SIEM? | **Tool:** Splunk SIEM

---

## 1. Executive Summary
In this lab, I used Splunk to investigate a compromised web server. By writing SPL queries to dig into the logs, I managed to track the attacker's path from their initial noisy scans to the exact moment they stole the data. Also, the attacker abused web vulnerabilities to poke around the server and grab sensitive files. While I successfully tracked down what happened, this exercise really highlighted a big takeaway: we need stricter outbound firewall rules to stop data from leaving the network in the first place!

## 2. Incident Investigation Log
This simulation was all about putting the puzzle pieces together using both web and firewall logs.

### Incident A: Web Attacks - SQLi & Path Traversal 

* **Severity:** Critical - True Positive
* **Affected Entities:** Internal Web Server, Attacker IP (198.51.100.55).
* **Analysis:**
  * **Initial Vector:** I noticed a massive spike in web traffic using Splunk's `timechart` command on the `web_traffic` logs, which looked like a brute-force scan.
  * **Payload:** Looking closely at the `user_agent` field, I spotted the attacker using *Havij*—a tool specifically made for automated SQL Injections (SQLi). After that, I filtered the logs for patterns, which showed they switched to Directory Traversal to read backend system files.
  * **Impact Assessment:** The attacker successfully abused these vulnerabilities to navigate the server's backend and look for sensitive data.
  * **Conclusion:** The web app was actively and successfully exploited by an external attacker.

* **Remediation & Response:**
  * Escalate the alert to the incident response team.
  * Immediately block IP `198.51.100.55` on the firewall.
  * Report to the dev team to patch those SQLi and Directory Traversal vulnerabilities.

### Incident B: Data Exfiltration - Stolen Data

* **Severity:** Critical - True Positive
* **Affected Entities:** Internal Web Server, Attacker's Command & Control Server.
* **Analysis:**
  * **Initial Vector:** After breaking into the web app, the internal server started making outbound connections to the attacker's machine.
  * **Payload:** Switching over to the `firewall_logs` and matched the server's internal IP with the attacker's IP to track the outbound traffic.
  * **Impact Assessment:** By using the `stats sum(size_bytes)` command in Splunk, I was able to calculate the exact amount of data (13kb) the attacker managed to steal before the connection was dropped.
  * **Conclusion:** The initial web breach led directly to a confirmed data leak.

* **Remediation & Response:**
  * Quarantine the web server from the rest of the network to stop any more data leaks or lateral movement.
  * Force a password reset for all accounts tied to that server.

## 3. Performance Review & Post-Mortem

* **Outcome:** Successfully mapped the whole attack chain, identified the attacker, and calculated exactly how much data was compromised.
* **Metrics Analysis:**
  * **Visibility:** Achieved 100% visibility into the attack by combining application logs (`web_traffic`) with network logs (`firewall_logs`).
* **Lesson Learned:** The web app bug was the main issue, but the fact that data got out shows that the network rules need to be updated. Moving forward, web servers shouldn't be allowed to connect to random external IPs. Setting up a "Default Deny" rule for outbound traffic would have stopped the data theft dead in its tracks, even if the web app was compromised.