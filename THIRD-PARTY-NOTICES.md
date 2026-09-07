# Third-party notices

OZFA Disk Control is built on the .NET platform and a small number of libraries. Everything in
this list is redistributable under a permissive licence, and every one of them is a package the
release artifacts actually contain — this is an inventory of what ships, not a list of what the
repository happens to reference.

The interface carries no third-party content at all.

The OZFA Disk Control mark and the horizontal wordmark are the product's own artwork. Every other
icon is original geometry drawn for this application, and the only typefaces named are the ones
Windows already provides. There is no bundled icon set, icon font, illustration or stock artwork,
and so no attribution condition attached to anything the user sees.

## Runtime

| Component | Version | Licence | Why it is here |
| --- | --- | --- | --- |
| .NET Runtime, Windows Desktop Runtime, ASP.NET Core Runtime | 10.0 | MIT | The platform. Both release artifacts are self-contained, so these ship inside them. |

## Libraries

| Package | Version | Licence | Why it is here |
| --- | --- | --- | --- |
| CommunityToolkit.Mvvm | 8.4.0 | MIT | `ObservableObject` and the relay commands behind every view model. |
| Microsoft.Data.Sqlite | 10.0.0 | MIT | The local history, library, alert, test, audit and sync database. |
| SQLitePCLRaw.bundle_e_sqlite3 (and core, lib, provider) | 3.0.5 | Apache-2.0 | The native SQLite engine under the above. Pinned to 3.0.x rather than the 2.1.11 that Microsoft.Data.Sqlite 10.0.0 resolves, because that version's bundled native library carries advisory GHSA-2m69-gcr7-jv3q. |
| Microsoft.Extensions.Hosting | 10.0.0 | MIT | The host and dependency-injection container the application is composed in. |
| Microsoft.Extensions.Configuration.Binder | 10.0.0 | MIT | Reads the shipped configuration file into the application's options. |
| Microsoft.Extensions.Logging.Abstractions | 10.0.0 | MIT | The logging interface every component is written against. |
| Microsoft.Extensions.Options | 10.0.0 | MIT | Options plumbing for the provider and database settings. |
| Serilog.Extensions.Hosting | 9.0.0 | Apache-2.0 | Structured logging, wired into the host. |
| Serilog.Sinks.File | 7.0.0 | Apache-2.0 | The rolling log file under the data directory. |
| Serilog.Sinks.Debug | 3.0.0 | Apache-2.0 | Debugger output. Only enabled when a debugger is attached. |
| System.Management | 10.0.0 | MIT | WMI queries: disk enumeration and the health provider. |
| System.Security.Cryptography.ProtectedData | 10.0.0 | MIT | DPAPI protection for the stored shared-library access token. |

## Build-time only

These are used to produce a release and are not part of any shipped artifact.

| Component | Version | Licence | Why it is here |
| --- | --- | --- | --- |
| xunit | 2.9.3 | Apache-2.0 | The test suite. |
| xunit.runner.visualstudio | 3.1.0 | Apache-2.0 | Test discovery and execution. |
| Microsoft.NET.Test.Sdk | 17.14.1 | MIT | Test host. |
| Inno Setup | 6.x | Inno Setup licence (free, including commercial use) | Compiles the Windows installer. Not redistributed; the installer it produces contains no Inno Setup source. |
| Pillow | 11.x | MIT-CMU | Renders the brand assets at build time. Build tool only; not shipped. |

## Not bundled

**Hard Disk Sentinel Professional** is a separate commercial product by Janos Mathe. OZFA Disk
Control reads the data it publishes over WMI when it is present and licensed on the same machine.
It is not included in any OZFA Disk Control artifact, is not installed, is not redistributed, and
none of its icons, artwork, strings or code are used. See the notes on the About page and in the README.
