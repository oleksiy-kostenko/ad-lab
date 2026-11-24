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



<img width="1019" height="717" alt="{F5C43E50-C62A-41B1-9B6A-246D01BBC4C2}" src="https://github.com/user-attachments/assets/920431b7-f8ec-4e66-9f70-b8420df68911" />


<img width="1031" height="767" alt="{5D0CBCF3-54D3-4297-84C4-8CFD52194556}" src="https://github.com/user-attachments/assets/a97c8dc9-8e6e-43c4-8dbc-ceb76b2b386b" />

promoting domain controler...

<img width="1030" height="759" alt="{0BB9ABE1-FDCB-479C-9EB3-D3641CF9229D}" src="https://github.com/user-attachments/assets/b43e978c-3bf1-4e81-a6ca-06d0831a83a3" />
Now when i log into the server it should the domain name /adminstrator which it does. 
This concludes the basic ground work for the windows server for this Active Directory.

Lets start this off with a basic setup of the active directory. Next i configured the respective departments, locations and Organizations units. In this project, they are named step 2: Basic AD setup.

<img width="1028" height="735" alt="ad-before png" src="https://github.com/user-attachments/assets/7d73d8df-f18d-4161-9c05-a2098c277d94" />

