---
# required metadata

title: Remotely assist users that are authenticated by your organization. 
description: With the Remote Help app, provide remote assistance to authenticated users who also run the Remote Help app.
keywords:
author: Smritib17
ms.author: smbhardwaj
manager: dougeby
ms.date: 12/08/2022
ms.topic: how-to
ms.service: microsoft-intune
ms.subservice: remote-actions
ms.localizationpriority: high
ms.technology:
ms.assetid: 

# optional metadata

#ROBOTS:
#audience:
ms.reviewer: nehashah
ms.suite: ems
search.appverid: MET150
#ms.tgt_pltfrm:
ms.custom: intune-azure
ms.collection:
- tier1
- M365-identity-device-management
- highpri
- highseo
---
 
# Use Remote Help with Microsoft Intune

[!INCLUDE [intune-add-on-note](../includes/intune-add-on-note.md)]

Remote Help is a cloud-based solution for secure help desk connections with role-based access controls. With the connection, your support staff can remote connect to the user's device. During the session, they can view the device's display and if permitted by the device user, take full control. Full control enables a helper to directly make configurations or take actions on the device.

This feature applies to:  
- Windows 10/11

In this article, we'll refer to the users who provide help as *helpers*, and users that receive help as *sharers* as they share their session with the helper. Both helpers and sharers sign in to your organization to use the app. It's through your Azure Active Directory (Azure AD) that the proper trusts are established for the Remote Help sessions.

Remote Help uses Intune role-based access controls (RBAC) to set the level of access a helper is allowed. Through RBAC, you determine which users can provide help and the level of help they can provide.

The Remote Help app is available from Microsoft to install on both devices enrolled with Intune and devices that aren’t enrolled. The app can also be deployed through Intune to your managed devices.

## Remote Help capabilities and requirements

The Remote Help app supports the following capabilities:

- **Enable Remote Help for your tenant** – By default, Intune tenants aren't enabled for Remote Help. If you choose to turn on Remote Help, its use is enabled tenant-wide. Remote Help must be enabled before users can be authenticated through your tenant when using Remote Help.

- **Use Remote Help with unenrolled devices** – Disabled by default, you can choose to allow help to devices that aren't enrolled with Intune.

- **Requires Organization login** - To use Remote Help, both the helper and the sharer must sign in with an Azure Active Directory (Azure AD) account from your organization. You can’t use Remote Help to assist users who aren’t members of your organization.

- **Compliance Warnings** - Before connecting to a user's device, a helper will see a non-compliance warning about that device if it’s not compliant with its assigned policies. This warning doesn’t block access but provides transparency about the risk of using sensitive data like administrative credentials during the session. 
    
    - Helpers who have access to device views in Intune will see a link in the warning to the device properties page in the Microsoft Intune admin center. This allows a helper to learn more about why the device is not compliant.
 
    - Unenrolled devices are always reported as non-compliant. This is because until a device enrolls with Intune it can’t receive policies from Intune and as such is unable to establish its compliance status.

- **Role-based access control** – Admins can set RBAC rules that determine the scope of a helper’s access, like:
  - The users who can help others and the range of actions they can do while providing help, like who can run elevated privileges while helping.
  - The users that can only view a device, and which can request full control of the session while assisting others.

- **Elevation of privilege** - When needed, a helper with the correct RBAC permissions can interact with the  UAC prompt on the sharer's machine to enter credentials. For example, your Help Desk employees might enter their administrative credentials to complete an action on the sharer’s device that requires administrative permissions.

- **Monitor active Remote Help sessions, and view details about past sessions** – In the Microsoft Intune admin center you can view reports that include details about who helped who, on what device, and for how long. You’ll also find details about active sessions.

  For unenrolled devices, auditing and reporting about the Remote Help sessions is limited.

## Prerequisites

