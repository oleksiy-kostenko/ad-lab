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


## 5. Importing 300 users

**What I did:** I needed a lot of users, and typing 300 by hand wasn't realistic. So I used AI to help me write two PowerShell scripts: one makes a list of 300 made-up employees, and the other imports them into Active Directory. I ran a dry run first with -WhatIf to see what it would do, then the real import. Each user lands in the OU for their region and department, and gets added to that department's security group.

**Why:** Doing 300 accounts by hand would take hours, and I'd probably make typos. A script does it the same way every time, and the dry run lets me check before anything changes.

**Result:** 180 users in Europe and 120 in the USA, each in the right department OU. Everyone has a temporary password and has to change it the first time they sign in.

<img width="1021" height="764" alt="PowershellUsercreation" src="https://github.com/user-attachments/assets/4e200c6d-da73-47ff-8d7f-9a3d8bf820f8" />

(picture: the -WhatIf output)
<img width="1025" height="760" alt="PowerShellSuccess" src="https://github.com/user-attachments/assets/155668c3-9fe8-409d-946e-a8a292c6ff2b" />

(picture: Operations-USA with its 24 users and two groups)
<img width="1021" height="211" alt="ConfirmationPSworked" src="https://github.com/user-attachments/assets/8cb80e33-d44b-4717-b632-ddcb93940752" />


## 6. Creating Group Policies (GPOs)

I'm creating several policies, linking each one to the OU where it should apply. Once the client PC joins the domain (step 7), I'll test each one from the client and add the screenshots here.

### 6.1 Domain password policy

**What I did:** I created a GPO called Password Policy and linked it to the domain. It sets a minimum of 12 characters, complexity on, 15 old passwords remembered, a 90-day maximum age and a 1-day minimum age.

**Why:** Password rules only work when they're set at the domain level, so I linked this one to the domain itself and not to an OU. I picked 90 days and 1 day so users aren't forced to change their password all the time, but can change it if they need to.

**Result:** Local Security Policy on the DC shows my values.
<img width="1015" height="472" alt="{CEAD0D1D-82D0-4BDE-8451-AC3BF26FD41F}" src="https://github.com/user-attachments/assets/2e0365d9-9304-4dcf-a690-208490506c49" />


**Troubleshooting:**
<img width="1032" height="777" alt="PasswordpolicyLink" src="https://github.com/user-attachments/assets/d747e58f-c783-44dc-ac49-73c56d31627b" />

| Problem | Cause | Fix |
|---|---|---|
| My password policy didn't apply. Local Security Policy still showed 7 characters and 42 days | The built-in Default Domain Policy was Link Order 1, so it won over mine | Moved my Password Policy to Link Order 1, ran gpupdate /force, and checked again |






### 6.2 Screen lock policy

**What I did:** I created a GPO called Screen Lockout and linked it to Users-EU and Users-USA. It turns on the blank screen saver, locks the screen after 10 minutes (600 seconds) of no use, and asks for the user's password to unlock it.

**Why:** If someone leaves their desk, nobody else can sit down and use their account. I linked it to the user OUs and not the whole domain, so it doesn't lock the server administrator out of the domain controller every ten minutes.

**Linked to:** Users-EU and Users-USA.

**Result:** In the editor, the four settings show as Enabled.

**Test:** This one is hard to show in a screenshot, since a picture can't prove the screen locked after 600 seconds (10 minutes) of no activity. I tested it, and it works.

<img width="1029" height="771" alt="screenlockoutpolicy" src="https://github.com/user-attachments/assets/be78ff6e-48fa-49c7-913c-eb284267d1d7" />

### 6.3 Block Control Panel

**What I did:** I created a GPO called Control Panel Block and linked it to Users-EU and Users-USA. It turns on "Prohibit access to Control Panel and PC settings", so users can't open Control Panel or the Settings app.

**Why:** Most users never need to change system settings. When someone changes the wrong thing, like the network settings, it turns into a helpdesk ticket. Blocking it means fewer tickets and fewer accidents.

**Linked to:** Users-EU and Users-USA, so it covers every department.

**Result:** In the editor the setting shows as Enabled, and the policy appears under both user OUs.
<img width="1034" height="780" alt="Control Pannel block but it can use it" src="https://github.com/user-attachments/assets/eaaee1fe-be55-43b1-91eb-86be819072c3" />

