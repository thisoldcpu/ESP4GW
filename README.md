# ESP4GW

<img width="150" height="180" alt="esp4gw_logo_150" src="https://github.com/user-attachments/assets/3d868c5a-6b33-46e0-a44d-3dc92d09733f" />

**ESP4GW is a native DOS-era compatibility layer and `<dos.h>` hardware abstraction layer for Espressif ESP32 microcontrollers.**

Its goal is to make it easier to port source code originally written for DOS, Watcom C/C++, DOS/4GW, and similar protected-mode environments to modern ESP32 hardware without first rewriting every DOS-facing part of the program.

ESP4GW does **not** run DOS.

ESP4GW does **not** emulate a PC.

ESP4GW does **not** execute DOS binaries.

Instead, legacy source code is recompiled natively for the ESP32, while ESP4GW provides compatible interfaces for the DOS-era services that the source expects.

---

## The Idea

A typical DOS game or application from the early-to-mid 1990s may contain perfectly portable game, application, file-format, math, rendering, or simulation code surrounded by assumptions such as:

```c
#include <dos.h>
#include <conio.h>
#include <i86.h>

_dos_findfirst(...);
_dos_open(...);
_dos_read(...);

int86(...);
intdos(...);

inp(...);
outp(...);
```

On a modern embedded target, the useful code may still compile cleanly once those platform dependencies are resolved.

ESP4GW exists to provide that resolution layer.

Instead of forcing every port to replace DOS APIs individually:

```text
Legacy Application Source
          |
          v
       ESP4GW
          |
          +---- Filesystem
          +---- Memory
          +---- Timers
          +---- Input
          +---- Display
          +---- Audio
          +---- DOS compatibility services
          +---- DPMI-inspired services
          |
          v
       ESP-IDF
          |
          v
        ESP32
```

The application remains recognizable as the original source.

The machine underneath it does not.

---

## What ESP4GW Is

ESP4GW is intended to become a reusable compatibility layer for **native source ports** of DOS-era software.

The project aims to provide familiar interfaces used by DOS applications while translating those operations into appropriate ESP32 and ESP-IDF facilities.

Where practical, legacy source should be able to retain calls such as:

```c
_dos_findfirst()
_dos_findnext()

_dos_open()
_dos_close()
_dos_read()
_dos_write()

_dos_getdate()
_dos_gettime()

int86()
intdos()
```

without each application having to invent its own incompatible replacement.

ESP4GW is therefore best thought of as:

> **A DOS programming environment personality for native ESP32 applications.**

It preserves useful interfaces and semantics.

It does not preserve the original machine.

---

## What ESP4GW Intends To Be

ESP4GW is intended to grow into several related compatibility layers.

### DOS C Runtime Compatibility

Provide commonly encountered DOS and Watcom-style structures, constants, functions, and headers.

Examples include:

```text
dos.h
conio.h
i86.h
io.h
direct.h
```

Only interfaces useful to real source ports need to be implemented.

The goal is practical compatibility, not archaeological completeness.

### DOS Service Translation

DOS filesystem and operating-environment requests can be translated into ESP-IDF, VFS, FATFS, POSIX, FreeRTOS, or other native facilities.

For example:

```text
DOS file search     -> ESP filesystem directory enumeration
DOS file handles    -> native file descriptors
DOS date/time       -> ESP/native time services
DOS disk queries    -> mounted filesystem information
```

The caller sees the familiar interface.

ESP4GW performs the translation.

### DPMI-Inspired Compatibility

DOS/4GW applications commonly lived in a DPMI protected-mode environment.

ESP4GW may eventually provide compatible abstractions for useful portions of that environment, including areas such as:

- memory allocation
- large-memory management
- memory locking
- virtual memory
- descriptor-like objects
- callback and vector registration
- interrupt-style service dispatch
- legacy register structures
- DOS service requests

These services will be implemented in terms appropriate to the ESP32.

An ESP32 does not have an x86 Global Descriptor Table simply because a legacy source file refers to selectors.

Where the underlying architecture differs, ESP4GW will translate the **meaning** of the operation rather than attempting to recreate unnecessary x86 machinery.

### Hardware Abstraction

Some legacy programs communicate with hardware through mechanisms such as:

```c
inp()
outp()
```

or through BIOS/DOS interrupt interfaces.

Where useful, ESP4GW may provide translation layers for commonly encountered DOS-era hardware concepts.

Potential examples include:

```text
VGA palette operations
timers
keyboard input
mouse input
joystick input
audio devices
storage devices
```

This does not mean ESP4GW will emulate those devices.

A request associated with an old PC device may instead be translated into the native ESP32 subsystem serving the same purpose.

