# Reading Crestron Device Logs #

A device log is the closest thing a control system has to a flight recorder. The program on the
processor tells you what the system was *supposed* to do. The log tells you what it actually did:
- what booted, and in what order
- what each device was told and whether it answered
- which modules choked on their input
- when the network went away
- who logged in, and from where

Service calls should usually start with the log -- but be a professional -- capture the logs to your 
service laptop, to an appropriate client folder, so that further analysis can be done later.

This primer is about reading logs as an engineer: what is in them, how the lines are put together,
what the common messages mean, and how to get answers out of a few hundred thousand lines quickly.
[Forensics.md](Forensics.md) covers the evidence side: preservation, chain of custody, and what to
capture, preferably from a proper forensics copy of the device's SDCARDs.

## Contents ##

1. [Why reading logs matters](#1-why-reading-logs-matters)
2. [Where the logs live and how they are named](#2-where-the-logs-live-and-how-they-are-named)
   - [2.1 3-Series (Windows CE)](#21-3-series-windows-ce)
   - [2.2 2-Series](#22-2-series)
   - [2.3 Toolbox system information](#23-toolbox-system-information)
   - [2.4 4-Series (Linux)](#24-4-series-linux)
   - [2.5 Finding logs in a pile of project files](#25-finding-logs-in-a-pile-of-project-files)
3. [Anatomy of a line](#3-anatomy-of-a-line)
   - [3.1 3-Series persistent log](#31-3-series-persistent-log)
   - [3.2 2-Series ERRLOG](#32-2-series-errlog)
4. [The cast: who writes what](#4-the-cast-who-writes-what)
5. [The rhythm of a boot](#5-the-rhythm-of-a-boot)
6. [Messages worth knowing](#6-messages-worth-knowing)
   - [6.1 The network and devices](#61-the-network-and-devices)
   - [6.2 The program under strain](#62-the-program-under-strain)
   - [6.3 SIMPL+ and SIMPL# module errors](#63-simpl-and-simpl-module-errors)
   - [6.4 Crashes at the operating-system level](#64-crashes-at-the-operating-system-level)
   - [6.5 Who was on the console](#65-who-was-on-the-console)
   - [6.6 When the log stops talking](#66-when-the-log-stops-talking)
7. [A working method](#7-a-working-method)

---

## 1. Why reading logs matters ##

- **Programs fail quietly.** A SIMPL module that overflows its buffer does not stop the program. It
  drops data and writes one line to the log, possibly thousands of times. A module stuck in a
  reconnect loop can fill a log with connect/disconnect pairs while nobody notices, because the
  device still mostly works.
- **Load causes are visible.** Logic that never settles, queue overflows, pipe-full messages and
  watchdog warnings show a processor under strain long before anyone sees a symptom.
- **Program's can continue working while functionality is dropped.** Crestron devices are known for
  continuing to run a program slot that is only partially functioning. The logs often provide indicators
  of why this happened.
- **The log is the only record of timing.** "The lights went off at 9:14" can be checked: a program
  restart, a power-on or a device dropping offline at 09:14 is either in the log or it is not.
- **It tells you what is really installed.** Model, firmware version, program slots, which devices
  checked in, which IP-IDs have no IP table entry. That is often more accurate than the
  documentation.
- **Your own code shows up in it.** Every unhandled SIMPL# exception arrives with a stack trace.

---

## 2. Where the logs live and how they are named ##

### 2.1 3-Series (Windows CE) ###

```
\Sys\PLog\CurrentBoot\Crestron_00.log ... Crestron_09.log    this boot session, _00 newest
\Sys\PLog\PreviousBoot\Crestron_00.log ...                    the previous boot session
\Sys\PLog\ZippedLogs\PLog_YYYY-MM-DD_HH-MM-SS.zip             older sessions, one zip each
```

The zipped bundles are the long memory. Each holds the `Crestron_NN.log` files of one archived
session, and the zip name is its timestamp. Touch panels keep the same structure, plus
`MsgLog.txt`, `MsgLogPrev.txt` and `flashlog.txt`.

### 2.2 2-Series ###

2-Series logs were captured from the console with `ERRLOG` and saved as
`ERRLOG_YY-MM-DD_H-MM-SS.TXT`, which is the naming of a Toolbox capture.

### 2.3 Toolbox system information ###

"System Info" in Toolbox saves a text dump as `*CrestronSysInfo*.txt` (often renamed with the model
or site). It is not a log, but it is the best companion to one. Its sections are:
- `[Product Info]` and `[Hardware Info]`
- `[Firmware Capabilities]` and `[Card Info]`
- `[Ethernet]` and `[IP Table]`
- `[Cresnet Devices]`
- `[System Clock]`
- `[Internal Memory Usage]` and `[Compact Flash Usage]`
- `[Error Log]`: a copy of the current log
- `[EBoot|OS|Updater.Product Info]`
- `[ProgramNN.Program]`, plus the IP table, hardware and card info for every program slot

### 2.4 4-Series (Linux) ###

The 4-Series moved to Linux, but kept the console vocabulary. It still answers `ERRLOG {SYS |
PLOGCURRENT | PLOGPREVIOUS | PLOGALL | NOTICE | WARNING | ERROR | FATAL}` and adds `ERRLOG LASTHOURS
<1-99>`. The persistent logs live under `/logs/`, and log bundles are saved there as `.tgz` files.
`csstartup.log` records the reboot reason (`User Reboot`, `Watchdog Reboot`, `Power On`, ...) at
every boot.

### 2.5 Finding logs in a pile of project files ###

Look for these names inside folders and inside zips:

| Pattern | What it is |
|---|---|
| `PLog_####-##-##_##-##-##.zip` | Archived 3-Series persistent-log session |
| `Crestron_##.log` | Persistent-log file (rotated 00 to 09) |
| `ERRLOG_*.TXT` | 2-Series console error log saved by Toolbox |
| `*CrestronSysInfo*.txt` | Toolbox system information dump |
| `MsgLog.txt`, `MsgLogPrev.txt`, `flashlog.txt` | Touch-panel logs |
| `csstartup.log`, `/logs/*.tgz` | 4-Series boot record and log bundles |

---

## 3. Anatomy of a line ##

### 3.1 3-Series persistent log ###

```
Notice: LogicEngine.exe [App 1] # 14:05:12  6-02-2020  # **Program 1 Started**
^^^^^^  ^^^^^^^^^^^^^^^ ^^^^^^^   ^^^^^^^^  ^^^^^^^^^    ^^^^^^^^^^^^^^^^^^^^^
severity process        slot      time      date M-D-Y   message
```

- **Severity:**
  - `Info`, `Notice`, `Warning`, `Error` and `Fatal`.
  - `Ok` (a status line) and `Debug` when enabled.
  - Error lines are not rare, and on their own they mean little; how often they repeat and what
    comes next is what matters.
- **Process:** the program that wrote the line, not necessarily the one at fault (see section 4).
- **`[App N]`:** the program slot (1 to 10). The same message from two slots is two problems.
- **Time and date:** local time, as the processor understood it at that moment. A clock that was
  wrong before NTP corrected it gives impossible orderings. Look for `TLDM.exe … Time Change event
  received` / `System Time Updated`: those mark the point where the clock was changed.

Not every line carries that header. Some lines are **untagged**: kernel and driver output
(`setIP: Registry static IP setting = …`, USB hub chatter, `Ethernet auto-negotiated 100Mbps, Full
Duplex`), Windows CE exception dumps, and the continuation lines of .NET stack traces. Treat an
untagged line as belonging to the nearest tagged line before it.

**Lines can interleave.** Two processes writing at once can splice into each other. Yes! err may or
may not lock your device up but there's nothing like a properly written logging API:

```
at CrestError: splusmanagerapp.exe [App 1] # 10:22:31  6-02-2020  # ron.SimplSharp.CrestronIO...

```

That is "Crestron.SimplSharp…" from one stack trace with another process's header dropped in the
middle. When a line looks garbled, look for a second `Severity: process #` inside it.

### 3.2 2-Series ERRLOG ###

Two physical lines per event:

```
Warning: Could not solve logic within 1000 waves. (0004:0000:0004)
     TimeStamp: 14:05:12  6-02-20   UpTime: 0 days 14:32:39.60  Task: LogicPro
```

`UpTime` is the 2-Series' great gift: it survives clock errors and shows directly whether a reboot
happened between two events. `Task` is the internal task that wrote the event, for example
`LogicPro` (logic solver), `CRHostH` (Cresnet host), `SchedHel` (scheduler), `TCP_Task` and
`TCP_Read` (CTP/console network) and `CONSOLE`. The bracketed triple is an internal error code.

---

## 4. The cast: who writes what ##

The 3-Series is Windows CE with a set of cooperating Crestron processes. These processes write
most of what the log shows:

| Process | Role | What it logs |
|---|---|---|
| `nk.exe` | The CE kernel image (all built-in drivers) | Boot reason, `System startup: <MODEL> [v<firmware> (<build date>) #<id>]`, DHCP, USB, watchdog (`WDG:`) |
| `CRESLOG.exe` | The persistent-log writer itself | `Logging is started!`, `System Reboot Notification`, and **suspension** (below) |
| `TLDM.exe` | Top-level device manager: starts and stops programs, talks to the I/O processor | `TLDM Starting..`, `THAL loading now ..`, `Program N Stopped`, time changes, heartbeat/IOP faults |
| `LogicEngine.exe [App N]` | The SIMPL logic engine, one per slot | Program load, symbol init, steady state, device update requests, IP table problems, pipe-full |
| `splusmanagerapp.exe [App N]` | Hosts SIMPL+ and SIMPL# modules | Module runtime errors (`Module S-1.2.3 : Name at line N: …`), TCP client activity, .NET exceptions |
| `CIPCommandProcessor.exe` | CIP/Ethernet device and gateway server | Devices offline/back online, gateway connects, TCP client failures, SSL, link loss |
| `ConsoleServiceCE.exe` | Console (CTP, Telnet, SSH shell) | `CTP/TELNET/SHELL Connection from: <ip>`, `Another console session is open` |
| `SimplSharpPro.exe` | SIMPL# Pro host | `Starting Programs...!`, server socket status |
| `SSHD.exe`, `TFTPServer.exe`, `CloudClient.exe`, `CrestronMonitor.exe`, `CrestronTimerEventEngine.exe`, `RouterTransportProcess.exe` | Services | Their own lifecycle and errors |

`nk.exe`, the SIMPL+ manager and the CIP processor usually write the bulk of the lines. That is
why a busy third-party module can bury everything else.

---

## 5. The rhythm of a boot ##

A healthy 3-Series boot reads like this:

1. **Why it restarted:** `nk.exe` `Power On` (power was lost), `User Reboot` (someone typed REBOOT
   or Toolbox did it) or `External Warm Reset` (a hardware reset, button or watchdog).
   `IOCTL_HWDOG_ACT_REBOOT: reason 0x…` is the hardware watchdog's own reason code.
2. `System startup: <MODEL> [v<firmware> (<build date>) #...]`: the model, firmware and build date.
   This is the line to search first.
3. `CRESLOG` `PersistentLog: Logging is started!`, then `TLDM Starting..` and `THAL loading now ..`.
4. Network: static IP from the registry or DHCP options, then `Start a clear/secure TCP gateway server`.
5. Per slot: `Loading program from \SIMPL\app01` → `Initializing Symbols` → `InitializeSymbols
   Done` → `SendInitDoneToSPlusManager Done` → `Going to steady state now` → `**Program 1
   Started**`.
6. Devices check in: `Update request from Slot-01.IP-ID-03 in Program 01`, gateway connections and
   panels.

Anything missing or out of order in that sequence is the finding.

A program that **stops** logs its own shutdown in detail: `Stopping Program.`, `BREAKING FOR
PAUSE/STOP @ N ticks`, `begin/end TLDM_TO_APP_STOP_MESSAGE`, `~CSimplPlus Unloading S+`, then TLDM's
`**Program N Stopped**`. A stop without that sequence was not a clean stop.

---

## 6. Messages worth knowing ##

Grouped by what they tell you.

### 6.1 The network and devices ###

| Message | Meaning |
|---|---|
| `Ethernet Device program N CIPID A5 : Not Responding (IP = …). Device Offline` / `CIP Device A5: Back Online` | An Ethernet device (panel, expander, gateway) dropped and came back. Pair them up: the gaps between them are the outage. Thousands of pairs for a single IP-ID mean a flapping device. |
| `TCP Client App N IP-ID x: Unable to connect to TCP server (…)` | A program-defined TCP client cannot reach its target. A dead third-party box can be retried for months and fill the log with these. |
| `Gateway Connection (ID :N) from … on port :N` / `Gateway Disconnect from …` | Panels, apps and gateways connecting over CIP (normally port 41794, or 41796 for secure CIP). |
| `TCP Server: SSL negotiation didn't complete on port N. Closing socket` | A client tried TLS and gave up: a wrong port, an old client or a scanner. |
| `Lost Ethernet link on LAN A` / `LAN` / `CS` | The physical link dropped on that port (CS is the control-subnet port). |
| `Cannot register Slot-NN.IP-ID-xx, there is no IP Table Entry for this device` | The program expects a device that the IP table does not list. A configuration error, not a network one. |
| `Did not receive End of Query Acknowledge from device at Slot-NN.IP-ID-xx` | A device checked in but did not finish its update handshake. |

### 6.2 The program under strain ###

| Message | Meaning |
|---|---|
| `Could not solve logic within 1000 waves` (2-Series `LogicPro`) | The logic solver could not reach a stable state: a feedback loop or an oscillating signal. It can repeat until it is nearly the whole log. Find the loop. |
| `Symbol Queue Overflow for symbol …` | A symbol received input faster than it could process it. The `Status` fields give the queue depths. |
| `Message From Device Slot-NN.IP-ID-A6: Cannot send to slave pipe … - Pipe Full` | A device is flooding the processor, or the processor is too busy to drain its pipes. |
| `SplusManager: Unable to send Module data back to the Logic Engine. Error Timeout for e_SIGNAL_TYPE_ANALOG/STRING/DIGITAL` | SIMPL+ produced output that the logic engine did not accept in time: an overloaded engine. |
| `StartupState/RuntimeState - Sysdog did not wake us up for N seconds` | The system watchdog thread was starved: the CPU was saturated. |
| `WDG: TLDM(…) Watchdog Fault` / `WDG: LogicEngine shutting down unexpectedly` | A watchdog caught a process and is restarting it. |
| `Simpl Plus Manager terminated. Restarting program` | The SIMPL+ host died and the program was restarted. Look at the lines just before it. |
| `Permanent String in symbol S# was declared as N characters - cannot be greater than M; truncating` | A design error caught at load time, logged on every load. |

### 6.3 SIMPL+ and SIMPL# module errors ###

Module messages name the module instance and source line:

```
Error: splusmanagerapp.exe [App 1] # … # Module S-2.3.4 : Example_Module_v3 at line 812: Overflow. Capacity = 4096. Length = 4120
```

- **`S-2.3.4`** is the module's position in the program. It maps to a `Sm` symbol in the .smw
- **`Overflow. Capacity = N. Length = M`:** a string or buffer was declared too small for the data a
  device sends. This is one of the most common module errors.
- **`TCP Client connection already in progress` / `Unable to resolve name` / `Direct Socket Buffer
  Full`:** module networking that needs a state machine or a bigger buffer.
- **`UpdateInputValue: Could not get m_BufferReadyEvent`:** the module could not accept new input
  fast enough.
- **`NVRAM Contents will be cleared. - NVRAM memory profile has changed`:** the module's stored
  variables no longer match its declaration, so saved settings are lost on this load.

SIMPL# exceptions arrive as `Exception:… - System.NullReferenceException: …`, followed by untagged
`at Namespace.Class.Method(...)` lines. The deepest of those frames is where the exception
started. Frames with short, meaningless names such as `at az.c(String A_0)` come from obfuscated
assemblies; the nearest frame with a readable namespace and class tells you which public API was
involved.

### 6.4 Crashes at the operating-system level ###

Windows CE writes native faults as two untagged lines:

```
Exception 'Data Abort' (0x4): Thread-Id=…(pth=…), Proc-Id=…(pprc=…) 'splusmanagerapp.exe', VM-active=… 'splusmanagerapp.exe'
PC=0032ebac(???+0x0032ebac) RA=0032eb08(???+0x0032eb08) SP=048dfa28, BVA=00000000
```

- **`Data Abort`:** a read or write to a bad address. `Prefetch Abort` is a jump to one. `Raised
  Exception (0xe06d7363)` is a C++ exception.
- **`BVA` is the bad address.** `BVA=00000000` (or any small value) is a null-pointer access, which
  is by far the most common case.
- The **process name** says who crashed. `splusmanagerapp.exe` is the usual one; `ConsoleServiceCE.exe`,
  `LogicEngine.exe`, `CIPCommandProcessor.exe` and `TFTPServer.exe` also appear.
- These are mostly **first-chance** faults. The .NET Compact Framework turns a null access into a
  `NullReferenceException`. The managed error line that follows in the same second shows whether
  code caught it. A Data Abort followed by a managed stack trace is a SIMPL# bug, not a firmware
  crash.
- `PC=…(coredll.dll+0x…)` or `(kernel.dll+0x…)` places the fault inside a named system DLL rather
  than in application code.

### 6.5 Who was on the console ###

`ConsoleServiceCE.exe` logs every connection with its source address: `CTP Connection from:`,
`TELNET Connection from:` and `SHELL Connection from:` (SSH). `Got Security Flag: N` follows at boot
and shows whether authentication was on. Toolbox, SIMPL Debugger and any script you run appear here
too, so expect to see yourself.

### 6.6 When the log stops talking ###

```
Notice: CRESLOG.exe # … # PersistentLog: Consecutive error states detected; logging is suspended
Notice: CRESLOG.exe # … # PersistentLog: Consecutive quiet states detected; logging is resumed
```

The persistent logger **suspends itself** when a component floods it with errors. A suspension is
not always followed by a resumption; when it is not, the gap lasts until the next reboot. **A quiet
stretch after a suspension notice is a gap in the recording, not a quiet system.** Rotation is the
other way history is lost: `Crestron_09.log` falls off the end, and only `ZippedLogs` keeps older
sessions.

---

## 7. A working method ##

1. **Start with identity.** Model, firmware and program slots come from `System startup:` and the
   SysInfo dump. Every message below means something slightly different on each firmware line.
2. **Lay out the sessions.** Find every reboot marker (`Power On`, `User Reboot`, `External Warm
   Reset`, `System Reboot Notification`, 4-Series `csstartup.log`) and split the log at them. Most
   questions are "what happened just before this restart?"
3. **Count before you read.** Reduce each message to a template (numbers, IPs, MACs, hex and quoted
   text masked), then count templates by severity and process. A few templates make up most of any
   log. Once those are explained, what is left is usually the interesting part.
4. **Pair the events.** Offline with back-online, connect with disconnect, stopped with started,
   suspended with resumed. The gaps and the unpaired lines are where the problems are.
5. **Line it up with the program.** Turn `Slot-NN.IP-ID-xx`, `S-2.3.4` and `[App N]` into names
   using the program's .smw. A log without its program is half a conversation.
6. **Mistrust the clock until it is confirmed.** Check for time-change events and compare against
   `UpTime` (2-Series) or the session order (3-Series) before you claim an exact time.
7. **Remember what is not logged.** Button presses, signal values and most of what the program
   does internally only appear if the program writes them. The log records the system's
   housekeeping, not the building.