**Excluding IT:** IT staff still need Control Panel for their work. So I used the Delegation tab to deny this policy to the groups GG-IT-EU and GG-IT-USA. Everyone else is blocked, and IT is left out.
<img width="1034" height="780" alt="Control Pannel block but it can use it" src="https://github.com/user-attachments/assets/f0c6c8e4-bb22-4344-9d80-0fc355eb2adf" />

**Test from the client:** I signed in as ana.fonseca, a Finance user, and tried to open Control Panel. It was blocked with a restrictions message, so the policy works.
<img width="640" height="214" alt="{09A4E388-AE2D-486E-BD66-1767B3E07FA5}" src="https://github.com/user-attachments/assets/46c05fbb-296c-47ea-bfef-45176e57ddfa" />

**Test from the client:** I signed in as Joao.Pinto, a IT user, and tried the same as before, opening the control pannel, and as you can see we created a delegation where that policy doesn't apply to this OU. 
<img width="948" height="657" alt="{7D56ED4B-F0BF-47CB-B5D3-0DBB1FF86BCF}" src="https://github.com/user-attachments/assets/9502e40f-ed73-4b48-b3a8-1ba97352433d" />


### 6.4 Block USB storage

**What I did:** I created a GPO called Block USB storage and linked it to Users-EU and Users-USA. It uses the user setting "All Removable Storage classes: Deny all access", so no one can use USB sticks or other removable drives.

**Why:** A USB stick is an easy way for company data to leave the building, and an easy way for malware to come in. Blocking it for all users is a simple rule that's easy to explain and check. I used a user policy so it follows the person to any PC they sign in on.

**Linked to:** Users-EU and Users-USA.

**Result:** In the editor the setting shows as Enabled.

<img width="1032" height="770" alt="USBstoragePolicy" src="https://github.com/user-attachments/assets/ad495ddf-9489-4b59-a2d4-2dd908cad264" />

**Test:** This one is hard to show reliably with screenshots, so there's no test picture here. The screenshot above shows how the policy is set up.

### 6.5 Windows Update

**What I did:** I created a GPO called Windows Updates and linked it to Europe and USA, so it flows down to the desktop and laptop OUs in each region. It uses "Configure Automatic Updates", so updates download automatically and install at a set time (03:00) local time of each department so it doesn't interfere with working hours.

**Why:** Unpatched PCs are one of the easiest ways in for attackers. A policy makes sure updates happen without users having to do anything.

**Result:** In the editor the setting shows as Enabled.

<img width="1035" height="776" alt="windowsupdatepolicy" src="https://github.com/user-attachments/assets/2e273ec5-da3a-4004-9788-8db189e282ab" />

**Test:** Updates install on a schedule, so this is hard to show reliably with screenshots. The screenshot above shows how the policy is set up.

## 7. Adding a Windows 11 client to the domain

**What I did:** I created a Windows 11 Enterprise virtual machine to act as a user's work PC, and tried to join it to my domain.

**Why:** So far I'd set everything up from the server. With a client PC, I can see what a real user would see and test my policies.

**The problem:** The PC said the domain controller could not be contacted. This was mostly an oversight on my part. I've been studying M365 basics, where devices connect over the internet, and it totally escaped me that my two VMs needed to be on the same network. Well, I won't forget that again!

<img width="844" height="479" alt="error joining the domain" src="https://github.com/user-attachments/assets/a7c10bee-ff4c-4e44-a80a-8a51a5c4c37c" />

**The fix:** My two virtual machines couldn't see each other, so I moved both to the same virtual network (NatNetwork). Then I set the client's DNS to the server's IP address (10.0.2.10) and made sure the server was turned on. My domain is private, so only my server knows where to find it.
<img width="1920" height="249" alt="creating a virtual network" src="https://github.com/user-attachments/assets/9237b403-e0ad-47e0-bb8f-9b1237701cad" />

**Result:** The client found the domain and asked for a username and password to join it.
<img width="982" height="679" alt="Successfully joined domain" src="https://github.com/user-attachments/assets/ef3ae416-c55b-42db-b91b-5bb341ff31cf" />

<img width="1023" height="769" alt="Password policy working" src="https://github.com/user-attachments/assets/83ed15bb-131d-4086-9eaf-98344d8e64b1" />

