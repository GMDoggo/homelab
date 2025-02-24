## Using `secretsdump.py`

### Dumping with a Password

`secretsdump.py domain/user:'Password'@ip_address`

- Focus on **Admin accounts** and other high-privilege accounts.
- The **DCC2** hash is especially valuable if it can be cracked.
- **Cleartext passwords** can be found in the registry, and `wdigest` on older versions of Windows may expose them.
    - If a **domain admin** logged into an older server, you could dump their password from `wdigest`.
- **Iterate** across every machine you access to find more and more accounts.

### Dumping with a Hash

`secretsdump.py user:@ip_address -hashes <hash>`

- This allows you to use a **hash** to dump credentials from a remote system, avoiding the need for cleartext passwords.
### Cracking NTLM Hashes

### Preparing Hashes for Cracking

1. Extract NTLM hashes (e.g., from `secretsdump.py` or other tools).
2. Save them to a text file (`ntlm.txt`).
3. Crack the hashes using **Hashcat**:

`hashcat -m 1000 ntlm.txt /usr/share/wordlists/rockyou.txt`

---
### Attack Pathway

1. **LLMNR Poisoning** → Steal **user hashes**.
2. **Crack the hashes** to reveal passwords.
3. **Password spraying** with cracked passwords to access new accounts.
4. Dump credentials using `secretsdump.py` to find **local admin hashes**.
5. Respray the network with **local admin passwords** to escalate access.

---
## Mitigation Strategies

- **Limit account re-use**:
    
    - Avoid using the same local admin password across multiple machines.
    - Disable **Guest** and **Administrator** accounts.
    - Implement **least privilege**: restrict who can be a local administrator.
- **Utilize strong passwords**:
    
    - Use **long passwords** (preferably >14 characters).
    - Avoid common dictionary words.
    - Long **passphrases** (e.g., a sentence) are more secure.
- **Privilege Access Management (PAM)**:
    
    - Implement **check-in/check-out** for sensitive accounts to limit exposure.
    - **Rotate passwords** automatically when checked in or out.
    - Regular password rotation reduces the risk of attackers exploiting static hashes.