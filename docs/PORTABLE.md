# OZFA Disk Control — Portable edition

Professional disk diagnostics, history and management. Nothing to install.

## Start here

1. Run **`OzfaDiskControl.exe`** from this folder.
2. That is all.

Everything the application writes — its database, your settings, its logs — goes into the
**`Data`** folder beside the executable. Nothing is written to the registry, nothing is
installed, and nothing is left behind on the machines you run it on. Copy this folder to another
stick and your entire history goes with it.

The file `portable.marker` in this folder is what selects this behaviour. Deleting it makes the
application store its data under `%LOCALAPPDATA%\OZFA\DiskControl` instead, like an installed
copy. The `Data` folder is left where it is if you do.

## What it does without anything else installed

- Finds every physical disk and identifies it properly — model, serial, firmware, bus, capacity,
  sector sizes — rather than by drive letter.
- Shows partitions, volumes, capacity and what is protected.
- Runs read-only tests: a quick check, a full surface scan with a block map, and a sequential
  read benchmark.
- Records history, raises alerts, and produces technician reports as PDF, JSON or CSV.
- Plans disk operations, checks them against the safety rules, and runs them as dry runs.

## What needs something else

**Advanced health data — the health percentage, the SMART attribute table, temperature and SSD
wear — comes from Hard Disk Sentinel Professional over WMI.** That is a separate commercial
product by a different author. It is not included here, is not installed by this application, and
you need your own legitimate licence for it. Enable **Report status to WMI** in its configuration
and press **Rescan**.

Without it everything above still works. The health readings are reported as unavailable rather
than guessed at.

**Executing a disk operation** — formatting, partitioning, assigning a drive letter — needs
administrator rights. Right-click `OzfaDiskControl.exe` and choose **Run as administrator**.
Planning, reviewing and dry runs all work without. The status bar says which you have.

## Things worth knowing

- **Closing the window does not stop it.** It goes to the notification area and keeps monitoring.
  Its right-click menu has Open, Rescan disks and Exit. Turn this off under
  **Settings → Application** if you would rather the close button quit.
- **Start with Windows is refused from a USB stick**, with the reason shown beside the setting.
  It would fail at every sign-in when the stick was elsewhere.
- **If this folder is read-only** the application falls back to storing data under your user
  profile and says so on the Settings page. It will not pretend to be portable when it is not.
- **One copy at a time per folder.** An installed copy and this one keep separate databases and
  can run side by side; the window caption says which is which.

## Safety

This application can destroy data. It refuses to target the Windows system and boot disk, refuses
to target EFI, reserved and recovery partitions, never selects a target by drive letter alone,
shows the full identity of the device before acting, requires explicit confirmation, and
re-checks the device immediately before execution.

**Real destructive execution has not yet been verified against hardware.** The planner, the
safety checks, the confirmation flow and the dry run are complete and tested; a real format or
repartition has so far only been run against a test backend. The application says so at the point
you choose a real run. Use a disk you can afford to lose.

---

`READ-ME-FIRST.md` · OZFA Disk Control v1.6.0 · KLOWZY · see `LICENSE` and
`THIRD-PARTY-NOTICES.md`
