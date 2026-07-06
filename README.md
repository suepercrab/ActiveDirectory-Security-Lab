# Active Directory Security Lab

An isolated Active Directory environment built from scratch to practice both attacking and defending a Windows domain. I stood up the domain, deployed a SIEM, ran recognized attacks from a Kali box, and wrote the detections that catch each one — all on a network isolated from everything else.

**Built by:** Hooper 

## The lab at a glance

```
+-------------------------------------------------+
| Internal Network "labnet" - no internet access  |
+-------------------------------------------------+
   |            |            |            |
+--------+  +--------+  +--------+  +--------+
| DC01   |  | WIN11  |  | KALI   |  | SIEM   |
| Server |  | Win 11 |  |attacker|  | Splunk |
| AD/DNS |  | client |  | Linux  |  | Free   |
| /DHCP  |  |(joined)|  |        |  |        |
|.40.10  |  | (DHCP) |  |.40.50  |  |.40.30  |
+--------+  +--------+  +--------+  +--------+
```

- **Domain:** `corp.lab` · subnet `10.0.40.0/24`
- **Host:** Linux Mint laptop running VirtualBox
- **Isolation:** internal-only network, no outside access 

## What I built

- A Windows Server 2022 **domain controller** running AD DS, DNS, and DHCP for the `corp.lab` forest.
- A populated directory with OUs, users, and security groups using **BadBlood** to generate a realistic messy enterprise, plus deliberately seeded weak accounts for the credential attacks.
- A **Windows 11 client** joined to the domain.
- **Group Policy** — a legal logon banner and an advanced audit policy that turns on the logging the SIEM depends on.
- A **Splunk Free SIEM** on a dedicated VM, collecting Windows Security, System, Directory Service, and Sysmon logs from DC01 and WIN11 via Universal Forwarders.
- A **Kali attacker** used to run the attack-to-detect loop below.


## Attacks and detections

| Attack | Tool | Detection (Event ID) | Notes |
|--------|------|----------------------|-------|
| Recon / port scan | nmap | — (intentionally quiet) | Pure recon barely touches Windows logs — documented as a detection gap |
| Password spray | NetExec | **4625** burst + **4624** success | Found the seeded weak account from a wall of failures |
| Kerberoasting | Impacket + hashcat | **4769** RC4 (`0x17`) | Cracked a service-account password offline |
| AD enumeration | BloodHound | **4624** NTLM logon burst | LDAP enum was quiet on 4662; pivoted to the NTLM auth burst instead |

Full detail with screenshots and Splunk queries lives in [`detections/`](./detections) and [`docs/04-attacks-detect.md`](./docs/04-attack-detection.md).


## Documentation

- [`docs/01-build.md`](./docs/01-build.md) — DC promotion, domain population, WIN11 join
- [`docs/02-group-policy.md`](./docs/02-group-policy.md) — logon banner + audit policy
- [`docs/03-siem.md`](./docs/03-siem.md) — Splunk SIEM + Universal Forwarders
- [`docs/04-attacks-detect.md`](./docs/04-attack-detection.md) — the four attacks and their detections
- [`detections/`](./detections) — per-attack detection write-ups (query, Event ID, remediation)

---

Skills demonstrated: Active Directory (Server 2022, DNS, DHCP, GPO) · Splunk SIEM engineering and log onboarding · detection writing by Windows Event ID (4625, 4769, 4624) · offensive tooling (NetExec, Impacket, hashcat, BloodHound) 

