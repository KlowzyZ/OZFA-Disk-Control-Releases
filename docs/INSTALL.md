# Installing OZFA Disk Control

There are two editions. They are built from one publish of the same code and behave identically;
the only difference is where they keep their data.

| | Installer | Portable |
| --- | --- | --- |
| File | `OZFA-Disk-Control-v1.4.0-Setup.exe` | `OZFA-Disk-Control-v1.4.0-Portable.zip` |
| Installs to | `C:\Program Files\OZFA Disk Control` | wherever you unzip it |
| Data, settings and logs | `%LOCALAPPDATA%\OZFA\DiskControl` | `Data\` beside the executable |
| Start Menu and uninstall entry | yes | no |
| Leaves anything behind | data folder above | nothing |
| .NET prerequisite | none | none |

Both are self-contained: the .NET runtime is inside the download, so nothing has to be installed
first. That is deliberate — this is a tool you reach for on a machine that is already in trouble.

## Requirements

- Windows 10 version 2004 (build 19041) or later, or Windows 11. 64-bit.
- No .NET installation required.
- **For advanced health data:** your own licensed installation of **Hard Disk Sentinel
  Professional**, with WMI support enabled. See [Health data](#health-data) below.

## Installer

1. Run `OZFA-Disk-Control-v1.4.0-Setup.exe`.
2. Windows SmartScreen may warn that the publisher is unrecognised — the release is not
   code-signed. Choose **More info → Run anyway** if you are satisfied the download is genuine;
   the checksums are published with the release.
3. Accept the licence and choose a location. The default installs for every user on the machine
   and needs administrator rights; if you do not have them, the privileges dialog lets you
   install into your own profile instead.
4. Optional tasks:
   - **Desktop shortcut.**
   - **Start when I sign in to Windows.** This writes the same registry entry the application's
     own Settings page manages, so the two always agree, and it is removed when you uninstall.
   - **The shared library server.** Only needed if this machine will host the optional shared
     library for other workstations. See [the README](../README.md#running-the-server).

### Uninstalling

Use **Settings → Apps → Installed apps → OZFA Disk Control**, or the Start Menu entry.

Uninstalling removes the program, the shortcuts and — when the account running the uninstaller is
the one the entry was written for, which is the usual case — the sign-in entry. If it survives,
remove it from **Task Manager → Startup apps**; it is listed as *OZFA Disk Control*. It **does
not** remove your history database, settings or logs — a diagnostics tool should not silently destroy the
record of every disk it has ever seen. To remove those as well, delete:

```text
%LOCALAPPDATA%\OZFA\DiskControl
```

## Portable

1. Unzip `OZFA-Disk-Control-v1.4.0-Portable.zip` anywhere — a USB stick, a technician's folder, a
   network share you have write access to.
2. Run `OzfaDiskControl.exe`.

Everything it writes goes into the `Data` folder beside the executable. Nothing is written to the
registry, nothing is installed, and nothing is left on the machine you ran it on.

The window caption, the About page and the status bar all say **Portable** so you can tell at a
glance which edition you are looking at when both are open on one machine. They keep separate
databases and can run side by side.

Two things behave differently in the portable edition, both on purpose:

- **Start with Windows is refused when the folder is on removable or network storage**, with the
  reason shown next to the setting. Registering it would produce a Windows error at every sign-in
  for as long as the stick is somewhere else.
- **If the folder cannot be written to** — a read-only share, a write-protected stick — the
  application falls back to the per-user location and says so on the Settings page rather than
  quietly behaving like the installed edition.

## Running it

The application does not need administrator rights to discover disks, read health, SMART,
temperature and partitions, record history, run read tests, or produce reports. The status bar
says **Read-only** when it is not elevated.

Administrator rights are needed only to **execute** a disk operation — formatting, partitioning,
changing a drive letter. Planning one, reviewing it and running it as a dry run all work without
elevation. Right-click the shortcut and choose **Run as administrator** when you need to execute.

## Closing and the notification area

Closing the window puts OZFA Disk Control in the notification area rather than ending it, so
history capture, alerting, temperature events, hot-plug detection and the shared-library queue
keep running. Double-click the icon to bring the window back; its right-click menu has **Open**,
**Rescan disks**, the current monitoring status, and **Exit**, which stops the application
properly.

If you would rather the close button ended the application, turn off **Keep running in the
notification area** under **Settings → Application**.

Only one copy runs at a time per data folder. Starting it again brings the running one to the
front instead of launching a second copy.

## Health data

OZFA Disk Control's version 1 reads advanced health data — the health percentage, the SMART
attribute table, temperature and the SSD wear figures — from **Hard Disk Sentinel Professional**
over WMI. That is a separate commercial product. It is **not bundled**, **not installed** and
**not redistributed** by OZFA Disk Control, and you need your own legitimate licence for it.

To make it available:

1. Install Hard Disk Sentinel **Professional** on the same machine.
2. In Hard Disk Sentinel, enable WMI support: **Configuration → Advanced → Integration →
   Report status to WMI** (the exact wording varies by version).
3. In OZFA Disk Control, press **Rescan**. The status bar's **Health data** segment shows
   `Connected` and how many of the detected devices reported in.

Without it, everything else still works: device discovery, disk identity, partitions and volumes,
capacity, read tests, surface scans, benchmarks, reports, history of what it can see, the shared
library and the disk tools. The health readings are reported as unavailable rather than guessed.

The **Diagnostics** page shows exactly what the provider returned, field by field, including the
raw values, which is the first place to look if a device is detected but has no health data.
