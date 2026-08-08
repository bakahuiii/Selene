# Windows Local Daily Narrative

[English](DAILY_NARRATIVE.md) | [Simplified Chinese](DAILY_NARRATIVE.zh-CN.md)

SELENE Windows reads an already-built daily combined file and creates a more readable Chinese daily narrative locally. It uses deterministic rules only: no network access, cloud translation, or remote model. It never writes back to the combined JSON.

## Location and Use

Use **Generate local readable narrative** in the Windows sync area, or wait for the ten-second sync maintenance cycle. Input and generated files are:

```text
<data-root>/Archive/Merged/SELENE-merged-v1-YYYY-MM-DD.json
<data-root>/Archive/Narrative/SELENE-narrative-v1-YYYY-MM-DD.zh-CN.json
<data-root>/Archive/Narrative/SELENE-narrative-v1-YYYY-MM-DD.zh-CN.md
```

The narrative is rebuilt after new Android or Windows snapshots are archived, so late-arriving data appears in the next rebuild for that local day.

## Rules and Content

The JSON records the input filename and SHA-256, and reports:

- deduplicated-event and source-snapshot counts;
- total foreground-active time, leading applications, and activity spans of at least one minute;
- each application's source platform; matching names are explicitly labelled as phone or computer,
  for example `QQ (phone)` and `QQ (computer)`;
- local mapping of common application package names;
- location-sample and local place-tag counts;
- confirmed continuous movement segments, duration, distance, and distance-over-time average speed;
- the latest available device battery percentage.

Activity spans are de-jittered per application: adjacent records for the same
application with a gap of no more than two minutes are treated as one
continuous span, so brief cross-device sampling interleaves do not produce a
list of fragments.

Current combined files remain backward compatible: SELENE first reads an
event's `platform`; when it is absent, the stable-ID prefixes `SELENE-auto-*`
(Android) and `SELENE-windows-*` (Windows) identify the source. Unrecognised
historical or third-party events are labelled as an unspecified device instead
of being assigned to phone or computer incorrectly.

A segment with a maximum speed above `8 m/s` is labelled as a likely location jump. The peak is not presented as real walking speed; average speed continues to use the segment's recorded distance and duration. Narratives do not expose raw coordinates, window titles, executable paths, browser URLs, or other raw sensitive metadata.

## Data Boundary

Both `Archive/Merged` and `Archive/Narrative` are derived data. The immutable Android and Windows `SELENE-v1-*` snapshots remain the facts of record. Deleting a narrative only causes it to be rebuilt on the next maintenance pass or manual generation. Do not use it as a HYPERION import source or delete source snapshots from it.

Generation is serialized inside the Windows process. It completely writes and closes a temporary file before atomically replacing the target, avoiding races between manual generation and automatic maintenance. A briefly locked target is retried a bounded number of times; failure preserves the last healthy narrative and is reported in the SELENE UI and log.
