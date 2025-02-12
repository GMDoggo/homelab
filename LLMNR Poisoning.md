### **Overview:**

Link Local Multicast Name Resolution (LLMNR) is used when DNS fails to resolve a name. It was previously known as **NBT-NS** (NetBIOS over TCP/IP).

**Key Flaw in LLMNR:**  
LLMNR has a significant vulnerability where services use a user's username and NTLMv2 hash inappropriately when responding to requests. This makes it possible for an attacker to capture and crack NTLMv2 hashes through man-in-the-middle attacks.

We can exploit this vulnerability to intercept and capture NTLMv2 hashes by leveraging **LLMNR** with a tool like **Responder**. This allows us to catch hash requests, such as those from "fat finger" DNS lookups.

---

### **Lab Setup:**

**1. Set up Responder on the Kali Machine:**

To begin intercepting LLMNR traffic, we first need to set up **Responder** on our Kali machine:
`sudo responder -I eth0 -dw`
![](Images/Pasted%20image%2020250212172006.png)
This command sets up Responder to listen on the local interface (`eth0`), with flags for **DHCP** and **Wpad**.

**2. Generate LLMNR Traffic:**
On the virtual machine (WS-01) with IP address `192.168.163.10`, we log in as a normal user (e.g., `j.smith`). We will then attempt to navigate to our malicious host (`192.168.163.15`).

As a result, Responder intercepts the LLMNR request, capturing the NTLMv2 hash for `j.smith`.
![](Images/Pasted%20image%2020250212172108.png)

---

### **Mitigation:**

The best method to mitigate LLMNR-based attacks is to **disable LLMNR and NBT-NS** through **Group Policy Objects (GPO)**.

If disabling LLMNR and NBT-NS via GPO is not an option, the next best course of action includes:

1. **Require Network Control:**  
    Implement tighter network access controls and use network segmentation to limit the scope of attacks.
    
2. **Require Strong Passwords:**  
    Enforce strong password policies to make it more difficult for attackers to crack captured hashes.