**Password reset:** The PC joined the domain, but when I went to sign in as ana.fonseca, I had forgotten the password. That gave me a chance to practise a real helpdesk task. I reset it in Active Directory and ticked "User must change password at next logon". When changing it, I tried a short password first, and it was rejected. A password that met my rules worked, which shows my password policy applies to domain users.
<img width="598" height="697" alt="passwrodpolicyworkinV1" src="https://github.com/user-attachments/assets/a1c4868e-7c2d-4ceb-9700-cb8d92fa1a39" />

**Step 8:** Setting up an Shared file.

**What I did:** On the server, I created a folder called "SharedFolder" and shared it over the network. In the share permissions, I gave Everyone read access, then denied access to one group, GG-HR-EU.
<img width="816" height="535" alt="{D9C3FAB3-3DBD-405E-898D-176723254291}" src="https://github.com/user-attachments/assets/760ba28a-c81a-4f46-97e9-f7c3cb6077d0" />

**Why:** Since this is a home lab, and my goal is to show whoever is reading this that I can do the job, I kept it simple: create a shared folder and set who can and can't access it. Using a group means I don't have to set permissions user by user. Anyone added to GG-HR-EU gets blocked too, from their next sign-in.

**Result:** The folder is shared, and GG-HR-EU is denied. Deny wins over Allow here, so its members can't get in over the network.
<img width="891" height="542" alt="{604B15C8-FCDC-4A9D-ADC5-D14C488589FE}" src="https://github.com/user-attachments/assets/b57b5d71-65c3-4a3d-bdaa-221786f90a9e" />


**Test from the client:** Next, I tested whether a user could connect to the folder. I signed in on the Windows 11 PC as joao.pinto, who isn't in the blocked group. The whoami command shows who's signed in, and SharedFolder shows up as network drive S: (\\DC01\SharedFolder).
<img width="895" height="633" alt="{EC826BE0-7757-4E68-A62E-4DF96BB79A37}" src="https://github.com/user-attachments/assets/dd7536ca-1692-4f61-8c1c-5cef5d0a96f3" />

### Mapping the drive with a Group Policy

**What I did:** I created a GPO called "Mapped Drives". I set it up under User Configuration > Preferences > Windows Settings > Drive Maps, with these settings:

| Setting | Value | What it means |
|---|---|---|
| Action | Update | Creates the drive if it's missing, or updates it if it already exists |
| Location | \\DC01\SharedFolder | The shared folder on my server |
| Drive letter | S: | The letter set for the drive |
| Label | Shared | The name I gave the drive |

**Why:** Users shouldn't have to type the folder path or set up the drive themselves. The GPO does it for them when they sign in, and if the path ever changes, I only need to update it in one place.

**Access:** The GPO only adds the drive. It doesn't give anyone permission. The folder's permissions still decide who can open it, and GG-HR-EU is denied.

(picture: the S: drive settings in the GPO editor)
<img width="1031" height="710" alt="{47A76BC0-B4FD-4CD2-9CF0-7929D36E7CCC}" src="https://github.com/user-attachments/assets/2dac1a8a-d032-46ce-85e4-402d7496be5d" />

**Test from the client:** I signed in on the Windows 11 PC as joao.pinto and ran whoami to confirm which user was signed in. Then I ran gpupdate /force to get the latest policies, and opened This PC.

**Result:** The policy update completed successfully, and the drive shows as Shared (S:), with the label I set in my GPO.
<img width="967" height="687" alt="{6F20ED66-6A1E-4C94-AB54-95D3C13CCE7A}" src="https://github.com/user-attachments/assets/3506b1f2-6a2f-4faa-9779-3ae01dcb855a" />

**Test with a blocked user:** Next, I signed in as carla.sousa, an HR-EU user, and tried to open the Shared (S:) drive.

**Result:** Access was denied. carla.sousa still sees the drive, but can't open it. This shows that the GPO only adds the drive, while the folder's permissions decide who can get in. The deny in OU GG-HR-EU works.
<img width="1009" height="720" alt="{09E1EFC1-6F1C-4DDE-BDB9-95D01B654D4E}" src="https://github.com/user-attachments/assets/cb9b770a-d054-4e33-a005-d193dfc0397b" />








