# IPv6 DNS Takeover via mitm6
IPv6 attacks are similar to traditional relaying attacks but tend to be more reliable due to the use of IPv6. In many environments, IPv6 may be enabled even when the network primarily uses IPv4. This often leads to a lack of DNS server support for IPv6 traffic, creating opportunities for attackers.

By listening for all IPv6 traffic and spoofing as a DNS server, attackers can act as a middleman (MitM) for IPv6 communication. This allows them to authenticate to the domain controller via LDAP or SMB. Notably, the attacker doesn't need administrative privileges to gather valuable information—unlike in SMB Relay attacks.

In this guide, we will use **Mitm6** alongside **ntlmrelayx** to perform a relay attack.
### Step 1: Prepare the Mitm6 Command
First, open a terminal and prepare to run Mitm6 with the following command: `sudo mitm6 -d quigley.local

### Step 2

`ntlmrelayx.py -6 -t ldaps://192.168.163.5 -wh fakewpad.quigley.local -l lootme`

- `-6` specifies IPv6.
- `-t` defines the target (LDAPS in this case).
- `-wh` points to the WPAD (Web Proxy Auto-Discovery Protocol) URL.
- `-l` specifies the folder where looted information will be saved.

![Pasted image 20250217172438](https://github.com/user-attachments/assets/43df7b27-cfe5-4f35-a38f-7449d8bf49cb)


After running this command, switch back to the terminal running Mitm6.

**Important Note:** Do not leave Mitm6 running unattended, as it can cause network outages.

### Step 3: Watch for Authentications

From the `ntlmrelayx` terminal, you can monitor successful authentications to the domain controller.

![Pasted image 20250217172746](https://github.com/user-attachments/assets/9026fb37-5cf8-430d-9340-08ffffa12526)


### Step 4: Explore the Loot

When you open the `lootme` folder, you'll find files generated from the LDAP domain dump.

![Pasted image 20250217172932](https://github.com/user-attachments/assets/226700ed-0736-42c9-ba77-009996f0b00f)


#### Domain Computers

Opening the HTML file for domain computers will show all devices within the domain.

![Pasted image 20250217173046](https://github.com/user-attachments/assets/78987196-f9a2-4a9d-8cf6-512b90d3ca88)


#### Domain Users

Similarly, opening the domain users file reveals user accounts. Below, we see an example where a domain admin left a password in a description, mistakenly believing it couldn't be seen.

![Pasted image 20250217173117](https://github.com/user-attachments/assets/2debf6c7-0e12-4f4d-a8b3-5e0dd63f6094)


Some columns may contain additional useful information, such as accounts that have never been logged into—potentially indicating a honeypot account that attackers should avoid.

Notably, our high-value users include **SQLService** and **R.Quigley**.

### Step 5: Simulating an Administrator Logon

Next, simulate an administrator logging in locally to one of the workstations. Upon logon, we successfully authenticate to the domain controller as a Domain Admin account.

![Pasted image 20250217175352](https://github.com/user-attachments/assets/e7540563-50b6-4269-aec0-d804a85ea30f)

When this happens, `ntlmrelayx` automatically generates a new account for us.

![Pasted image 20250217175427](https://github.com/user-attachments/assets/b08e4ad3-198c-439b-8ffd-e533a0298543)


Looking at the domain controller, we can see that the newly created account is a member of the **Enterprise Admins** group, allowing us to run **secretsdump** against the domain.

![Pasted image 20250217175601](https://github.com/user-attachments/assets/9374e1f9-043b-4722-8de9-66b79dff8ff7)


## Mitigation Strategies

To protect against IPv6 poisoning and similar relay attacks, consider the following mitigation strategies:

- **Block IPv6 Traffic:** IPv6 poisoning leverages the fact that Windows queries for an IPv6 address even in IPv4-only environments. If your network doesn't use IPv6, block DHCPv6 traffic and incoming router advertisements in the Windows Firewall using Group Policy. Disabling IPv6 entirely may cause side effects, so instead, set the following rules to "Block":
    
    - (Inbound) Core Networking - Dynamic Host Configuration Protocol for IPv6 (DHCPv6-In)
    - (Inbound) Core Networking - Router Advertisement (ICMPv6-In)
    - (Outbound) Core Networking - Dynamic Host Configuration Protocol for IPv6 (DHCPv6-Out)
- **Disable WPAD (If Not in Use):** If WPAD is not required, disable it via Group Policy and turn off the `WinHttpAutoProxySvc` service.
    
- **Mitigate LDAP and LDAPS Relaying:** LDAP and LDAPS relaying can be mitigated by enabling both **LDAP signing** and **LDAP channel binding**.
    
- **Protect Administrative Users:** Add administrative users to the **Protected Users** group or mark their accounts as "sensitive and cannot be delegated." This will prevent the impersonation of these users via delegation.
