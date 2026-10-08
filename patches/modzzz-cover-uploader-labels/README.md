# Modzzz cover-upload label repair

Opening the News or Announcements creation form on UNA 15.0.0-RC2 produced
`Undefined array key "form_field_covers_uploader_simple"` and
`Undefined array key "form_field_covers_uploader_html5"` in
`modules/base/text/classes/BxBaseModTextFormEntry.php`, lines 32–33.

Both installed modules already contained the corresponding translated
"Upload" labels. Their configuration did not expose those labels through
the `CNF['T']` names required by UNA's shared text form.

The patch adds the two missing mappings to each module's configuration:

- `modules/modzzz/news/classes/MzNewsConfig.php`
- `modules/modzzz/announcements/classes/MzAnnouncementsConfig.php`

It reuses each module's existing `_mz_*_form_entry_input_covers_uploader_{simple,html5}_title`
translation keys. The UNA base/text class is unchanged.

## Verification

Applied on myunaworld.com at 2026-10-08 02:09:33 UTC (October 7, 2026, 10:09:33 PM Eastern).

- Reproduced both warnings on `/create-news` and `/create-announcement` before repair.
- PHP 8.3.6 syntax checks passed for both changed files.
- Both authenticated creation forms reloaded without PHP warnings after repair.
- Header Image, the Upload label, and Publish controls remained present.
- All five installed Modzzz modules with cover fields resolved both uploader labels.
- No Microphone registration remained in the installed-module list.
- Replayed this exact patch on the saved originals; resulting SHA-256 values matched the deployed files.
- Confirmed the live repaired files still matched those values before saving to Git.

The upload and publish actions were not submitted during verification.

## Application

Use an administrator-maintained checkout or staging copy of the target UNA site.
Back up the two configuration files. Check the current module version, file
contents and installed translation keys before applying. The SHA-256 values
for the verified My UNA World source are in [verification.json](verification.json).

From the UNA site root, with this patch available at a known path:

```bash
git apply --check /path/to/repair.patch
git apply /path/to/repair.patch
php -l modules/modzzz/news/classes/MzNewsConfig.php
php -l modules/modzzz/announcements/classes/MzAnnouncementsConfig.php
```

Preserve the patch's line endings: the verified News configuration uses CRLF,
while Announcements uses LF. Apply only if both preflight and syntax checks
succeed. Reload the relevant creation/editing forms and check that no PHP
warning appears and that Header Image still renders. If a server caches PHP
with timestamp validation disabled, refresh the relevant PHP opcode cache
through the site's established maintenance process.

My UNA World already contains this repair; applying it there again is unnecessary.
Other module versions may require the same mappings at a different location.
A future module update can replace these files, so check the mappings after updating.

## Rollback

Restore the two saved original configuration files and syntax-check them, or,
if no intervening edits occurred, preflight and reverse this patch from the
UNA site root:

```bash
git apply --reverse --check /path/to/repair.patch
git apply --reverse /path/to/repair.patch
```

The patch is a compatibility delta for separately installed Modzzz modules;
the complete third-party module packages are not included.
