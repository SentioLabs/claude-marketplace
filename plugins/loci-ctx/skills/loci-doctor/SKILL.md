---
name: loci-doctor
description: >
  Check loci system health. Displays runtime availability, server status,
  database health, and configuration.
  Trigger: /loci-doctor, "loci doctor", "check loci health", "loci diagnostics"
---

# loci doctor

Run system health diagnostics.

## Instructions

Run the following command and display the output as a checklist:

```bash
loci doctor --json
```

Format each check as:
- ✓ Check name — detail (if passed)
- ✗ Check name — detail (if failed)

Checks include: server connectivity, database status, runtime availability
(JavaScript, Python, Shell, etc.), index health, and configuration validity.
