# 3.Platform-as-a-Service-Deploying-and-monitoring-App-services-
Provisioned a Windows App Service in Azure, then used Kudu's debug console to navigate the site's actual file system (wwwroot, deployments, locks), edit the default hostingstart.html directly, and confirm the change reflected on the live public URL.

# Azure App Service Deployment Lab (Provisioning a Windows Web App and Editing Live Content via Kudu)

`Microsoft Azure` `App Service` `Kudu` `PowerShell` `ASP.NET` `Cloud Deployment`

## Overview
This lab was hands on practice standing up a web app from scratch in Microsoft Azure, then going past the portal UI to interact with the underlying file system through Kudu's advanced tools console. The goal was to understand what actually happens behind an App Service deployment, not just click Create and assume it works, but confirm the site directory structure, find the default landing page, and edit it directly to prove the app was really live and that I understood where its files lived on disk.

## Objective
Provision a new Azure Web App with the correct runtime and region settings, then use the Kudu debug console to navigate the site's file system, locate the default content file, modify it, and verify the change by loading the live public URL.

## Environment
- **Cloud provider:** Microsoft Azure, subscription "Azure subscription 1"
- **Resource group:** web site hp (newly created for this lab)
- **App name:** mywebsite2024
- **Public hostname:** mywebsite2024 a0geaadqa4gbhzae.canadacentral-01.azurewebsites.net
- **Runtime stack:** ASP.NET V4.8
- **Operating system:** Windows
- **Region:** Canada Central
- **Access method:** Kudu / SCM debug console (PowerShell shell) at the site's .scm.canadacentral-01.azurewebsites.net/DebugConsole endpoint

## Tools I Used

| Tool | What It Does | Why I Used It |
|------|--------------|----------------|
| **Azure Portal** | Web console for provisioning cloud resources | Used the Create Web App wizard to configure subscription, resource group, name, runtime, OS, and region |
| **Kudu (SCM site)** | Diagnostic and management console that runs alongside every App Service | Gave me direct access to the site's actual file system rather than only the public facing app |
| **Kudu Debug Console (PowerShell)** | In browser shell running against the app's file system | Used `cd` to move through the site directory tree and confirm folder structure |
| **Kudu file browser** | Web based file explorer inside Kudu | Used to locate, open, and edit `hostingstart.html` directly |

## What I Did

### Provisioning the Web App
1. Opened the Create Web App blade in the Azure Portal and set the subscription to "Azure subscription 1."
2. Created a new resource group named `web site hp` rather than reusing an existing one, to keep this lab isolated from other resources.
3. Named the app `mywebsite2024`, which Azure automatically paired with a unique default hostname (`mywebsite2024 a0geaadqa4gbhzae.canadacentral-01.azurewebsites.net`) with the "secure unique default hostname" option left on.
4. Set the publish method to Code (not Container), the runtime stack to ASP.NET V4.8, the operating system to Windows, and the region to Canada Central.
5. Reviewed and created the app rather than continuing further into optional settings like a database, since the goal of this lab was deployment fundamentals, not a full application stack.

### Exploring the File System Through Kudu
1. Instead of only checking that the app loaded, I opened the app's Kudu debug console at `/DebugConsole/?shell=powershell` on the `.scm.` subdomain to see the file system Azure was actually running the site from.
2. From the default `C:\home` prompt, moved into `C:\home\site` and confirmed three folders existed: `deployments`, `locks`, and `wwwroot`. This matched what I expected: `wwwroot` is the actual web root that gets served publicly, while `deployments` and `locks` are Azure's own deployment bookkeeping and not part of the served content.
3. Moved into `C:\home\site\wwwroot` and found a single file, `hostingstart.html`, which is the default placeholder page Azure ships with a fresh Windows/ASP.NET App Service before any real code is deployed.

