---
name: loci-stats
description: >
  Show loci context savings statistics. Displays token consumption,
  context savings ratio, and per-tool breakdown.
  Trigger: /loci-stats, "loci stats", "context stats", "context savings"
---

# loci stats

Show context window savings for this session.

## Instructions

Run the following command and display the output:

```bash
loci stats --json
```

Format the JSON output as a readable summary:
- Total bytes intercepted vs returned
- Context savings ratio (percentage)
- Per-tool breakdown (which tools saved the most)
- Session duration and event count
