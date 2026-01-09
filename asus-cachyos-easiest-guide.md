## BIOS & Windows Preparation (Very Important)

Before booting the CachyOS installer, **you must adjust a few ASUS BIOS settings**.
Skipping this step is one of the **most common reasons for installation failures**.

> **Note**
> These steps are standard for ASUS laptops and are covered in almost every CachyOS / Arch / ASUS dual-boot YouTube tutorial.

---

### Required BIOS Changes

Enter BIOS (usually by pressing **F2** or **DEL** during boot) and make sure:

* **Disable Secure Boot**
* **Disable Fast Boot**
* **Disable Intel VMD (Volume Management Device)**

> **Why this matters**
> Intel VMD hides NVMe drives from the Linux installer, causing disks to not appear at all.

---

### Windows-Specific (Dual Boot Only)

If you are dual-booting with Windows:

* **Disable BitLocker** in Windows *before* installing Linux

  * Either fully turn it off
  * Or decrypt the Windows drive

> **Important**
> Failing to disable BitLocker can:

* Prevent Linux from accessing disks
* Cause Windows boot issues later
* Lock you out of your own data

---

### Final Reminder

> **Do not skip this step**
> BIOS misconfiguration causes more issues than Linux itself.

If you’re unsure:

* Search YouTube for:
  **“ASUS CachyOS dual boot”** or **“ASUS Arch Linux install”**
* Follow any recent guide — the BIOS steps are nearly identical.

Once this is done, proceed with the CachyOS installation normally 🚀


# The Easiest Linux Guide for ASUS ROG, TUF & Similar Laptops (CachyOS)

> This guide is part of **Tetricks** — a personal collection of **tested fixes, workarounds, and setups** that I’ve actually experienced and verified.
>
> Anyone is welcome to adapt, improve, or suggest changes. Linux is collaborative by nature 🚀

---

## Why This Guide Exists

ASUS laptops (ROG, TUF, ProArt, etc.) are powerful, but Linux setup **does not have to be painful**.

This guide focuses on the **most effortless, least error-prone way** to run Linux on ASUS laptops — without unnecessary tweaking, repo juggling, or breaking your system.

---

## Recommended Approach (TL;DR)

* **Just use CachyOS**
* Follow this guide
* Enjoy Linux 😄

No exotic patches. No manual kernel builds. No complicated Arch rituals.

---

## Important Notes Before Starting

> **Note**

* This guide uses **CachyOS Rolling**
* CachyOS Stable is **not tested**, but should behave very similarly

> **Tested Hardware**

* ✅ **ASUS TUF F15 (FX506HF)**
* Works flawlessly on this device

> **Scope**

* Wayland-focused (modern standard)
* Also works on X11 (nothing here depends on compositor/display server)

---

## Why CachyOS for ASUS Laptops?

CachyOS comes **pre-optimized** for modern hardware, especially gaming laptops.

### Key Advantages

* Ships with a **custom kernel** that includes:

  * ASUS / Armoury-related drivers
  * Performance and scheduler patches
* **SSD TRIM enabled by default**
* **ZRAM enabled by default**

  * Excellent for gaming laptops with limited RAM
* Latest **proprietary NVIDIA drivers** included
* Minimal manual setup required

> **Why this matters**
> Most ASUS laptops *just work* out of the box with CachyOS — far better than vanilla Arch or many other distros.

---

## Desktop Environment (DE) Choice

### Recommended: **COSMIC DE (System76)**

* Tested and **works the best** for this setup
* Extremely stable in real-world use
* Wayland-native
* Still in early development, but already very reliable
* Expected to reach stable release soon

> **Note**
> Other options are completely valid:

* KDE Plasma 6 → tested, but behaves **very poorly** on this hardware
* GNOME → tested, but very strict networking policies noticeably reduce upload/download speeds
* Hyprland, Sway, i3, etc. → all fine if you know what you’re doing

---

## Installation Steps

### 1. Install CachyOS

* Install CachyOS like any normal Linux distro
* If dual-booting with Windows:

  * Plenty of **good YouTube and online guides** exist
  * Follow any standard CachyOS + Windows dual-boot tutorial

> **Personal Preference**

* During installation, select **only COSMIC DE**
* Add other DEs or window managers later if needed

---

### 2. Post-Install Sanity Check

After first boot:

```bash
sudo pacman -Syu
```

> **Why?**
> CachyOS uses an online installer and usually ships fresh packages — this is just a sanity check.

---

### 3. NVIDIA & Drivers

> **Good news 🎉**

* CachyOS already includes:

  * Latest proprietary NVIDIA drivers
  * Required firmware and dependencies

➡️ **No manual driver installation needed**

---

## ASUS-Specific Tools (The Important Part)

### Install ASUS Control Tools

```bash
sudo pacman -S asusctl rog-control-center
```

That’s it.
No extra repos. No services to manually enable.

> **Important Note**

* The official ASUS Linux Arch guide mentions enabling `power-profiles-daemon`
* That guide is for **vanilla Arch**
* **CachyOS already handles this properly**

---

### Using ASUS Features

* Prefer **ROG Control Center (GUI)** over CLI
* You can still use `asusctl` if you like terminal control

#### Strongly Recommended

> **Set a charge limit immediately if your laptop supports it**

* Ideal value: **80%**
* Can be done via:

  * ROG Control Center (GUI)
  * `asusctl` command

This significantly improves long-term battery health 🔋

> **Note**

* “ROG Control Center” is just a name
* It works perfectly on:

  * ASUS ROG
  * ASUS TUF
  * ASUS ProArt laptops

---

## About supergfxctl (Optional)

If your laptop has a **physical MUX switch** (hardware GPU power switch):

* You *may* install `supergfxctl`
* Check your laptop specs online to confirm MUX support

> **Important Caveat**

* `supergfxctl` is currently **suspended**
* No active development by ASUS Linux team
* Not personally tested (my laptop doesn’t have a MUX)
* According to official docs, it *should* still work

---

## ZRAM Consideration (Older Systems Only)

> **Optional**

* If your system is **very old**, you may disable/remove ZRAM
* ZRAM slightly increases CPU usage
* Negligible on modern systems
* Can impact performance on *very old hardware*

---

## Final Words

That’s it.
You’re done 🎉

Enjoy Linux on your ASUS laptop — **simple, fast, and stable**.

> **Final Note**

* This guide may not fully apply to *future* hardware
* The tech world changes fast
* OEMs are aggressively pushing AI-powered laptops
* Still, **most features should just work**

On my system, Linux:

* Outperforms Windows
* Uses less power
* Provides better battery life
* Feels more responsive overall

I genuinely believe this setup works great for **many ASUS laptops**, especially those not locked behind heavy AI firmware layers.

---

**Contributions welcome. Improvements encouraged.**
That’s what **Tetricks** is all about 🤝
