# NVIDIA GPU Passthrough with Distrobox using CDI (Arch / Arch-based)

This document records a **tested, working solution** for slow, stuck, or hanging NVIDIA GPU initialization when using **Distrobox** containers.

It targets **Arch Linux and Arch-based distributions** (Arch, CachyOS, EndeavourOS, etc.) and assumes **Podman** as the container runtime.

> **Tetricks** collects *real-world, verified fixes and workarounds* — not theoretical or copy-pasted recipes.

---

## Notes on Docker

Docker is **not tested** with this workflow in **Tetricks**.

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
* If you **don’t need GPU access**, do **not** use `--nvidia` or CDI
* If you **do need GPU access**, use **CDI**
* **Never mix** legacy OCI hooks and CDI
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

✅ CDI capability exists — continue to Step 3.

⚠️ This command only shows *capability*, not whether the CDI spec is valid.

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

## Step 3 — Generate NVIDIA CDI specification (manual)

Generate the CDI spec once manually:

```bash
sudo mkdir -p /etc/cdi
sudo nvidia-ctk cdi generate --output=/etc/cdi/nvidia.yaml
```

Verify:

```bash
nvidia-ctk cdi list
```

Expected:

```text
nvidia.com/gpu=all
nvidia.com/gpu=0
```

---

## Step 3.1 — IMPORTANT: Why CDI must be regenerated on boot

On many systems (especially laptops with iGPU + NVIDIA dGPU):

* `/dev/dri/cardX` numbering is **not stable**
* Device order may change on every reboot
* CDI specs contain **hard-coded device paths**

This can cause errors like:

```
failed to stat CDI host device /dev/dri/cardX
```

➡️ **This is expected behavior, not a bug.**

The correct fix is to **regenerate CDI on every boot**.

---

## Step 3.2 — Enable automatic CDI generation (RECOMMENDED)

Create a systemd service:

```bash
sudo nano /etc/systemd/system/nvidia-cdi.service
```

```ini
[Unit]
Description=Generate NVIDIA CDI device spec
After=multi-user.target

[Service]
Type=oneshot
ExecStart=/usr/bin/mkdir -p /etc/cdi
ExecStart=/usr/bin/nvidia-ctk cdi generate --output=/etc/cdi/nvidia.yaml

[Install]
WantedBy=multi-user.target
```

Enable it:

```bash
sudo systemctl daemon-reload
sudo systemctl enable nvidia-cdi.service
```

This ensures:

* CDI always matches current device layout
* Containers survive reboots
* No manual intervention required

---

## Step 4 — Disable legacy NVIDIA OCI hooks (if present)

Legacy hooks **must not coexist with CDI**.

Check:

```bash
ls /usr/share/containers/oci/hooks.d/
```

### If `oci-nvidia-hook.json` exists

```bash
sudo mv /usr/share/containers/oci/hooks.d/oci-nvidia-hook.json \
        /usr/share/containers/oci/hooks.d/oci-nvidia-hook.json.bak
```

Restart Podman:

```bash
systemctl --user restart podman
```

### If no hook exists

That is normal — no action required.

---

## Step 5 — Recreate the Distrobox container (mandatory)

⚠️ Containers **created with `--nvidia` must be recreated**.

```bash
distrobox stop <name>
distrobox rm <name>
```

Create a CDI-based container:

```bash
distrobox create \
  --name <name> \
  --image <image> \
  --additional-flags "--device nvidia.com/gpu=all --security-opt=label=disable"
```

Rules:

* ❌ Do NOT use `--nvidia`
* ❌ Do NOT enable legacy hooks
* ✅ Use CDI only

---

## Step 6 — Verify GPU access inside container

```bash
distrobox enter <name>
```

```bash
nvidia-smi
```

Expected:

* Instant output
* No delay or hang
* GPU visible

---

## Optional — Lightweight GPU warm-up (recommended for laptops)

Avoid persistence mode.

### Fish

```fish
alias warm-gpu 'nvidia-smi > /dev/null 2>&1'
funcsave warm-gpu
```

### Bash / Zsh

```bash
alias warm-gpu='nvidia-smi > /dev/null 2>&1'
```

Usage:

```bash
warm-gpu
distrobox enter <name>
```

---

## When to use CDI vs when NOT to

### Use CDI when:

* You need CUDA / Vulkan / OpenGL inside containers
* You use Distrobox with Podman
* You want fast, predictable startup

### Do NOT use CDI when:

* GPU is not required
* Container is CPU-only
* You rely on legacy Docker-only workflows

### Never use:

* `distrobox --nvidia`
* Mixed CDI + OCI hooks
* Manual editing of `/etc/cdi/*.yaml`

---

## Summary

* No legacy hooks
* No `--nvidia`
* Auto-regenerated CDI
* Reboot-safe
* Kernel-update safe
* Laptop-friendly

This is the **cleanest and most future-proof** NVIDIA + Distrobox setup on Arch-based systems.

---

## End notes

Tested on real systems with:

* NVIDIA proprietary drivers
* Podman 5.x
* Distrobox 1.8+

Adapt freely for other distributions or container workflows.
