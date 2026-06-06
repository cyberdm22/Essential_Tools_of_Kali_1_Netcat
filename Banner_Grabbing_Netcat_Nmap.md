# Banner Grabbing using Netcat and Nmap

**Banner Grabbing** is a fundamental reconnaissance technique used to collect software, OS, and service version information from an open port. A "banner" is the text or binary metadata a service sends when a connection is first established.

## 🎯 Fundamentals of Banner Grabbing

* **The Objective:** Transition from knowing a port is open to knowing *exactly* what software and version is running behind it, paving the way for CVE (Common Vulnerabilities and Exposures) matching.
* **The "Silence" Factor:** If a connection hangs indefinitely without a response, the port is likely **Filtered** (dropped by a firewall) or waiting for specific client-side input.
* **Port States in Reconnaissance:**
  * **Open:** Service is actively listening and accepted the connection.
  * **Closed:** Host actively rejected the connection (RST packet).
  * **Filtered:** Firewall intercepted and dropped the packet; no response sent.

---

## 💻 Essential Commands & Functions

### 1. Manual Banner Grabbing (Netcat)
Used for simple, plaintext protocols (like FTP, HTTP, SSH).
```bash
nc -nv <Target_IP> <Target_Port>
```
*-n: Disables DNS resolution (speed and stealth).*

*-v: Verbose mode. Absolutely critical; without it, Netcat will not report connection failures or successes.*

### 2. Automated Service Versioning (Nmap)
Used when services communicate in complex binary (like SMB or RPC) rather than plaintext, or when scanning at scale. Nmap sends protocol-specific probes to force a response and translates the binary into readable text.
```bash
nmap -sV -p <Target_Port> <Target_IP>
```
*-sV: Instructs Nmap to interrogate the service to determine its exact version.*
*-p: Specifies the port(s) to scan (e.g., -p 21, -p- for all ports, or -p 135,139,445).*

### 3. RPC Endpoint Enumeration (rpcinfo)
When Nmap discovers open high-number ports (49152–65535) returning unrecognized binary data, these are often dynamic Microsoft RPC services. rpcinfo bypasses standard banner grabbing by querying the RPC Endpoint Mapper (Port 135) directly.
```bash
rpcinfo -p <Target_IP>
```
*Function: Dumps the entire registry of routing tables, revealing the exact internal services (like Task Scheduler or Event Viewer) mapped to those high ports.*

### 4. Creating a Fake Banner (Defensive/Lab Testing)
Used to simulate a vulnerable service or test firewall configurations.
```DOS
echo "220 Fake Secret FTP Server v1.0" | ncat.exe -lvnp 21
```

---

# 🔴 Red Teaming Insights
1. Map the Firewall, Don't Fight It: If a port is filtered, banner grabbing is physically impossible because the packets never reach the application layer. Red Teamers pivot to ports that the host operating system natively leaves open for business operations (e.g., Port 135 for RPC, Port 445 for SMB).

2. Network Profile Fingerprinting: If standard file sharing ports (139, 445) show up as filtered on a Windows target, the host's network adapter is likely set to a "Public" profile. If they are open, it is set to "Private," signaling that lateral movement via SMB exploits may be viable.

3. Infrastructure Fingerprinting via Artifacts: Identifying specific ports immediately reveals the target's underlying infrastructure. For example, discovering Port 2179 (vmrdp) open instantly informs the attacker that the target machine is acting as a Hyper-V host for virtual machines.

---

# 🚀 Practical Applications in Hacking
1. Precision Exploitation: Banner grabbing prevents "spray and pray" attacks. If a banner reveals Microsoft FTP Service 7.0, the attacker will only deploy exploits specifically engineered for that exact version, reducing network noise and lowering the chance of crashing the service.

2. OS Fingerprinting: Banners often bleed underlying operating system data (e.g., an Apache banner might read Apache/2.4.41 (Ubuntu)), confirming the OS without needing a dedicated OS-level scan.

3. Targeting Native Infrastructure: By enumerating RPC (Port 135) and SMB (Port 445), attackers identify pathways for devastating lateral movement techniques, including Pass-the-Hash, NTLM relay attacks, and remote code execution via tools like Impacket or PsExec.
