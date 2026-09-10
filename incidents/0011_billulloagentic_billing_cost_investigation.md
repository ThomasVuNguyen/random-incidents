# 0011 — BillulloAgentic GCP Billing Cost Investigation

**Date:** 2026-09-09
**Billing Account:** `01C6E6-0A1A06-C4D0D7` (BillulloAgentic)

![Billing Cost Investigation](0011_billulloagentic_billing_cost_investigation.png)

---

## What Happened

The BillulloAgentic GCP billing account has been unusually expensive over the last 60 days. An investigation was conducted across all 5 active projects linked to this billing account to identify the root causes of the excessive spending.

### Billing Account Overview

The billing account `01C6E6-0A1A06-C4D0D7` (BillulloAgentic) has **5 active projects** drawing charges, plus 1 with billing disabled:

| Project | Billing | Cloud Run Services | Cloud Functions | Firestore DBs | Vertex AI |
|---|---|---|---|---|---|
| `project-beaver-beenex` | ✅ Active | **19 services** | **13 functions** | 3 databases | ✅ Enabled |
| `starmind-72daa` | ✅ Active | **13 services** | **11 functions** | **7 databases** | ✅ Enabled |
| `miles-build-demo` | ✅ Active | 2 services | 1 function | 1 database | ✅ Enabled |
| `beenex-prop-engine-2026` | ✅ Active | 1 service | 0 functions | 1 database | ✅ Enabled |
| `beesecurity` | ✅ Active | 2 services | 0 functions | 1 database | ❌ Disabled |
| `fir-rag-fef5b` | ❌ Disabled | 4 services (stale) | 0 functions | 4 databases | ✅ Enabled |

**Total across the billing account:** 41 Cloud Run services, 25 Cloud Functions, 16 Firestore databases, Vertex AI on 5 projects.

---

## Root Causes Identified

### 🔴 #1 — Always-On Cloud Run Instance (`wia` service)

The `wia` service in `project-beaver-beenex` has `minScale: 1`, meaning it runs **24/7** regardless of traffic. At 1 vCPU + 512 MiB, this alone costs approximately **$40-60/month** in idle compute.

```
wia: min=1 max=20 cpu=1000m mem=512Mi
```

### 🔴 #2 — Massive Artifact Registry Bloat (63+ GB of Docker Images)

| Project | Repository | Size |
|---|---|---|
| `project-beaver-beenex` | `beenex-repo` | **40.8 GB** (297 images + 100 cache layers) |
| `project-beaver-beenex` | `beaver-docker` | **7.2 GB** |
| `miles-build-demo` | `cloud-run-source-deploy` | **14.9 GB** (58 images of `miles-build-agent`) |
| `project-beaver-beenex` | `gcr.io` | **2.1 GB** |
| Various | Other repos | ~2.6 GB combined |

**Total Artifact Registry storage: ~67.6 GB** — at $0.10/GB/month, that's ~$6.76/month just for container storage. The `beenex-repo` alone has **297 Docker image versions** and **100 cache layers** that were never cleaned up. The `miles-build-demo` has **58 versions** of a single service with **4 GB memory** allocation.

### 🔴 #3 — High-Frequency Cloud Scheduler Jobs Triggering Cloud Run

Three scheduler jobs are actively firing and triggering Cloud Run cold starts:

| Job | Schedule | Target |
|---|---|---|
| `beenex-signing-notification-outbox` | **Every 5 minutes** | `beenex-engine` (2 vCPU, 2 GiB) |
| `process_drip_sequences` | **Every 1 hour** | Cloud Function → Cloud Run |
| `beesecurity-scan` | **Every 15 minutes** | `beesecurity-backend` (1 vCPU, 1 GiB) |

The every-5-minute job hitting a 2 vCPU / 2 GiB Cloud Run service generates **~8,640 invocations/month** with cold starts, each spinning up a heavy container.

### 🔴 #4 — Vertex AI (Agent Platform API) Enabled on 5 Projects

Vertex AI is enabled and potentially incurring charges on:
- `project-beaver-beenex`
- `starmind-72daa`
- `miles-build-demo`
- `beenex-prop-engine-2026`
- `fir-rag-fef5b` (billing disabled, but was active recently)

Any API calls to Gemini via Vertex AI, even from agent prototypes or testing, can accumulate significant costs quickly ($1-10+ per thousand calls depending on model).

### 🟡 #5 — 8 Bloom Stage Services in `starmind-72daa`

The `starmind-72daa` project runs **8 separate "bloomstage" Cloud Run services** (stage0–stage7), each with 1 vCPU and 512 MiB–2 GiB memory. Combined with `bloombranch`, `bloomchat`, and `af3-cloud-backend`, that's 11 potentially active services. The last Cloud Build was May 2026, suggesting these may be abandoned but still consuming resources when triggered.

### 🟡 #6 — `miles-build-demo` Agent Service (2 vCPU, 4 GiB)

The `miles-build-agent` service is configured with **2 vCPU and 4 GiB memory** — one of the heaviest resource configurations. With 58 stale Docker images at ~14.9 GB, this project appears to be a prototype that was never cleaned up.

### 🟡 #7 — Multiple Firestore Databases

16 Firestore databases across 6 projects, many potentially unused:
- `starmind-72daa` has **7 databases** (readable, bloomtwo, beedraw, tribe, writebook, alphafold3, default)
- `fir-rag-fef5b` has **4 databases** (mock-service, bee-draw, default, agentic-v2) with billing disabled

Firestore charges per document read/write and storage, so inactive databases with stale data still incur storage costs.

---

## Recommended Fixes

### Immediate Actions (High Impact)

