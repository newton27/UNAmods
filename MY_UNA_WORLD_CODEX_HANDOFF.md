# My UNA World — Codex Handoff

## Goal
Build **My UNA World v1.0** as a **Studio-only UNA module** for full-site backup/export.

The module must not expose member-facing pages or public menus. All controls belong in UNA Studio.

## Core requirements

1. **Studio-only module**
   - No frontend/member pages.
   - Studio dashboard for backup creation, history, diagnostics, settings, and installed-module inventory.

2. **World globe Studio icon**
   - Use a globe/world icon as the module's primary `std-icon.svg`.
   - The Studio tile should visually communicate site-wide backup/archive scope.

3. **Installed module discovery**
   - Read the authoritative UNA installed-module registry (`sys_modules`).
   - Inventory all installed modules on every backup run.
   - Track module name, title, version, enabled state, vendor/path where available.
   - Classify modules as dedicated support, generic support, no persistent data, or review-needed.

4. **Full database backup**
   - Back up the complete UNA database, not only known module tables.
   - Include schema and row data in a portable form.
   - Prefer SQL dump plus JSONL/JSON manifests where practical.
   - Never store or print database passwords.

5. **Module data coverage**
   - Maintain a per-module table inventory so unknown third-party modules are still captured by the full DB backup.
   - Dedicated adapters may provide richer exports for known modules.

6. **Original media/storage backup**
   - Preserve original physical files, not only resized/transcoded derivatives.
   - Use UNA storage metadata and storage paths to locate source files.
   - Include photos, videos, files, covers, avatars, attachments, and other module storage objects as applicable.
   - Log missing/orphaned files instead of failing the whole backup.

7. **Reuse proven album-export behavior**
   The earlier working album exporter used `bx_albums_files` fields such as:
   - `id`
   - `profile_id`
   - `path`
   - `file_name`
   - `mime_type`
   - `ext`
   - `size`
   - `added`
   - `private`

   It resolved candidate source paths from UNA storage metadata, copied originals, and wrote a manifest containing source and export paths. Preserve this philosophy in the module.

8. **Backup archive structure**
   Suggested archive layout:

   ```text
   my-una-world-YYYY-MM-DD-HHMMSS.zip
   ├── manifest.json
   ├── report.html
   ├── checksums.sha256
   ├── database/
   │   ├── full.sql
   │   ├── schema.json
   │   └── tables/*.jsonl
   ├── modules/
   │   ├── installed.json
   │   └── source/<vendor>/<module>/...
   ├── storage/
   │   └── ... original UNA storage files ...
   └── diagnostics/
       ├── missing-files.json
       ├── orphaned-records.json
       └── warnings.log
   ```

9. **Private backup location**
   - Store archives outside `public_html` when server layout permits.
   - Suggested default: one directory above document root, e.g. `<home>/my_una_world_backups/`.
   - If forced inside web root, protect with server access controls and show a Studio warning.

10. **Backup history**
    - Record backup ID, start/end timestamps, status, archive filename/path, size, checksum, module count, table count, file count, missing-file count, warnings/errors.
    - Provide Studio actions to inspect/download/delete backups.

11. **Integrity verification**
    - SHA-256 checksum manifest for exported files and final archive.
    - Verify archive can be opened before marking backup successful.

12. **Safety / resilience**
    - Backup should be read-only against existing site content.
    - Use temporary working directory, then atomic finalization/rename.
    - Clean temporary files after failure.
    - Avoid fragile fixed-schema assumptions where possible.
    - PHP 8.3 compatible.
    - Avoid dynamic-property deprecations.
    - Guard null/false/array assumptions.

13. **No restore in v1.0**
    - Export/backup only.
    - Design archive format so restore/import can be added later.

## Studio UX

Suggested sections:

- **Dashboard** — last backup, backup health, installed module count, storage/database size summary.
- **Create Backup** — run full backup, show progress/status.
- **Modules** — installed module inventory and coverage status.
- **Media & Storage** — storage objects/files found and missing-file diagnostics.
- **Backup History** — archives, status, size, checksum, actions.
- **Settings** — backup path, retention, compression, source-code inclusion, exclusions.
- **Diagnostics** — missing files, orphaned metadata, write-permission problems, unsupported storage drivers.

## Proposed module path/name

```text
modules/newton/my_una_world/
```

Suggested internal module name:

```text
newton_my_una_world
```

Suggested module title:

```text
My UNA World
```

## Suggested class structure

```text
modules/newton/my_una_world/
├── module.php
├── std-icon.svg
├── classes/
│   ├── MyUnaWorldModule.php
│   ├── MyUnaWorldConfig.php
│   ├── MyUnaWorldDb.php
│   ├── MyUnaWorldBackup.php
│   ├── MyUnaWorldArchive.php
│   ├── MyUnaWorldDatabaseExporter.php
│   ├── MyUnaWorldStorageExporter.php
│   ├── MyUnaWorldModuleScanner.php
│   ├── MyUnaWorldIntegrity.php
│   └── adapters/
│       ├── BaseAdapter.php
│       ├── AlbumsAdapter.php
│       ├── ForumAdapter.php
│       └── GenericAdapter.php
├── install/
│   ├── config.php
│   ├── install.sql
│   ├── uninstall.sql
│   └── langs/en.xml
├── studio/
│   └── ... Studio controllers/forms/templates ...
└── template/
```

Adjust class/file naming to match the UNA version and patterns already present in this repository.

## Existing reference behavior

The previously successful album export resolved originals from `bx_albums_files`, copied files by profile/file ID, and wrote CSV manifest rows with original filename, MIME, extension, size, privacy flag, timestamps, source path, and export path.

The previous forum export also used schema-adaptive detection for likely media tables/columns and exported readable JSON/CSV/HTML plus media. Reuse that defensive/adaptive style.

## Definition of done for first Codex pass

- Installable UNA module skeleton under `modules/newton/my_una_world/`.
- Globe `std-icon.svg` included.
- Studio entry visible and no frontend page exposed.
- Installed modules scan works.
- Full DB export works using UNA's configured DB connection without exposing credentials.
- Storage/media export foundation works and preserves originals.
- ZIP archive + manifest + checksums produced.
- Backup history persisted.
- PHP files lint clean on PHP 8.3.
- README with install/test instructions.
- No restore/import feature yet.

## Important implementation rule

Do not blindly assume table/page/storage schemas are identical across UNA versions. Inspect actual available schema/UNA APIs and degrade safely. The backup should prefer collecting too much over silently omitting a third-party module's persistent data, while avoiding transient caches/logs where clearly unnecessary.
