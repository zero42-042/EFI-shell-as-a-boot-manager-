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

```text
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

Final note

EFI Shell was never meant to be user-friendly.
It was meant to be honest.

And sometimes, that’s exactly what we need.

> Booting doesn’t have to be complicated —
we just forgot what the firmware already gives us.




---

