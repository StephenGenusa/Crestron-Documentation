# Forensics and Crestron Devices #

Crestron devices sit quietly in racks and closets in government, commercial and
residential buildings, controlling door hardware, lighting, shades, HVAC, AV and
security-system integration. They keep logs. Most investigators do not know these
devices exist, let alone that they record anything.

This document is for three audiences:

- **Investigators and examiners** who need to know what is on these devices, how
  to get it without destroying it, and what it does and does not prove.
- **First responders and site staff** who need to make the right call in the first
  ten minutes. Section 2 is written for you and assumes no Crestron knowledge.
- **Businesses and counsel** assessing what their own building systems retain, for
  incident response, insurance, employment matters or litigation hold.

## A note on verification ##

Forensic value lives in specifics, and wrong specifics are worse than none. Every
path, filename and command below was verified in one of two ways:

- Read directly out of a physical acquisition of a real device, or
- Read out of Crestron's own per-device console help.

Numeric reason values in section 8.3 are reported as observed on the exhibits
examined. They are not authoritative: read the description the device itself prints
alongside the number, and rely on that rather than on the table.

Anything I could not confirm that way is tagged **[unverified]**. Treat those as
leads to check on your own exhibit, not as findings.

The verified material here is grounded in **3-Series hardware running Windows CE**,
specifically an MC3 acquisition. Touch panels, 4-Series, and Android-based devices
follow the same general shape but their specifics differ and are marked accordingly.

## Contents ##

