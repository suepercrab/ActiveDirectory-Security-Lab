# Detection: Password Spraying

## The attack in one sentence

One common password (`Password123!`) tried against many domain accounts at once — designed to slip under account-lockout thresholds — to find any user with a weak password.

## How I ran it

From KALI, against the DC:

```bash
nxc smb 10.0.40.10 -u users.txt -p 'Password123!' --continue-on-success
```

The seeded weak account came back as a success while the rest failed:

```
[+] corp.lab\asnow:Password123!
```

In a small lab, I ran it several times so the failed-logon burst was visible in the SIEM rather than a single event per account.

![Password spray - Kali](../screenshots/KaliSpray.png)

## The detection

**Event ID 4625** (failed logon) is the signal — many failures across different accounts from one source IP in a tight window, with the occasional **4624** (success) marking the account that fell.

Raw events showing the burst and the success together:

```
index=wineventlog (EventCode=4625 OR EventCode=4624)
| table _time, EventCode, Account_Name, Source_Network_Address
| sort _time
```

Aggregated detection view:

```
index=wineventlog EventCode=4625
| stats count by Account_Name, Source_Network_Address
| sort - count
```

![Password spray - 4625 detection](../screenshots/SprayLogs.png)

The tell is many 4625s from a single source (KALI) in seconds — no human logs in that fast against that many accounts.

## Remediation

- Enforce strong, unique passwords and ban common patterns (a weak account is a domain-wide problem).
- Account lockout policy to slow spraying, with monitoring so the spray itself alerts.
- MFA where possible.
- Alert on the pattern of many failed logons from distinct accounts in a short window

