---
title: 3CX Defender for Endpoint Hunting Queries
author: LJ
date: 2019-08-08 11:33:00 +0800
categories: [Blogging, Demo]
tags: [typography]
pin: true
math: true
mermaid: true
image:
  path: /commons/devices-mockup.png
  lqip: data:image/webp;base64,UklGRpoAAABXRUJQVlA4WAoAAAAQAAAADwAABwAAQUxQSDIAAAARL0AmbZurmr57yyIiqE8oiG0bejIYEQTgqiDA9vqnsUSI6H+oAERp2HZ65qP/VIAWAFZQOCBCAAAA8AEAnQEqEAAIAAVAfCWkAALp8sF8rgRgAP7o9FDvMCkMde9PK7euH5M1m6VWoDXf2FkP3BqV0ZYbO6NA/VFIAAAA
  alt: Responsive rendering of Chirpy theme on multiple devices.
---
# Introduction

3CX is a software development company with a large outreach, over 600,000 companies worldwide and around 12 million daily users. 3CX provide desktop applications which allows users to control voice calls from various different platforms.

On 29th of March 2023, CrowdStrike observed malicious activity originating from signed files in their clients' environments, the files were 3CX desktop application. Crowstrike claim that the MacOS (version 18.11.1213) and Windows (versions 18.12.407 18.12.416) platforms have currently been identified as infected.

The word signed here is particularly important. Without going on too much of a tangent, file signing is technique software vendors utilise to prove the authenticity of files. If a file is signed by a particular vendor, it *should* be safe. Malicious actors often try to imitate genuine software to evade detection but can't provide the signature of authenticity to go with it. They become unstuck here, as any keen eyed security analyst will notice the absence of such a signature.

So, why is a signed file exhibiting signs of malicious activity?

Pierre Jourdan, CISO of 3CX states in a blog post that a library bundled into their Windows Electron App has caused the issue but it is unclear at this stage as to what this library is, and as to how exactly this caused their desktop applications to become infected. We just know that this is how the attackers were able to embed malware into the genuine software.

Does it really matter anyway? Yeah it does, it matters hugely but we won't know until 3CX say so lets focus on what we do know. The attack vector:

1. The attack starts when the MSI file is downloaded from the 3CX website or an update is pushed to an already installed application.
2. MSI installer extracts two malicious files ffmpeg.dll and d3dcompiler.dll
3. Sophos state that ffmpeg is sideloaded and used to extract and decrypt a payload from d3dcomiler.
4. The decrypted shell code is executed and in-turn downloads icon files from a GitHub repository (I've left this out of the detection as the repo has long been taken down).
5. Base64 encoded strings attached to the end of these icon files are used to download a final DLL file which is used to steal information from the infected device.

---

# Detection

Onto the juicy bit. I work with the MS security stack so this detection logic is going to written in KQL. We are primarily going to be using the Device tables of Microsoft Defender for Endpoint here, so the device needs to have a Defender for Endpoint agent installed in order to see this activity.

In my mind, the first stage in detecting this is to detect if any of the computers in the organisation are running 3CX desktop applications:

```sql
DeviceProcessEvents
| where FileName contains "3CX" or InitiatingProcessFileName contains "3CX"
//Software version specific tailoring - Comment out lines 5 and 7 to search for all versions of the software
//Comment out the line below  if trying to discover devices on Windows - Line below is for Mac
| where ProcessVersionInfoProductVersion contains "18.11.1213"
	//Comment out the line below if trying to discover devices on Mac - Line below iqf for Windows versions affected
| where ProcessVersionInfoProductVersion has_any ("18.12.407","18.12.416")
| distinct DeviceName, FileName, SHA256
```

With the KQL above you can search for any mention of 3CX in file names by commenting out lines detailed in the query you can search for specific windows versions, specific mac versions or (as I would recommend) do a flat search for 3CX despite what version it's on.

This first step allows you to identify any shadow IT you have by looking for where 3CX processes are running on your devices. If you don't have any results, good for you you can go relax.

If you have identified 3CX desktop apps running on your endpoints, uninstall them at your discretion. The infected 3CX versions side load two malicious dll files - **ffmpeg.dll** and **d3dcompiler.dll.** Lets detect them.

```
DeviceFileEvents
| where ActionType == "FileCreated" or ActionType == "FileModified"
| where InitiatingProcessFileName contains "msi"
| where FileName has_any ("ffmpeg.dll","d3dcompiler.dll")
```

*Note: I think Jabra use a dll called ffmpeg.dll so don't panic if you see this it might not be malicious. Check the hashes below:*

**Malicious ffmpeg.dll -** c485674ee63ec8d4e8fde9800788175a8b02d3f9416d0e763360fff7f8eb4e02

7986bbaee8940da11ce089383521ab420c443ab7b15ed42aed91fd31ce833896

**Malicious d3dcompiler.dll -**

11be1803e2e307b647a8a7e02d128335c448ff741bf06bf52b332e0bbf423b03

Still with me?

Okay, lets try and identify some malicious behaviours of the application. We know that it beacons out to C2 instances. Crowdstrike have provided us with some domains that the infected software is known to be connecting to. Going forward these will be pretty futile (as the domains will likely get shut down, or the attackers will switch to avoid detection) but this could be handy if you want to detect whether any data has already been exfiltrated out of your network.

```
let maliciousDomains = dynamic(["akamaicontainer.com", "akamaitechcloudservices.com", "azuredeploystore.com","azureonlinecloud.com","azureonlinestorage.com","dunamistrd.com","glcloudservice.com","journalide.org", "msedgepackageinfo.com","msstorageazure.com","msstorageboxes.com","officeaddons.com","officestoragebox.com","pbxcloudeservices.com","pbxphonenetwork.com","pbxsources.com","qwepoi123098.com","sbmsa.wiki","sourceslabs.com", "visualstudiofactory.com", "zacharryblogs.com"]);
DeviceNetworkEvents
| where RemoteUrl has_any (maliciousDomains)
| where InitiatingProcessFileName contains "3CX"
```

Finally, here is one for detection going forward let's continue to monitor whether the 3CX application is spawning any suspicious processes:

```
let susTools = dynamic(["py","bsh","dll","ps1","psh","cmd","bat", "bash"]);
DeviceProcessEvents
| where InitiatingProcessFileName contains "3CX"
| where ProcessCommandLine has_any (susTools) oThey need to move to Arcr FileName has_any (susTools)
```

I hope this can help some people out there. Remember, Keep Calm and KQL.

## Resources

Some resources I used to help me write this article:

[Bleeping Computer](https://www.bleepingcomputer.com/news/security/hackers-compromise-3cx-desktop-app-in-a-supply-chain-attack/)

[CrowdStrike](https://www.crowdstrike.com/blog/crowdstrike-detects-and-prevents-active-intrusion-campaign-targeting-3cxdesktopapp-customers/)

[TrendMicro](https://www.trendmicro.com/en_us/research/23/c/information-on-attacks-involving-3cx-desktop-app.html)