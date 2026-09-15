# Changelog

## 1.3.0 — 2026-09-10

Two problems found using 1.2 on the bench, one of them serious.

### Fixed

- **A swapped-in disk could briefly show the health of the disk it replaced.** With two drives in a
  dock, taking both out and putting two different ones in produced, for a few seconds, the old
  drives' health figures sitting under the new drives. A manual Rescan corrected it.

  Three things lined up to cause it. Windows publishes the new devices within a second or two and
  hands them the disk numbers the old ones had. Hard Disk Sentinel scans on its own schedule, so it
  is still describing the drives that were pulled out — and nothing inside those records marks them
  as stale, because every field in them is a real reading of real hardware. And a dock that answers
  for its drives gives both bays the same product string and no serial, which leaves the health
  provider's own device numbering as the only thing left to match on. Correct evidence, correct
  numbering, wrong drives.

  The application now keeps track of which health record it put on which disk, and of exactly what
  the provider was publishing at the time. When the machine changes — the device list differs, or
  Windows reports a storage arrival or removal — a record that a disk was wearing before the change
  is held back for as long as the provider's output has not moved, because output that has not moved
  is a provider that has not looked again. While anything is held back, matching a disk by the
  provider's device number is switched off entirely, since position is precisely the evidence that
  goes wrong across a swap.

  Holding a record back is not throwing it away: a serial or a unique identifier agreeing is proof
  the same device is back, and proof still wins. A drive unplugged and plugged into another port
  keeps its own reading, and so does every disk that reports a serial of its own. The hold is
  scoped to records worn by a disk that is either gone or indistinguishable from a replacement —
  a machine's internal disks are not affected by a USB stick being plugged in.

  Every state the application publishes now carries a number identifying the set of devices it
  describes, and a state describing a set that has already been replaced is dropped rather than
  displayed. That closes the last gap: an answer about the previous machine arriving after the new
  one is on screen.

- **A disk being waited on now says so, instead of reporting no health data.** "No health data" is a
  verdict about a drive. A device attached three seconds ago that is still being asked about has not
  earned one, and a technician who reads it as a verdict puts a working disk on the reject pile.
  Those devices now read **Detecting health data…** until the answer arrives or the attempt is over.

### Changed

- **The wait for a health provider to catch up is longer, and it starts after a swap as well as
  after an arrival.** What is really being waited on is the provider's own scan interval, which is
  measured in tens of seconds, so the backoff now runs nine attempts across a minute rather than
  five across nineteen seconds, and stops the moment the answer arrives. It re-asks the provider,
  not the machine: discovery is not re-run, a device that has been attached for a while is never
  re-asked about, and the retry is abandoned outright if the machine changes underneath it. When it
  ends without an answer, the device reports no health data — which by then is the truth.

  History capture and alerting continue to sit the wait out, so a reading that was merely late does
  not become a "health data disappeared, then came back" pair in a disk's history or an alert about
  nothing.

- **Disks behind a USB adapter are now named after the drive even when the adapter's name looks
  plausible.** In 1.2 the drive's real model was substituted only when what Windows reported was
  obviously not a model — empty, numeric, or a bridge chipset's name. That missed the case this was
  reported for: the same two-bay dock passes both real models through with two disks in it, and
  reports its own perfectly ordinary-looking product name with one disk in it. "Does the Windows
  name look plausible" is therefore not a usable test.

  Where a health record has been correlated to a device behind an enclosure and that record names
  real media, the record now wins — it was read from the drive, while Windows read whatever answered
  on the bus. The one exception is a provider model that is merely a shortened form of the one
  Windows already has, since providers routinely clip the field (Hard Disk Sentinel stops at
  twenty-five characters) and trading a complete name for a truncated one describing the same drive
  would be a loss dressed up as a fix. The enclosure is still named alongside the drive.

  Nothing about this changes which device is which: correlation, history and the check that runs
  immediately before a destructive operation all continue to use the identity Windows reported, and
  the Tools page keeps displaying that identity so a confirmation always shows the identity it is
  actually checking.

### Not verified on hardware

