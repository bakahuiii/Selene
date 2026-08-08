# Daily Combined JSON

Windows 0.5.4 creates a derived daily view after each synchronization
maintenance pass. It reads immutable `SELENE-v1-.../context-events.json`
snapshots from both the durable Android archive and the selected Windows export
directory, then writes:

```text
%LOCALAPPDATA%\SELENE\Archive\Merged\SELENE-merged-v1-YYYY-MM-DD.json
```

The date is the Windows system-local calendar date of each event's `startAt`.
An event is assigned to one day only. If an event has no parseable `startAt`,
the source snapshot's `generatedAt` is used as a conservative fallback.

## Preservation and Idempotency

- Android source snapshots remain in `Archive\SELENE-v1-*` and continue to be
  acknowledged and retained by the existing 24-hour cleanup policy.
- Windows source snapshots remain in the user-selected export directory.
- Daily files are derived output. They are atomically replaceable when late data
  arrives, and never used as a source for another merge.
- A failed or incomplete source scan leaves the previous healthy daily file
  unchanged and appears in the Windows sync status.

Events are deduplicated by their stable `id`. When the same ID appears in
overlapping snapshots, the copy with the newest `capturedAt` is retained. The
event object itself is not rewritten, rounded, or stripped of fields.

## Envelope

The daily file keeps the `selene-context-events/v1` schema and the required
`producer.layout: immutable-snapshot-v1`, so existing HYPERION importers remain
compatible. It adds optional metadata under `aggregation`:

```json
{
  "aggregation": {
    "schema": "selene-daily-merge/v1",
    "period": "day",
    "day": "2026-08-08",
    "timezone": "China Standard Time",
    "sourceSnapshotCount": 12,
    "sourceEventCount": 184,
    "uniqueEventCount": 176,
    "duplicateEventCount": 8
  }
}
```

`events` contains the original event JSON objects. Android `0.5.3` snapshots
are accepted without migration; the merged producer version is `0.5.4`.

## Verification

The Windows contract test verifies Android `0.5.3` input compatibility, mixed
Android/Windows source discovery, stable-ID deduplication, HYPERION schema
compatibility, and preservation of the Android source directory.
