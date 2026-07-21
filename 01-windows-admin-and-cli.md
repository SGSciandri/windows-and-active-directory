# Windows System Administration & CLI Cheat Sheet

## Core Administrative Consoles
* **Computer Management (`compmgmt.msc`):** Primary hub for Task Scheduler, Event Viewer (`eventvwr.msc`), Shared Folders, Local Users & Groups (`lusrmgr.msc`), and Disk Management.
* **Registry Editor (`regedit.exe`):** Central hierarchical database storing OS settings, user profiles, hardware configs, and startup persistence locations.
* **Volume Shadow Copy Service (VSS):** System restore/backup subsystem. 
  * *Security Context:* Ransomware routinely executes `vssadmin delete shadows /all /quiet` to prevent localized data recovery.

## High-Yield Diagnostic Tools
| Tool | Executable | Primary Security / Sysadmin Use |
| :--- | :--- | :--- |
| **System Info** | `msinfo32.exe` | Read-only summary of hardware specs, patch states, and environment variables. |
| **Resource Monitor** | `resmon.exe` | Deep tracking of CPU, memory, disk, and network. Used to identify process handles and lock files. |
| **Performance Monitor**| `perfmon.msc` | Real-time and historical resource telemetry logging. |

## Essential Windows CLI Commands

### Networking & Socket Forensics
```cmd
:: Detailed network adapter configuration (MAC, DNS, DHCP)
ipconfig /all

:: Audit active TCP connections, listening ports, and associated PIDs
netstat -ano

:: Map active connections showing executable names (Requires Admin)
netstat -anob

:: Perform DNS lookup against default or custom resolver
nslookup tryhackme.com 8.8.8.8