- The stale-health fix, the retry and the adapter naming are covered by tests that reproduce the
  reported scenarios against constructed machines — including the swap where the two device lists
  are indistinguishable from each other. **None of it has been run against a real dock**: the
  development machine has no USB-to-SATA adapter, and Hard Disk Sentinel could not be started
  during this build, so no live correlation was exercised at all. What a specific dock publishes,
  and how long a specific Sentinel installation takes to rescan, decide what you will actually see.
- **No real format or repartition has been executed against a disposable disk.** Unchanged from
  1.2: planning, the safety checks, the confirmation flow and the dry run are complete and tested;
  the moment the backend writes has only ever been exercised against a test backend.

## 1.2.0 — 2026-09-09

Three problems found while using 1.1 on the bench. Nothing else changed.

### Fixed

- **A secondary disk was refused simply for containing an old Windows.** Protection was decided by
  what a disk looked like: any EFI System Partition, Microsoft Reserved Partition or recovery
  partition made the whole disk untouchable. A technician's second disk is usually a disk pulled
  out of another computer and carries exactly those, so the disk that most needed erasing was the
  one the application would not touch.

  Protection is now decided by a single question — **does the Windows installation that is running
  right now depend on this disk?** The boot disk, the system disk, the volume Windows is running
  from, any partition holding an active page file or hibernation state, and the recovery partition
  Windows itself names all still protect their disk absolutely, and no confirmation can unlock
  them. A partition *type* on a disk nothing on this machine uses no longer refuses anything: it
  produces a warning that names what is about to be destroyed, including the mounted volumes and
  the fact that another computer may stop starting.

  The two verdicts are now said in two different sets of words, because collapsing them was half
  the problem. **PROTECTED** means the application refuses. **DESTRUCTIVE WARNING** means it will
  proceed and the data will be gone. Identity checking is unchanged and still fails closed: a
  device with no trustworthy serial, no capacity or an unreadable partition table is refused, and
  the target is re-resolved and re-judged immediately before anything is written.

- **Needing administrator rights was presented as though the disk were protected.** Disk
  management requires an elevated process, and a refusal for that reason read like a safety
  verdict about the drive. It is now reported as what it is, and the page offers **Continue as
  administrator**, which restarts the application elevated and reopens the same device.

  The plan does not travel. Only the device's identity does — the elevated copy re-reads the
  machine, re-evaluates protection, and asks for the plan to be reviewed and confirmed again,
  because the device list can change while the consent prompt is on screen. Dry runs continue to
  work without elevation, exactly as before.

- **A disk plugged in while the application was running could show "No health data".** Windows
  publishes a device as soon as the bus enumerates it, which for some drives is before Hard Disk
  Sentinel has finished its own scan — so the disk appeared complete in every respect except its
  health, and pressing Rescan a few seconds later fixed it.

  The application now waits that race out by itself: for a newly connected device only, it re-asks
  the health provider five times over nineteen seconds and stops the moment a reading arrives. It
  re-asks the provider, not the machine — discovery is not re-run — and a device that has been
  attached for a while is never re-asked about, so this can never become a permanent poll. History
  and alerting sit out the wait, so a reading that was merely late does not become a "health data
  disappeared, then came back" pair in the disk's history or an alert about nothing.

### Changed

- **Disks in USB adapters are now named after the drive, not the adapter.** Some adapters answer
  the Windows storage stack with the bridge chipset's own product string, or with a placeholder as
  bare as `0`, while Hard Disk Sentinel reads the drive's real identity through the same adapter.
  Where a health reading has already been correlated to the device, the drive's model — and its
  serial and firmware where the adapter clearly supplied neither — is what the disk list, the
  header, the Overview page and the report now show, with the enclosure named alongside it.

  The substitution is display only, and narrow: it happens only for a device reached through an
  enclosure, only where a reading was already matched to it, and only where what Windows reported
  is recognisably not a drive model and what the provider reported is. A drive that names itself
  is never renamed. Everything that decides *which device is which* — correlation, history, and
  the check that runs immediately before a destructive operation — continues to use the identity
  Windows reported, and the Tools page keeps showing that identity so the confirmation always
  displays the identity it is actually checking.

