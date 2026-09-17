# 0015 — Large ingestion exhausted VM memory headroom

**Date:** 2026-09-12

**Service:** `*.beenex.cloud` application host  
**Status:** Resolved

![A memory-constrained server stabilized by a separate swap safety buffer](0015_ingestion_memory_pressure_swap_safety_net.png)

## What happened

During a large data-ingestion job, the shared application VM repeatedly approached its 15.6 GiB physical-memory ceiling. A live inspection found only about 322–386 MiB available, approximately 14.6 GiB of anonymous memory, and no swap space.

The machine was not CPU-bound. Memory pressure forced aggressive reclaim and storage activity: 26 processes were blocked in uninterruptible I/O sleep, short-window full memory pressure exceeded 60%, full I/O pressure exceeded 80%, and the CPU spent roughly 85–89% of sampled time waiting on I/O. The largest consumers included Twenty's Redis container at about 5.4 GiB and the Teable application at about 3.2 GiB, both without container memory limits.

This explains why the VM could accept network connections while failing to complete SSH and HTTPS handshakes. The active ingestion made changing Redis limits or eviction behavior unsafe because that could reject writes, evict required data, or interrupt queued work.

After the immediate mitigation, the owner confirmed that Twenty was retired and authorized a complete wipe. The owner had also deleted Teable through Coolify and asked for an independent host-level residue check.

## What I did to fix it

- Added a dedicated 16 GiB swap file with owner-only permissions.
- Activated it immediately and added it to the boot filesystem table so it persists after restart.
- Set `vm.swappiness=10` at runtime and in a persistent system configuration, keeping swap as a low-priority safety buffer rather than normal working memory.
- Backed up the prior boot filesystem table before changing it.
- Left Redis configuration, container limits, and the ingestion rate unchanged while the live job continued.
- Verified the swap file was active, the persistent mount configuration passed validation, and the VM began using the buffer.

Immediately after mitigation, available memory rose to about 926 MiB, blocked processes fell from 26 to 1, short-window full memory pressure dropped to about 11%, and short-window full I/O pressure dropped to about 13%. All eight affected public hostnames returned their expected application, authentication, redirect, or unassigned-route responses in three consecutive checks.

After the owner confirmed Twenty was no longer used, I permanently removed its four orphaned containers, PostgreSQL data volume, Redis data volume, local application-storage volume, application image, and the protected rate-limit backup. The old Coolify resource and Compose directory were already absent, so deletion was performed against the exact live Docker objects after validating that none of the volumes were shared with another workload.

The follow-up Teable audit found no remaining Coolify project or resource, Docker container, volume, network, image, or deployment path. Its retired `data.beenex.cloud` route returned the expected Traefik 404. The same proof was obtained for Twenty at `20.beenex.cloud`, while unaffected public applications continued returning HTTP 200.

After cleanup, VM memory use fell from about 13 GiB to 4 GiB, available memory increased to about 10 GiB, swap use fell to about 376 MiB, and no processes remained blocked on I/O.

## What should prevent it next time

- Keep the 16 GiB low-swappiness buffer enabled to avoid abrupt exhaustion, but do not treat swap as additional application capacity.
- Alert on available memory, swap growth, memory pressure, I/O pressure, blocked tasks, and endpoint latency rather than RAM percentage alone.
- Remove retired workloads from both the Coolify control plane and the Docker host; a deleted metadata record is not proof that its containers and volumes are gone.
- Add evidence-based memory limits to remaining large containers only after confirming their peak working sets.
- Throttle future ingestion jobs if sustained swap use or I/O wait rises enough to affect interactive traffic.
- Consider a larger-memory VM if this ingestion profile is routine; heavy swap use would preserve processes at the cost of severe latency.

— Codex, 2026-09-12
