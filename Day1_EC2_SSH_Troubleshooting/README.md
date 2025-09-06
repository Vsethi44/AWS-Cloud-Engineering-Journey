# EC2 SSH Troubleshooting

## Problem Statement
Critical EC2 instance became inaccessible via SSH due to misconfigured Security Group (SG) and Network ACL (NACL), creating a potential downtime risk.

## Environment Setup
- **Region:** ap-south-1 (Mumbai)  
- **AMI:** Amazon Linux 2023  
- **Instance Type:** t2.micro  
- **Key Pair:** key-pair.pem  
- **Security Group:** Initially allowed SSH (22) from laptop IP  
- **Network ACL:** Custom NACL with SSH inbound/outbound rules 
- **Public IPv4:** <EC2 Public IP, changes on restart>

## Steps to Reproduce
1. Launched EC2 instance and connected successfully. 
   ![Connected Terminal](./screenshots/01-connected-terminal.png)

2. Captured baseline system info: (`uname -a`)  
   ![EC2 Baseline](./screenshots/02-ec2-baseline.png)

3. Verified initial SSH connection working.  
   ![SSH Connect](./screenshots/03-ssh-connect.png)

4. Simulated issue: 
- Removed SG port 22 rule OR denied NACL rule.  
- SSH connection timed out.
   ![SSH Timeout](./screenshots/04-ssh-timeout.png)

5. Checked rules:
- NACL inbound: denied SSH (22).
- NACL outbound: denied all traffic. 
   ![NACL Inbound](./screenshots/05-nacl-inbound.png)  
   ![NACL Outbound](./screenshots/06-nacl-outbound.png)

6. Verify network reachability from laptop  
   ![Network Reachability](./screenshots/07-network-reachability.png)

## Error Observed
- SSH connection failed in PowerShell.
- Test-NetConnection -ComputerName <PublicIP> -Port 22 → failed.  

## Investigation
- Verified SG rules → port 22 missing / incorrect source.
- Checked NACL → SSH inbound/outbound blocked.
- Confirmed public IP changed after stop/start.
- Inside instance logs (after regaining access):(`sudo journalctl -u sshd -n 50`)

## Root Cause
Access control layers (SG + NACL) were misconfigured, blocking SSH port 22.
This broke admin access and could have caused extended downtime if not quickly identified.

## Solution
1. Added inbound SG rule: SSH (22) from laptop IP.
2. Fixed NACL: allowed inbound 22, outbound all traffic.
3. Re-tested SSH → successful.
4. Verified with: ('whoami && hostname -f')

## Verification

### SSH Success + Logs
![SSH Success + Logs](./screenshots/08-ssh-success-tail.png)

### User & Hostname Verification
![Whoami + Hostname](./screenshots/09-whoami-hostname.png) 

## Impact
1. Restored secure SSH access within minutes.
2. Prevented extended downtime by quickly identifying SG/NACL misconfiguration.
3. Improved confidence in AWS troubleshooting workflow.

## Commands Used (Reference)

ssh -i key-pair.pem ec2-user@<Public_IP>
sudo journalctl -u sshd -n 50
whoami
hostname