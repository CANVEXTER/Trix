# NVIDIA GPU Passthrough with Distrobox using CDI (Arch / Arch-based)

This document records a **tested, working solution** for slow, stuck, or hanging NVIDIA GPU initialization when using **Distrobox** containers.

It targets **Arch Linux and Arch-based distributions** (Arch, CachyOS, EndeavourOS, etc.) and assumes **Podman** as the container runtime.

> **Repository intent**
> **Trix** collects *real-world, verified fixes and workarounds* — not theoretical or copy-pasted recipes.

---

## Notes on Docker
Docker is **not tested** with this workflow in **Trix**.

It may work, but behavior can differ.

**Podman remains the recommended and verified engine.**

---

## Background (important context)

If a Distrobox container:

* Takes a very long time to start
* Appears to freeze during NVIDIA initialization
* Was created using the `--nvidia` flag

➡️ **This is not an OS issue, not a driver bug, and not Podman’s fault.**

The root cause is Distrobox’s `--nvidia` flag relying on **legacy NVIDIA OCI hooks**, which are:

* Deprecated
* Slow due to runtime probing
* Fragile on modern systems
* Unnecessary today

The modern and supported solution is **Container Device Interface (CDI)**.

---

## Quick principles (read first)

* Always use the **latest proprietary NVIDIA drivers** for your kernel
* If you **don’t need GPU access**, do **not** use `--nvidia`
* If you **do need GPU access**, use **CDI**
* CDI is faster, simpler, and future-proof

---

## Step 1 — Verify NVIDIA and CDI support (do this first)

Before installing or changing anything, **verify what already works**.

### 1.1 Verify NVIDIA driver (host)

```bash
nvidia-smi
```

Expected:

* Driver version shown
* GPU listed
* No errors or delay

If this fails, **fix your NVIDIA driver first**.

---

### 1.2 Verify NVIDIA Container Toolkit

```bash
nvidia-ctk --version
```

If this command exists, continue to **Step 1.3**.
If it does **not**, install the required packages (see Step 2).

---

### 1.3 Verify CDI device availability

```bash
nvidia-ctk cdi list
```

If you already see entries like:

```text
nvidia.com/gpu=all
nvidia.com/gpu=0
```

✅ CDI is already available — you may skip directly to **Step 4**.

If **nothing is listed**, CDI is not generated yet — continue below.

---

## Step 2 — Install required packages (if missing)

Install the required host packages:

```bash
sudo pacman -S nvidia nvidia-utils nvidia-container-toolkit podman distrobox
```

After installation, **reboot is recommended**, then re-run:

```bash
nvidia-smi
nvidia-ctk --version
```

Do not continue until both succeed.

---

## Step 3 — Generate NVIDIA CDI specification

Generate the CDI spec manually:

```bash
sudo nvidia-ctk cdi generate --output=/etc/cdi/nvidia.yaml
```

Verify again:

```bash
nvidia-ctk cdi list
```

Expected output:

```text
nvidia.com/gpu=all
nvidia.com/gpu=0
```

If devices appear, CDI is now active.

---

## Step 4 — Disable legacy NVIDIA OCI hooks (if present)

Legacy hooks can **conflict with CDI**.

Check for them:

```bash
ls /usr/share/containers/oci/hooks.d/
```

### If `oci-nvidia-hook.json` exists

Disable it safely:

```bash
sudo mv /usr/share/containers/oci/hooks.d/oci-nvidia-hook.json \
        /usr/share/containers/oci/hooks.d/oci-nvidia-hook.json.bak
```

Restart Podman:

```bash
systemctl --user restart podman
```

### If no NVIDIA hook is present

That is normal on clean systems — no action required.

---

## Step 5 — Recreate the Distrobox container (mandatory)

⚠️ If the container was **ever created with `--nvidia`**, it **must be recreated**.

### Remove the old container

```bash
distrobox stop <name>
distrobox rm <name>
```

---

### Create a new container using CDI

```bash
distrobox create \
  --name <name> \
  --image <image> \
  --additional-flags "--device nvidia.com/gpu=all --security-opt=label=disable"
```

Notes:

* **Do NOT use `--nvidia`**
* CDI exposes the GPU directly
* `label=disable` avoids SELinux/AppArmor overhead

---

## Step 6 — Verify GPU access inside the container

Enter the container:

```bash
distrobox enter <name>
```

Run:

```bash
nvidia-smi
```

Expected:

* Immediate output
* No hang or delay
* GPU visible instantly

---

## Optional — Lightweight GPU warm-up (no persistence)

On laptops or power-sensitive systems, persistence mode is unnecessary.

Instead, initialize the GPU **once on demand**.

---

### Fish

```fish
alias warm-gpu 'nvidia-smi > /dev/null 2>&1'
funcsave warm-gpu
```

---

### Bash

```bash
alias warm-gpu='nvidia-smi > /dev/null 2>&1'
```

---

### Zsh

```zsh
alias warm-gpu='nvidia-smi > /dev/null 2>&1'
```

Usage (all shells):

```bash
warm-gpu
distrobox enter <name>
```

This avoids:

* Boot-time GPU wakeups
* Idle power drain
* Persistent NVIDIA services

---

## Summary — What this achieves

* No legacy NVIDIA hooks
* No `--nvidia` flag
* Fast, predictable container startup
* GPU initializes only when needed
* Reliable on Arch-based systems

This is currently the **cleanest and most future-proof** NVIDIA + Distrobox setup.

---

## End notes

Tested on real systems with:

* NVIDIA proprietary drivers
* Podman 5.x
* Distrobox 1.8+

Adapt freely for other distributions or container workflows.