### Not verified on hardware

Honest about what this release has and has not been run against:

- The protection rules were verified on a real two-disk machine: the system/boot disk is refused
  with its dependencies named, and a secondary NVMe carrying a leftover Microsoft Reserved
  Partition and a mounted volume is now correctly allowed with a warning that names both.
- The USB-adapter identity change and the hot-plug retry are covered by tests, but the development
  machine has no USB-to-SATA adapter attached, so neither has been seen working against a real
  bridge. The behaviour with a specific adapter depends on what that adapter publishes.
- No real format or repartition has been executed against a disposable disk. Planning, the safety
  checks, the confirmation flow and the dry run are complete and tested; the moment the backend
  actually writes has still only been exercised against a test backend. The Tools page says so
  where the choice is made.

## 1.1.0 — 2026-09-08

### Fixed

- **Disks in USB adapters and docks showed "No health data".** A USB-to-SATA/NVMe bridge answers
  the Windows storage stack on the drive's behalf and reports the serial number baked into the
  adapter rather than the one printed on the drive, while Hard Disk Sentinel reaches past the
  bridge and reports the real one. One device therefore presented two unrelated serials, and the
  rule that two different serials prove two different devices — correct for a directly attached
  disk — threw the reading away before the model or the capacity was ever looked at. Health,
  temperature, performance and the SMART table now appear for externally attached disks.

  Adapters that go further and publish the bridge chipset's own name as the model, leaving nothing
  in the record that describes the drive, are handled by a last-resort pass over the health
  provider's own device numbering — but only after that numbering has been proved against the
  disks in the same scan that matched on their own identity. Capacity is only ever a veto, never
  the evidence.

  Safety is unchanged. A reading matched this way is labelled in the interface as matched on
  limited evidence, and can never authorize a format or a repartition: those still require a
  unique identifier. Two disks that cannot be told apart leave both readings unattached, because
  showing one disk's health under another disk's name would be worse than showing none.

- **Correlation failures are now explained in the log.** "No health data" looked identical whether
  the provider never saw the disk, saw it and rejected it on an identifier, or matched two disks
  equally well and dropped the reading as ambiguous. Each is now a different sentence naming which
  fields disagreed — never what they contained.

### Changed

- **A sign-in launch now stays in the notification area by default.** Asking Windows to start
  something when you sign in is a request for it to be watching, not a request for a window in
  front of whatever you signed in to do. Monitoring, history capture, alerting and hot-plug
  detection run exactly as before; starting the application yourself still shows the window. An
  existing installation keeps whatever it was already set to. The choice is also now disabled
  until "start when I sign in" is switched on, since it has nothing to decide before then.

- **Corrected the update and privacy documentation.** The 1.0.0 notes said no update repository was
  configured, which was already untrue of the published build. Automatic checking is on by default
  and contacts GitHub for this product's list of releases; it sends no machine, user or disk
  information, and it cannot download or install anything.

### Known limitation

- The USB correlation fix has **not yet been confirmed against real bridge hardware** — none was
  available on the build machine. It is covered by tests and is fail-closed by construction: it
  suspends a disproof and can never raise a match to the confidence a destructive operation
  requires. If your adapter still reports no health data, the log now says which fields disagreed.

## 1.0.0 — 2026-09-07

The first public release.

### Highlights

- **Two editions.** `OZFA-Disk-Control-v1.0.0-Setup.exe` installs normally, with Start Menu and
  uninstall entries and an optional sign-in start. `OZFA-Disk-Control-v1.0.0-Portable.zip` runs
  from anywhere and keeps its database, settings and logs in a `Data` folder beside the
  executable. Both are self-contained: no .NET installation is required.
- **It keeps monitoring when you close the window.** Closing goes to the Windows notification
  area rather than ending the application, so history capture, alerting, temperature events,
  hot-plug detection and the shared-library queue all continue. The icon's menu has Open, Rescan
  disks, the current monitoring status, and Exit.
