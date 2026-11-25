# ad-lab By Oleksiy Kostenko (Junior IT helpdesk L1)
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

Let's start this off with a basic setup of the active directory. Next i configured the respective departments, servers,locations and Organizations units. In this project, they are named 

## 2. Basic AD setup

**Before:** default domain, only the built-in containers.
<img width="1028" height="735" alt="ad-before png" src="https://github.com/user-attachments/assets/7d73d8df-f18d-4161-9c05-a2098c277d94" />
**After:** regional OUs for Europe and USA, each with desktops, laptops and seven departments.

**Result:** I created two regional OUs, Europe and USA. Each has separate OUs for desktops, laptops and users, and the users are split into seven departments (Sales, Support, Operations, Finance, Marketing, HR and IT). Servers have their own OU, split by region. This lets me apply policies to a whole region, one department, or only the laptops.
<img width="1038" height="771" alt="ad-AFTER png (2)" src="https://github.com/user-attachments/assets/dab3da73-def5-4b08-9da9-a6320ed3ab1e" />


**Troubleshooting:**

| Problem | Cause | Fix |
|---|---|---|
| I created an OU with the wrong name and couldn't delete it | New OUs are protected from accidental deletion by default | Turned on View > Advanced Features, opened the OU's Properties > Object tab, unticked "Protect object from accidental deletion", then deleted it. Kept this setting enabled by default to all the OUs that i wanted to keep.


## 3. Security groups

**What I did:** Created one global security group for each department in each region, 14 in total, inside the department's own OU. For example, GG-Finance-EU is in Finance-EU.

**Why:** Permissions are given to groups, not to individual people. This way, when new users join or someone moves within the company, I can simply move them between groups instead of changing the folder permissions for each user.

**Result:** A search for "GG-" returns 14 groups.

**Next:** The groups are empty for now. I'll add the users and give the groups access to the shared folders in a later step. For now I'm laying the groundwork for the company.

<img width="1024" height="773" alt="SecurityGroups-AD png" src="https://github.com/user-attachments/assets/b1c28b14-c517-48ae-80a6-a3ede2989035" />


## 4. Distribution groups

**What I did:** Next to the security groups, I made 14 distribution groups, one for each department in each region. For example, DL-Finance-USA sits in Finance-USA.

**Why:** A distribution group is basically an email list. Send one message to it and everyone in the group gets it. It can't give anyone access to a folder, which is why I keep it separate from the security groups.

**Result:** 14 distribution groups, one next to each security group. A search for "DL" returns all 14, and the second picture shows both kinds in the same OU: GG-Finance-USA is the security group and DL-Finance-USA is the distribution group.

**Next:** These groups only hold the list for now. They'll start sending mail once I add a mail server like Exchange or Microsoft 365. This is planned for my 365 lab later.

<img width="1035" height="778" alt="Distrib-list-14 PNG" src="https://github.com/user-attachments/assets/ef5d371e-0654-4130-83db-39a6050ab846" />


<img width="1029" height="778" alt="Distribuition groups png" src="https://github.com/user-attachments/assets/73a1f90b-e9c6-4305-9c98-491aef74a952" />