### Editing the Default Page and Verifying the Change
1. Used Kudu's file browser edit function (the pencil icon next to `hostingstart.html`) to open the file directly in the browser rather than redeploying through source control, since the point of this step was understanding the raw file system, not setting up a CI/CD pipeline.
2. Replaced the default placeholder content with a short custom line of text so the change would be obvious and easy to verify.
3. Saved the file, which updated its modified timestamp and confirmed the write went through.
4. Opened a new tab and loaded the public hostname directly. The live page rendered the custom text I had just written, confirming that the file I edited inside Kudu really was the same content being served to the public internet, not a cached or separate copy.

## What's in This Repo

```
azure-webapp-deployment-lab/
├── README.md                          # This file
└── screenshots/
    ├── 01-create-web-app-config.png       # Portal configuration for the new App Service
    ├── 02-kudu-site-directory.png         # Kudu console showing deployments, locks, wwwroot
    ├── 03-kudu-wwwroot-file-listing.png   # wwwroot contents before and after edit
    └── 04-live-site-updated-content.png   # Public URL rendering the edited page
```

## Skills I Picked Up
- **Understanding App Service's actual file layout,** seeing firsthand that `wwwroot` is the real served content while `deployments` and `locks` are internal Azure metadata folders, rather than just trusting that abstraction from documentation.
- **Using Kudu as a diagnostic tool, not just a deployment target,** getting comfortable dropping into the PowerShell based debug console to move through the file system the same way I would on a local machine.
- **Connecting portal configuration to what's running underneath,** tracing the runtime stack and OS choices made in the Create Web App wizard through to the actual folder structure and file the app served.
- **Verifying a change end to end,** editing a file at the source and then independently confirming the live public site reflected that exact change, instead of assuming a save in the console meant the job was done.

## How This Applies in the Real World
Most real world troubleshooting on a managed platform like Azure App Service happens exactly here, in the gap between what the portal shows you and what the underlying file system and process are actually doing. Being able to open a debug console, confirm what folder is really being served, and check timestamps on files is a basic but genuinely useful skill when a deployment isn't behaving the way the portal says it should. It's also a reminder that "the cloud" is still a real file system and a real process underneath the abstraction, and knowing how to get to that layer when something needs verifying is worth more than trusting the dashboard by itself.

## Where I'm Coming From
I'm making the jump into cybersecurity from a background in healthcare. Cloud platforms like Azure are a big part of that shift for me, since so much of the infrastructure I'll eventually be securing or assessing lives here rather than on premises. Labs like this one are how I'm building the baseline comfort with cloud consoles and file systems that I'm missing on paper right now compared to my hands on healthcare experience.

## What I Want to Learn Next
- Deploying an actual application through source control (GitHub Actions or Azure DevOps) instead of editing files directly through Kudu
- Reviewing App Service access logs and diagnostic logs to understand what normal versus suspicious activity looks like on a live app
- Locking down the app with authentication, custom domains, and TLS rather than leaving it on the default public hostname
- Comparing this Windows/ASP.NET stack against a Linux based App Service to see how the file layout and console access differ

## Limitations & What I'd Do Differently in Production
- **Editing files directly through Kudu is fine for a lab, but it isn't how real deployments should work.** A production app should be deployed through source control and a proper CI/CD pipeline, not hand edited in a live file browser.
- **No access controls or custom domain were configured.** The app is sitting on its default public hostname with no additional authentication layer, which would need to change before this represented anything close to a production posture.
- **No monitoring or logging was reviewed.** This lab focused on provisioning and file system verification only; a real deployment would need diagnostic logging and alerting configured from the start.
- **Single environment, no staging slot.** In production I would use a staging deployment slot to test changes before swapping them into production rather than editing the live site directly.

## References
- [Azure App Service Documentation](https://learn.microsoft.com/en-us/azure/app-service/)
- [Kudu Documentation](https://github.com/projectkudu/kudu/wiki)
- [Azure App Service on Windows vs Linux](https://learn.microsoft.com/en-us/azure/app-service/overview)
- CompTIA Security+ (SY0 701) Exam Objectives
