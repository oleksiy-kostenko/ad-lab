# ad-lab
Active Directory home lab on Windows Server 2025: users, groups, shared folders, Group Policy and documented troubleshooting cases.

# Active Directory Home Lab

Active Directory home lab on Windows Server 2025: users, groups, shared folders, Group Policy and documented troubleshooting cases.

Each section follows the same order: **what I did, why, the result, and what I had to troubleshoot.**

## 1. Server setup

**What I did:** Named the server DC01, set the correct time zone and gave it a static IP.

**Why:** Clients use a domain controller for DNS and sign-in, so its address must not change. The clock matters because Kerberos rejects sign-ins when the time is too far off.

**Result:** `ipconfig /all` shows the name DC01, DHCP disabled and the static address.

**Troubleshooting:**

| Problem | Cause | Fix |
|---|---|---|
| No connection | Set 10.10.10.1 with no gateway while VirtualBox NAT used 10.0.2.x | Used an address and gateway in the right network |
| No internet | DNS pointed to 127.0.0.1 before the DNS role was installed | Used 1.1.1.1 for now |
| Name didn't change | (write what you had done wrong) | Renamed it again and restarted |


<img width="1019" height="717" alt="{F5C43E50-C62A-41B1-9B6A-246D01BBC4C2}" src="https://github.com/user-attachments/assets/920431b7-f8ec-4e66-9f70-b8420df68911" />
