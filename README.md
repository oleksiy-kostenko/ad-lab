# Active Directory Home Lab

### CompTIA A+ · Aspiring IT Helpdesk / Service Desk (L1)

Hi, I'm **Oleksiy Kostenko**. After several years in hospitality, where I learned to stay calm and help people, I'm moving into IT.

- **Looking for:** my first IT Helpdesk / Service Desk (L1) job in the Porto area, Portugal
- **Certification:** CompTIA A+
- **Languages:** Portuguese, English and Russian (native level)

I like learning by doing, so I built this lab to practise what a helpdesk deals with. I tested key parts as different users, and wrote down what broke and how I fixed it.
(documented)
### Skills

- **Practised in this lab:** Active Directory · Group Policy · File permissions · DNS troubleshooting · Windows 11 and Server 2025
- **A+ knowledge:** TCP/IP and DHCP · VPN · Remote Desktop · Hardware and printers
- **Currently learning:** Microsoft 365 admin · Entra ID basics
- **Additional skills:** Ticketing practice (simulator) · Documentation · Using knowledge bases · Clear communication · Friendly, patient and attentive

### Also working on

- **Microsoft 365|Entra ID lab:** a small lab to practise administration.
- **Service desk simulator:** I'm practising troubleshooting, talking to users and documenting realistic tickets. It's practice, not real work experience, but it matters because I haven't worked in IT yet. My goal is around 200 tickets over 30–50 hours, with the harder ones documented from start to finish.
  
