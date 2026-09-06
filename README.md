# Madrid Job Radar state

This repository stores the shared machine-readable state and configuration for the Madrid job radar.

It contains no personal profile data. It stores employer universe/ownership, job requisition state, and run health needed across Radar A, Radar B, Radar C, Safety Net and Daily Audit.

## Authoritative employer universe

`config/employers.json` is the only authoritative source for employer membership and A/B/C ownership.

Current validated universe:

- 198 employers total
- Radar A: 66
- Radar B: 66
- Radar C: 66
- 198 unique employer IDs
- 198 unique employer names

The file was derived from `Madrid_Job_Radar_Source_Map_2026-09-06_v11.csv`. Its recorded SHA-256 identifies the exact source-map version used.

Every scheduled task must fetch and validate `config/employers.json` before scanning. A/B/C derive their employer lists strictly from the `radar` ownership field. Task/chat memory must never add, omit, rename or move an employer. If the master config is unavailable or invalid, emergency fallback may preserve coverage, but the run cannot be marked COMPLETE.

## Shared requisition ledger

`state/requisitions.json` is the primary deduplication ledger.

A role is identified first by stable employer + requisition/job ID. If no stable requisition exists, use employer + canonical official role URL. Task-history mentions are secondary and must not override this ledger.

Each scheduled radar should:

1. Fetch and validate `config/employers.json`
2. Fetch `state/requisitions.json` before classifying a candidate as new
3. Merge observations by `duplicate_key`
4. Preserve existing records when an employer source fails or renders ambiguously
5. Never reset age merely because LinkedIn reposted the same requisition
6. Write only material new/updated requisition state
7. Use GitHub optimistic concurrency: fetch file + blob SHA, update with that SHA, and on conflict refetch/merge/retry once

## Run heartbeats

Run state is stored under `state/runs/` for Radar A, Radar B, Radar C, Safety Net and Daily Audit.

A/B/C must record the config SHA used, maintain recent run history, account for every one of their 66 config-owned employers, and only mark COMPLETE when coverage and ledger checks succeed. Safety Net acts as watchdog/recovery. Daily Audit reviews the cross-task history and the complete 198-employer universe.

## Important rules

- Missing from task/chat history does not mean new
- Present in the ledger means previously known unless the official employer created a genuinely new stable requisition ID
- Reappearance of the same ID is normally not new
- Source failure is never equivalent to zero vacancies
- Request failure is not automatically source failure
- Known PARTIAL/WEAK/BROKEN source behavior is handled separately from unexpected healthy-source failures
- A closed/removed official role remains in the ledger with updated status rather than being deleted
- `alerted=true` prevents duplicate alerts for the same requisition unless a genuinely new official requisition ID is created
- A later successful run never erases evidence of an earlier missing or incomplete run

The initial requisition ledger was seeded from `Madrid_Job_Radar_Ledger_2026-09-06_v2.xlsx` on 2026-09-06.