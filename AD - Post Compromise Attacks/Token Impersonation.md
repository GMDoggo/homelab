# Token Impersonation Overview

## Token Impersonation with Delegation Tokens — MITRE ATT&CK: [Access Token Manipulation (T1134)](https://attack.mitre.org/techniques/T1134/)

Tokens are temporary keys that allow access to a system or network without requiring credentials for every action. These tokens are generated when a user authenticates to a system with their username and password.

The **Local Security Authority Subsystem Service (LSASS)** is responsible for verifying these credentials:

- **Local accounts**: LSASS verifies with the **Security Account Manager (SAM)**.
- **Domain accounts**: LSASS forwards the verification request to the **domain controller**.

### Token Types

#### Delegation Tokens

- Created when users **interactively** log into a system using their credentials, such as through:
    - Remote Desktop Protocol (RDP)
    - Virtual Network Computing (VNC)

#### Impersonation Tokens

- Created when users **non-interactively** log in to a system, typically when accessing resources without being prompted for credentials, such as:
    - Shared drives on a network

---

## Token Impersonation Attack

Token impersonation is a post-exploitation technique on Windows systems. It allows an attacker to steal the access token of a logged-in user without knowing their credentials, enabling the attacker to perform actions with the user's privileges.

### Domain Escalation Example

- **Delegation tokens** can be used for domain escalation because they contain authentication credentials.
- If a **Domain Admin** is logged into a workstation (e.g., `WS-01`), an attacker can impersonate that token, escalate privileges, and compromise the domain (e.g., by dumping secrets from the Domain Controller).

---

# Lab: Token Impersonation Attack Using Metasploit and Incognito

This lab walks through a token impersonation attack using Metasploit and Incognito.

### General Steps

1. **Open msfconsole**  
    `msfconsole`
    
2. **Load psexec**  
    Set up the options in the `psexec` module. ![Pasted image 20250224162533](https://github.com/user-attachments/assets/c1be7368-2ab8-4a4c-b800-fa633528aefe)

    
3. **Run the psexec module**  
    This opens a shell. ![Pasted image 20250224162624](https://github.com/user-attachments/assets/096f7920-cc3e-4325-a651-a37dd82e7c2b)
    
    In the shell, you'll notice that the shell is currently `nt authority\system`. ![Pasted image 20250224162738](https://github.com/user-attachments/assets/d82c95cb-5b9c-4a9c-83b3-8aca159f4979)

4. **Load Incognito for Token Impersonation**  
    Use `use incognito` to load Incognito. ![Pasted image 20250224163022](https://github.com/user-attachments/assets/6c56544c-acc1-4081-b859-2b753eb18881)

5. **List Available Tokens**  
    Use the `list_tokens -u` command to list tokens by username. ![Pasted image 20250224163400](https://github.com/user-attachments/assets/d14c856e-5464-4f5e-884e-8ef363a056dc)
    
In this example, we have the `j.smith` user in an interactive session on `WS-01`, which generates a delegation token.
    
6. **Impersonate the Token**  
    Use `impersonate_token quigley\\j.smith` to impersonate the token.
    
    **Note:** Ensure you have double-forward slashes between the domain name and the username. ![Pasted image 20250224163703](https://github.com/user-attachments/assets/250bcd29-99ee-4f7d-885e-0f47ac5b8ecf)

    
    After impersonation, dropping into the shell shows the impersonated `j.smith` user. ![Pasted image 20250224163728](https://github.com/user-attachments/assets/071b1e2b-5389-4331-84fb-5f7aaa3f5e5b)

    
### Privilege Escalation

1. **Find Active Domain Admin Sessions**  
    Use Bloodhound to locate Domain Admin sessions, which helps identify where Domain Admin delegation tokens are present.
   
   ![Pasted image 20250224164845](https://github.com/user-attachments/assets/7406c078-733d-45e2-a80b-f95ea9c3cb93)

3. **Repeat Token Impersonation Steps**  
    Use the steps from above to impersonate a Domain Admin. ![Pasted image 20250224164950](https://github.com/user-attachments/assets/c8ab3f99-9cd8-4109-a507-1ca730b44507)

4. **Successfully Impersonate Domain Admin**  
    After successfully impersonating a Domain Admin, you can perform privileged actions. ![Pasted image 20250224165051](https://github.com/user-attachments/assets/29e46753-f2d3-4e82-b589-2db668963950)

5. **Create a New Domain Admin Account (for Persistence)**  
    For persistence, create a new Domain Admin account. ![Pasted image 20250224165250](https://github.com/user-attachments/assets/de67b885-968c-43cd-9e4e-1b51116c750c)

    
6. **Dump Secrets from Domain Controller**  
    With Domain Admin privileges, use `secretsdump` on the Domain Controller to extract secrets. Only a Domain Admin would have access to do this. ![Pasted image 20250224165455](https://github.com/user-attachments/assets/88ddab45-2232-4441-95be-53b0748580ed)

---

# Mitigation Strategies for Token Impersonation

### Limit Token Creation Permissions

- Restrict which users/groups can create tokens to prevent abuse.

### Account Tiering

- Implement account tiering to limit exposure of privileged accounts, separating high-value accounts from general user accounts.

### Local Administrator Restriction

- Restrict local administrator privileges on workstations, especially where sensitive accounts may log in.
