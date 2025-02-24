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
    Set up the options in the `psexec` module. ![psexec setup](../../../Images/Pasted%20image%2020250224162533.png)
    
3. **Run the psexec module**  
    This opens a shell. ![shell opened](../../../Images/Pasted%20image%2020250224162624.png)
    
    In the shell, you'll notice that the shell is currently `nt authority\system`. ![nt authority\system](../../../Images/Pasted%20image%2020250224162738.png)
    
4. **Load Incognito for Token Impersonation**  
    Use `use incognito` to load Incognito. ![Load incognito](../../../Images/Pasted%20image%2020250224163022.png)
    
5. **List Available Tokens**  
    Use the `list_tokens -u` command to list tokens by username. ![List tokens](../../../Images/Pasted%20image%2020250224163400.png)
    
    In this example, we have the `j.smith` user in an interactive session on `WS-01`, which generates a delegation token.
    
6. **Impersonate the Token**  
    Use `impersonate_token quigley\\j.smith` to impersonate the token.
    
    **Note:** Ensure you have double-forward slashes between the domain name and the username. ![Impersonate token](../../../Images/Pasted%20image%2020250224163703.png)
    
    After impersonation, dropping into the shell shows the impersonated `j.smith` user. ![Impersonation success](../../../Images/Pasted%20image%2020250224163728.png)
    

### Privilege Escalation

1. **Find Active Domain Admin Sessions**  
    Use Bloodhound to locate Domain Admin sessions, which helps identify where Domain Admin delegation tokens are present. ![Bloodhound search](../../../Images/Pasted%20image%2020250224164845.png)
    
2. **Repeat Token Impersonation Steps**  
    Use the steps from above to impersonate a Domain Admin. ![Impersonate DA](../../../Images/Pasted%20image%2020250224164950.png)
    
3. **Successfully Impersonate Domain Admin**  
    After successfully impersonating a Domain Admin, you can perform privileged actions. ![Success DA impersonation](../../../Images/Pasted%20image%2020250224165051.png)
    
4. **Create a New Domain Admin Account (for Persistence)**  
    For persistence, create a new Domain Admin account. ![New DA account](../../../Images/Pasted%20image%2020250224165250.png)
    
5. **Dump Secrets from Domain Controller**  
    With Domain Admin privileges, use `secretsdump` on the Domain Controller to extract secrets. Only a Domain Admin would have access to do this. ![secretsdump](../../../Images/Pasted%20image%2020250224165455.png)
    

---

# Mitigation Strategies for Token Impersonation

### Limit Token Creation Permissions

- Restrict which users/groups can create tokens to prevent abuse.

### Account Tiering

- Implement account tiering to limit exposure of privileged accounts, separating high-value accounts from general user accounts.

### Local Administrator Restriction

- Restrict local administrator privileges on workstations, especially where sensitive accounts may log in.