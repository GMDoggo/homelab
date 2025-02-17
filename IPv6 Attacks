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

![Command Execution](../../../Images/Pasted%20image%2020250217172438.png)

After running this command, switch back to the terminal running Mitm6.

**Important Note:** Do not leave Mitm6 running unattended, as it can cause network outages.

### Step 3: Watch for Authentications

From the `ntlmrelayx` terminal, you can monitor successful authentications to the domain controller.

![Successful Authentications](../../../Images/Pasted%20image%2020250217172746.png)

### Step 4: Explore the Loot

When you open the `lootme` folder, you'll find files generated from the LDAP domain dump.

![LDAP Domain Dump](../../../Images/Pasted%20image%2020250217172932.png)

#### Domain Computers

Opening the HTML file for domain computers will show all devices within the domain.

![Domain Computers](../../../Images/Pasted%20image%2020250217173046.png)

#### Domain Users

Similarly, opening the domain users file reveals user accounts. Below, we see an example where a domain admin left a password in a description, mistakenly believing it couldn't be seen.

![Domain Admin Password](../../../Images/Pasted%20image%2020250217173338.png)

Some columns may contain additional useful information, such as accounts that have never been logged into—potentially indicating a honeypot account that attackers should avoid.

Notably, our high-value users include **SQLService** and **R.Quigley**.

### Step 5: Simulating an Administrator Logon

Next, simulate an administrator logging in locally to one of the workstations. Upon logon, we successfully authenticate to the domain controller as a Domain Admin account.

![Domain Admin Authentication](../../../Images/Pasted%20image%2020250217175352.png)

When this happens, `ntlmrelayx` automatically generates a new account for us.

![Account Creation](../../../Images/Pasted%20image%2020250217175427.png)

Looking at the domain controller, we can see that the newly created account is a member of the **Enterprise Admins** group, allowing us to run **secretsdump** against the domain.

![Enterprise Admins Account](../../../Images/Pasted%20image%2020250217175601.png)

## Mitigation Strategies

To protect against IPv6 poisoning and similar relay attacks, consider the following mitigation strategies:

- **Block IPv6 Traffic:** IPv6 poisoning leverages the fact that Windows queries for an IPv6 address even in IPv4-only environments. If your network doesn't use IPv6, block DHCPv6 traffic and incoming router advertisements in the Windows Firewall using Group Policy. Disabling IPv6 entirely may cause side effects, so instead, set the following rules to "Block":
    
    - (Inbound) Core Networking - Dynamic Host Configuration Protocol for IPv6 (DHCPv6-In)
    - (Inbound) Core Networking - Router Advertisement (ICMPv6-In)
    - (Outbound) Core Networking - Dynamic Host Configuration Protocol for IPv6 (DHCPv6-Out)
- **Disable WPAD (If Not in Use):** If WPAD is not required, disable it via Group Policy and turn off the `WinHttpAutoProxySvc` service.
    
- **Mitigate LDAP and LDAPS Relaying:** LDAP and LDAPS relaying can be mitigated by enabling both **LDAP signing** and **LDAP channel binding**.
    
- **Protect Administrative Users:** Add administrative users to the **Protected Users** group or mark their accounts as "sensitive and cannot be delegated." This will prevent the impersonation of these users via delegation.