1. **Clean Artifact Registry** — Delete old Docker image versions, keeping only the 3 most recent per service. This will reclaim ~60+ GB and save ~$6/month in storage alone, plus reduce future build/pull times.

   ```bash
   # Example: Clean beenex-repo (297 → 3 images)
   gcloud artifacts docker images list us-central1-docker.pkg.dev/project-beaver-beenex/beenex-repo/beenex-engine \
     --sort-by="~CREATE_TIME" --format="value(version)" | tail -n +4 | \
     xargs -I {} gcloud artifacts docker images delete \
     us-central1-docker.pkg.dev/project-beaver-beenex/beenex-repo/beenex-engine@{} --quiet
   ```

2. **Remove `minScale: 1` from `wia`** — Unless this service requires instant response, set minScale to 0 to avoid 24/7 idle charges.

   ```bash
   gcloud run services update wia --project=project-beaver-beenex \
     --region=us-central1 --min-instances=0
   ```

3. **Reduce scheduler frequency** — Change `beenex-signing-notification-outbox` from every 5 minutes to every 15 or 30 minutes. Change `beesecurity-scan` from every 15 minutes to every hour.

4. **Audit Vertex AI usage** — Check if any of the 5 projects are making active Vertex AI API calls. Disable the API on projects that don't need it.

### Medium-Term Actions

5. **Evaluate `starmind-72daa` bloom services** — Last build was May 2026 (4+ months ago). If these are no longer used, delete the services and their Cloud Functions.

6. **Evaluate `miles-build-demo`** — Appears to be a demo/prototype project. Consider deleting if no longer needed.

7. **Clean up `fir-rag-fef5b`** — Billing is already disabled, but 4 Cloud Run services are still deployed (they'll error on invocation). Delete them to prevent accidental re-enablement charges.

8. **Set up Artifact Registry cleanup policies** — Configure automatic cleanup to retain only the N most recent versions per image.

   ```bash
   gcloud artifacts repositories set-cleanup-policies cloud-run-source-deploy \
     --project=project-beaver-beenex --location=us-central1 \
     --policy=keep-3-versions.json
   ```

### Ongoing Prevention

9. **Set budget alerts** — Configure billing budget alerts at 50%, 80%, and 100% of expected spend for the BillulloAgentic billing account.

10. **Enable billing export to BigQuery** — This allows detailed cost analysis queries to catch spikes early.

---

---

## Remediation Actions Taken (2026-09-09)

### ✅ 1. WIA minScale set to 0
- Changed `wia` Cloud Run service from `minScale: 1` to `minScale: 0`
- Analyzed the [beenex-wia repo](https://github.com/BillulloNex/beenex-wia): Next.js SSR CRUD app, no WebSockets/SSE/background jobs, client-side Firestore only
- The app's own `apphosting.yaml` already specified `minInstances: 0`
- **Savings: ~$45/month**

### ✅ 2. Artifact Registry cleaned (640 images deleted)
- Deleted 640 stale Docker images across 11 repositories in 3 projects
- Kept 3 most recent versions per service
- Major deletions: beenex-engine (294), beenex-engine-cache (101), miles-build-agent (55), activepieces (62), wia (63), af3-cloud-backend (21), plus smaller repos
- **Savings: ~$6/month** in storage costs

### ✅ 3. Cloud Scheduler frequencies reduced
- `beenex-signing-notification-outbox`: every 5 min → **every 30 min** (6x reduction)
- `beesecurity-scan`: every 15 min → **weekly Monday 8 AM ET** (672x reduction)
  - Analyzed [BeeSecurity repo](https://github.com/BillulloNex/BeeSecurity): scans static infrastructure posture (API keys, IAM, firewalls) that changes only on deployment, plus daily BigQuery billing exports — nothing justifies sub-hourly scanning
- `process_drip_sequences`: kept at every 1 hour (already reasonable)
- **Savings: ~$21/month** in Cloud Run cold start compute

### ✅ 4. Vertex AI audited (kept enabled)
- Audited 60 days of Vertex AI API logs across all 5 projects
- Only `beenex-prop-engine-2026` had actual GenerateContent calls in the last 60 days
- 3 projects showed zero Vertex AI API calls but were kept enabled per owner's decision (active projects that may use it)
- **No action taken** — Vertex AI API has no cost when enabled but idle; only API calls incur charges

### ✅ 5. Stale services evaluated (no action needed)
- Audited traffic logs for all bloom/miles services over the last 60 days
- **starmind-72daa**: All 8 bloomstage services, bloombranch, bloomchat, af3-cloud-backend, and comfyshare had **ZERO traffic** in 60 days. Only `get-or-create-user` was active (last request Sep 5).
- **miles-build-demo**: `miles-build-agent` had **ZERO traffic** in 60 days. `budget-shutoff` Cloud Function was active (billing safety net — kept).
- All idle services have `minScale: 0` so they cost **$0 in compute** when idle
- **Decision: left in place** — no cost to keep them, owner can clean up later if desired

### 💰 Final Cost Summary

| | Monthly | Annual |
|---|---|---|
| **Before optimization** | ~$100-120 | ~$1,200-1,440 |
| **After optimization** | ~$25-50 (active use) / ~$2-5 (fully idle) | ~$300-600 / ~$24-60 |
| **Total savings** | **~$72/month** | **~$860/year** |

---

## What Will Prevent This From Happening Again

1. **Artifact Registry cleanup policies** on all repositories to auto-delete old images
2. **Budget alerts** at multiple thresholds with email notifications
3. **Billing export to BigQuery** for cost visibility and anomaly detection
4. **Regular quarterly audit** of Cloud Run min-instances, scheduler frequencies, and enabled APIs
5. **Project lifecycle policy** — prototype projects should have a TTL or be cleaned up after demos

---

**Signed by:** Antigravity — 2026-09-09