1. [First response — do this now](#1-first-response--do-this-now)
2. [Why these devices matter to an investigation](#2-why-these-devices-matter-to-an-investigation)
3. [Evidence preservation and spoliation](#3-evidence-preservation-and-spoliation)
4. [Recognizing the equipment on site](#4-recognizing-the-equipment-on-site)
5. [Acquisition](#5-acquisition)
6. [Device families and operating systems](#6-device-families-and-operating-systems)
7. [Artifact catalog](#7-artifact-catalog)
8. [Interpreting the evidence](#8-interpreting-the-evidence)
9. [Off-device corroboration](#9-off-device-corroboration)
10. [Anti-forensics and evidence destruction](#10-anti-forensics-and-evidence-destruction)
11. [Legal and evidentiary considerations](#11-legal-and-evidentiary-considerations)
12. [Quick reference](#12-quick-reference)

---

## 1. First response — do this now ##

**The clock is running.** These devices write logs to flash storage that is
continuously reused. Every minute a device stays powered and running, it is
overwriting the record of what it did. This is not hypothetical: log data ages out
by ordinary operation, with no one intending to destroy anything.

If you take one thing from this document, take this: **a Crestron processor that
stays powered is actively consuming its own evidence.**

### Triage checklist ###

**Do:**

1. **Photograph everything before touching it.** The rack, each device's front and
   rear, all cabling, all indicator LEDs, and any labels. Labels frequently carry
   the integrator's name, a job number and a site name — all of which identify
   records you can obtain later.
2. **Identify the processors first.** They hold the most. See section 4 for what
   they look like. Touch panels are second priority.
3. **Pull power to the processors.** Unplug at the device. This freezes the flash.
4. **Then collect touch panels.** Newer panels are powered over Ethernet; older
   ones run on Cresnet or a 24 V supply.
5. **Record the wall-clock time you pulled power**, against a known-good reference
   (your phone, synced). You will need this to correct for potential clock drift later.
6. **Seize the whole device**, not just a memory card. Case, card, and any
   externally attached storage.
7. **Note anything that looks like power management** — a rack PDU, a timer, an
   IP-controlled outlet strip. It changes how you read reboot events (section 8).

**Do not:**

1. **Do not boot the device to "have a look."** The persistent log holds only the
   current and the previous session. Booting a seized device once pushes the
   evidence session into `PreviousBoot`. Booting it **twice destroys it.**
2. **Do not connect it to a network.** It may be enrolled with a cloud service or a
   remote monitoring provider that can reach in and update, reset or reboot it.
3. **Do not "reset it to check something."** `RESTORE` and the factory-default path
   are destructive and are recorded as such.
4. **Do not let site staff or the integrator "help" by pulling logs.** A live log
   pull returns a small rotating window. Imaging recovers far more, including
   deleted data. Well-meaning assistance destroys evidence.
5. **Do not assume the SD card is the whole story.** Some devices keep data on
   internal flash that never leaves the board.

### If you genuinely cannot seize the device ###

Business continuity sometimes wins — the system runs the building's door hardware
and cannot go down. If so, document the constraint, and see
[section 5](#5-acquisition) on live collection. Capture the persistent log, the
audit log and the reboot reason **before** anyone reboots anything, and record every
command you issue. Live collection is a fallback, not a substitute.

---

## 2. Why these devices matter to an investigation ##

A Crestron control system is a general-purpose automation computer wired into the
physical building. Depending on how it was programmed, it may know:

| Question | Possible source |
|---|---|
| Was the door locked or unlocked, and when? | Access/lock control logic, relay states |
| Were the lights on in that room at that hour? | Lighting control and scene events |
| Was anyone in the space? | Occupancy sensors, HVAC setback, "room in use" logic |
| Was a meeting recorded or streamed? | AV routing, DM switching, codec control |
| Was the alarm armed? | Security-panel integration |
| Did someone override the schedule? | Timer events vs. user-driven events |
| Who connected to the system, from where? | Console connection logs with source IP |
| Was the system reconfigured, and by whom? | Audit log, program load events |
| When was the device restarted, and why? | Reboot reason code in NVRAM |

Two caveats that matter more than the list:

**A Crestron system logs its own behavior, not the building's ground truth.** It
records that it *commanded* a lock to open. Whether the lock physically opened, and
whether a person walked through, are separate questions needing separate evidence.

**What is recorded depends entirely on how the system was programmed.** There is no
standard schema. One site's processor is a rich event recorder; another's logs
nothing but boot messages. You cannot know which you have until you look. This is
why the program itself (section 7) is an artifact worth as much as the logs.

---

## 3. Evidence preservation and spoliation ##

The core problem is that **these devices destroy evidence as a side effect of
running normally.** Nobody has to act in bad faith. Three mechanisms:

1. **Log rotation.** Message logs rotate through a fixed number of files. Once the
   ring wraps, the oldest content is gone.
2. **Session-scoped persistent logs.** The persistent log keeps the current boot
   session and the previous one. Two reboots and the window you cared about is gone.
3. **Flash reuse.** Deleted data survives only until its space is reallocated. A
   busy processor reallocates continuously.

The practical consequence is that **delay itself causes spoliation.** If you are
counsel and you anticipate that a building system holds relevant evidence, a
litigation hold that says "preserve documents" does nothing here. The hold must
specifically direct that the control processors be powered down and imaged, and it
must reach the integrator, who may be the only party with remote access.

The party most likely to destroy this evidence is usually not a wrongdoer. It is
a well-meaning technician doing a routine reboot, a firmware update, or a nightly
automated restart, days after the incident and weeks before anyone thinks to ask.

---

## 4. Recognizing the equipment on site ##

You are looking for a rack, a closet, a cabinet above a ceiling, or a run of DIN
rail in an electrical room.

**Processors** — the priority. Typically a 1U or half-rack black or silver box, or
a DIN-rail module. Front panel usually shows a `PWR`/`NET`/`MSG` indicator group.
Model names on the chassis include `MC3`, `RMC3`, `CP3`, `CP3N`, `PRO3`, `AV3`,
`DIN-AP3`, `MC4`, `CP4`, `AV4`, `DMPS3-*`, `PYNG-HUB`. Anything beginning `MC`,
`CP`, `PRO`, `AV`, `DIN-AP` or `DMPS` is a processor.

**Touch panels** — wall-mounted or tabletop screens, model names beginning `TS`,
`TSW`, `TPMC`, `TST`. Second priority, and worth taking: newer ones run Android and
keep their own artifacts.

**Other devices worth noting** (lower priority, but photograph and inventory):
DM matrix switchers and endpoints, `CEN-*` network devices, `ZUM-*` lighting,
Cresnet keypads and occupancy sensors, DSPs.

**On the network.** Crestron's assigned MAC OUI is **`00:10:7F`**. Devices commonly
listen on TCP 41794 (CIP) and 41795 (CTP console), plus Telnet 23, HTTP 80, HTTPS
443, FTP 21 and SSH 22 depending on configuration. A network scan or a DHCP lease
table will often reveal a control system nobody mentioned. See `DeviceSecurity.md`
in this repository for the default open-port set.

**Labels are evidence.** Integrator name, job number, site name and rack elevations
are routinely printed on labels and on the devices themselves. Photograph them.
They tell you whose records to request.

---

## 5. Acquisition ##

### 5.1 Order of preference ###

1. **Physical acquisition of the storage** — best. Full filesystem, unallocated
   space, deleted files.
2. **Physical acquisition of the whole device** where storage is not removable.
3. **Live console collection** — a fallback that gets you a fraction of the data
   and alters the device while you do it.

### 5.2 Physical acquisition ###

Many Crestron devices store their operating system and data on a removable card.
Others use soldered flash.

Getting to the card generally means removing case screws and, on several models, a
metal shield screwed over the board. Looking at the opened device from several
angles makes the card much easier to spot than looking straight down. This
repository's `NoRMARepairs.md` covers opening these devices and working on them
without going through Crestron's RMA process, which is the same physical access
problem viewed from the maintenance side.

Once you have the card, image it with whatever you normally use. Write-block it as
you would any other exhibit. Hash before and after.

**What you will find is not a normal disk.** On a verified MC3 acquisition:

- **No MBR and no partition table.** Byte 0 of the image is `FE 00 00 EA` — boot
  code, not a partition-table signature. The image begins with a bootloader, so
  tools that expect a partition table will report it as unrecognized data.
- The data volume began at **byte offset 103,285,760**, with the backup boot sector
  exactly 12 sectors later. *That offset is specific to this device and firmware —
  do not assume it. Find your own.*
- The filesystem is **TexFAT**, Microsoft's transaction-safe exFAT variant used by
  Windows CE. Expect `$TEX_FAT` entries and duplicated `$ALLOC_BITMAP` structures.
  Because TexFAT maintains redundant metadata for crash safety, **prior directory
  states may be recoverable** from the alternate structures. This is an advantage
  over ordinary exFAT and is worth exploiting.

Practical method for locating the volume when there is no partition table: scan the
image for sector-aligned 512-byte blocks ending in `55 AA` whose first byte is `EB`
or `E9`, then validate the candidate as a boot sector. On the verified image this
returned the volume start and its backup immediately.

Once you know the offset, standard tooling works:

```
fls -f exfat -r -p -o <sector_offset> image.dd
```

Note that `fsstat` may hang or run extremely long on these volumes; `fls` was
reliable where `fsstat` was not. Extracting the volume to fast local storage first
makes a large difference.

### 5.3 Live console collection ###

Use only when you cannot power the device down, and understand that you are
changing the exhibit.

Access is by the CTP console (TCP 41795), Telnet, SSH, or a direct serial
connection, typically via Crestron Toolbox. **Every console connection you make is
itself logged**, with your source IP — which is good practice-hygiene, because your
own activity is distinguishable from the subject's later.

Collect in this order, before anything reboots:

| Order | Command | Gets you |
|---|---|---|
| 1 | `NVRAMREBOOT SHOW` | Why the device last restarted — volatile in the sense that the next reboot overwrites it |
| 2 | `PLOGPREVIOUS` | Persistent log, previous boot session — lost on next reboot |
| 3 | `PLOGCURRENT` | Persistent log, current session |
| 4 | `PLOGALL` | Both sessions |
| 5 | `GETAUDITLOG` / `PRINTAUDITLOG ALL` | Audit log, if audit logging was ever enabled |
| 6 | `ERRLOG` | Message log at the chosen severity |
| 7 | `UPTIME`, `PROGUPTIME` | How long the system and each program have been running |
| 8 | `TIMEDATE`, `TIMEZONE` | The device's own idea of the time — record this against real time |
| 9 | `WHO`, `IPIDWHO` | Connected users and CIP peers |
| 10 | `SHOWHW` / device info | Model, firmware, serial |

Capture the whole session to a transcript, timestamped, and hash it.

**Never issue** `CLEARAUDITLOG`, `RESTORE`, `ERASE`, or any reboot command on an
exhibit.

### 5.4 Documentation ###

Record, at minimum: device model and serial; the exact wall-clock time of power-down
against a trusted reference; the device's own reported time at that moment; who had
physical and network access; and whether the device was enrolled with any cloud or
remote monitoring service (section 9).

---

## 6. Device families and operating systems ##

What you can recover depends heavily on the platform:

- **3-Series processors** run **Windows CE** (6 or 7 depending on model). This is
  the platform the verified material in this document comes from. Artifacts include
  a Windows CE registry, TexFAT volumes, and the `Sys\PLog` log structure below.
- **4-Series processors** run **Linux**, not Windows CE, and are laid out entirely
  differently — a partitioned eMMC with ext3/ext4 volumes. Verified here from a CP4
  acquisition. See section 7.8.
- **Windows CE touch panels** (the TSW-x50 generation and similar) are verified
  here from a TSW-750 acquisition. They share the processor's `Sys\PLog` structure
  but add artifact classes of their own — see section 7.7.
- **Newer touch panels and DSPs run Android**, which is why some panel software
  ships as an APK. See section 7.9. **[unverified]** in specifics — no Android
  device acquisition was examined.

`DeviceOperatingSystems.md` in this repository has a per-model OS table. It is not
current, but it is a good starting point for deciding what method to apply to a
given model.

---

## 7. Artifact catalog ##

Everything in this section was read from a real MC3 acquisition unless marked.

### 7.1 Top-level layout ###

```
Documents and Settings/   Windows CE registry hives
Sys/                      logs, config, keys, scheduled events
Simpl/                    program slots (the site's actual control program)
Nvram/                    non-volatile app data, project files
Program Files/
Application Data/
AutoUpdateLogs/           firmware update history
User/  FTP/  HTML/        user-writable areas, web interface
My Documents/  Temp/  Recycled/  ROMDISK/  SSHBanner/  Windows/
```

### 7.2 Logs — the primary artifact ###

```
Sys/PLog/CurrentBoot/Crestron_00.log     active session
Sys/PLog/PreviousBoot/Crestron_00.log    previous session
Sys/PLog/ZippedLogs/PLog_2017-08-13_22-08-02.zip
```

Three things matter here:

1. **Logs are split by boot session**, not kept as one continuous file. This is the
   mechanism behind the "do not boot it twice" warning.
2. **`ZippedLogs` is the sleeper.** Archived logs carry their own date and time in
   the filename and can extend history well beyond the live rotation. Always check
   it.
3. Files rotate as `Crestron_00.log` through `Crestron_09.log`, `_00` being current.
   **The depth is configurable**, via `LOGGERNUMBACKUPLOGS[:program#] {N}`. Do not
   assume ten. Check the device's actual setting, and note that a suspiciously small
   configured value is itself worth asking about.

**Log line format**, verified:

```
<Severity>: <Process>.exe [App N] # <timestamp> # <message>
```

Severities observed: `Info`, `Notice`, `Warning`, `Error`; `OK` and `Fatal` are also
selectable via `ERRLOG`.

Real examples, with addresses replaced by documentation-range equivalents:

```
Notice: nk.exe # 2017-06-19 20:43:07  # User Reboot
Notice: nk.exe # 2017-06-19 20:43:07  # System startup: MC3 [v1.501.0043 (Nov 07 2016) #007B9E5E]
Notice: ConsoleServiceCE.exe # 2017-08-04 15:22:24  # CTP Connection from: 198.51.100.18
Warning: ConsoleServiceCE.exe # 2017-08-04 15:23:11  # Deprecated CTP connection accepted from address 198.51.100.18:49887.
Notice: ConsoleServiceCE.exe # 17:36:43  3-17-2016  # SHELL Connection from: 198.51.100.13
Info: LogicEngine.exe [App 1] # 17:37:42  3-17-2016  # Loading program from \SIMPL\app01
Notice: LogicEngine.exe # 17:37:43  3-17-2016  # **Program 1 Started**
```

High-value line types:

| Line | Why it matters |
|---|---|
| `System startup: <MODEL> [<firmware> (<build date>) #<hash>]` | Model, firmware and build, per boot — builds a firmware timeline |
| `User Reboot` | A *commanded* restart. Read section 8.3 before drawing conclusions |
| `CTP Connection from: <ip>` | Remote console access, with source address |
| `Deprecated CTP connection accepted from address <ip>:<port>` | Source address **and source port** |
| `SHELL Connection from: <ip>` | Shell access |
| `Loading program from \SIMPL\appNN` / `**Program N Started**` | Program load and restart timeline |

### 7.3 Persistent log, audit log, scheduled events ###

```
Sys/AuditLog/            audit log storage
Sys/TimerEvents/         scheduled events
Sys/Authentication/      authentication data
```

**The audit log is the closest thing to a security log.** When enabled it records
logons, logoffs, account management and console commands. It is **off by default**,
so its absence proves nothing about conduct — only that nobody turned it on. Where
it exists it is high-value and attributable.

**`Sys/TimerEvents/` is the direct answer to "scheduled or user-driven?"** — it holds
the system's scheduled events. Compare an event's time against this to determine
whether it was automation or a person. See section 8.4.

### 7.4 Configuration and identity ###

| Path | Contains |
|---|---|
| `Sys/Setup Configuration.txt` | Device setup |
| `Sys/MyCrestron Configuration.txt` | Cloud service enrolment — see section 9 |
| `Sys/hwConfig.ini` | Hardware configuration |
| `Sys/~.iptable0.dip` | CIP/IP table: which peers this device talks to |
| `Sys/SSH/ssh_host_rsa_key` | Device SSH host key — ties network captures to *this* device |
| `Sys/pems/fw_public_key.pem` | Firmware signing key material |
| `Sys/pufversion.ini` | Firmware/update version state |
| `Sys/AutoUpdate/` + `AutoUpdateLogs/` | Update history: when firmware changed, and to what |

The **IP table** is worth dwelling on. It enumerates the other devices this
processor communicates with, which lets you reconstruct the control topology — and
tells you what *else* to go seize.

**Built-in accounts.** Crestron devices ship with vendor accounts that exist
independently of any account the owner or integrator created. Two things follow for
an investigation. First, **attribution**:
access through a built-in vendor account in the audit log or console-connection log
is not the owner and not the integrator — it is vendor-level access, and it should
be read as such rather than attributed to site personnel. Second, **exposure**: the
existence of standing vendor accounts is one of the reasons a control system should
never have been reachable from an untrusted network, and their presence in the logs
of a device that was internet-exposed is worth noting. This document does not publish
the credentials for these accounts; their forensic relevance is that they exist and
that activity under them is distinguishable from site activity.

### 7.5 The control program ###

```
Simpl/App00/  Simpl/App01/  ...
```

Program slots hold the site's actual control program: compiled logic, SIMPL+
modules and SIMPL# libraries. See `ProgramSlots.md` and `3SAppManagers.md` in this
repository for how slots and application managers work.

**This is often the single most valuable artifact**, because it defines what the
system was capable of doing and what it was set up to record. Without it, a log line
saying a relay fired is uninterpretable. With it, you can determine which physical
device that relay drives.

Note that log messages may reference a slot as `\SIMPL\app01` while the on-disk
directory is `Simpl/App00`. Do not assume the numbering matches; verify against the
device.

### 7.6 Registry and deleted data ###

```
Documents and Settings/System.hv          system registry hive
Documents and Settings/default/user.hv    user registry hive
Program Files/SaveRegistry/MC3Reg.lgr     (was deleted; recoverable)
```

Windows CE registry hives can be parsed offline. Device configuration state is
held under `HKEY_LOCAL_MACHINE\Crestron\System`.

**Deleted data is abundant.** The verified acquisition had **74 deleted entries**
recoverable in a single volume, including a project file whose filename carried the
integrator's job number, the client name and the site name. Filenames alone can
identify the site, the integrator, and the project records worth subpoenaing —
which is precisely why you image rather than copy files off a running device.

### 7.7 Touch panel artifacts (Windows CE panels) ###

Verified from a TSW-750 acquisition. Panels are the second seizure priority and
carry artifact classes the processors do not.

**Storage layout differs from a processor.** The panel carried **two** filesystems,
not one: a FAT32 volume roughly 400 MiB in, which was effectively empty, and the
main exFAT volume around the 1 GiB mark. The exFAT volume did **not** begin on a
neatly aligned sector — it started one sector past the round number. Scan for the
volume rather than assuming an offset, and do not stop after finding the first
filesystem.

**Logs — same structure, plus a much deeper archive:**

```
Sys/PLog/CurrentBoot/Crestron_00.log
Sys/PLog/PreviousBoot/Crestron_00.log
Sys/PLog/ZippedLogs/PLog_<date>_<time>.zip
```

The examined panel held **46 archived log bundles spanning more than four years**.
On a panel, `ZippedLogs` may be the single richest source in the building. Panels
also use additional log names not seen on the processor, including `MsgLog.txt` and
`MsgLogPrev.txt` (current and previous message log) and `flashlog.txt`.

**Crash dumps** — `Sys/DumpFiles/` and `Sys/ExtraDumpFiles/`, with no processor
equivalent. Dumps can contain memory contents from the moment of failure.

**Registry hives live somewhere else.** On the panel they are under
`Windows/Registry/` (`system.hv`, `default/user.hv`, plus `Windows/Keys/*.mky`),
not under `Documents and Settings/` as on the processor. Look in both places.

**Web cache.** The panel carried a full Internet Explorer cache tree at
`Windows/Profiles/guest/Temporary Internet Files/Content.IE5/`, including deleted
cached documents. Panels fetch weather, scheduling and other feeds, so this can
establish what the panel retrieved, from where, and when — and cached feed content
may itself carry location information.

**Flash local shared objects.** The Core3/SmartGraphics interface is Flash-based
(`SmartGraphics.swf`, `container.swf`, `Scheduler.swf` and theme bundles under
`ROMDISK/`). Consequently the panel keeps
`Application Data/Macromedia/Flash Player/#SharedObjects/` — Flash LSOs, which can
persist interface state, and `Adobe/Flash Player/AssetCache/`. This is a
panel-specific artifact class worth checking; general Flash LSO forensic method
applies.

**The panel project.** `ROMDISK/user/Display/*.vtz` is the compiled panel project —
the interface definition. It tells you what buttons existed, what they were
labelled and what they were wired to, which is what makes panel-driven events
interpretable. The examined panel had one in deleted state, alongside a deleted
`Program Files/ProjectBackup` directory. Panels also run their own program slot
(`Simpl/App00/`, with `ProgramInfo.config`).

**Configuration.** `Core3config.ini` holds panel configuration. `Sys/hwConfig.ini`,
`Sys/~.iptable0.dip`, `Sys/AuditLog/` and `Sys/AutoUpdate/` mirror the processor.
The panel also carried both RSA **and** DSA SSH host keys under `Sys/SSH/`, and a
deleted `sshd_config`.

### 7.8 4-Series processors (Linux) ###

Verified from a CP4 acquisition. The 4-Series is a different machine from the
3-Series and almost nothing in section 7.1–7.6 transfers. It is an ARM Linux system
(a BusyBox userland on a Linux `armhf` kernel), so treat it as a Linux exhibit and
use ordinary Linux forensic tooling throughout.

**Storage is a partitioned card with a normal MBR**, unlike the 3-Series raw NAND
dump. Boot media on the examined device was an 8 GB microSD. The layout is roughly a
dozen partitions, ext3/ext4, and it is deliberately **mirrored**: paired boot
volumes (`sys1` and its mirror `sys`) and paired root volumes (`rootfs` and its
mirror `rootfs1`), then `sdcard`, `data`, `var`, `varlog` and `persist`. The
mirroring means **two system images are on the device, one of them the previous
version** — worth diffing, because it can show what changed and when. Note that
Crestron content lives on the *second* of each mirrored pair on the running system
(`rootfs1`), so check both members rather than assuming the first is authoritative.

**Logs have their own partition**, `varlog`, mounted at `/logs`:

```
/logs/CurrentBoot/Crestron_00.log
/logs/PreviousBoot/Crestron_00.log
/logs/ZippedLogs/plog_<date>_<time>.tar.gz
/logs/MsgLog.txt
/logs/reboot_pid.log
/logs/sdcardWriteCounter.log
/logs/diskFullnessAtReboot.log
/logs/deviceTable.txt
/logs/ramdiskLogs/
```

The `CurrentBoot` / `PreviousBoot` / `ZippedLogs` pattern carries over from the
3-Series, but archives are `.tar.gz` and lowercase `plog_`. The examined device held
**37 archives across five months.**

Three of these have no 3-Series equivalent and are worth going straight to:

- **`reboot_pid.log`** — records the process associated with a restart. This is the
  most useful single addition in the 4-Series: it attacks the attribution problem in
  section 8.3 directly, potentially naming what initiated a reboot rather than
  leaving you with "software-initiated."
- **`sdcardWriteCounter.log`** — a running flash write count. Bears on how hard the
  device has been overwriting itself, and on storage-wear questions.
- **`diskFullnessAtReboot.log`** — storage pressure at restart, which bears on how
  aggressively logs were being discarded.

**`ramdiskLogs/` was empty on the acquired image, and that is expected** — the name
and the device's mount configuration indicate RAM backing, so anything buffered
there is lost on power-down. This does not change the
advice in section 1: the flash-resident history is vastly larger, and a ramdisk
buffer is lost on any restart anyway. But if you are doing live collection under
section 5.3, capture before you power down.

**Other partitions:**

| Path | Contains |
|---|---|
| `/data/crestron/registry/HKEY_LOCAL_MACHINE/Crestron/System/` | The registry survives as a **directory tree**, not a hive file — browsable directly |
| `/data/crestron/config/rc.conf` | Device configuration |
| `/data/crestron/config/errLogServer.conf` | Error-log server settings — check for off-box log forwarding (section 9) |
| `/data/crestron/hostname` | Device hostname, often site-derived |
| `/data/Authen/`, `/data/ssh/` | Authentication and SSH state |
| `/data/coredump/` | Crash dumps |
| `/data/datastore.bin`, `/data/crestdata.tab` | Data stores |
| `/data/timezone`, `/data/localtime` | **Time configuration — read this before interpreting any timestamp** |
| `/data2` (`persist`) | Persistent data surviving updates |
| `/mnt/sdcard/crestron/program01` … `program10` | The ten program slots |
| `/mnt/sdcard/ROMDISK/` | Firmware packages, certificates, update logs and archives |
| `/mnt/sdcard/webtmp/` | **Web-server upload temporaries** — program and firmware uploads traverse the web interface and leave recoverable remnants here |
| `/var/log/` | Standard Linux logs |

The examined `sdcard` volume had **237 deleted entries out of 417** — including all
ten program-slot directories — so carving is likely to be productive on 4-Series.

**The 4-Series keeps a Redis key-value store, and it is a primary artifact.**
`/data/cresstore/` holds a Redis database dump (`dump_<n>.rdb`) that stores registry
settings, certificate material and configuration URLs as key-value pairs. It is a
plain on-disk database file: load it into any Redis-compatible reader, or carve
strings from it directly, and search it for `http`/`https` URLs. Two things in
particular live here — the certificates the device trusts, and the update-catalog
URL the auto-updater fetches (see below) — so this single file can establish what
the device trusted and where it phoned for updates. The registry is *also* browsable
as a directory tree under
`/data/crestron/registry/HKEY_LOCAL_MACHINE/Crestron/System/`; the Redis store and
the tree overlap, so check both.

**Startup and service configuration is readable as text.** The device is
systemd-based, and the unit files under `/lib/systemd/system/` (on the root volume)
enumerate the Crestron daemons by function — among them an **audit-log service**, an
**error-log server** (its config is the `errLogServer.conf` named above), an
**IP-blocker service** (a blocklist records addresses blocked after failed logins),
an **account-management service**, an **NVRAM service**, the **console listener**
(`ctpd`) and the **auto-updater**. Reading these units tells you which logging and
security services *should* have been running, which is a useful cross-check against
what you actually recovered — a service whose logs are absent is worth a question.

**Auto-update leaves a trail.** The Redis store holds an update-catalog URL, and
`/mnt/sdcard/ROMDISK/` plus the update logs hold the update plugins and artifacts.
Together they evidence the device's firmware history and its network update source —
corroborate
against the network logs in section 9.

**Built-in accounts on 4-Series.** The Linux password file names Crestron service
accounts (a `crestron` account and a `userprogs` account); on the examined device
their `shadow` entries were empty or disabled, meaning those accounts are not
password-authenticated in the ordinary sense — access is gated by other means. As in
section 7.4, treat the possibility of standing vendor accounts as live on 4-Series
too: the forensic point is that such accounts exist and that activity under them is
vendor-level, distinguishable from site activity. This document does not publish
account names or credentials.

The web-server upload temporaries deserve particular attention: dozens were present
in deleted state. Because program loads and firmware pushes traverse the web
interface, these can evidence *what was loaded and when*, even where the loaded
artifact is gone.

**Partition timestamps disagreed usefully** on the examined device: most volumes
last written on one date, the `persist` volume six months later. Volume-level mount
and write times are worth recording per partition rather than treating the device as
having a single "last used" time.

### 7.9 Android-based devices (newer panels and DSPs) ###

**[unverified] throughout.** No Android device acquisition was examined for this
document. This section says what to expect and which method to apply. Treat all of
it as leads to confirm on your own exhibit.

The newer touch panels and the DSPs run Android — which is why some Crestron panel
software ships as an APK, and why `DeviceOperatingSystems.md` lists Android 4.x and
5.x against those models. The practical consequence is the important part:

**Expect these to behave as ordinary Android devices wearing a Crestron badge** —
vendor applications on top of a stock platform. If that holds on your exhibit,
**your normal Android forensic method applies end to end** — the same tooling, the
same artifact locations, the same interpretation. Confirm the platform build on the
device before committing to that approach, but do not assume you are facing an
exotic embedded target requiring special technique — treat it as an Android handset
that happens to be screwed to a wall.

**Artifact classes worth pulling**, in rough order of investigative value:

| Artifact | Why |
|---|---|
| **Bluetooth pairing records** | Device names of phones that paired with the unit. See the note below — this is the significant one |
| Per-application databases under the standard app-data paths | Vendor applications store state here, alongside everything else |
| Wi-Fi configuration and known-network history | Places the device, and sometimes places other devices |
| Browser history and cache | Panels fetch feeds and may host a browser |
| Download and media providers | Ordinary Android artifacts on a device nobody thinks to seize |
| Calendar and contacts providers | Present in the platform; may be populated on scheduling panels |
| Crash traces and tombstones | Fault history, and memory contents at failure |
| Account records | What services the device was signed in to |

**Bluetooth pairings are a genuine exception to section 8.5.** Elsewhere this
document is emphatic that a control system tells you a command was issued, not who
issued it, and that occupancy sensing detects presence rather than identity. An
Android device that pairs with phones is different: pairing records carry
user-assigned device names, which are frequently a person's name. That is
identity-adjacent evidence of a specific handset having been in Bluetooth range.
Corroborate it — a pairing record is not proof a person was present at a given
moment, and names are trivially editable — but do not overlook it, and do not repeat
the "these systems cannot identify people" line without checking.

**DSPs deserve separate thought.** The DSP is the device in the audio path. If any
part of the system touches microphones, conferencing or recording, the wiretap and
eavesdropping considerations in section 11 apply with full force, and they are
materially different from the rules governing stored records. Establish the legal
position before collection, not after. A DSP may also carry evidence of call and
conference activity — when audio paths were active, and between which endpoints —
which is often the question actually being asked.

**Acquisition notes.** These are embedded devices, so the usual Android acquisition
route may not be open: developer options and USB debugging are typically not
enabled, there may be no accessible recovery, and physical acquisition of the eMMC
may be the only path. Establish the device's encryption state early, because it
determines whether a physical acquisition yields anything readable. Where the device
can be treated as a normal Android target, prefer that; where it cannot, fall back
to the physical methods in section 5.2.

---

## 8. Interpreting the evidence ##

This is where cases are won and lost. The artifacts are easy to collect and easy to
misread.

### 8.1 Timestamps: two formats ###

Crestron changed log timestamp format between firmware generations. Both appear in
the wild, and **both can appear in one evidence set** if the device was updated.

```
Newer:   2017-08-04 15:22:24        year-month-day, then time
Older:   17:34:47  3-17-2016        TIME FIRST, then month-day-year
```

The older format leads with the time and uses a US-style date. **A naive
lexicographic sort will scramble it**, and a parser expecting ISO order will either
fail or, worse, silently misparse. Normalize deliberately, and state in your notes
which format each source file used.

### 8.2 Time is not trustworthy until you prove it ###

Before you assert that anything happened at a particular moment, establish what the
device thought time was and why.

- **The real-time clock is battery-backed.** 3-Series processors use a DS3231 RTC
  part. A dead or failing battery means a wrong or reset clock. This repository's
  `BatteryReplacements.md` lists the battery by device — relevant here because a
  device that has been in service a decade may well have a dead cell.
  This is not theoretical. The examined TSW-750 held ten archived log bundles
  stamped `2006-01-01 12:00:3x`, seconds apart — a device booting repeatedly with
  no valid clock and falling back to a default date, years before the surrounding
  archives. **Timestamps clustered on an implausible date seconds apart are a
  dead-clock signature, not evidence of activity on that date.**
- **`SNTP`** may or may not be configured. If it is, timestamps are as good as the
  time source and the network. If not, drift accumulates freely.
- **`TIMEZONE` and `TIMEDATE`** are per-device settings. Two processors in one
  building can disagree.
- A device that lost power and came back with no RTC and no SNTP may start counting
  from an epoch or a build date, producing timestamps that are internally
  consistent and completely wrong.

**Always record the device's own reported time against a trusted reference at the
moment of seizure.** It is the only way to compute the offset afterward, and you
cannot go back for it.

### 8.3 Reboots: what they do and do not mean ###

A restart is one of the most common events you will see, and one of the most
frequently overread.

The device stores a **reboot reason code in NVRAM**, retrievable with
`NVRAMREBOOT SHOW`, but the number alone is not a finding. On 3-Series equipment the command prints the value
together with a short description, so **read the device's own printed text and
record both** — the raw value and the description, verbatim.

The values below were seen in the course of examination. They are a starting point
for recognizing what you find, nothing more: the correspondence is not guaranteed
across models or firmware generations, and the list is certainly incomplete.

| Value | Text the device printed | Reading |
|---|---|---|
| 0 | power cycle | Plug pulled, outage, or PDU |
| 2 | application hung | Watchdog — software fault, not a person |
| 3 | soft reset | Commanded soft reset |
| 4 | restore defaults | **Configuration destroyed** |

Other values occur, including large values in the tens of thousands, and small
values can be ambiguous — the same integer may be reported in different
circumstances, and the meaning can shift by model and firmware. **Do not interpret a
bare number.** Record the raw value and the device's printed description; if the
device prints no description for the value you recover, report it as an uninterpreted
raw value rather than mapping it from another exhibit.

The one distinction worth holding onto is that a **power cycle reports differently
from a commanded restart** — which lets you separate a pulled plug from a deliberate
reboot rather than guessing from absence.

Now the pitfalls:

**A `User Reboot` line does not mean a user was present.** It means the restart was
software-initiated rather than a power event. That can be a technician typing
`REBOOT` at a console, a program restarting the system, or a remote/cloud-initiated
restart with no operator present. On the exhibits examined, `User Reboot` lines
appeared with no corresponding console connection logged, so the line does not by
itself indicate that anyone was connected.

**On 4-Series, check `/logs/reboot_pid.log` before concluding anything.** It
records the process associated with the restart, which may let you name what
initiated it rather than stopping at "software-initiated." The 3-Series has no
equivalent.

**Therefore: absence of a console connection before a reboot proves nothing.** Do
not reason "no CTP connection is logged, so nobody did this." Check the reason code
and the cloud enrolment instead.

**Most real restarts are power cycles.** This equipment is well known for needing
them, and installers routinely fit power-management hardware — a PDU, a timer, an
IP outlet strip — that power-cycles devices overnight as a matter of course.

**A regular nightly restart at a fixed time is automation, not occupancy.** If you
see a device restarting at 03:00 every night, that is almost certainly a scheduled
power cycle. Reading it as human activity would be a serious error, and it is
exactly the kind of error that a competent opposing expert will find.

### 8.4 Scheduled or user-driven? ###

This is usually the central question — did a person do this, or did the building do
it on a timer?

Work through it in this order:

1. **Check `Sys/TimerEvents/`.** If an event matches a scheduled event, that is your
   answer.
2. **Look for cadence.** Events on an exact interval, or at the same clock time
   daily, or aligned to business hours, are automation. Human activity is irregular.
3. **Look for the trigger, not the effect.** A lighting change logged at 18:00 sharp
   every weekday is a schedule. The same change at 18:37 on one Tuesday is not.
4. **Correlate with an independent occupancy signal** — badge records, alarm
   arm/disarm, network authentication.
5. **Read the control program.** It defines which events are schedule-driven and
   which are exposed to a user interface at all. Some functions simply cannot be
   user-triggered on a given site.

Astronomical schedules deserve special care: lighting is frequently driven by
sunrise/sunset offsets, so the trigger time **drifts across the year** while still
being pure automation. A time that shifts by a minute or two per day is a
sun-tracking schedule, not a person becoming gradually more punctual.

### 8.5 Attribution and its limits ###

What you can often show: that a console session came from a given source IP and
port at a given time; that an authenticated action occurred, if audit logging was
enabled; that a program was loaded or restarted.

What you generally **cannot** show from the device alone:

- **Which human being** was at the keyboard. IP addresses identify hosts, and
  Crestron systems are frequently accessed from shared technician laptops and
  shared credentials.
- **Whether a physical event actually occurred.** The system logs the command it
  issued, not the outcome.
- **Who was in the room.** Occupancy sensing detects motion or presence, not
  identity, and is easily triggered by cleaning staff, pets or HVAC airflow.
  *Partial exception:* Android-based panels and DSPs may hold Bluetooth pairing
  records naming specific handsets — see section 7.9.
- **That nothing happened.** Absence of a log entry is weak evidence. Logging depends
  on how the site was programmed, audit logging is off by default, and rotation
  destroys history routinely.

State these limits in your report before someone else states them for you.

---

## 9. Off-device corroboration ##

Assume the device is one of several records, and often not the most complete.

- **Remote syslog.** `REMOTESYSLOG` can forward messages, and with `-A` the audit
  log, to an external server. If it was configured, **a copy of the evidence may
  exist off-box and beyond the reach of anyone who wiped the device.** Check
  `Sys/` configuration and the device's syslog settings early.
- **Cloud enrolment and telemetry.** `Sys/MyCrestron Configuration.txt` records
  cloud service enrolment. Beyond enrolment, 3-Series processors periodically beacon
  to Crestron cloud hosts — pushing system and program details, and querying for the
  site's current public IP address. That activity is visible in network logs, DNS
  query records and firewall/NetFlow data even when nothing was captured on the
  device itself, so a site's own network infrastructure may show the device phoning
  home on a schedule. Where a device is enrolled, the provider may also hold
  connection history, update history and remote-command records — obtainable by
  legal process, and potentially surviving the device's own destruction.
- **Third-party monitoring.** Many integrators run their own or third-party
  monitoring platforms with their own logs and their own remote-control ability.
  The site labels and project filenames tell you which integrator to ask.
- **The integrator's records.** The source program, commissioning notes, job files
  and service tickets. The compiled program on the device is far easier to
  interpret with the source project alongside it.
- **Network infrastructure.** DHCP leases, switch MAC tables and NetFlow place the
  device on the network and may capture sessions to it. The device's SSH host key
  ties captured sessions to that specific unit.
- **The rest of the control system.** The IP table (`Sys/~.iptable0.dip`) names the
  peers. Touch panels, DM endpoints and other processors hold their own records.

---

## 10. Anti-forensics and evidence destruction ##

Both deliberate and routine. Watch for:

| Mechanism | Signature |
|---|---|
| `CLEARAUDITLOG` | Audit log empty or starting abruptly; gap against other sources |
| `RESTORE` / factory defaults | Reboot reason describing a restore-to-defaults |
| Format | Reboot reason describing a format |
| Firmware reflash | `AutoUpdateLogs/`, `pufversion.ini`, a firmware change in the boot lines |
| Reducing log retention | `LOGGERNUMBACKUPLOGS` set unusually low |
| Disabling audit logging | `AUDITLOGGING OFF` |
| Repeated reboots | Persistent log window pushed out; `PreviousBoot` overwritten |
| Card swap | Card that does not match the device's age, wear or labeling |

Two observations worth carrying into a report:

**Routine maintenance is indistinguishable from cleanup unless you look at reason
codes and timing.** A firmware update days after an incident may be entirely
innocent. Establish whether it was scheduled, who performed it, and whether it was
consistent with prior practice.

**Repeated reboots are the most effective and least suspicious way to destroy this
evidence.** Nobody has to delete anything. Two restarts empty the persistent log
window. If you see an unexplained cluster of restarts shortly after the period of
interest, that is worth pursuing.

---

## 11. Legal and evidentiary considerations ##

Not legal advice. Jurisdiction-specific; consult counsel. **[unverified]** as to any
particular jurisdiction.

**Authority.** Establish who owns and controls the device before seizing it. In
commercial buildings the processor may belong to the landlord, the tenant, or the
integrator, and may be under a service contract giving a third party remote access.
In residential settings a control system may cover shared and private spaces, and
may hold data about people who are not subjects of the investigation. Consent from
one occupant may not cover all of it.

**Scope.** These systems can be far more revealing than they appear. Occupancy,
movement between rooms, sleep and waking patterns, and presence or absence over
months may be reconstructable. That breadth cuts both ways: it makes the evidence
valuable, and it makes an overbroad seizure a genuine privacy problem. Frame
warrants and requests to the question actually being asked.

**Audio and video.** If any part of the system touches microphones, cameras or
conferencing, wiretap and eavesdropping statutes may apply, with materially
different rules from those governing stored records. Get this cleared before
collection, not after.

**Chain of custody.** Standard practice applies. Note especially that these devices
are often physically accessible to many people — contractors, cleaners, facilities
staff — so document physical security at the site, not just after seizure.

**Preservation.** See section 3. A generic litigation hold will not preserve this
evidence. The hold must direct that processors be powered down and imaged, and it
must reach the integrator.

**What an opposing expert will attack**, in roughly the order they will try:

1. **The clock.** Unverified time source, dead RTC, no SNTP, timezone confusion,
   the two timestamp formats. This is the most productive line of attack and the
   easiest to defend against with contemporaneous notes.
2. **Attribution.** Shared credentials, shared technician laptops, remote access by
   the integrator or cloud service, no audit logging.
3. **Scheduled versus human.** Precisely the confusion described in section 8.4.
4. **The gap between command and physical reality.** The log says the system
   commanded a lock to open; that is not the same as the door opening.
5. **Preservation conduct.** Whether you booted the device, connected it to a
   network, or let site staff pull logs first.
6. **Completeness.** Rotation and session-scoped logs mean you have a window, not a
   history. Be explicit about the window's edges.

**Reporting.** State what the device recorded, what that means mechanically, and
what it does not establish, as three separate things. The commonest failure in this
material is presenting a device's internal record of its own commands as if it were
an observation of the physical world.

---

## 12. Quick reference ##

### Console commands ###

| Command | Purpose |
|---|---|
| `NVRAMREBOOT [SHOW]` | Display the last reboot reason stored in NVRAM |
| `PLOGCURRENT` / `PLOGPREVIOUS` / `PLOGALL` | Persistent log, current / previous / both sessions |
| `ERRLOG {OK\|NOTICE\|WARNING\|ERROR\|FATAL}` | Message log at severity |
| `AUDITLOGGING [ON\|OFF] {ALL\|...}` | Configure audit logging (off by default) |
| `GETAUDITLOG` / `PRINTAUDITLOG [ALL]` | Retrieve audit log |
| `REMOTESYSLOG -S: -A -I:<addr> -P:<port>` | Remote syslog, `-A` includes audit log |
| `LOGGERNUMBACKUPLOGS[:program#] {N}` | Number of backup log files retained |
| `UPTIME` / `PROGUPTIME[:program#]` | System / program uptime |
| `TIMEDATE` / `TIMEZONE` / `SNTP` | Device time, zone, network time sync |
| `WHO` / `WHOAMI` / `IPIDWHO` | Sessions and CIP peers |
| `TASKSTAT` / `SSPTASKS` | Running tasks |

**Destructive — never on an exhibit:** `CLEARAUDITLOG`, `RESTORE`,
`PLATFORMDATARESTORE`, `REBOOT`, `FORCEDREBOOT`, `REBOOTOPTION`, `ERASE`.

### Paths (3-Series / Windows CE, verified on MC3) ###

| Path | Contains |
|---|---|
| `Sys/PLog/CurrentBoot/Crestron_NN.log` | Current session message logs |
| `Sys/PLog/PreviousBoot/Crestron_NN.log` | Previous session message logs |
| `Sys/PLog/ZippedLogs/PLog_<date>_<time>.zip` | Archived logs — extends history |
| `Sys/AuditLog/` | Audit log |
| `Sys/TimerEvents/` | Scheduled events |
| `Sys/Authentication/` | Authentication data |
| `Sys/~.iptable0.dip` | CIP/IP peer table |
| `Sys/MyCrestron Configuration.txt` | Cloud enrolment |
| `Sys/Setup Configuration.txt` / `Sys/hwConfig.ini` | Device and hardware config |
| `Sys/SSH/ssh_host_rsa_key` | Device SSH host key |
| `Sys/AutoUpdate/`, `AutoUpdateLogs/`, `Sys/pufversion.ini` | Update history |
| `Simpl/AppNN/` | Control program slots |
| `Nvram/` | Non-volatile app data, project files |
| `Documents and Settings/System.hv`, `default/user.hv` | Windows CE registry hives |
| `Recycled/`, `User/`, `FTP/`, `Temp/` | User-writable and deleted-item areas |

### Paths (Windows CE touch panels, verified on TSW-750) ###

Panel differences from the processor layout above:

| Path | Contains |
|---|---|
| `Sys/PLog/ZippedLogs/` | Archived logs — on the examined panel, 4+ years of them |
| `MsgLog.txt` / `MsgLogPrev.txt` | Panel message log, current and previous |
| `Sys/DumpFiles/`, `Sys/ExtraDumpFiles/` | Crash dumps (no processor equivalent) |
| `Windows/Registry/system.hv`, `default/user.hv` | Registry hives — **different location** from the processor |
| `Windows/Profiles/guest/Temporary Internet Files/Content.IE5/` | Web cache, incl. deleted cached feeds |
| `Application Data/Macromedia/Flash Player/#SharedObjects/` | Flash local shared objects |
| `Application Data/Adobe/Flash Player/AssetCache/` | Flash asset cache |
| `Application Data/Macromedia/Flash Player/Logs/flashlog.txt` | Flash log |
| `ROMDISK/user/Display/*.vtz` | Compiled panel project — the interface definition |
| `Core3config.ini` | Panel configuration |
| `Sys/SSH/` | RSA **and** DSA host keys on panels |

Storage note: the examined panel had **two** filesystems — a near-empty FAT32
volume and the main exFAT volume, the latter starting one sector past a round
offset. Scan for volumes; do not assume one, and do not assume alignment.

### Paths (4-Series, Linux — verified on CP4) ###

| Path | Contains |
|---|---|
| `/logs/CurrentBoot/`, `/logs/PreviousBoot/` | Session message logs |
| `/logs/ZippedLogs/plog_<date>_<time>.tar.gz` | Archived logs |
| `/logs/reboot_pid.log` | **Process associated with a restart** |
| `/logs/sdcardWriteCounter.log`, `/logs/diskFullnessAtReboot.log` | Flash wear, storage pressure |
| `/logs/ramdiskLogs/` | RAM-backed — lost on power-down |
| `/data/cresstore/dump_<n>.rdb` | Redis key-value store — registry, certificates, config/update URLs |
| `/data/crestron/registry/HKEY_LOCAL_MACHINE/Crestron/System/` | Registry as a directory tree (overlaps the Redis store) |
| `/lib/systemd/system/` | systemd units — enumerates Crestron logging/security daemons |
| `/data/crestron/config/rc.conf`, `errLogServer.conf` | Config, off-box log forwarding |
| `/data/timezone`, `/data/localtime` | Time configuration |
| `/data/coredump/`, `/data/Authen/`, `/data/ssh/` | Dumps, auth, SSH state |
| `/mnt/sdcard/crestron/program01`…`program10` | Program slots |
| `/mnt/sdcard/webtmp/` | lighttpd upload remnants |
| `/mnt/sdcard/ROMDISK/` | Firmware, certs, update logs |
| `/var/log/` | Standard Linux logs |

Storage note: partitioned eMMC with an MBR, ext3/ext4, paired A/B boot and root
volumes — the previous system image is still on the device.

### Network ###

| Item | Value |
|---|---|
| MAC OUI | `00:10:7F` |
| CIP | 41794/tcp, 41794/udp |
| CTP console | 41795/tcp |
| Others | 21 FTP, 22 SSH, 23 Telnet, 80 HTTP, 443 HTTPS |

---

## Corrections and contributions ##

This document is grounded in device acquisitions: an MC3 processor and a TSW-750
touch panel (3-Series, Windows CE), and a CP4 processor (4-Series, Linux). Coverage
of Android-based panels and DSPs is thinner and marked `[unverified]` where it is
inference rather than observation. Corrections, additions and pull requests
are welcome — particularly verified artifact paths from platforms other than the
ones examined here.
