# Madrid Job Radar state

This repository stores the shared machine-readable state for the Madrid job radar.

It contains no personal profile data. It stores employer/job requisition state needed for deduplication and reconciliation across the scheduled Radar A, Radar B, Radar C, Safety Net and Daily Audit tasks.

## Shared source of truth

`state/requisitions.json` is the primary deduplication ledger.

A role is identified first by stable employer + requisition/job ID. If no stable requisition exists, use employer + canonical official role URL. A task-history mention is secondary only and must not override this ledger.

Each scheduled radar should:

1. Fetch `state/requisitions.json` before classifying a candidate as new
2. Merge observations by `duplicate_key`
3. Preserve existing records when an employer source fails or renders ambiguously
4. Never reset age merely because LinkedIn reposted the same requisition
5. Write new/updated records back after the scan
6. Use GitHub optimistic concurrency: fetch the file and blob SHA, then update with that SHA
7. If the update is rejected because the SHA changed, refetch once, merge its own changes into the newest file, and retry once

## Important state rules

- Missing from a task's chat history does not mean new
- Present in the ledger means previously known unless the official employer created a genuinely new stable requisition ID
- Reappearance of the same ID is normally not new
- Source failure is never equivalent to zero vacancies
- A closed/removed official role remains in the ledger with a closed/removed status rather than being deleted
- `alerted=true` prevents duplicate alerts for the same requisition unless a genuinely new official requisition ID is created
- The ledger is shared across all five radar tasks

The initial ledger was seeded from `Madrid_Job_Radar_Ledger_2026-09-06_v2.xlsx` on 2026-09-06.