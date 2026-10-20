---
Hey all — sorry for the slow turnaround on this one. Big thanks to @SeldomNosePicker —
your diagnosis was exactly right and it cracked the whole thing open.

There were actually **two separate bugs**, and the May fix only addressed half of one:

1. **Permissions (the one you hit).** The container image runs as UID 1001 internally,
   but freshly-created appdata folders on Unraid are owned by root/nobody — so the
   first-boot model download silently failed with `PermissionError [Errno 13]`, leaving
   RapidOCR missing its files and crashing the container on boot. The template was also
   showing PUID/PGID fields that this image never actually used (my mistake — that's a
   LinuxServer convention docling-serve doesn't implement), which made things extra
   confusing.

2. **Broken image tags.** The template's cpu/cu128/cu126 branch selector pointed at image
   tags that don't exist upstream (only `latest` is published per variant), so picking
   the CPU branch could fail at pull.

**What changed (template update):**
- The template now runs the container with `--user 0`, so first-run downloads can write
  to appdata regardless of folder ownership. Fresh installs just work now.
- Branch tags fixed to `latest` on each image variant (base / cpu / cu128). The cu126
  branch is gone — upstream stopped publishing it.
- Removed the fake PUID/PGID fields and two obsolete env vars.

**If your install is currently broken**, either:
- Edit the container → add `--user 0` in Extra Parameters → restart, **or**
- From the Unraid host: `chown -R 1001:1001 /mnt/user/appdata/docling` then restart
  (exactly what @SeldomNosePicker did — `id` inside the container console confirms the UID).

First boot after that will download the models (~a few GB — keep `DOCLING_SERVE_ENABLE_REMOTE_SERVICES=true`
until the cache is populated, then you can flip it off for fully-offline operation).

Please re-install or update from the latest template and let me know if anyone still hits
download failures.