# 0001 — Mac mini and Coolify storage exhaustion

**Date:** 2026-08-21

**Status:** Resolved

**Service:** `https://cloud.comfyspace.tech`

**Host:** Mac mini

**Runtime:** Production Lima VM

![Mac mini Coolify storage incident](0001_mac_mini_coolify_storage_exhaustion.png)

## What happened

The public Coolify control plane began returning HTTP 500. DNS, TLS, Cloudflare, and the tunnel edge were reachable, which localized the failure to the Mac mini or the Coolify runtime behind it.

Both storage layers were exhausted:

- The macOS Data volume had only **864 MiB available** and reported 100% capacity.
- The production Lima VM had only **739 MiB available** on its 77 GiB root filesystem and also reported 100% usage.
- Docker held 65.91 GB of images, including 37.55 GB reclaimable, plus 11.73 GB of build cache.
- The nearly full macOS volume caused the running Lima guest to degrade into filesystem I/O errors.
- After a clean stop and restart attempt, Ubuntu could not complete boot or reach SSH.
- An offline filesystem check confirmed ext4 corruption: a corrupted orphan inode list, a deleted inode with invalid deletion time, and an extent tree requiring repair.

The storage incident therefore had two related causes: disposable macOS data had consumed the host's remaining headroom, while unused Docker images and build cache had filled the Coolify guest. Operating at effectively zero free space then damaged the guest filesystem.

## What Codex did to fix it

### 1. Restored host headroom

Removed approximately 17 GiB of regenerable browser, developer-tool, virtualization, and updater caches. No Coolify application data, databases, Docker volumes, backups, downloads, or personal documents were touched during this recovery step.

### 2. Diagnosed and repaired the Coolify VM filesystem

- Confirmed the correct production Lima VM without changing a separate non-production VM.
- Stopped the failed production VM.
- Inspected its 80 GiB raw disk read-only from a separate Linux environment.
- Installed `e2fsprogs` on macOS, attached the raw Lima disk with `hdiutil`, and ran an offline `e2fsck` repair against the Linux root partition.
- Ran a second read-only filesystem check before detaching the disk.
- Restarted the VM and confirmed that Ubuntu, SSH, Docker, and the Coolify containers came back normally.

### 3. Removed unused Docker storage and returned space to macOS

- Pruned unused Docker images and build cache while preserving running and stopped containers and all volumes.
- Ran `fstrim` inside the guest so the freed virtual blocks were returned to the sparse Lima disk on macOS.
- This changed the first recovery checkpoint to:
  - macOS: **864 MiB free → 70 GiB free**
  - Coolify VM: **739 MiB free → 53 GiB free**

### 4. Removed an abandoned application

- Deleted the abandoned resource from the live Coolify inventory.
- Verified its containers, network, and image were gone.
- Found three volumes that remained after Coolify's queued deletion and explicitly removed its PostgreSQL, MinIO, and Valkey volumes.
- Confirmed the origin route no longer served the application. Cloudflare could still return a cached copy of the old frontend, but the app and origin workload were gone.

### 5. Performed the approved computer cleanup

After reviewing the Mac's storage inventory with the operator, permanently removed an obsolete development VM, unused local AI models, disposable browser and developer-tool data, and Docker volumes confirmed to be unused. Production data and unrelated virtual machines were preserved.

### 6. Verified the recovery

- `https://cloud.comfyspace.tech` followed its expected redirect and returned HTTP 200 at `/login`.
- All six core Coolify services reported healthy.
- macOS finished with **93 GiB free**.
- The Coolify VM finished with **59 GiB free**.
- Docker finished with only six active volumes and no reclaimable orphan volumes.

## What should prevent this from happening again

1. Monitor the macOS Data volume and the Coolify guest root filesystem as separate storage layers. Alert at 70%, 80%, and 90% utilization.
2. Keep at least 15–20% free space on both the physical host and the VM.
3. Schedule conservative Docker cleanup for unused images and build cache. Do not automatically prune volumes because they may hold database data from recoverable or temporarily stopped applications.
4. Configure Docker JSON log rotation and alert on rapidly growing container logs.
5. Run or verify periodic `fstrim` in the Lima guest so deleted guest blocks are returned to macOS.
6. Review old UTM/Lima VMs, local AI models, browser profiles, and developer caches quarterly.
7. Keep Coolify database and application backups on storage outside the Lima system disk.
8. Treat filesystem I/O errors after a disk-full condition as possible filesystem corruption; stop the VM and perform an offline check before repeated restart attempts.

---

**Solved and documented by Codex — 2026-08-21**
