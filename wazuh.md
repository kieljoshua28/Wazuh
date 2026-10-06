## Overview
A hands-on home lab simulating a real-world attack and detection scenario. 
I deployed a Wazuh SIEM to monitor a Linux server, simulated an SSH 
brute-force attack using Kali Linux and Hydra, then investigated and 
documented the resulting security alerts.

## Tools Used
- **Wazuh 4.14.8** — SIEM / log monitoring
- **Kali Linux** — attack simulation (Hydra, rockyou.txt wordlist)
- **VirtualBox / VMware** — virtualization

## What I Did

## Step 1: Deploy the SIEM

Downloaded the official Wazuh 4.14.8 OVA (pre-built virtual appliance).

Imported it into Oracle VirtualBox as a VM with 8GB RAM, 4 CPU cores, 50GB disk.

Powered on the VM and logged in via console using default credentials.
<img width="797" height="706" alt="Screenshot 2026-10-06 195208" src="https://github.com/user-attachments/assets/17f4e8da-8894-4a93-8507-f23fcec8a3cf" />

Retrieved the VM's IP address with ip a command (192.168.0.132).
<img width="797" height="692" alt="Screenshot 2026-10-06 195402" src="https://github.com/user-attachments/assets/a599aa81-bcf7-4948-a1ce-b64a4dac2744" />

Accessed the Wazuh web dashboard at https://192.168.0.132 from the host browser and logged in with the dashboard credentials.
<img width="1917" height="975" alt="Screenshot 2026-10-06 195601" src="https://github.com/user-attachments/assets/3981cb51-aff6-45e7-ae0a-4508c1eb17de" />

The Wazuh Dashboard
<img width="1917" height="892" alt="Screenshot 2026-10-06 195644" src="https://github.com/user-attachments/assets/81fe205f-adbd-4672-b027-5558b897050c" />

## Step 2: Set up the attacker machine

Imported a Kali Linux VM into VMware.

Configured its network adapter to the same reachable network as the Wazuh VM (NAT).

Verified connectivity between the two machines using ping <Wazuh-IP>.
<img width="1890" height="957" alt="Screenshot 2026-10-06 200051" src="https://github.com/user-attachments/assets/8ed4796b-2b1d-4f60-ab56-b893fcc6bca0" />

## Step 3: Prepare the attack

Located the rockyou.txt password wordlist at /usr/share/wordlists/. (Cheat sheet for security testing)
<img width="1702" height="912" alt="Screenshot 2026-10-06 200508" src="https://github.com/user-attachments/assets/d1f2cdfe-ae0f-4a7f-9fa3-3290a1aaa674" />

Decompressed with gunzip since it ships as a .gz file (I've done it the last time i practice so i just need to validate if it exist). I need a higher priviledge to run the command so i use sudo su to have root access.
<img width="842" height="180" alt="Screenshot 2026-10-06 200816" src="https://github.com/user-attachments/assets/aa0b5dc9-5b66-4463-a279-36b9f794cfe7" />

## Step 4: Simulate the attack
Ran Hydra from Kali Linux to perform a brute-force SSH login attempt against the Wazuh server using the rockyou.txt wordlist. The attack was allowed to run for several minutes, generating hundreds of failed login attempts, then manually stopped. A full exhaustive run against the entire 14-million-entry wordlist would take over 1,000 hours, so the attack was stopped once sufficient log data was generated for the detection and analysis phase, matching how a real intrusion attempt would typically be interrupted by detection and response.
<img width="1675" height="280" alt="Screenshot 2026-10-06 202151" src="https://github.com/user-attachments/assets/d697ee0b-d926-4910-86ca-9a31ea6a9654" />



## Key Result
Alert volume rose from 2 to 899 medium severity alerts and 8 to 5,562 low severity alerts within minutes of the simulated attack, 
demonstrating real-time detection of brute-force authentication attempts.
<img width="1917" height="895" alt="Screenshot 2026-10-06 203107" src="https://github.com/user-attachments/assets/c934e94a-d4af-4e15-8919-0e299ed8c33c" />









