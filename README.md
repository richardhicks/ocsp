# Install-OcspServer.ps1

A PowerShell script to install and configure the Active Directory Certificate Services (ADCS) Online Responder role service on Windows Server. The Online Responder provides Online Certificate Status Protocol (OCSP) services, allowing clients to check certificate revocation status efficiently without downloading full Certificate Revocation Lists (CRLs).

## Overview

This script automates the initial installation and security hardening of an OCSP server. Specifically, it performs the following tasks.

* Installs the ADCS Online Responder role service (`ADCS-Online-Cert`) with management tools
* Configures the Online Responder service using `Install-AdcsOnlineResponder`
* Enables Windows audit policy for **Certification Services**, **Registry**, and **Other System Events** (success and failure)
* Enables OCSP service auditing by setting the `AuditFilter` registry value to `11` (0x0B), which captures service start/stop events, revocation configuration changes, and signing certificate events
* Restarts the Online Responder service (`OcspSvc`) to apply the audit filter setting
* Enables the built-in Windows firewall rules for the Online Responder service, restricted to the **Domain** profile
* Launches the OCSP management console (`ocsp.msc`) to complete revocation configuration (on servers with the Desktop Experience)

> \\\*\\\*Note:\\\*\\\* Registry and Other System Events success auditing only generates events for objects with SACLs defined, so the impact on security event log volume is minimal.

## Requirements

* Windows Server with the ADCS Online Responder role service available
* PowerShell running in an **elevated** session (the script enforces `#Requires -RunAsAdministrator`)
* Domain-joined server (the firewall rules enabled by this script are scoped to the Domain profile)

## Installation

This script is published to the [PowerShell Gallery](https://www.powershellgallery.com/packages/Install-OcspServer/). Run the following command in an elevated PowerShell window to install it.

```powershell
Install-Script Install-OcspServer
```

To update a previously installed version of the script, run the following command.

```powershell
Update-Script Install-OcspServer
```

Alternatively, download the script directly from the [GitHub repository](https://github.com/richardhicks/ocsp).

## Usage

Install and configure the Online Responder on the local server.

```powershell
.\\\\Install-OcspServer.ps1
```

Preview all changes without applying them.

```powershell
.\\\\Install-OcspServer.ps1 -WhatIf
```

Display detailed progress information during execution.

```powershell
.\\\\Install-OcspServer.ps1 -Verbose
```

## Post-Installation Configuration

This script installs and hardens the Online Responder service, but additional configuration is required before the server can respond to certificate status requests. Administrators must create a revocation configuration for each issuing Certification Authority (CA) using the Online Responder management console (`ocsp.msc`). This includes enrolling for an OCSP Response Signing certificate and specifying the CRL distribution points the responder will use.

* On servers with the Desktop Experience, the script automatically launches the OCSP management console after installation completes.
* On **Server Core** installations, the script displays a warning. Open the OCSP management console on an administrative workstation with the Remote Server Administration Tools (RSAT) installed to complete the configuration remotely.

In addition, the issuing CA must be configured to include the OCSP URL in the Authority Information Access (AIA) extension of issued certificates.

## Restart Handling

If installing the Online Responder role service requires a system restart, the script displays a warning and exits with code **3010** (the standard Windows convention for success with a restart required). Restart the server and run the script again to complete the configuration.

## Exit Codes

|Code|Meaning|
|-|-|
|0|Success|
|1|An error occurred during installation or configuration|
|3010|Success, but a system restart is required. Restart and run the script again.|

## Support

This script is provided as-is without warranty or formal support. However, feedback is welcome. Please open an [issue](https://github.com/richardhicks/ocsp/issues) on GitHub to report problems or suggest improvements.

## License

This project is licensed under the [MIT License](https://github.com/richardhicks/ocsp/blob/main/LICENSE).

## Author

**Richard Hicks**
Richard M. Hicks Consulting, Inc.
[https://www.richardhicks.com/](https://www.richardhicks.com/)