- [Intune subscription](../fundamentals/licenses.md)
- [Remote Help add on license or an Intune Suite license](intune-add-ons.md#available-add-ons) for all IT support workers (helpers) and users (sharers)
- Windows 10/11
- The Remote Help app for Windows. See [Install and update Remote Help](#install-and-update-remote-help)

> [!NOTE]
> Remote Help has the following limitations:  
>
> - Remote Help is not supported on GCC, GCC High or DoD Tenants.
> - You cannot establish a Remote Help session from one tenant to a different tenant.
> - May not be available in all markets or localizations.

### Network considerations

Remote Help communicates over port 443 (https) and connects to the Remote Assistance Service at `https://remoteassistance.support.services.microsoft.com` by using the Remote Desktop Protocol (RDP). The traffic is encrypted with TLS 1.2.

Both the helper and sharer must be able to reach these endpoints over port 443:

| Domain/Name                       | Description                                           |
|-----------------------------------|-------------------------------------------------------|
|\*.aria.microsoft.com             | Used for accessibility features within the app|
|\*.events.data.microsoft.com      | Microsoft Telemetry Service |
|\*.monitor.azure.com              | Required for telemetry and remote service initialization|
|\*.support.services.microsoft.com | Primary endpoint used for the Remote Help application|
|\*.trouter.skype.com              | Used for Azure Communication Service for chat and connection between parties|
|\*.aadcdn.msauth.net              | Required for logging in to the application (AAD)|
|\*.aadcdn.msftauth.net            | Required for logging in to the application (AAD)|
|\*.edge.skype.com                 | Used for Azure Communication Service for chat and connection between parties|
|\*.graph.microsoft.com            | Used for connecting to the Microsoft Graph service|
|\*.login.microsoftonline.com      | Required for Microsoft login service. Might not be available in preview in all markets or for all localizations|
|\*.remoteassistanceprodacs.communication.azure.com|Used for Azure Communication Service for chat and connection between parties|
|[Allow list for Microsoft Edge endpoints](/deployedge/microsoft-edge-security-endpoints) |The app uses Edge WebView2 browser control. This article identifies the domain URLs that you need to add to the allow list to ensure communications through firewalls and other security mechanisms|

### Data and privacy

Microsoft logs a small amount of session data to monitor the health of the Remote Help system. This data includes the following information:

- Start and end time of the session. This information is stored on Microsoft servers for 30 days.
- Who helped whom and on what device. This information is stored on Microsoft servers for 30 days.
- Errors arising from Remote Help itself, such as unexpected disconnections. This information is stored on the sharer's device in the event viewer.
- Features used inside the app such as view only and elevation. This information is stored on Microsoft servers for 30 days.

Remote Help logs session details to the Windows Event Logs on the device of both the helper and sharer. Microsoft can't access a session or view any actions or keystrokes that occur in the session.

The helper and sharer both see the following information about the other individual, taken from their organizational profiles:

- Their organization profile picture (if present)
- Company name
- Verified domain
- First and Last name
- Job title

Microsoft does not store any data about either the sharer or the helper for longer than 30 days.

## Install and update Remote Help

Remote Help is available as download from Microsoft and must be installed on each device before that device can be used to participate in a Remote Help session. By default, users will be opted into automatic updates and Remote Help will update itself when an update is available.

For users that opted out of automatic updates, when an update to Remote Help is required, users are prompted to install that version of Remote Help when the app opens. You can use the same process to download and install Remote Help to install an updated version. There's no need to uninstall the previous version before installing the updated version.

- Intune admins can download and deploy the app to enrolled devices. For more information about app deployments, see [Install apps on Windows devices](../apps/apps-windows-10-app-deploy.md#install-apps-on-windows-10-devices).
- Individual users who have permissions to install apps on their devices can also download and install Remote Help.

> [!NOTE]
> - In May 2022, existing users of Remote Help will see a recommended upgrade screen when they open the Remote Help app. Users will be able to continue using Remote Help without upgrading.
> - On May 23, 2022, existing users of Remote Help will see a mandatory upgrade screen when they open the Remote Help app. They will not be able to proceed until they upgrade to the latest version of Remote Help.
> - Remote Help will now require Microsoft Edge WebView2 Runtime. During the Remote Help installation process, if Microsoft Edge WebView2 Runtime is not installed on the device, then Remote Help installation will install it. When uninstalling Remote Help, Microsoft Edge WebView2 Runtime will not be uninstalled.

### Download Remote Help

Download the latest version of Remote Help direct from Microsoft at [aka.ms/downloadremotehelp](https://aka.ms/downloadremotehelp).

The most recent version of Remote Help is **4.1.1.0**

### Deploy Remote Help as a Win32 app

To deploy Remote Help with Intune, you can add the app as a Windows win32 app, and define a detection rule to identify devices that don’t have the most current version of Remote Help installed.  Before you can add Remote Help as a Win32 app, you must repackage *remotehelpinstaller.exe* as a *.intunewin* file, which is a Win32 app file you can deploy with Intune. For information on how to repackage a file as a Wind32 app, see [Prepare the Win32 app content for upload](../apps/apps-win32-prepare.md).

After you repackage Remote Help as a *.intunewin* file, use the procedures in [Add a Win32 app](../apps/apps-win32-add.md ) with the following details to upload and deploy Remote Help. In the following, the repackaged remotehelpinstaller.exe file is named *remotehelp.intunewin*.

1. On the App information page, select **Select app package file**, and locate the *remotehelp.intunewin* file you’ve previously prepared, and then select **OK**.

   Add a *Publisher* and then select **Next**. The other details on the App Information page are optional.

2. On the Program page, configure the following options:

   - For *Install command line*, specify **remotehelpinstaller.exe /quiet acceptTerms=1**
   - For *Uninstall command line*, specify **remotehelpinstaller.exe /uninstall /quiet acceptTerms=1**

To opt out of automatic updates, specify enableAutoUpdates=0 as part of the install command **remotehelpinstaller.exe /quiet acceptTerms=1 enableAutoUpdates=0**

   > [!IMPORTANT]
   > The command line options *acceptTerms* and *enableAutoUpdates* are always case sensitive.

   You can leave the rest of the options at their default values and select **Next** to continue.

3. On the Requirements page, configure the following options to meet your environment, and then select **Next**:

   - *Operating system architecture*
   - *Minimum operating system*

4. On the Detection rules page, for *Rules format*, select **Manually configure detection rules**, and then select **Add** to open the *Detection rule* pane. Configure the following options:

   - For *Rule type*, select **File**
   - For *Path*, specify **C:\Program Files\Remote Help**
   - For *File or folder*, specify **RemoteHelp.exe**
   - For *Detection method*, select **String (version)**
   - For *Operator*, select **Greater than or equal to**
   - For *Value*, specify the [version of Remote Help](#download-remote-help) you are deploying. For example, **10.0.22467.1000**
   - Leave *Associated with a 32-bit app on 64-bit clients* set to **No**

5. Proceed to the Assignments page, and then select an applicable device group or groups that should install the Remote Help app.

6. Complete creation of the Windows app to have Intune deploy and install Remote Help on applicable devices.

## Configure Remote Help for your tenant

To configure your tenant to support Remote Help, review and complete the following tasks.

### Task 1 – Enable Remote Help

1. Sign in to [Microsoft Intune admin center](https://go.microsoft.com/fwlink/?linkid=2109431) and go to **Tenant administration** > **Remote Help**.

2. On the **Settings** tab:
   1. Set **Enable Remote Help** to **Enabled** to allow the use of remote help. By default, this setting is *Disabled*.
   2. Set **Allow Remote Help to unenrolled devices** to **Enabled** if you want to allow this option. By default, this setting is *Disabled*.
   3. Set **Disable chat** to **Yes** to remove the chat functionality in the Remote Help app. By default, chat is enabled and this setting is set to **No**.  

3. Select **Save**.

> [!NOTE] 
> When you purchase licenses or start a trial, it could take a while to become active (anywhere between 30 minutes to 8 hours). 
> When you try to create a Remote Help session you may continue to see messages indicating that Remote Help isn't enabled for the tenant even if you enabled Remote Help in the tenant after activation. 
### Task 2 – Configure permissions for Remote Help

The following Intune RBAC permissions manage the use of the Remote Help app. Set each to *Yes* to grant the permission:

- Category: **Remote Help app**
- Permissions:
  - **Take full control** – Yes/No
  - **Elevation** – Yes/No
  - **View screen** – Yes/No

By default, the built-in **Help Desk Operator** role sets all of these permissions to **Yes**. You can use the built-in role or create custom roles to grant only the remote tasks and Remote Help app permissions that you want different groups of users to have. For more information on using Intune RBAC, see [Role-based access control](../fundamentals/role-based-access-control.md).

> [!IMPORTANT]  
> During a Remote Help session, when a helper has the *Elevation* permission, the helper will not automatically be able to view the sharer's UAC prompt. Instead, for a non-admin sharer, a button will appear on the helper's Remote Help toolbar that will allow them to request access to the UAC prompt on the sharer's device. Once requested and accepted, the helper will be able to perform elevated actions on the sharer's device. 
> When the sharer ends the Remote Help session, they will be shown a dialog box that will warn them that if they continue, they will be logged off.
> If the helper ends the session, the sharer will not be logged off.

### Task 3 – Assign user to roles

After creating the custom roles that you'll use to provide different users with Remote Help permissions, assign users to those roles.

1. Sign into [Microsoft Intune admin center](https://go.microsoft.com/fwlink/?linkid=2109431) and go to **Tenant administration**  > **Roles** >  and select a role that grants Remote Help app permissions.

2. Select **Assignments**  > **Assign** to open **Add Role Assignment**.

3. On the *Basics* page, enter an **Assignment name** and optional **Assignment description**, and then choose **Next**.

4. On the **Admin Groups** page, select the group that contains the user you want to give the permissions to. Choose **Next**.

5. On the **Scope (Groups)** page, choose a group containing the users/devices that the member above will be allowed to manage. You also can choose all users or all devices. Choose **Next** to continue.

   >[!IMPORTANT]
   >If a sharer’s device isn’t in the scope of a helper, that helper cannot provide assistance.

6. On the **Review + Create** page, when you're done, choose **Create**. The new assignment is displayed in the list of assignments.

## How to use Remote Help

The use of Remote Help depends on whether you're requesting help or providing help.

### Request help

To request help, you must reach out to your support staff to request assistance. You can reach out through a call, chat, email, and so on, and you'll be the sharer during the session. Be prepared to enter a security code that you'll get from the individual who is assisting you. You'll enter the code in your Remote Help instance to establish a connection to the helper's instance of Remote Help.

As a sharer, when you’ve requested help and both you and the helper are ready to start:

1. Start the Remote Help app on the device and sign in to authenticate to your organization. The device might not need to be enrolled to Intune if your administrator allows you to get help on unenrolled devices.

2. After signing into the app, get the security code from the individual assisting you and enter that code below *Get Help*, and then select **Submit**.

3. After submitting the security code from the helper, the helper will see information about you including your full name, job title, company, profile picture, and verified domain. As the sharer, your app displays similar information about the helper.

   At this time, the helper might request a session with full control of your device or choose only screen sharing. If they request full control, you can select the option to *Allow full control* or choose to *Decline the request*. Full control must be established before the help session starts. If full control is required after the sessions starts, both users must disconnect and restart the Remote Help session.

4. After establishing the type of session (full control or screen sharing), the session is established, and the helper can now assist in resolving any issues on the device.

   During assistance, helpers that have the *Elevation* permission can enter local admin permissions on your shared device. *Elevation* allows the helper to run executable programs or take similar actions when you lack sufficient permissions.

5. After the issues are resolved, or at any time during the session, both the sharer and helper can end the session. To end the session, select **Leave** in the upper right corner of the Remote Help app. Upon the end of a session, the sharer is automatically signed out of their device as a security precaution to ensure all connections between the devices close.

### Provide help

As a helper, after receiving a request from a user who wants assistance by using the Remote Help app:

1. Start the Remote Help app on your device. You can start the app from within the Microsoft Intune admin center:

   1. Sign into [Microsoft Intune admin center](https://go.microsoft.com/fwlink/?linkid=2109431) and go to **Devices** > **All devices** and select the device on which assistance is needed.

   2. From the remote actions bar across the top of the device view, select **New Remote Help session**. This action opens the Remote Help app.

   Alternately, or for devices not enrolled in Intune, locate the Remote Help app on your device and manually start it. After Remote Helps opens, you'll need to sign in to authenticate to your organization.

2. When Remote Help opens you must sign in to authenticate to your organization. After signing into the app, under *Give help* select **Get a security code**. Remote Help generates a security code that you’ll share with the person who has requested assistance. They'll enter this code in their instance of Remote Help to establish a connection to your Remote Help instance.

3. After the sharer enters the security code, as the helper you'll see information about the sharer, including their full name, job title, company, profile picture, and verified domain. The sharer will see similar information about you.

   At this time, you can request a session with full control of the sharer's device or choose only screen sharing. If you request full control, the sharer can choose to *Allow full control* or to *Decline the request*. Full control must be established before the help session starts. If full control is required after the sessions starts, both users must disconnect and restart the Remote Help session.

4. After establishing that the session will use a shared display or full control, Remote Help will display a **Compliance Warning* if the sharer's device fails to meet the conditions of its assigned compliance policies.

   During assistance, helpers that have the *Elevation* permission can enter local admin permissions on your shared device. *Elevation* allows the helper to run executable programs or take similar actions when you lack sufficient permissions.

5. After the issues are resolved, or at any time during the session, both the sharer and helper can end the session. To end the session, select **Leave** in the upper right corner of the Remote Help app. Upon the end of a session, the sharer is automatically signed out of their device as a security precaution to ensure all connections between the devices close.

## Monitoring and reports

You can monitor the use of Remote Help from within the Microsoft Intune admin center.

1. Sign into the [Microsoft Intune admin center](https://go.microsoft.com/fwlink/?linkid=2109431) and go to **Tenant admin** > **Remote Help**.

2. On the Monitor tab, you’ll see a count of active sessions and historical data about past sessions.

3. On the Remote Help sessions tab, you’ll see the records of past sessions, including:
   - The helper (Provider ID) and sharer (Recipient ID) of each session.
   - The device that received assistance.
   - The start and end time of the Remote Assistance session.

## Log files

Remote Help logs data during installation and during Remote Help sessions, which can be of use when investigating issues with the app.

**Installation of Remote Help** - When Remote Help installs or uninstalls, the following two logs are created in the device users' Temp folder, for example, `C:\Users\<username>\AppData\Local\Temp`. The \* in the log file name represents a date and time stamp of when the log was created.

- Remote_help_*_QuickAssist_Win10_x64.msi.log
- Remote_help_*.log

**Operational logs** - During use of Remote Help, operational details are logged in the Windows Event Viewer:

- Event Viewer > Application and Services > Microsoft > Windows > RemoteHelp

## Installation details

Automatic firewall rule creation from the Remote Help installer has been removed. However, if needed, System administrators can create firewall rules.

Depending on the environment that Remote Help is utilized in, it may be necessary to create firewall rules to allow Remote Help through the Windows Defender Firewall. In situations where this is necessary, these are the Remote Help executables that should be allowed through the firewall:

 - C:\Program Files\Remote help\RemoteHelp.exe
 - C:\Program Files\Remote help\RHService.exe
 - C:\Program Files\Remote help\RemoteHelpRDP.exe

## Languages Supported

Remote Help is supported in the following languages:
- Czech
- Danish
- Dutch
- English
- Finnish
- French
- German
- Greek
- Hungarian
- Italian
- Japanese
- Korean
- Norwegian
- Polish
- Portuguese (Portugal)
- Romanian
- Russian
- Spanish
- Swedish
- Turkish

> [!NOTE] 
> The Message function in Remote Help only supports single byte characters.

## Known Issues

- When setting a conditional access policy for apps **Office 365** and **Office 365 SharePoint Online** with the grant set to **Require device to be marked as compliant**, if a user's device is either unenrolled or non-compliant, then the Remote Help session won’t be established. 
If a conditional access policy is configured as described above and if the devices participating in the remote assistance session are unenrolled or non-compliant, the tenant will not be able to use Remote Help. 

## What's New for Remote Help 

Updates for Remote Help are released periodically. When we update Remote Help, you can read about the changes here.

### February 6, 2023

Version: 4.1.1.0 - Changes in this release:

With Remote Help 4.1.1.0 a new Laser Pointer feature has been added to better assist a Helper guide a Sharer during a session. This Laser Pointer can be used by a Helper in both Full Control and View Only sessions. Additional updates include improvements to localization, and error handling.

Various bug fixes included in this release:

- Fixed an issue where in some cases a Helper is unable to interact with elevated applications

- Resolved an accessibility issue where a Helper was unable to use some keyboard navigation hotkeys

- Reliability fixes and improved logging for WebView2 integration

### September 6, 2022

Version: 4.0.1.13 - Changes in this release:

With Remote Help 4.0.1.13 fixes were introduced to address an issue that prevented people from having multiple sessions open at the same time. The fixes also addressed an issue where the app was launching without focus, and prevented keyboard navigation and screen readers from working on launch.

For more information, go to [Use Remote Help with Intune and Microsoft Endpoint Manager](../remote-actions/remote-help.md)

### July 26, 2022

Version: 4.0.1.12 - Changes in this release: 

Various fixes were introduced to address the 'Try again later' message that appears when not authenticated. The fixes also include an improved auto-update capability.

### May 11, 2022

Version 4.0.1.7 - Webview 2 release 

### April 5, 2022

Version 4.0.0.0 - GA release

## Next steps

[Get support in Microsoft Intune admin center](../../get-support.md)
  