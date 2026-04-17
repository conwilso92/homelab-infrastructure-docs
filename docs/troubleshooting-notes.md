# Troubleshooting Notes

This page documents real problems encountered in the lab, along with the cause, fix, and lesson learned from each one.

---

## 1. TrueNAS web interface was not accessible after initial setup

### Problem
After installation, the server reported that the web interface could not be accessed, which blocked normal management from the browser.

### Likely Cause
The issue appeared to be related to network configuration and IP assignment during initial setup.

### Fix
I reviewed the network configuration, corrected addressing issues, and ultimately stabilized access by using a DHCP reservation for the server on the router.

### Lesson Learned
Infrastructure services need predictable addressing. In a home lab, DHCP reservation can provide stable management access without the overhead of manual static configuration.

---

## 2. Emby activation and metadata issues were caused by DNS problems

### Problem
Emby had issues with activation, metadata retrieval, and artwork population.

### Cause
The root problem was DNS resolution inside the application environment, not the app itself.

### Fix
I corrected DNS settings in the relevant environment and redeployed / reconfigured the app as needed.

### Lesson Learned
Application issues are not always application problems. When services fail to reach external resources, DNS should be one of the first things checked.

---

## 3. Download and media apps were not agreeing on file paths

### Problem
Apps in the media stack were producing path mismatch warnings and could not consistently find completed downloads or mapped media folders.

### Cause
Container host paths and internal container paths were not standardized across services.

### Fix
I aligned host path mappings and container mount paths so all related services referenced the same storage structure consistently.

### Lesson Learned
In containerized environments, consistent path mapping is critical. Even when services are technically running, bad path alignment can break automation between them.

---

## 4. Remote access took multiple attempts to get working cleanly

### Problem
External access was inconsistent while testing different approaches for remote connectivity.

### Cause
The issue was not one single failure point, but a combination of connectivity design choices, DNS behavior, and tunnel/service configuration.

### Fix
I moved toward a cleaner model using Tailscale for private access and Cloudflare Tunnel for selected external access use cases.

### Lesson Learned
Remote access design should be deliberate. Using modern overlay networking and tunnels can reduce complexity, but only if the roles of each tool are clearly defined.

---

## 5. Networking issues sometimes looked like service issues

### Problem
During troubleshooting, some apps appeared broken when the real issue was actually network or name resolution related.

### Cause
Upstream network problems, DNS behavior, or service mapping issues created symptoms inside the applications.

### Fix
I started troubleshooting by isolating layers: network, DNS, storage paths, then app configuration.

### Lesson Learned
The fastest way to waste time is to assume the visible symptom is the root cause. A layered troubleshooting approach is much more reliable.

---

## 6. Immich failed to install due to pgvecto_upgrade container crashing on startup

### Problem
When installing Immich from the TrueNAS app catalog, the deployment failed repeatedly. The app lifecycle log showed a generic error: `service "pgvecto_upgrade" didn't complete successfully: exit 1`. The Immich containers would start and then immediately die.

### Investigation

This turned into a deep troubleshooting session that touched permissions, ZFS ACLs, Docker behavior, and container internals. Here's the path it took:

**Initial theory — dataset permissions:**
The Immich postgres and library datasets were owned by root, but the postgres container runs as user 999. I changed ownership to match the expected UID. This didn't fix it.

**Second theory — run-as user mismatch:**
I had configured other apps to run as UID 3000 and tried matching Immich to the same pattern. Ownership was updated and verified, but the install still failed with the same error.

**Catching the container before it died:**
The pgvecto_upgrade container was crashing instantly, so I couldn't pull logs from it in time. I used `docker events --filter 'name=immich'` and `docker ps -a | grep immich` to try to catch it, but it self-destructed too quickly. Eventually I inspected the container image directly by running it manually with `sudo docker run --rm` and examining the upgrade script inside.

**Reading the upgrade script:**
Inside the postgres-upgrade image, the `/upgrade.sh` script runs three checks before proceeding: `check_writable` on the base directory, `check_writable` on `/var/run/postgresql`, and `check_dir_owner_match` on the base directory. If any of these fail, the script exits 1 — which is exactly what was happening.

**Third theory — ZFS ACL type blocking writes:**
After confirming the container runs as root and the ownership looked correct, I ran the upgrade container manually with the dataset mounted. The error was `chmod: changing permissions: Operation not permitted` and `mkdir: cannot create directory: Permission denied`. This pointed to a ZFS-level restriction rather than standard Unix permissions.

I checked the ZFS properties and found that the postgres dataset had `acltype=nfsv4` set — likely from when I had previously configured an SMB share on the Immich datasets. NFSv4 ACLs override standard Unix permissions, so even though `chown` showed correct ownership, the ACL was silently blocking writes.

**Fixing the ACL type:**
I switched the ACL type to POSIX on both the postgres dataset and the parent immich dataset, discarded the inherited ACL mode, and reset ownership. The `mkdir` permission error changed, which confirmed progress — but it still failed.

**Leftover directory from failed installs:**
The postgres dataset contained a leftover `18` directory from a previous failed install attempt. The upgrade script detected this directory, looked for a `PG_VERSION` file inside it, didn't find one, and exited. Removing the stale directory with `sudo rm -rf /mnt/tank/immich/postgres/18` cleared this issue.

**Still failing — parent dataset ACL inheritance:**
Even after cleaning the child dataset, the parent `tank/immich` dataset still had `acltype=nfsv4` and was owned by root. Since postgres is a child dataset, the parent's NFSv4 ACL was interfering. I fixed the parent dataset the same way — set acltype to POSIX, discarded aclmode, inherited aclinherit, and reset ownership and permissions.

**Final root cause — container UID mismatch:**
After all the permission and ACL fixes, the container was still failing. I inspected the Docker image directly and found that `/var/lib/postgresql` inside the container is owned by the `postgres` user, which runs as **UID 999** — not UID 568 (the TrueNAS apps user) as the documentation suggested. The dataset ownership needed to match UID 999 for the pgvecto_upgrade container to write to it.

### Fix

The final fix required three things:
1. Set ZFS ACL type to POSIX on both `tank/immich` and `tank/immich/postgres` datasets (and discard NFSv4 ACL settings)
2. Remove the leftover `postgres/18` directory from previous failed installs
3. Set dataset ownership to UID **999** (the postgres user inside the container) and update the User ID and Group ID in the TrueNAS Immich installer to 999

After making all three changes, the install completed successfully.

### Lesson Learned
This issue had multiple overlapping causes — ZFS ACL type, stale data from failed installs, and a UID mismatch between the documentation and the actual container. No single fix would have resolved it alone.

Key takeaways:
- ZFS ACL types (NFSv4 vs POSIX) can silently override Unix permissions and block container writes even when ownership looks correct
- Failed app installs can leave behind stale directories that interfere with future attempts
- Always verify the actual UID a container process runs as rather than trusting documentation — inspect the image directly if needed
- When a container dies immediately, inspecting the image manually (`docker run --rm`) is more reliable than trying to catch logs before it self-destructs
- Layered troubleshooting matters — this problem couldn't be solved by fixing any single layer in isolation
