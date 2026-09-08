# Changelog

## 1.1.0 — 2026-09-07

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