### Porting Diagnostics

Unsupported operations should fail visibly and honestly.

A compatibility layer that silently pretends something worked is considerably more dangerous than one that says:

```text
ESP4GW: unsupported service
ESP4GW: unsupported interrupt
ESP4GW: unsupported I/O port
```

One long-term goal is for ESP4GW to help identify the remaining architecture-specific assumptions inside a legacy application.

In that role it becomes both a compatibility library and a porting tool.

---

# What ESP4GW Is NOT

The boundaries of this project are deliberate.

They are not temporary limitations waiting to become features.

## ESP4GW Is Not An Operating System

ESP4GW will **never become an operating system**.

It does not boot hardware into DOS.

It does not provide a DOS command shell.

It does not provide `COMMAND.COM`.

It does not attempt to replace FreeDOS, MS-DOS, DR-DOS, or any other operating system.

ESP-IDF and the underlying ESP32 runtime remain responsible for the actual system.

ESP4GW is a library used by an application running inside that environment.

---

## ESP4GW Is Not An Emulator

ESP4GW will **never become a PC emulator or x86 emulator**.

It will not emulate:

- an 8086
- an 80286
- an 80386
- an 80486
- x86 protected mode
- an IBM-compatible PC
- a complete BIOS
- DOS as a machine environment

There is no virtual x86 CPU executing instructions.

There is no cycle-by-cycle recreation of PC hardware.

There is no intention to turn ESP4GW into DOSBox.

---

## ESP4GW Does Not Run DOS Executables

ESP4GW does not load and execute:

```text
.EXE
.COM
LE/LX
DOS/4GW binaries
Watcom executables
```

Programs must be available as source code and compiled for the target ESP32 architecture.

For example:

```text
Original:

DOS source
    |
Watcom C/C++
    |
DOS/4GW
    |
80386

ESP4GW:

DOS source
    |
ESP-compatible compiler
    |
ESP4GW
    |
ESP-IDF
    |
ESP32
```

ESP4GW targets **source compatibility**, not binary compatibility.

---

## ESP4GW Is Not DOS/4GW Ported To Xtensa

Despite the name, ESP4GW is not intended to be a literal architecture port of the original DOS/4GW executable.

DOS/4GW contains machinery specifically designed around x86 protected mode, DOS, BIOS services, descriptors, interrupts, and 386-class CPUs.

Recreating all of that machinery on another architecture would provide little benefit to a native source port.

ESP4GW instead asks:

> What service was the application trying to obtain from DOS/4GW?

That service can then be implemented natively.

The interface may survive.

The implementation does not have to.

---

## ESP4GW Is Not A Compatibility Excuse

ESP4GW will not claim that unsupported behavior works when it does not.

Some DOS concepts do not have useful equivalents on ESP32.

Some can be translated.

Some can be represented by lightweight compatibility objects.

Some require application changes.

Some should simply return:

```text
DOS_ERR_NOT_SUPPORTED
```

That distinction is intentional.

Correct failure is preferable to fake compatibility.

---

# Compatibility Philosophy

ESP4GW follows several basic rules.

### Preserve source when practical

If a DOS-era interface can be supported cleanly, keep the interface.

Avoid forcing application-specific rewrites for problems that can be solved once inside ESP4GW.

### Translate semantics, not architecture

A DOS API requesting a directory search does not require DOS.

It requires a directory search.

A timer interrupt may not require an x86 interrupt.

It may require a periodic callback.

A protected-mode memory allocation does not necessarily require an x86 descriptor.

It requires memory with the expected lifetime and behavior.

ESP4GW attempts to preserve the useful semantic contract.

### Never fake success

If ESP4GW cannot perform an operation correctly, it should report that fact.

### Avoid application-specific hacks

ESP4GW is intended to become reusable infrastructure.

Application-specific workarounds belong in the application unless they represent a generally useful DOS compatibility behavior.

### Native first

The resulting application should remain an ESP32 application.

ESP-IDF, FreeRTOS, native peripherals, PSRAM, flash, SD storage, native display controllers, Bluetooth, USB, and other ESP facilities should be usable directly.

ESP4GW should help legacy code join that environment rather than hiding it behind a simulated PC.

---

# Current Status

ESP4GW is at a very early stage.

The initial `<dos.h>` compatibility layer already defines a number of classic DOS-facing data structures and APIs, including:

```text
REGS
SREGS
DOSERROR
find_t
dosdate_t
dostime_t
diskfree_t
```

as well as a growing collection of `_dos_*`, `int86`, `intdos`, and related interfaces.

The current implementation already provides native ESP32-backed functionality for several filesystem and runtime operations.

Examples include:

