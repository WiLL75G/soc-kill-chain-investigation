# Attack Narrative From Recon to Exfiltration

**Author:** William Gokah
**Project:** SOC Kill-Chain Investigation (portfolio synthesis)
**Environment:** Home detection lab Kali `192.168.64.15` (attacker), Windows `JAMES-VM` and Ubuntu targets, Splunk indexer + universal forwarders, Suricata IDS, Microsoft Sentinel.

---

## Purpose of this document

This narrative describes a single, staged intrusion that runs the full kill chain from reconnaissance to data exfiltration. It is the spine of the capstone: each stage below was **detected by one or more detections I have already built** in my existing repos, and each is investigated as an analyst would on shift.

The point of the project is not to demonstrate a new technique it is to demonstrate the **investigative thinking that connects the techniques**: one intrusion, seen from multiple layers (host, network, cloud SIEM), correlated into a single story, and mapped to both MITRE ATT&CK and the underlying networking concepts.

> **Honesty note:** every detection referenced corresponds to real lab work in my existing repositories. Where a stage is simulated rather than fully executed end-to-end, it is labelled as such. No results are invented.

---

## Cast

| Role | Host | IP | Notes |
|------|------|-----|-------|
| Attacker | Kali (`Attacker-Tier4`) | `192.168.64.15` | Source of all offensive activity |
| Target (Windows) | `JAMES-VM` | *(lab-assigned)* | Endpoint forensics subject (Sysmon + PowerShell logging) |
| Target (Linux) | Ubuntu Server | *(lab-assigned)* | SSH brute-force target |
| Detection: host | Wazuh + Sysmon → Splunk | — | Host-layer telemetry |
| Detection: network | Suricata IDS | — | Custom rules `sid 1000001` / `1000002` |
| Detection: cloud SIEM | Microsoft Sentinel | — | KQL hunts |

---

## The kill chain, stage by stage

### Stage 1 Reconnaissance
**Attacker action:** From `192.168.64.15`, the attacker runs a TCP SYN scan (`nmap -sS`) against the target to enumerate open services mapping the attack surface before committing.

**What it looks like on the wire:** a burst of SYN packets from one source to many destination ports, with SYN-ACK from open ports and RST from closed ones the classic half-open scan fingerprint.

**Detected by (existing work):** Suricata custom rule **`sid 1000001`** (port scan, T1046) from `detection-engineering-labs/03-suricata-ids`. Fired on live Kali scan traffic in the lab.

**Analyst read:** recon is a *precursor*, not yet impact but it tells you an actor is present and profiling. The value is catching it early, before the chain progresses.

---

### Stage 2 Initial Access
**Attacker action:** Having found SSH (port 22) open, the attacker launches a Hydra brute-force against the Linux target, cycling credentials until one succeeds.

**What it looks like:** a rapid sequence of failed authentications from a single source, terminated by a success a volume-then-success pattern.

**Detected by (existing work):**
- **Host layer:** Wazuh SSH brute-force detection `detection-engineering-labs/01-wazuh-ssh-bruteforce` (rules 5760/5503/5763/5551/2502/40112, level-12 compromise, T1110).
- **Network layer:** Suricata custom rule **`sid 1000002`** (SSH brute-force, T1110) same attack caught *network-side*, giving defense-in-depth.
- **Cloud SIEM:** the Sentinel SSH brute-force hunt (`sentinel-soc-lab-setup`) confirmed the same pattern (88 failed logins, 8 successes) via KQL.

**Analyst read:** this is the first CIA hit **Confidentiality** (credential compromise). The *same* attack detected at three independent layers (host, network, cloud) is the defense-in-depth story made concrete: no single blind spot hides it.

---

### Stage 3 Foothold / Command & Control
**Attacker action:** With access established, the attacker executes an obfuscated PowerShell payload on the Windows endpoint a base64-encoded command containing a download cradle and a discovery sequence, then establishes a periodic beacon.

**What it looks like:** PowerShell Script Block Logging (Event ID 4104) capturing the deobfuscated command; Sysmon Event ID 1 recording process creation; regular, machine-timed outbound connections (beaconing).

**Detected by (existing work):** PowerShell investigation lab `detection-engineering-labs/04-powershell-investigation` (Sysmon EID 1 + Script Block Logging 4104, recovered deobfuscated base64 `-EncodedCommand`, download cradle, discovery sequence, T1059.001). The Sentinel malicious-PowerShell hunt (`sentinel-soc-lab-setup`) corroborates at the cloud layer.

**Analyst read:** execution + C2 = the attacker now has a controlled foothold. CIA: **Integrity** (remote control) and continued **Confidentiality** risk. The deobfuscated command is the key artifact it reveals intent.

---

### Stage 4 — Lateral Movement & Exfiltration
**Attacker action:** From the foothold, the attacker probes internally (SMB/RDP toward other hosts east-west), then stages and exfiltrates data over a covert channel (DNS tunneling / encrypted outbound).

**What it looks like:** SMB (445) / RDP (3389) connections fanning from one internal host to several behaviour a normal user never exhibits; then high-volume or high-frequency DNS queries with long, high-entropy subdomains, or a large encrypted outbound transfer.

**Detected by (existing work):** the AI-era detection lab (`soc-ai-era-detection-lab`) demonstrates the exfil-correlation pattern (Splunk SPL detections, correlated kill chain, IOC-based). The Sentinel workspace provides the KQL hunting surface for the lateral and DNS-anomaly signals.

**Analyst read:** this is where **network segmentation** would have mattered on a flat network the lateral movement is silent east-west traffic; forcing it through a segmented, logged chokepoint is what turns it visible. Exfil is the terminal **Confidentiality** breach.

---

### Stage 5 Impact & Investigation Close
**Outcome:** data exfiltration achieved (in the scenario). The investigation reconstructs the full chain by correlating the host, network, and cloud detections into a single timeline, extracts IOCs, and documents response actions (isolate the endpoint, reset credentials, block the C2 destination, add/enforce segmentation).

**Deliverable:** the incident report (`05-incident-report/INCIDENT-REPORT.pdf`) presents this as Summary → Methodology → Findings/IOCs → Response → Conclusion.

---

## The correlation view (why this is a capstone, not a lab)

Each stage above already exists as a standalone detection in my portfolio. What this project adds is the **thread**: one attacker (`192.168.64.15`), one continuous intrusion, detected at **three layers** and reconstructed by an analyst into a coherent kill chain. Individual detections prove I can build rules; this correlation proves I can *run an investigation*.

| Stage | Layer(s) that caught it | CIA leg | Source repo |
|-------|------------------------|---------|-------------|
| Recon | Network (Suricata) | Precursor | detection-engineering-labs/03 |
| Initial Access | Host + Network + Cloud | Confidentiality | labs/01 + labs/03 + sentinel-soc-lab-setup |
| Foothold/C2 | Host (Sysmon/4104) + Cloud | Integrity + Confidentiality | detection-engineering-labs/04 |
| Lateral + Exfil | Cloud SIEM + SPL correlation | Confidentiality | soc-ai-era-detection-lab + sentinel |
| Impact/close | Correlated across all | — | this repo |
