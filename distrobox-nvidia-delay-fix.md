# NVIDIA GPU passthrough with Distrobox using CDI (Arch / Arch-based)

This document records a **tested, working workaround** for slow or stuck NVIDIA GPU initialization when using **Distrobox** containers.

It is written for **Arch Linux and Arch-based distributions** (Arch, CachyOS, EndeavourOS, etc.) and assumes **Podman** as the container engine.

> Repository intent: **Trix** collects *practical fixes and workarounds that are verified in real systems*, not theoretical recipes.

---

## Background (important context)

Before anything else, understand this clearly:

* If your Distrobox container **takes a very long time to start** or appears to hang during NVIDIA integration
* And you are using the `--nvidia` flag

➡️ **This is not your OS, not your GPU, and not Podman’s fault.**

This is a **known limitation of Distrobox’s `--nvidia` flag**, because it relies on **legacy NVIDIA OCI hooks**.

Those hooks are increasingly deprecated and are **slow, fragile, and unnecessary** on modern systems.

A better approach exists: **Container Device Interface (CDI)**.

---

## Quick principles (read this first)

* Always use the **latest proprietary NVIDIA drivers** available for your kernel
* The **simplest workaround** is: **do not use `--nvidia` at all**
* If you *do* need GPU access inside containers → **use CDI**
* CDI is faster, simpler, and future-proof

---

## Requirements (Arch / Arch-based)

Make sure these are installed and working on the host:

```bash
sudo pacman -S nvidia nvidia-utils nvidia-container-toolkit podman distrobox
```

> Docker is **not covered** in this guide. You may experiment at your own risk.
>
> **Podman is strongly recommended** and fully tested with this approach.

---

## Step 1 — Verify the host is healthy

Do **not** proceed unless all of the following work correctly.

### NVIDIA driver

```bash
nvidia-smi
```

Expected:

* Driver version shown
* GPU listed
* No errors

---

### Podman

```bash
podman --version
```

---

### Distrobox

```bash
distrobox --version
```

---

### NVIDIA Container Toolkit (CDI provider)

```bash
nvidia-ctk --version
```

If any of these commands fail, fix that **before continuing**.

---

## Step 2 — Why NOT to use `--nvidia`

The `--nvidia` flag:

* Uses **legacy OCI hooks**
* Performs slow device probing
* Can block container startup
* Is no longer the recommended path

If you **do not need GPU access**, simply **avoid `--nvidia` entirely**.

If you **do need GPU access**, continue below.

---

## Step 3 — Generate NVIDIA CDI specification

CDI exposes GPUs as **standard container devices**.

Generate the spec on the host:

```bash
sudo nvidia-ctk cdi generate --output=/etc/cdi/nvidia.yaml
```

Verify it:

```bash
nvidia-ctk cdi list
```

Expected output includes entries such as:

```text
nvidia.com/gpu=all
nvidia.com/gpu=0
```

If nothing is listed, CDI is not active yet.

---

## Step 4 — Handle legacy NVIDIA OCI hooks (if present)

On some systems, legacy hooks may still exist and **conflict with CDI**.

Check:

```bash
ls /usr/share/containers/oci/hooks.d/
```

### If you see `oci-nvidia-hook.json`

Disable it (do not delete permanently):

```bash
sudo mv /usr/share/containers/oci/hooks.d/oci-nvidia-hook.json \
        /usr/share/containers/oci/hooks.d/oci-nvidia-hook.json.bak
```

Restart Podman:

```bash
systemctl --user restart podman
```

### If nothing NVIDIA-related is shown

That is **completely fine**. No action is required.

---

## Step 5 — Recreate the Distrobox container (important)

If your container was **ever created using `--nvidia`**, recreate it.

### Remove old container

```bash
distrobox stop <name>
distrobox rm <name>
```

---

### Create container using CDI (recommended)

```bash
distrobox create \
  --name <name> \
  --image <image> \
  --additional-flags "--device nvidia.com/gpu=all --security-opt=label=disable"
```

Notes:

* **Do NOT use `--nvidia`**
* CDI handles GPU exposure
* `label=disable` avoids SELinux/AppArmor overhead

---

## Step 6 — Verify GPU inside the container

Enter the container:

```bash
distrobox enter <name>
```

Run:

```bash
nvidia-smi
```

Expected:

* Instant output
* No freezing or long delay
* GPU visible immediately

---

## Optional — Lightweight GPU warm-up (no persistence)

On laptops or power-sensitive systems, **persistence mode is not required**.

Instead, you may optionally warm the GPU *on demand*.

This triggers NVIDIA initialization once, without keeping the GPU awake.

---

### Fish shell

```fish
alias warm-gpu 'nvidia-smi > /dev/null 2>&1'
funcsave warm-gpu
```

Usage:

```bash
warm-gpu
distrobox enter <name>
```

---

### Bash

Add the alias to `~/.bashrc` (or `~/.bash_profile` if preferred):

```bash
alias warm-gpu='nvidia-smi > /dev/null 2>&1'
```

Reload shell:

```bash
source ~/.bashrc
```

Usage:

```bash
warm-gpu
distrobox enter <name>
```

---

### Zsh

Add the alias to `~/.zshrc`:

```zsh
alias warm-gpu='nvidia-smi > /dev/null 2>&1'
```

Reload shell:

```zsh
source ~/.zshrc
```

Usage:

```bash
warm-gpu
distrobox enter <name>
```

---

This approach avoids:

* Boot-time GPU wakeups
* Idle power drain
* Background NVIDIA services

---

## Summary (what this setup achieves)

* No legacy NVIDIA hooks
* No `--nvidia` flag
* Fast container startup
* GPU initializes only when needed
* Works reliably on Arch-based systems

This is currently the **cleanest and most future-proof** way to use NVIDIA GPUs with Distrobox.

---

## Notes on Docker

Docker is **not tested** with this setup in this repository.

You may experiment, but behavior may differ.

Podman remains the **recommended and verified** engine.

---

## End

If you are reading this in the **Trix** repository:

* This method has been tested on real systems
* NVIDIA proprietary drivers
* Podman 5.x
* Distrobox 1.8+

Feel free to adapt this guide for other distributions.
