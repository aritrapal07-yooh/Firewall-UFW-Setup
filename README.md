# Firewall-UFW-Setup
Task 4: Setup and Use a Firewall on Linux (UFW)

Cyber Security Internship | Elevate Labs Author: Arti Pal Date: 13 November 2026

1. Overview

This task covers the basics of host firewall management on Linux using UFW (Uncomplicated Firewall). The work involved installing UFW, setting a default policy, allowing SSH, creating and removing a rule for port 23 (Telnet), and checking the rule list after each change.

2. Objective

Configure and test basic firewall rules to allow or block network traffic, and understand how a firewall filters traffic.

3. Environment
Item	Details
Operating system	Kali Linux 2026.3 (VMware virtual machine)
Firewall tool	UFW 0.36.2-10
Other tools	nc (netcat) for a test listener, nmap for port checks
Privileges	root (commands run with sudo)
4. Procedure
Step 1: Install UFW and check its state
bash
sudo apt update && sudo apt install ufw -y
sudo ufw status verbose

UFW was not installed on the machine, so it was installed from the Kali repository. Immediately after installation the firewall reported Status: inactive, which is the starting point for the task (Figure 1).

Step 2: Set the default policy
bash
sudo ufw default deny incoming
sudo ufw default allow outgoing

All incoming connections are denied unless a rule allows them, and all outgoing connections are allowed. This default-deny approach means only services that are opened on purpose are reachable.

Step 3: Allow SSH (port 22)
bash
sudo ufw allow 22/tcp

SSH is allowed before the firewall is switched on, so remote access is not lost when the policy takes effect.

Step 4: Enable the firewall and list the rules
bash
sudo ufw enable
sudo ufw status numbered

UFW reported that the firewall was active and enabled on system startup. The numbered list shows one rule for IPv4 and one for IPv6 (Figure 2).

Step 5: Start a test listener on port 23
bash
sudo nc -lvnp 23

Netcat was used to listen on port 23 so that there is something on the port to connect to when testing a rule. It reported listening on [any] 23 and was stopped with Ctrl+C.

Step 6: Block inbound Telnet and test it
bash
sudo ufw deny 23/tcp
sudo ufw status numbered

To test the rule, connect to port 23 from a second machine on the same network:

bash
nmap -p 22,23 <kali-ip-addresss>

With the deny rule active, port 23 is reported as filtered while port 22 stays reachable once the SSH service is running (sudo systemctl start ssh).

Note on local testing: UFW does not filter loopback traffic, and a connection to the machine's own IP address also travels over the loopback interface. A test run from the same machine (for example nc 127.0.0.1 23) will therefore connect even when a deny rule exists. Testing from another host gives a valid result.

Step 7: Remove the test rule
bash
sudo ufw delete deny 23/tcp
sudo ufw status numbered

The rule is removed to restore the original state. In the captured session this command returned Could not delete non-existent rule, which means no port 23 rule was present at that point. The listing that followed shows only the SSH rules (Figure 2).

5. Evidence
Figure 1: UFW installation and initial state

UFW installation and inactive status

A connectivity check to 8.8.8.8 was run first to confirm internet access. UFW 0.36.2-10 was then installed with apt, and ufw status showed Status: inactive.

Figure 2: Default policy, SSH rule, enable, and rule listing

UFW default policy, SSH rule and numbered status

The screenshot shows the default policy being set (incoming deny, outgoing allow), the SSH rule being added, the firewall being enabled, and the numbered rule listing. It then shows the netcat listener on port 23, followed by the cleanup command and the final rule list.

6. Results
Rules in place after configuration
No.	To	Action	From
1	22/tcp	ALLOW IN	Anywhere
2	22/tcp (v6)	ALLOW IN	Anywhere (v6)
Setting	Value
Default incoming policy	deny
Default outgoing policy	allow
Firewall state	active, enabled at system startup
Port 23 rule after cleanup	none
7. How a Firewall Filters Traffic

A firewall sits between a host or network and the traffic that reaches it. Each packet is compared with an ordered list of rules that describe the direction, protocol, port and source or destination address. The first rule that matches decides whether the packet is allowed or dropped. If no rule matches, the default policy applies.

With UFW the default policy here is deny incoming, allow outgoing. A service such as SSH is reachable only because a rule opens port 22, while other incoming ports, including 23, are closed to outside connections unless a rule says otherwise. UFW is a simpler front end for the kernel's packet filtering (iptables/nftables), so one short command such as ufw allow 22/tcp replaces several lines of low-level syntax.

Telnet (port 23) is a common port to block because it sends usernames and passwords in clear text. SSH provides the same remote access with encryption.

8. Key Learnings
Installing and enabling UFW and reading its status output.
Choosing a default-deny policy and adding exceptions for needed services.
Allowing a service before enabling the firewall so access is not lost.
Adding, listing (status numbered) and deleting rules.
Understanding that local loopback tests do not exercise UFW rules, so a second host is needed.
9. Conclusion

UFW was installed and configured with a default-deny incoming policy, an allow rule for SSH on port 22, and the firewall was enabled. The rule list was checked after each stage and the final state contains only the SSH rules. The task gave practical experience with host firewall management and with how rules and default policies decide which network traffic is accepted.

10. Disclaimer

This task was carried out on a personal lab virtual machine for educational purposes.
