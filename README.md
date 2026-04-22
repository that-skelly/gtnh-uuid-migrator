# Better Questing UUID Migrator

A small CLI tool for migrating Better Questing progress from one player UUID to another.

This exists because the common one-off scripts floating around only cover one storage layout. This tool supports both of the layouts people are most likely to hit:

- **Legacy single-file layout** using `betterquesting/QuestProgress.json`
- **GTNH / per-player layout** using `betterquesting/QuestProgress/<uuid>.json`

It can also optionally update `QuestingParties.json` so the target UUID is added to the same party membership as the source UUID.

## What it does

### Legacy single-file mode
If your server stores quest progress in a single `QuestProgress.json`, the tool rewrites every internal reference from the source UUID to the target UUID inside that file.

### Per-player mode
If your server stores quest progress in `QuestProgress/<uuid>.json`, the tool:

1. loads the source player's quest progress file
2. rewrites every internal occurrence of the source UUID to the target UUID
3. writes the result to the target player's quest progress file

That solves the exact GTNH-style problem where renaming or copying the file alone is not enough because the old UUID is still embedded all over the JSON.

## Features

- automatic layout detection
- safe `.bak` backups by default
- dry-run mode
- optional party membership sync
- optional full UUID replacement inside `QuestingParties.json`

## Requirements

- Python 3.9+

## Quick start

### 1. Back up your world first
Do this before touching Better Questing files.

### 2. Stop the server
Do not edit live Better Questing files while the server is running.

### 3. Run the tool

#### GTNH / per-player layout

```bash
python -m bq_uuid_migrator.cli \
  /path/to/world/betterquesting \
  11111111-1111-1111-1111-111111111111 \
  22222222-2222-2222-2222-222222222222
```

This will detect `QuestProgress/<source_uuid>.json`, rewrite internal UUID references, and write the migrated result to `QuestProgress/<target_uuid>.json`.

#### Legacy single-file layout

```bash
python -m bq_uuid_migrator.cli \
  /path/to/world/betterquesting/QuestProgress.json \
  11111111-1111-1111-1111-111111111111 \
  22222222-2222-2222-2222-222222222222
```

## Party membership

### Add the target UUID to any party containing the source UUID

This is the safer option if you want both accounts to remain in the party.

```bash
python -m bq_uuid_migrator.cli \
  /path/to/world/betterquesting \
  11111111-1111-1111-1111-111111111111 \
  22222222-2222-2222-2222-222222222222 \
  --sync-party-membership
```

### Replace the source UUID inside `QuestingParties.json`

Use this only if you are fully migrating away from the source account.

```bash
python -m bq_uuid_migrator.cli \
  /path/to/world/betterquesting \
  11111111-1111-1111-1111-111111111111 \
  22222222-2222-2222-2222-222222222222 \
  --replace-party-uuid
```

## Dry run

```bash
python -m bq_uuid_migrator.cli \
  /path/to/world/betterquesting \
  11111111-1111-1111-1111-111111111111 \
  22222222-2222-2222-2222-222222222222 \
  --sync-party-membership \
  --dry-run
```

## Windows example

```powershell
py -m bq_uuid_migrator.cli `
  C:\path\to\your\world\betterquesting `
  11111111-1111-1111-1111-111111111111 `
  22222222-2222-2222-2222-222222222222 `
  --sync-party-membership
```

## Output examples

### Per-player mode

```text
Mode: per-player-file
Progress target: C:\path\to\your\world\betterquesting\QuestProgress\11111111-1111-1111-1111-111111111111.json
Replaced 1518 UUID references in source player file.
Wrote migrated progress to: C:\path\to\your\world\betterquesting\QuestProgress\22222222-2222-2222-2222-222222222222.json
Done.
```

## Safety notes

- Always back up your `betterquesting` folder first.
- Keep the server offline while migrating files.
- Review the `.bak` files before deleting them.
- If your client still looks desynced after migration, reconnect and use `/bq_user refresh`.

## License

MIT
