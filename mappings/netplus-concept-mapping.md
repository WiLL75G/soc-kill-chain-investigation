# Network+ Concept Mapping

**How each stage of the intrusion is explained by a networking fundamental.**

This is the document that makes this project a *capstone* rather than another detection lab. It maps every stage of the kill chain to the underlying Network+ concept that explains *why* the attack works and *why* the detection catches it. It is the applied output of a full networking-fundamentals study cycle, viewed entirely through a defender's lens.

The organising idea learned across that cycle: **networking and security are not separate subjects — every networking concept is a security lens.** Subnetting is scoping. DNS is an exfil channel. NAT is attribution. Ports are intent. Segmentation is blast-radius control. Monitoring is the pipeline the analyst lives in.

---

## Stage-by-stage concept mapping

### Stage 1 — Reconnaissance (port scan)
| Concept | How it applies |
|---------|----------------|
| **Ports & protocols** | The scan enumerates well-known ports (0–1023) to find listening services; the port *is* the intent signal. |
| **TCP three-way handshake** | A SYN scan sends SYN and reads the reply — SYN-ACK = open, RST = closed — never completing the handshake (half-open). |
| **OSI layering** | Recon lives at L4 (transport); knowing the layer tells you it's a network-visible event, not an endpoint one. |
| **IP addressing / subnetting** | The scan's source (`192.168.64.15`) is instantly classifiable as internal (RFC 1918); scoping it to the lab subnet is a subnetting judgement. |

### Stage 2 — Initial Access (SSH brute-force)
| Concept | How it applies |
|---------|----------------|
| **Ports & protocols** | Port 22 (SSH) is the target — a "tattoo-six" port and a lateral-movement favourite. |
| **Plaintext vs encrypted** | SSH is the *secure* replacement for Telnet (23); the attack is against the credential, not the transport. |
| **Network devices / logging** | The brute-force is visible in host auth logs, network IDS, and the SIEM — "which device saw it, and did it log it?" is answered three ways. |
| **CIA triad** | First Confidentiality hit (credential compromise). |

### Stage 3 — Foothold / C2 (obfuscated PowerShell + beacon)
| Concept | How it applies |
|---------|----------------|
| **Ports & protocols** | Beacon typically rides port 443 to blend with normal HTTPS — the port is a hint, not proof; behaviour (timing) is the detection. |
| **DNS** | C2 can use DNS resolution for check-ins; DNS is the under-monitored, usually-allowed channel attackers favour. |
| **Monitoring pipeline (SIEM)** | Endpoint events (Sysmon EID 1, PowerShell 4104) are shipped by forwarders → normalized → correlated → alerted — the analyst lives downstream of this pipeline. |
| **Encapsulation** | Reading the packet/log layers to recover the deobfuscated command mirrors peeling the encapsulation onion. |

### Stage 4 — Lateral Movement & Exfiltration
| Concept | How it applies |
|---------|----------------|
| **Segmentation / VLANs** | Lateral movement is east-west traffic; on a flat network it's a blind spot, on a segmented one it's forced through a logged L3 chokepoint. |
| **North-south vs east-west** | The exfil crosses the perimeter (north-south, heavily logged); the lateral movement stays internal (east-west, under-monitored) — the core reason both must be watched. |
| **DNS tunneling** | Exfil over DNS uses long, high-entropy subdomains and abnormal query volume — the tunneling fingerprint. |
| **NAT / PAT** | Any outbound to an external C2/exfil endpoint is attributed back to the internal host via the NAT translation table (source-port mapping). |
| **CIA triad** | Terminal Confidentiality breach (data leaves the boundary). |

### Stage 5 — Impact & Investigation Close
| Concept | How it applies |
|---------|----------------|
| **Troubleshooting methodology** | Identify → theory → test → plan → implement → verify → document *is* the incident-response skeleton and the structure of the final report. |
| **Defense-in-depth** | The chain was caught at host + network + cloud layers; each layer is an independent log source, and investigation = correlating across them. |
| **Non-repudiation** | The logs are the proof of who did what and when — provided they weren't cleared (cleared logs are themselves an IOC). |

---

## The five through-lines (the load-bearing concepts)

Every stage above rests on one or more of these five recurring ideas — the ones worth carrying into any SOC role:

1. **"Which device saw this, and did it log it?"** — visibility is everything; you can't defend what you can't see.
2. **Every alert → CIA leg → fingerprint → log source** — the identification chain that structures triage.
3. **Benign cause + malicious twin, held simultaneously** — the dual-hypothesis discipline that catches incidents hiding as tickets.
4. **Trust boundaries are control and monitoring points** — perimeter, VLANs, cloud identity.
5. **Automated detection has a ceiling; the analyst lives in the gap** — signature misses novel, anomaly drowns in noise; the human resolves ground truth.

---

## Why this matters for the role

Detection labs prove I can *build* rules. This mapping proves I understand the *networking substrate the rules sit on* — why C2 hides on 443, why exfil favours DNS, why segmentation changes what's detectable, why NAT logs are the attribution key. That "why" under the detections is the difference between writing a rule and engineering detection.
