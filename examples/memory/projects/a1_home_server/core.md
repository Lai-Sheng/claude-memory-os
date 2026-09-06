---
name: a1_home_server_core
description: 🚨 Zoned project — single entry point. Read the zone table first and load only the zone picked. Move everything off the Synology before its warranty ends 2026-06.
metadata:
  type: project
  tier: core
---

# A1 Home Server Migration — core (permanent layer + router)

**Started**: 2025-11-02

> 🚦 This project is **zoned**. It is four jobs wearing one name, so `state.md`
> was replaced by one `state.md` per zone. See
> [../../../../docs/srp-zoning.md](../../../../docs/srp-zoning.md).

---

## Why this project exists

The Synology DS918+ holds everything — photos, the media library, backups for
three machines, and the only copy of the pre-2019 archive. Its warranty ends
**2026-06** and one drive bay has already been flaky. This is not a hobby
upgrade; it is a deadline against a single point of failure.

## Definition of done

Every service runs on the new box, the Synology has been **wiped and powered
off for 30 days** with no one noticing, and the 3-2-1 backup rule holds without
it. Then this gets frozen.

---

## 🔒 Inviolable rules

- 🚨 **Never move the last copy of anything.** Copy, verify checksums, *then*
  delete. *Why*: I lost the 2016 photo folder to a "move" that half-completed on
  a dropped SMB mount. Restored from a backup that happened to be 3 weeks stale.
- 🚨 **One service migrated at a time, with a rollback path.** *Why*: I moved DNS
  and the reverse proxy in the same evening and spent it debugging without
  working DNS to look anything up.
- 🚨 **No service migrates before its backup is verified restoring**, not just
  running.

---

## 🔀 Zone router

> Pick a zone, then load **only** that zone. Do not load all four.

| Zone | Owns | File |
|---|---|---|
| 🧱 **Hardware** | The new box, drives, the RAID rebuild, physical install | `hardware/state.md` |
| 📦 **Services** | Per-service migration: Plex, Paperless, photos, backups | `services/state.md` |
| 🔐 **Networking** | DNS, reverse proxy, VLANs, remote access | `networking/state.md` |
| 💎 **Facts** | Verified facts every zone needs: drive serials, capacities, IPs, warranty dates | `facts/*.md` |

**Zoning rules**

1. **Facts sink** — a serial number, an IP, a capacity lives in `facts/`, once.
   The moment it exists in two zones they will disagree and I will not notice.
2. **State belongs to whoever drives it** — the RAID rebuild is Hardware's, even
   though I first hit it while trying to migrate Plex.
3. **Cross-zone links only, never copies** — `[[a1-facts-drives]]`, not a
   pasted table.

🚨 Sorting a request into a zone is your job. If I ask a Services question while
"in" Hardware, just answer it and note `📌 filed under Services` in one line.

## 🚧 Current blocker (whole project)

RAID rebuild running since 2026-01-12, ~40 h estimated. **Services zone is
blocked until it finishes** — nothing new gets written to the array mid-rebuild.
Hardware and Networking can proceed.

---

## Related memory

- `[[a1-facts-drives]]` — serials, capacities, purchase dates, warranty ends
