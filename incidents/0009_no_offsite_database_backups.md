# 0009 — No Offsite Database Backups Across Coolify Infrastructure

![Backup Disaster Recovery](0009_no_offsite_database_backups.png)

## What happened

A backup audit of the entire Coolify infrastructure (5 servers, 18 projects, 6 databases, 15 applications, 11 services) revealed critical gaps:

- **4 of 5 PostgreSQL databases had zero backup schedules** — comfymail-postgres, ymv2-postgres, metamcp-db, metamcp-postgres
- **The one database with backups (peopledb-postgres)** had daily local-only dumps stored on the same server — useless if the server is lost
- **No S3 or offsite storage** was configured anywhere in Coolify
- **No Redis backup** for comfymail-redis
- **No Coolify instance backup** (configuration, env vars, domains, secrets)

In a total loss scenario, all database contents, service data (Twenty CRM, Mautic, Activepieces), environment variables, and persistent volumes would be permanently destroyed. Only application source code (in GitHub) would survive.

### Critical databases at risk

| Database | Purpose | Risk |
|----------|---------|------|
| ymv2-postgres | Custom EHR (patient data) | 🔴 Critical — zero backups |
| comfymail-postgres | Email system | 🔴 Critical — zero backups |
| peopledb-postgres | People Intelligence DB | 🟡 High — local-only backups |
| metamcp-db | MCP aggregator | 🟠 Medium — zero backups |
| metamcp-postgres | MCP aggregator (legacy) | ⚫ Down + zero backups |

## What was done to fix it

### 1. Created GCS Backup Destination
- Created a GCS bucket `comfyspace-coolify-backup` (Nearline, US multi-region) in the Starmind project
- Created a dedicated service account `coolify-backup@starmind-72daa.iam.gserviceaccount.com` with `storage.objectAdmin` on the bucket
- Generated HMAC keys (S3-compatible credentials) via the GCS Interoperability console (org policy blocked CLI key creation)
- Added GCS as an S3-compatible storage destination in Coolify (UUID: `juck6ll3ia1bw7ol1mk3s86y`)

### 2. Created Backup Schedules for All Databases
All configured with daily backups at midnight UTC (`0 0 * * *`), local + S3 upload:

| Database | Action | Backup UUID | S3 Enabled | Retention (local) | Retention (S3) |
|----------|--------|-------------|------------|-------------------|----------------|
| comfymail-postgres | Created new | `lzowl5m9gtqg8hst1yywz4jm` | ✅ | 7 days / 7 copies | 30 days / 30 copies |
| ymv2-postgres | Created new | `p1xzzqugmqfjkbkkm6z2220c` | ✅ | 7 days / 7 copies | 30 days / 30 copies |
| metamcp-db | Created new | `uwqoewhhmud63r7as16h4d8n` | ✅ | 7 days / 7 copies | 30 days / 30 copies |
| metamcp-postgres | Created new | `aox8ld3mn5wcjdiwvyjbvqzt` | ✅ | 7 days / 7 copies | 30 days / 30 copies |
| peopledb-postgres | Updated existing | `kyjylpiu2lck397fz1g8t9p7` | ✅ (was ❌) | 7 days / 7 copies | 30 days / 30 copies |

## What will prevent this from happening again

1. **Offsite backups are now automatic** — all PostgreSQL databases back up daily to GCS, surviving a total server loss
2. **30-day S3 retention** — provides a month of point-in-time recovery options
3. **The first backup will run at midnight UTC tonight** — verify tomorrow that all 5 databases show `success` status and `s3_uploaded` is true
4. **Remaining gaps to address**:
   - Redis backup (comfymail-redis) — Coolify doesn't support Redis backup scheduling; consider a custom cron
   - Coolify instance backup — export `/data/coolify/` periodically
   - Service data (Twenty CRM, etc.) — volume snapshots needed
   - Rotate the HMAC secret that was exposed during configuration

---

**Signed:** Antigravity · 2026-09-05
