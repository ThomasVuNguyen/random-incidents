# 0013 — Compute Engine RDP session establishment timeout

## What Happened

Remote Desktop connections to the Windows Compute Engine VM `platinum-it-windows-6cpu-24gb-prod` stalled at **Detecting network quality** and failed with Windows App error `0x108`: the session could not be established in time.

The VM itself was running and reachable. TCP port 3389, the RDP negotiation, and the TLS handshake all succeeded consistently. Windows also accepted the tested credentials, but each attempted session was torn down after roughly 90 seconds. Windows Remote Desktop Services logged event 20498, stating that it took too long to complete the client connection.

The failure was inside the VM's Remote Desktop Services session-establishment path, not the public network, firewall, credentials, disk capacity, or RDS licensing. The strongest service-level symptom was `UmRdpService` becoming stuck in `STOP_PENDING` while a targeted RDP service restart was attempted.

## What Was Done

1. Confirmed the VM was `RUNNING`, the external IP was reachable on TCP 3389, and the GCP firewall allowed RDP.
2. Proved that RDP protocol negotiation and TLS completed successfully from the client.
3. Correlated Windows authentication and Remote Desktop event logs: credentials were accepted, but the server timed out while completing the session.
4. Verified that disk capacity and Remote Desktop licensing were healthy. The server had available RDS licenses and no configured session ceiling.
5. Attempted the least disruptive recovery first by restarting the RDP-related services. `SessionEnv` restarted, but `UmRdpService` remained stuck and `TermService` could not restart cleanly while its dependent service was hung.
6. Rebooted the VM to clear the wedged RDP stack.
7. Verified the recovery with real Remote Desktop logons for both `thomas` and `ptuser06`. Windows created session ID 5 for `thomas` and session ID 6 for `ptuser06`; other users also established active RDP sessions after the reboot.
8. Confirmed `TermService`, `SessionEnv`, and `UmRdpService` were all running, then removed the temporary recovery services and closed the diagnostic tunnels.

## What Will Prevent This From Happening Again

1. Alert on repeated Terminal Services event 20498 or multiple RDP attempts that authenticate but fail to create a shell.
2. Keep the Windows VM on a controlled patch-and-reboot schedule so long-lived RDP service state is periodically cleared.
3. Use this recovery order: verify TCP/TLS and authentication, attempt a targeted RDP service restart, then perform a controlled reboot if `UmRdpService` is stuck or sessions still time out.
4. Do not reset user passwords or delete profiles when authentication succeeds; those actions would not repair this failure and could disrupt shared-user state.

---

**Signed:** Codex  
**Date:** 2026-09-11
