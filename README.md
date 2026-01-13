# EFI-shell-as-a-boot-manager-
Boot manager without a bootloader (UEFI only)

---

# EFI Shell as a Boot Manager  
> Remember that firmware is already a bootloader

## TL;DR

You don’t need GRUB.  
You don’t need Ventoy.  
You don’t even need PXE ROM.

**UEFI + EFI Shell + `.efi` payloads = complete boot solution.**

---

## Why this repository exists

Over time, boot chains became unnecessarily complex:

Firmware → shim → GRUB → kernel Firmware → PXE ROM → iPXE → installer Firmware → Ventoy → ISO → bootloader → kernel

But UEFI already gives us everything we need.

**EFI Shell is a fully capable pre-OS environment** that can:
- access disks
- read filesystems (FAT)
- execute EFI applications
- chainload network boot (iPXE)
- work without removable media

This repo exists to remind people:

> **The firmware is the bootloader.**

---

## Core idea

Use **EFI Shell** as a minimal, explicit boot manager.

UEFI firmware ↓ EFI Shell ↓ Choose what to run: ├─ Local EFI application (Linux EFIstub, WinPE, tools) └─ iPXE.efi (network boot via HTTP / PXE / iSCSI)

No magic. No hidden layers.

---

## What EFI Shell already provides

- Block I/O access
- FAT filesystem support
- Manual and scripted execution (`.nsh`)
- EFI application loading
- Vendor-independent behavior
- Works before any OS is loaded

EFI Shell is **not a hack**.  
It is an official UEFI reference application.

---

## What this setup replaces

| Traditional component | Replaced by |
|----------------------|-------------|
| GRUB                 | EFI Shell |
| PXE ROM              | `ipxe.efi` |
| Multiboot USB        | ESP partition |
| Rescue flash drives  | Firmware tools |

---

## Example ESP layout


ESP (FAT32)
└─ EFI/
   ├─ BOOT/
   │  └─ BOOTX64.EFI    # EFI Shell
   └─ TOOLS/
      ├─ ipxe.efi      # Network boot
      ├─ linux.efi     # Linux EFIstub
      ├─ winpe.efi     # Windows PE
      ├─ memtest.efi
      └─ shell.nsh


---

Example workflow

Local boot

fs0:
cd EFI\TOOLS
linux.efi

Network boot

fs0:
cd EFI\TOOLS
ipxe.efi

From there, iPXE handles:

DHCP

HTTP

PXE

iSCSI

wimboot


EFI Shell does not need to know how networking works.


---

Optional automation

EFI Shell supports startup.nsh:

echo "1) Network boot (iPXE)"
echo "2) Linux"
echo "3) WinPE"
pause

No configuration language.
No parser.
Just execution.


---

Philosophy

EFI Shell is stateless

.efi files are self-contained

Network boot is just another payload


This is not about convenience.
This is about clarity and control.


---

When this approach makes sense

Rescue environments

Lab machines

Diskless setups

Firmware experiments

Systems without Ethernet PXE

People who want to understand booting instead of hiding it



---

When it does NOT

Consumer desktops

Secure Boot-heavy environments (without custom signing)

Users who want “plug and play”


This repo is intentionally low-level.


---

Related projects

iPXE — network boot engine

EDK II — UEFI reference implementation

Linux EFIstub

systemd-boot (for comparison)

---

## Why not GRUB?

This repository is **not anti-GRUB**.  
GRUB is a powerful and mature bootloader.

However, its design goals are fundamentally different from what is explored here.

### GRUB is designed to:
- abstract hardware differences
- support many filesystems
- provide scripting and automation
- work identically across BIOS and UEFI
- act as a universal pre-OS environment

This comes at a cost:
- complex configuration language
- non-trivial runtime logic
- filesystem drivers and probing
- persistent bootloader state
- larger attack surface

For users who want:
- full automation
- menu-driven multiboot
- encrypted disks
- complex storage setups

**GRUB is the correct choice.**

This repository explores what happens when you *intentionally remove* that layer.

---

## Why Secure Boot does not work out of the box

EFI Shell and direct EFI execution intentionally avoid Secure Boot assumptions.

### Reasons:

- EFI Shell is often **unsigned**
- Direct EFIstub kernels are usually **not signed**
- iPXE custom builds are unsigned by default
- No shim layer is present to mediate trust

Secure Boot expects:

Firmware → Microsoft-signed shim → signed bootloader → signed kernel

This repository uses:

Firmware → EFI Shell → arbitrary EFI payload

These models are incompatible by default.

---

## Can Secure Boot be used anyway?

Yes — but **only with explicit user action**.

Possible approaches:
- Enroll custom keys (Machine Owner Keys)
- Sign EFI Shell, kernels, and tools manually
- Use custom firmware keys
- Disable Microsoft trust chain entirely

This is:
- intentional
- explicit
- user-controlled

But **not beginner-friendly**.

Secure Boot is about policy enforcement.  
This repository is about **operator control**.

---

## Why this approach is useless for some users

This setup is **not universal** and does not try to be.

It will be useless or inappropriate if you expect:
- automatic OS discovery
- graphical menus
- unattended installs
- consumer-friendly UX
- vendor-supported workflows
- “it just works” behavior

It requires:
- understanding UEFI concepts
- manual execution or scripting
- acceptance of explicit control
- willingness to debug firmware behavior

For many users, this is unacceptable — and that is fine.

---

## Who this *is* for

This approach makes sense if you:
- want to understand booting, not abstract it
- work with rescue, lab, or test systems
- need full control over boot paths
- dislike opaque boot chains
- are comfortable operating pre-OS environments
- value simplicity over automation

This is a **tool**, not a product.

---

## Design philosophy (explicitly stated)

- No hidden state
- No implicit automation
- No mandatory configuration language
- No bootloader persistence
- No attempt to be user-friendly

If something boots, it is because **you executed it**.

---

## Final clarification

This repository does not claim that EFI Shell should replace GRUB.

It demonstrates that:

> **For a specific class of users,  
> GRUB is optional, not mandatory.**

EFI Shell already exists.  
UEFI already provides the primitives.  
This repository merely connects the dots.

---

Final note

EFI Shell was never meant to be user-friendly.
It was meant to be honest.

And sometimes, that’s exactly what we need.

> Booting doesn’t have to be complicated —
we just forgot what the firmware already gives us.




---