- **It remembers where it was.** Window position, size and maximized state come back across
  launches — and are refused when they would land on a monitor that is no longer connected. The
  Light/Dark/Follow Windows choice is remembered too, with dark as the first-run default.
- **One copy per data folder.** Starting it again brings the running window to the front. An
  installed copy and a portable copy keep separate databases and can still run side by side.

### Changed

- **Removed the menu bar.** Every `File · Disk · View · Report · Configuration · Help` entry was
  a second route to something already one press away on the toolbar, on the tab strip, or on the
  Settings page — at the cost of a row of vertical space on every screen. The toolbar now carries
  the product mark at its head.
- **Rebuilt the Settings page** as a category rail with one group on screen at a time:
  Application, History capture, Alerts, Disk tools, Network, Local database. Six panels stacked
  in one scrolling column made every group narrow and put Save several screens away from most of
  what it saved.
- **New Application settings:** theme, keep running in the notification area, start with Windows,
  start minimized to the notification area, and disk connected/removed notifications.
- **Removed the Performance page.** Its tab was present, permanently disabled and explained
  itself, which is honest but is still a dead entry in a finished product's navigation.
  Benchmarking is on the Tests page.
- **Reworded the About page** and gave it the edition and data folder.
- **A generated application icon** with a separate simplified drawing for the 16- and
  24-pixel frames, so the mark stays legible at notification-area size.
- One notification-area icon for the whole application. Alerts previously raised a second one,
  which read as a different program.

### Added

- **Official branding.** The OZFA Disk Control mark is now the application, taskbar, notification
  area, shortcut and installer icon, as a proper multi-resolution `.ico` (16 through 256) whose
  small sizes are optically compensated rather than merely resampled. The horizontal lockup heads
  the About page and the README. Both are derived from the source artwork by
  the brand build script; the transparency is cut from the artwork's own edge rather than masked to
  a guessed corner radius, so there is no pale rim on a dark taskbar.
- **An update checker.** About shows the installed version, the status, when it last checked, and
  a Check for updates button; Settings gains an Updates category. It reads a version number and
  nothing else — it cannot download or install, and opening a release happens in your browser.
  Automatic checking is on by default, once a day, stable releases only, and one notification per
  new version rather than one per check.

### Fixed

- A `SectionPanel` title ran straight through a wide header badge at narrow widths. A `Grid` does
  not clip its children, so a star column squeezed to nothing still drew at full width.
- **The light theme's toolbar labels were nearly invisible after switching themes** — they
  measured 1.1:1 against the rail. An implicit `TextBlock` style sets the page text colour on
  every TextBlock in the application, including the labels inside toolbar buttons, so they never
  inherited the colour of the control hosting them; in the dark theme the wrong colour and the
  right one are nearly identical, which is why only the light theme showed it. Chrome labels and
  glyphs now take their colour from the control that hosts them, chrome state changes moved from
  template triggers to style triggers, and both palettes share one measured set of chrome values
  (primary ink 14.9:1, muted 8.2:1, disabled 4.2:1, selected fill carrying white at 4.9:1).
- The toolbar pushed its action buttons — Rescan among them — off the right-hand edge at the
  minimum window size. It now drops the navigation labels, and then the product name, before it
  drops anything you can press.
- A launch that was meant to start in the notification area still put the window on screen,
  because applying a saved window placement is itself what makes a window visible.

### Known limitations

- **Real destructive execution is not yet verified against hardware.** The planner, the safety
  checks, the confirmation flow and the dry run are complete and covered by tests; a real format
  or repartition has so far only been executed against a test backend, because both disks in the
  development machine hold protected roles. The application says so at the point a real run is
  chosen.
- **Advanced health data requires Hard Disk Sentinel Professional** with WMI enabled. It is a
  separate commercial product, is not bundled, and needs your own licence. Everything else works
  without it.
- **Native SMART and NVMe access is not implemented.** The provider boundary exists so it can be
  added without any page, report, database or safety rule changing.
- The release is **not code-signed**; Windows SmartScreen will warn on first run.