- file search
- file open/create/close
- file read/write
- file attributes
- file timestamps
- date and time queries
- filesystem free-space queries
- DOS-style error translation

Unsupported facilities currently report an explicit unsupported result rather than manufacturing DOS behavior.

This is intentional.

---

# Current `<dos.h>` Direction

ESP4GW's public API follows the familiar DOS-era programming interface while the implementation lives behind the compatibility layer.

For example:

```c
struct find_t file;

if (_dos_findfirst("*.PIG", _A_NORMAL, &file) == 0)
{
    do
    {
        printf("%s\n", file.name);
    }
    while (_dos_findnext(&file) == 0);

    _dos_findclose(&file);
}
```

The calling code looks like DOS source.

The implementation may be backed by ESP-IDF VFS, FATFS, POSIX APIs, or another native service.

That distinction is the core of ESP4GW.

---

# Future Areas

Potential future work includes:

- broader `<dos.h>` coverage
- `<conio.h>` compatibility
- `<i86.h>` compatibility
- DOS interrupt dispatcher
- DPMI service dispatcher
- native implementation of useful DPMI memory services
- PSRAM-aware memory management
- optional virtual-memory support
- interrupt/vector compatibility objects
- legacy timer translation
- I/O-port dispatch
- VGA-style palette translation
- keyboard compatibility
- mouse compatibility
- joystick compatibility
- legacy audio API translation
- improved DOS pathname handling
- drive/mount mapping
- compatibility test applications
- automated API coverage testing
- per-application compatibility reports

None of these require ESP4GW to become an emulator.

---

# Virtual Memory

ESP4GW may eventually explore DPMI-inspired virtual-memory facilities.

ESP32 systems can contain several distinct classes of storage and memory:

```text
internal SRAM
PSRAM
flash
SD storage
memory-mapped resources
```

A future ESP4GW memory manager may make it possible for legacy applications to request a larger logical memory space while ESP4GW determines how that memory should be represented physically.

Possible strategies include:

- internal-RAM allocations
- PSRAM allocations
- pageable PSRAM
- reloadable read-only assets
- SD-backed pages
- application-controlled locking
- resource-aware eviction

Such a facility would reproduce the **purpose** of DOS-extender virtual memory without pretending that an ESP32 is a 386.

This remains experimental work.

---

# Why ESP4GW?

Because a surprising amount of old software is not fundamentally dependent on DOS.

It is dependent on a collection of services that DOS happened to provide.

When source code is available, replacing an entire PC with an emulator may be unnecessary.

Sometimes the better question is:

> How much of the original program can simply become native code?

ESP4GW exists to make the answer:

> More than before.

---

# First Real-World Test

ESP4GW grew out of work porting the original **Descent** source to the ESP32-S3.

That project exposed a recurring pattern: much of the engine could be compiled and adapted normally, while relatively small pieces of DOS-facing infrastructure repeatedly required replacement or compatibility shims.

Rather than solving those problems only for one game, ESP4GW aims to turn them into reusable infrastructure for future ports.

Descent is therefore an important test case.

It is not the definition of the project.

---

# Target Platforms

Initial development targets:

```text
ESP32-S3
ESP-IDF
```

The architecture should avoid unnecessary assumptions that prevent support for other ESP32 variants where practical.

Support for non-Espressif platforms is not currently a project goal.

---

# Project Goals

In short, ESP4GW wants to make this:

```c
#include <dos.h>

/* 30-year-old source code */
```

less frightening on this:

```text
ESP32-S3
```

without putting an imaginary IBM PC between them.

---

# Non-Goals Summary

ESP4GW will not become:

- an operating system
- DOS
- FreeDOS
- an x86 emulator
- a PC emulator
- DOSBox
- a DOS virtual machine
- a DOS binary loader
- a DOS/4GW binary runtime
- a cycle-accurate PC hardware implementation

ESP4GW will remain:

> **A native compatibility and hardware abstraction layer for porting DOS-era source code to ESP32 hardware.**

---

## Project Status

**Experimental / Early Development**

APIs, internal organization, and compatibility behavior are expected to change substantially while the initial architecture is established.

Contributions, compatibility reports, test programs, and examples from real DOS-era source ports are welcome.

---

## Name

The name **ESP4GW** is a reference to the DOS/4GW era and to the project's goal of making protected-mode DOS source easier to bring to ESP32 hardware.

It does not imply binary compatibility with DOS/4GW or affiliation with the original DOS/4GW developers or rights holders.

All third-party product names and trademarks remain the property of their respective owners.
<img width="150" height="180" alt="esp4gw_logo_150" src="https://github.com/user-attachments/assets/f9443903-f3f6-4e5d-9e28-2561befb88ee" />