**Jump to:** [Proof it works](#proof-it-works) · [Problems I solved](#problems-i-solved) · [How I built it](#how-i-built-it) · [What's next](#whats-next)

## What I practised, in helpdesk terms

| When a user says... | What I did in my lab |
|---|---|
| [**"I forgot my password."**](#a-forgotten-password) | Reset a password in Active Directory, with a forced change at next sign-in |
| [**"My PC won't join the domain."**](#7-adding-a-windows-11-client) | Fixed it by checking the network, DNS and whether the server was on |
| [**"I can't open this folder."**](#8-file-shares-and-permissions) | Set up share and NTFS permissions, inheritance and Deny |
| [**"Where's my shared drive?"**](#proof-it-works) | Mapped a network drive for users with Group Policy |
| [**"Control Panel says it's blocked."**](#proof-it-works) | Blocked it for normal users with Group Policy, but left it open for IT |
| [**"We need accounts for new people."**](#5-adding-300-users-with-powershell) | Added 300 users to the right OUs and groups with PowerShell (AI wrote the scripts) |

## Proof it works

I signed in on a Windows 11 PC as different users to check my settings. Click any screenshot to see it full size.

**Same policy, different result.** Control Panel is blocked for normal users, but IT can still open it.

| Finance (`ana.fonseca`): blocked ✅ | IT (`joao.pinto`): opens ✅ |
|:---:|:---:|
| <img width="380" alt="Control Panel blocked for ana.fonseca with a restrictions message" src="https://github.com/user-attachments/assets/46c05fbb-296c-47ea-bfef-45176e57ddfa"> | <img width="380" alt="joao.pinto can open Control Panel" src="https://github.com/user-attachments/assets/9502e40f-ed73-4b48-b3a8-1ba97352433d"> |

**Same drive, different access.** Group Policy adds the S: drive for both users, but EU HR isn't allowed in.

| IT (`joao.pinto`): drive appears ✅ | HR (`carla.sousa`): access denied ✅ |
|:---:|:---:|
| <img width="380" alt="gpupdate succeeds and the Shared (S:) drive appears for joao.pinto" src="https://github.com/user-attachments/assets/3506b1f2-6a2f-4faa-9779-3ae01dcb855a"> | <img width="380" alt="carla.sousa gets access denied on the shared drive" src="https://github.com/user-attachments/assets/cb9b770a-d054-4e33-a005-d193dfc0397b"> |

<sub>Where you see a command window, I ran `whoami` to show who's signed in. In the drive test, I also ran `gpupdate /force` for `joao.pinto` to get the latest policies.</sub>

I also tested [joining the PC to my domain](#7-adding-a-windows-11-client) and [a forced password change](#a-forgotten-password).

## Problems I solved

Not everything worked the first time, and these taught me the most:

- **My PC couldn't find the domain controller.** I've been studying Microsoft 365 basics, where devices connect over the internet, and it totally escaped me that my two VMs needed to be on the same network. After that, I still had to point the PC's DNS at my server, and remember to turn the server on. Well, I won't forget that again! → [Step 7](#7-adding-a-windows-11-client)
- **My password policy didn't apply.** The built-in Default Domain Policy was above mine in the link order, so it won. I moved mine to the top and it worked. → [Step 6](#6-group-policy)
- **I couldn't delete an OU I'd named wrong.** New OUs are protected from accidental deletion, so I had to turn that protection off first. → [Step 2](#2-organising-active-directory)
- **My server had no connection and no internet.** One cause was an IP address from the wrong network. Another was pointing the server's DNS at itself before the DNS role was installed. → [Step 1](#1-server-setup)

## How I built it

<img width="1600" height="960" alt="lab-network" src="https://github.com/user-attachments/assets/0693a0b0-307e-48d8-bf19-0fd764a11308" />


- **Domain:** `OleksiyKostenko.local`. I used my own name on purpose, so it's clear this lab is my work.
- **Users:** 300 made-up employees in 2 regions and 7 departments, added with PowerShell and CSV files. AI helped me with this because I'm still developing my PowerShell skills, and I won't claim to know something I don't.
- **Groups:** 14 security groups and 14 distribution groups
- **Tools:** VirtualBox, Windows Server 2025, Windows 11 Enterprise, PowerShell

Each step starts with a short summary. Click any ▸ line to see the details and screenshots.

### 1. Server setup

I set up a Windows Server 2025 virtual machine, named it **DC01**, gave it a fixed IP address and made it the domain controller for my new domain.

<details>
<summary><b>Details:</b> why it matters, what went wrong, and 3 screenshots</summary>

I also set the correct time zone, then installed Active Directory Domain Services and DNS.

**Why:** Computers rely on the domain controller for DNS and sign-in, so its address must not change. The clock matters too: Kerberos, which handles domain sign-ins, rejects them if the time is too far off.

**Result:** `ipconfig /all` showed the name DC01, DHCP turned off and the static address. After the server became a domain controller, I signed in as `OLEKSIYKOSTENKO\Administrator`.

| Problem | Cause | Fix |
|---|---|---|
| No connection | I set 10.10.10.1 with no gateway, but VirtualBox NAT uses 10.0.2.x | Used an address and gateway from the right network |
| No internet | I pointed DNS to 127.0.0.1 before the DNS role was installed | Used 1.1.1.1 for the time being |

The static IP, checked with `ipconfig /all`:

<img width="750" alt="ipconfig on DC01 showing the static IP address" src="https://github.com/user-attachments/assets/920431b7-f8ec-4e66-9f70-b8420df68911">

Installing Active Directory Domain Services and DNS:

<img width="750" alt="AD DS and DNS roles installed on DC01" src="https://github.com/user-attachments/assets/a97c8dc9-8e6e-43c4-8dbc-ceb76b2b386b">

Creating the new domain:

<img width="750" alt="Creating a new forest called OleksiyKostenko.local" src="https://github.com/user-attachments/assets/b43e978c-3bf1-4e81-a6ca-06d0831a83a3">

</details>

### 2. Organising Active Directory

I organised the domain like a company with staff in **Europe** and the **USA**. Each region has organisational units (OUs) for desktops, laptops and users, with the users split into 7 departments.

<details>
<summary><b>Details:</b> before and after, and the OU I couldn't delete</summary>

The departments are Sales, Support, Operations, Finance, Marketing, HR and IT. Servers have their own OU, also split by region.

**Why:** This lets me apply a policy to a whole region, one department, or only the laptops.

| Before | After |
|:---:|:---:|
| <img width="380" alt="Active Directory before, with only the built-in containers" src="https://github.com/user-attachments/assets/7d73d8df-f18d-4161-9c05-a2098c277d94"> | <img width="380" alt="Active Directory after, with Europe, USA and Servers OUs" src="https://github.com/user-attachments/assets/dab3da73-def5-4b08-9da9-a6320ed3ab1e"> |

| Problem | Cause | Fix |
|---|---|---|
| I created an OU with the wrong name and couldn't delete it | New OUs are protected from accidental deletion by default | Turned on View > Advanced Features, opened the OU's Properties > Object tab, unticked "Protect object from accidental deletion", then deleted it. I left the protection on for the OUs I wanted to keep. |

</details>

### 3. Security groups

I made a security group for each department in each region, 14 in total, like `GG-Finance-EU`. Permissions go to groups instead of single people, so when someone joins or changes team, I just change their groups.

<details>
<summary><b>Details:</b> the 14 security groups</summary>

They're global groups, and each one sits in its department's OU. The import script in step 5 adds each user to their department's group.

<img width="750" alt="Search for GG showing 14 security groups" src="https://github.com/user-attachments/assets/b1c28b14-c517-48ae-80a6-a3ede2989035">

</details>

### 4. Distribution groups

I also made 14 distribution groups, which are basically email lists, like `DL-Finance-USA` (DL for distribution list). They can't give anyone access to a folder, so I keep them separate from the security groups.

<details>
<summary><b>Details:</b> 2 screenshots</summary>

I haven't set up email yet, so these can't send anything. That's for a Microsoft 365 lab later.

<img width="750" alt="Search for DL showing 14 distribution groups" src="https://github.com/user-attachments/assets/ef5d371e-0654-4130-83db-39a6050ab846">

Both types side by side in Finance-USA:

<img width="750" alt="Finance-USA with its security group and distribution group" src="https://github.com/user-attachments/assets/73a1f90b-e9c6-4305-9c98-491aef74a952">

</details>

### 5. Adding 300 users with PowerShell

Typing 300 users by hand wasn't realistic, so I used AI to write two PowerShell scripts. One makes a list of made-up employees, and the other adds them to the right OU and group. I did a dry run first, to check everything before anything changed.

<details>
<summary><b>Details:</b> the results and 3 screenshots</summary>

The dry run uses `-WhatIf`, which shows what would happen without changing anything. Each new user gets a temporary password that must be changed at first sign-in.

**Why:** Doing it by hand would take hours, and I'd probably make typos. A script does it the same way every time.

**Result:** I checked the numbers with PowerShell: 180 users in Europe, 120 in the USA, and 24 members in `GG-Operations-USA`.

<img width="750" alt="User counts of 180, 120 and 24" src="https://github.com/user-attachments/assets/8cb80e33-d44b-4717-b632-ddcb93940752">

The dry run with `-WhatIf`:

<img width="750" alt="PowerShell dry run with WhatIf" src="https://github.com/user-attachments/assets/4e200c6d-da73-47ff-8d7f-9a3d8bf820f8">

Starting the real import (the temporary password is hidden):

<img width="750" alt="Starting the real import with the password hidden" src="https://github.com/user-attachments/assets/155668c3-9fe8-409d-946e-a8a292c6ff2b">

</details>

### 6. Group Policy

Group Policy lets me manage settings for users and computers from one place. I made 6 policies. Five are below, and the sixth, a mapped network drive, is in [step 8](#8-file-shares-and-permissions).

| Policy | What it does | Applies to |
|---|---|---|
| Password policy | At least 12 characters, with complexity | Whole domain |
| Screen lock | Locks the screen after 10 minutes | EU and USA users |
| Block Control Panel | Blocks Control Panel, except for IT | EU and USA users |
| Block USB storage | Blocks USB sticks and other removable drives | EU and USA users |
| Windows Update | Installs updates at 03:00 | Europe and USA |

<details>
<summary><b>Password policy:</b> and why mine didn't work at first</summary>

I created a GPO called "Password Policy" and linked it to the domain. It requires at least 12 characters with complexity on, remembers the last 15 passwords, and sets a maximum age of 90 days and a minimum age of 1 day.

**Why:** With Group Policy, password rules for domain users only work at the domain level, so I linked it there and not to an OU. The 1-day minimum stops someone changing their password several times in a row to get back to an old one.

| Problem | Cause | Fix |
|---|---|---|
| My policy didn't apply. Local Security Policy still showed 7 characters and 42 days | The built-in Default Domain Policy was at Link Order 1, so it won over mine | Moved my policy to Link Order 1, ran `gpupdate /force` and checked again |

After that, my policy was at Link Order 1, and Local Security Policy on the domain controller showed my values:

<img width="750" alt="Password Policy at Link Order 1 and its values in Local Security Policy" src="https://github.com/user-attachments/assets/d747e58f-c783-44dc-ac49-73c56d31627b">

The settings inside the policy:

<img width="750" alt="Password Policy settings in the Group Policy editor" src="https://github.com/user-attachments/assets/2e0365d9-9304-4dcf-a690-208490506c49">

**Something I learned:** Newer guidance from Microsoft and NIST moves away from forcing regular password changes. I used 90 days here to practise setting it up.

</details>

<details>
<summary><b>Screen lock:</b> locks the screen after 10 minutes</summary>

I made a GPO called "Screen Lockout" and linked it to Users-EU and Users-USA. After 10 minutes (600 seconds) without activity, the blank screen saver starts, and the user needs their password to get back in.

**Why:** If someone leaves their desk, nobody else can use their account. I linked it to the user OUs because it's meant for staff accounts. Admin accounts and servers should lock too, which I would set up separately.

I tested it and it works. A screenshot can't really prove the 10-minute wait, so here are the settings:

<img width="750" alt="Screen Lockout linked to both user OUs with four settings enabled" src="https://github.com/user-attachments/assets/be78ff6e-48fa-49c7-913c-eb284267d1d7">

</details>

<details>
<summary><b>Block Control Panel:</b> and how I left IT out</summary>

I made a GPO called "Control Panel Block", turned on "Prohibit access to Control Panel and PC settings", and linked it to Users-EU and Users-USA.

**Why:** Most users don't need to change system settings. When someone changes the wrong thing, like network settings, it turns into a helpdesk ticket.

**Leaving IT out:** IT staff still need Control Panel for their work. So on the Delegation tab, I set Deny on "Apply group policy" for the groups `GG-IT-EU` and `GG-IT-USA`.

You can see it tested with two users in [Proof it works](#proof-it-works). This is the policy setting:

<img width="750" alt="Control Panel block setting enabled" src="https://github.com/user-attachments/assets/eaaee1fe-be55-43b1-91eb-86be819072c3">

</details>

<details>
<summary><b>Block USB storage:</b> blocks USB sticks and removable drives</summary>

I made a GPO called "Block USB storage" and linked it to Users-EU and Users-USA. It uses "All Removable Storage classes: Deny all access" to block USB sticks and other removable drives.

**Why:** A USB stick is an easy way for company data to walk out of the building, and for malware to get in. It's a user policy, so it follows the person to whichever PC they sign in on.

This one is hard to show reliably in screenshots, so here's how it's set up:

<img width="750" alt="Removable storage settings in the Block USB storage policy" src="https://github.com/user-attachments/assets/ad495ddf-9489-4b59-a2d4-2dd908cad264">

</details>

<details>
<summary><b>Windows Update:</b> installs updates at night</summary>

I made a GPO called "Windows Updates" and linked it to Europe and USA, so it flows down to the desktop and laptop OUs. With "Configure Automatic Updates", updates download automatically and install at 03:00 (each PC's local time), outside working hours.

**Why:** PCs without updates are an easy target for attackers, and users shouldn't have to remember to update.

Updates run on a schedule, so this is hard to show reliably in screenshots. Here's how it's set up:

<img width="750" alt="Configure Automatic Updates enabled" src="https://github.com/user-attachments/assets/2e273ec5-da3a-4004-9788-8db189e282ab">

</details>

### 7. Adding a Windows 11 client

I created a Windows 11 Enterprise virtual machine to act as a user's work PC, and joined it to my domain. Until then I'd done everything from the server, so this was my chance to see what a real user sees.

<details>
<summary><b>Details:</b> why it didn't join at first, and how I fixed it</summary>

The PC kept saying the domain controller couldn't be contacted.

| Problem | Cause | Fix |
|---|---|---|
| Domain join failed: the domain controller couldn't be contacted | My two VMs were on separate NAT networks | Put both on the same VirtualBox NatNetwork |
| The same error again | The client wasn't using my server for DNS, and the server wasn't turned on | Set the client's DNS to 10.0.2.10 and turned DC01 on |

My domain is private, so public DNS doesn't know it. That's why the client has to ask my server.

| Before: can't contact the domain controller | After: finds the domain and asks for a login |
|:---:|:---:|
| <img width="380" alt="Error saying the domain controller could not be contacted" src="https://github.com/user-attachments/assets/a7c10bee-ff4c-4e44-a80a-8a51a5c4c37c"> | <img width="380" alt="Client DNS settings and the domain join sign-in prompt" src="https://github.com/user-attachments/assets/ef3ae416-c55b-42db-b91b-5bb341ff31cf"> |

The NatNetwork in VirtualBox:

<img width="750" alt="NatNetwork in VirtualBox" src="https://github.com/user-attachments/assets/9237b403-e0ad-47e0-bb8f-9b1237701cad">

</details>

#### A forgotten password

After the PC joined the domain, I went to sign in as `ana.fonseca` and realised I had forgotten the password. That gave me a chance to practise one of the most common helpdesk tasks: I reset it in Active Directory and ticked "User must change password at next logon". At the next sign-in, Windows asked for a new password before letting me in.

<details>
<summary><b>Screenshots:</b> the forced password change</summary>

| Password must be changed | Setting the new password |
|:---:|:---:|
| <img width="380" alt="Message that the password must be changed before signing in" src="https://github.com/user-attachments/assets/83ed15bb-131d-4086-9eaf-98344d8e64b1"> | <img width="380" alt="Setting a new password at sign-in" src="https://github.com/user-attachments/assets/a1c4868e-7c2d-4ceb-9700-cb8d92fa1a39"> |

In the second screenshot, "Sign in to: OLEKSIYKOSTENKO" also shows that the PC is part of my domain.

</details>

### 8. File shares and permissions

I made shared folders and controlled who can get in. There are two layers of permissions: **share permissions** apply over the network, and **NTFS permissions** (the Security tab) apply to the files and folders themselves. Over the network both count, and the stricter one wins.

<details>
<summary><b>A shared folder with one group blocked</b></summary>

I shared a folder called "SharedFolder". In the share permissions, Everyone and Domain Users can read, and I denied access to one group, `GG-HR-EU`.

**Why:** I kept this one simple: share a folder and control who gets in. Using a group means I don't set permissions one user at a time. Anyone added to `GG-HR-EU` gets blocked too, from their next sign-in.

**Result:** Here, Deny wins over Allow, so members of `GG-HR-EU` can't get in over the network.

<img width="750" alt="Share permissions on SharedFolder" src="https://github.com/user-attachments/assets/760ba28a-c81a-4f46-97e9-f7c3cb6077d0">

<img width="750" alt="GG-HR-EU denied in the share permissions" src="https://github.com/user-attachments/assets/b57b5d71-65c3-4a3d-bdaa-221786f90a9e">

</details>

<details>
<summary><b>Mapping the S: drive</b> with Group Policy</summary>

I made a GPO called "Mapped Drives" and set up the drive under User Configuration > Preferences > Windows Settings > Drive Maps:

| Setting | Value | What it means |
|---|---|---|
| Action | Update | Creates the drive if it's missing, or updates it if it's already there |
| Location | `\\DC01\SharedFolder` | The shared folder on my server |
| Drive letter | S: | The letter set for the drive |
| Label | Shared | The name I gave the drive |

**Why:** Users shouldn't have to type the path or set up the drive themselves. And if the path ever changes, I only update it in one place.

**Good to know:** The GPO only adds the drive. It doesn't give anyone access. The permissions still decide who gets in, as the [drive test](#proof-it-works) shows.

<img width="750" alt="Drive Maps settings in the Mapped Drives policy" src="https://github.com/user-attachments/assets/2dac1a8a-d032-46ce-85e4-402d7496be5d">

</details>

<details>
<summary><b>Folder inheritance:</b> giving one project folder to one team</summary>

I created a folder called "Common" and shared it as `\\DC01\Common`, with subfolders like "Events" and "Project".

**Why:** I imagined one team working on a project. By default, a new folder inherits its permissions from the folder above it, and later changes to that parent flow down too. I didn't want Everyone's access to "Common" to carry over to "Project".

**What I did:** On "Project", I disabled inheritance, removed Everyone and gave the Operations team, `GG-Operations-USA`, Full control.

**Result:** The "Inherited from" column showed "None", and SYSTEM and the administrators kept their access.

| Inheritance disabled | Final permissions on "Project" |
|:---:|:---:|
| <img width="380" alt="Project with inheritance disabled" src="https://github.com/user-attachments/assets/44f24d83-9e29-4f01-8bd3-2bc2d64f3652"> | <img width="380" alt="Final permissions on Project" src="https://github.com/user-attachments/assets/a2082521-ac57-4453-8846-8cde027a2e89"> |

Sharing "Common":

<img width="750" alt="The Common folder is shared" src="https://github.com/user-attachments/assets/60ea1ae3-bd55-4fdc-a966-36434f908597">

</details>

<details>
<summary><b>Explicit deny:</b> stopping one group from reading a folder</summary>

I made a shared folder called "Project2" and used an explicit deny on one group, `GG-Sales-USA`.

**Why:** I wanted to learn how to stop one group from reading a folder while Everyone is still allowed on it.

| 1. Share permissions: Everyone has Full Control | 2. First, `GG-Sales-USA` can read |
|:---:|:---:|
| <img width="380" alt="Share permissions on Project2" src="https://github.com/user-attachments/assets/a60e3e4d-4767-4caa-8916-544e15e545ac"> | <img width="380" alt="GG-Sales-USA with read access" src="https://github.com/user-attachments/assets/1522c189-d17e-4fcd-a87a-945252f47922"> |
| **3. Then I ticked Deny on Read** | **4. Everyone is still allowed on the Security tab** |
| <img width="380" alt="GG-Sales-USA with Deny on Read" src="https://github.com/user-attachments/assets/3ba35719-8fb0-4f38-bf47-c2e67af3791b"> | <img width="380" alt="Everyone is still allowed on Project2" src="https://github.com/user-attachments/assets/eaa82a57-ba62-4f8a-802b-d5a35db59d0e"> |

People in `GG-Sales-USA` are also part of Everyone, which can read and write. But on this folder, **the Deny on Read wins over the Allow**.

**What I learned:** There are two kinds of deny. An **explicit deny** is when I tick Deny myself, like here. An **implicit deny** is when nothing gives you access, so you're refused by default.

</details>

> **Honestly:** inheritance and shares were the hardest part of this lab for me. I went through guides and documentation to get them right. I understand them now, and I'm still practising.

## What's next

**My plan:** After Network+, I'll focus on Security+ and CCNA, improve my PowerShell skills, and build a full network with security features (in progress).

This lab is a work in progress too. In it, I want to:

- Follow least privilege: give folder access only to the groups that need it, with Modify instead of Full control
- Add an account lockout policy and practise unlocking accounts
- Practise the full new starter, mover and leaver process
- Find a reliable way to show the USB and Windows Update policies working on the client
- Move the file shares to a separate file server instead of the domain controller
- Connect the lab to Microsoft Entra ID and try remote access over a VPN
- Write PowerShell scripts like these myself

If you took time from your day to read my CV and open this project, thank you. Getting into IT hasn't been easy, and some things take me a few tries, but I don't give up. Building a career in this field is my goal, and I'm serious about it. If you give me a chance, even a small one, I'll show up, work relentlessly, dedicate myself to improving every day, without excuses, and do everything I can to exceed your expectations. Thanks again, and enjoy exploring my lab!
**Thanks for reading!** I'm still learning, and I'll keep adding to this lab.
