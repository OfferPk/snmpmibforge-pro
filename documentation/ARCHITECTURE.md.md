\# SNMPMibForge Pro \- Technical Architecture & Security Specification

\#\# 1\. Executive Summary

Simple Network Management Protocol (SNMP) is the standard operational telemetry and management protocol deployed across millions of routers, switches, firewalls, servers, hypervisors, and data center facilities. Despite widespread use, enterprise SNMP configurations suffer from critical vulnerabilities:  
\- \*\*Cleartext SNMPv1/v2c Community Strings\*\*: Transmitting credentials like \`public\` or \`private\` in unencrypted UDP packets allows attackers to eavesdrop on sensitive device configurations or execute malicious remote SNMP Set operations.  
\- \*\*USM Complexity & Key Localization\*\*: SNMPv3 User-Based Security Model (USM / RFC 3414\) provides authentication and privacy, but key localization requires hash-loop stretching over authoritative EngineIDs, often leading to configuration errors and unencrypted fallback modes.  
\- \*\*Deprecated Cryptographic Ciphers\*\*: Continued use of broken MD5 or SHA-1 hashes and DES 56-bit encryption violates NIST SP 800-131A and FIPS 140-3 compliance baselines.  
\- \*\*Permissive VACM MIB Views & Read-Write Exposures\*\*: Exposing the entire \`.1.3.6.1\` internet MIB root or granting Read-Write (RW) permissions allows remote adversaries to extract private OIDs or reboot devices.

SNMPMibForge Pro provides a 100% client-side studio and standalone DevSecOps CLI tool to enforce Zero-Trust SNMPv3 standards, compute RFC 3414 / RFC 7860 localized keys, audit configurations against a 10-point Zero-Trust rubric (0–100%, Grades A+ to F), and synthesize configurations across Net-SNMP, Cisco Catalyst/IOS-XE, Juniper Junos, and Prometheus SNMP Exporter.

\---

\#\# 2\. Threat Models & Vulnerability Anatomy

\#\#\# 2.1 Cleartext Community String Sniffing (v1 / v2c)  
\- \*\*The Threat\*\*:  
  In SNMPv1 and v2c, requests and responses are sent in plaintext over UDP port 161\. An attacker on the local network segment can intercept the community string using passive packet capture, query device routing tables, interface counters, and ARP caches, or execute SNMP Set commands to alter administrative state.  
\- \*\*SNMPMibForge Pro Mitigation\*\*:  
  Mandates SNMPv3 with \`authPriv\` (both HMAC authentication and AES encryption) and blocks legacy v1/v2c modes.

\#\#\# 2.2 Replay Attacks & Authoritative EngineID  
\- \*\*The Threat\*\*:  
  Without an authoritative EngineID and synchronized message timers (\`snmpEngineBoots\` and \`snmpEngineTime\`), an attacker can capture legitimate SNMP packets and replay them to disrupt management systems or spoof alarms.  
\- \*\*SNMPMibForge Pro Mitigation\*\*:  
  Strictly validates EngineID format according to RFC 3411 Section 5 and ensures USM messages enforce windowed replay protection.

\#\#\# 2.3 Key Localization Algorithm (RFC 3414 / RFC 7860\)  
\- \*\*How It Works\*\*:  
  To prevent identical passphrases on different devices from sharing the same cryptographic key:  
  1\. The user's passphrase is repeated until it fills an exact buffer of 1,048,576 bytes (1MB).  
  2\. The master digest \$K\_1\$ is calculated using SHA-256 (RFC 7860\) or SHA-1 (RFC 3414).  
  3\. The master digest is concatenated with the binary authoritative \`EngineID\` and the master digest again:  
     \$\$\\text{Localized Key} \= \\text{Digest}(K\_1 \\,\\|\\, \\text{EngineID} \\,\\|\\, K\_1)\$\$  
\- \*\*Result\*\*: Even if two routers use the same password, their localized keys differ completely because their EngineIDs are unique.

\---

\#\# 3\. 10-Point Zero-Trust SNMPv3 Rubric

| \# | Security Control | Max Pts | Passing Requirement | Threat Mitigated |  
|---|------------------|---------|---------------------|------------------|  
| 1 | SNMP Protocol Version Hardening | 15 | SNMPv3 mandated; v1/v2c blocked | Cleartext community string sniffing |  
| 2 | USM Security Level Enforcement | 15 | \`authPriv\` (Auth \+ Privacy) enforced | Unencrypted telemetry eavesdropping |  
| 3 | Cryptographic Authentication Protocol | 15 | HMAC-SHA-256 or SHA-512 (RFC 7860\) | Packet tampering & hash collision attacks |  
| 4 | Cryptographic Privacy Encryption Protocol | 10 | AES-128 / AES-192 / AES-256-CFB | Payload eavesdropping & DES brute-force |  
| 5 | Authoritative EngineID Formatting | 10 | RFC 3411 enterprise hex octet structure | Key localization failure & replay attacks |  
| 6 | VACM Least Privilege Scoping | 10 | Restricted MIB views; internet root blocked | Proprietary OID & topology leakage |  
| 7 | Read-Only (RO) Monitoring Gating | 10 | ReadOnly enforced; Read-Write blocked | Unauthorized remote configuration alteration |  
| 8 | Transport ACL & Subnet Segmentation | 5 | Dedicated /24 or /32 management subnet | Unbounded network scanning & DoS |  
| 9 | Notification Security (Inform vs Trap) | 5 | Authenticated Inform with delivery ACK | Dropped or spoofed alarm notifications |  
| 10 | Passphrase Length & Shannon Entropy | 5 | \$\\ge 16\$ characters with high entropy | Offline dictionary & brute-force cracking |

\---

\#\# 4\. Multi-Vendor Synthesizer Matrix

| Target Platform | Output Format | Key Architectural Constructs |  
|-----------------|---------------|------------------------------|  
| \*\*Net-SNMP\*\* | \`snmpd.conf\` | \`engineID\`, \`createUser \<user\> SHA256 ... AES\`, \`rouser \<user\> priv \-V \<view\>\` |  
| \*\*Cisco IOS-XE / ASA\*\* | CLI Config | \`snmp-server view\`, \`snmp-server group ... v3 priv\`, \`snmp-server user ... auth sha-256 priv aes 256\` |  
| \*\*Juniper Junos\*\* | Hierarchical \`set\` | \`set snmp v3 usm ... authentication-sha256 privacy-aes256\`, \`set snmp view\` |  
| \*\*Prometheus Exporter\*\* | \`snmp.yml\` | \`walk:\`, \`version: 3\`, \`auth: { security\_level: authPriv, auth\_protocol: SHA256 }\` |  
| \*\*Python pysnmp\*\* | Python Script | \`UsmUserData(..., authProtocol=usmHMAC192SHA256AuthProtocol, privProtocol=usmAesCfb256Protocol)\` |  
