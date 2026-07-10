# SOC Kill-Chain Investigation

**A single intrusion, from reconnaissance to exfiltration, detected across host, network, and cloud layers and reconstructed into one correlated investigation.**

Author: **William Gokah** · Detection Engineer / SOC Analyst
GitHub: [WiLL75G](https://github.com/WiLL75G) · Focus: remote Tier 1/2 detection engineering

---

## What this project is

Most portfolio projects demonstrate a single detection technique. This one demonstrates the **investigative thinking that connects techniques** the skill that separates a detection engineer from a rule-writer.

I stage one continuous intrusion by a single attacker (`192.168.64.15`), let it run the full kill chain, and show how detections I have **already built across my portfolio** each catch a different stage. Then I do what an analyst actually does on shift: correlate those separate signals host, network, and cloud into a single timeline, extract the IOCs, map every stage to MITRE ATT&CK and to the underlying networking fundamentals, and write it up as a professional incident report.

**The thread is the point.** Any one detection proves I can write a rule. Stitching five stages across three telemetry layers into one coherent story proves I can run an investigation.

---

## The kill chain

| Stage | Attacker action | Detected at layer | CIA leg | MITRE |
|-------|-----------------|-------------------|---------|-------|
| 1. Reconnaissance | SYN port scan | Network (Suricata) | Precursor | T1046 |
| 2. Initial Access | SSH brute-force | Host + Network + Cloud | Confidentiality | T1110 |
| 3. Foothold / C2 | Obfuscated PowerShell + beacon | Host (Sysmon/4104) + Cloud | Integrity + Confidentiality | T1059.001 |
| 4. Lateral + Exfil | SMB/RDP fan-out, DNS tunneling | Cloud SIEM + SPL correlation | Confidentiality | T1021 / T1048 |
| 5. Impact / Close | Data exfiltration, IR | Correlated across all | — | — |

> Detections referenced are real work from my existing repositories. Simulated stages are labelled as such; no results are invented. See the [attack narrative](00-scenario/attack-narrative.md) for full detail.

---

## Repository structure

```
soc-kill-chain-investigation/
├── README.md                          <- you are here
├── 00-scenario/
│   └── attack-narrative.md            <- the staged intrusion, stage by stage
├── 01-recon/
│   └── detection-and-findings.md
├── 02-initial-access/
│   └── detection-and-findings.md
├── 03-foothold-c2/
│   └── detection-and-findings.md
├── 04-lateral-exfil/
│   └── detection-and-findings.md
├── 05-incident-report/
│   └── INCIDENT-REPORT.pdf            <- flagship deliverable
├── mappings/
│   ├── mitre-attack-mapping.md        <- every stage -> ATT&CK tactic/technique
│   └── netplus-concept-mapping.md     <- every stage -> networking fundamental
└── screenshots/
```

---

## Source detections (the layers this project orchestrates)

This capstone does not rebuild detections it correlates ones I have already built. Each stage links back to its source repo:

| Layer | Detection | Source repo |
|-------|-----------|-------------|
| Host Linux | SSH brute-force (Wazuh, rules 5760–40112, level-12 compromise) | `detection-engineering-labs/01` |
| Network | Suricata IDS, custom rules `sid 1000001` (scan) / `1000002` (brute-force) | `detection-engineering-labs/03` |
| Host Windows | PowerShell investigation (Sysmon EID 1 + Script Block Logging 4104, base64 deobfuscation) | `detection-engineering-labs/04` |
| Cloud SIEM | Sentinel KQL hunts (SSH brute-force + malicious PowerShell) | `sentinel-soc-lab-setup` |
| Correlation | Splunk SPL kill-chain correlation, IOC-based | `soc-ai-era-detection-lab` |

*Link targets are placeholders confirm each repo is public before publishing.*

---

## Lab environment

| Role | Host | IP |
|------|------|-----|
| Attacker | Kali (`Attacker-Tier4`) | `192.168.64.15` |
| Target Windows | `JAMES-VM` | *(lab-assigned)* |
| Target Linux | Ubuntu Server | *(lab-assigned)* |
| Detection stack | Splunk (indexer + forwarders), Suricata IDS, Microsoft Sentinel | — |

---

## What this demonstrates to a hiring manager

- **Investigation over isolated detection** correlating multi-layer telemetry into one narrative, the core Tier 1/2 skill.
- **Defense-in-depth in practice** the same attack caught independently at host, network, and cloud, proving no single blind spot.
- **ATT&CK fluency** every stage mapped to tactic and technique.
- **The "why" under the rules** the [networking-concept mapping](mappings/netplus-concept-mapping.md) shows I understand the substrate detections sit on (why C2 hides on 443, why exfil favours DNS, why segmentation changes what's detectable, why NAT logs are the attribution key).
- **Professional reporting** a structured incident report (Summary -> Methodology -> Findings -> Response -> Conclusion).

---

## Read next

-> [The attack narrative](00-scenario/attack-narrative.md) the full staged intrusion
-> [Networking-concept mapping](mappings/netplus-concept-mapping.md) the fundamentals under each stage
-> [Incident report (PDF)](05-incident-report/INCIDENT-REPORT.pdf) the flagship write-up
