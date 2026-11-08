# Codex Local Project Recovery

**An unofficial Windows utility to diagnose and recover missing local projects in the Codex Desktop sidebar when the local SQLite project catalog is still intact.**

> **Important:** This is an independent community project. It is not made, supported, endorsed, or affiliated with OpenAI. Codex Desktop's local storage format is an internal implementation detail and may change without notice.

- **Platform:** Windows 10/11
- **PowerShell:** 5.1 or newer
- **Python:** 3.9 or newer
- **Dependencies:** Python standard library only; no pip packages are required
- **License:** MIT — see [LICENSE](LICENSE)

## What problem does this address?

This tool is intended for one specific situation:

- Local projects have disappeared from the Codex Desktop Projects sidebar.
- The local Codex SQLite database still exists and passes its integrity check.
- The database still contains project records and project-root paths.
- The active desktop project catalog in .codex-global-state.json is empty or incomplete.
- A saved inactive account snapshot may contain project order and conversation-to-project assignments.

The tool reconstructs the local project catalog from the SQLite records and restores only conversation assignments that can be matched safely.

**This is not a general Codex repair tool.** It cannot recover deleted project folders, missing source files, deleted conversation contents, cloud account data, or a corrupted SQLite database.

## Maintainer's recovery experience

The issue that motivated this project occurred on the maintainer's Windows computer. A prior recovery procedure restored **9 local project entries and 37 saved conversation-to-project assignments**. The original inventory also found two project-root paths that no longer existed at their recorded locations; those folders were not recreated or moved.

That successful recovery is one real-world report, not a guarantee that this release will work with every Codex Desktop version or every missing-project incident.

This public release includes five synthetic Python tests. The exact PowerShell launcher and end-to-end recovery flow must still be validated on the target Windows/Codex installation before relying on them.

## What the tool does

When run, the PowerShell launcher and Python engine:

1. Detect the Codex data directory (CODEX_HOME, or %USERPROFILE%\\.codex by default).
2. Refuse to continue while known Codex Desktop processes are running.
3. Check the installed Python version and required SQLite tables/columns.
4. Open state_5.sqlite in read-only mode and run PRAGMA quick_check.
5. Inventory projects, project roots, idempotency keys, and thread records.
6. Resolve original desktop project IDs only when the available evidence is unambiguous.
7. Inspect inactive account snapshots and select the best-supported matching snapshot.
8. Stop rather than guess if equally strong snapshots contain different recovery data.
9. Rebuild/merge the local project catalog and project order.
10. Restore only saved conversation assignments that match existing thread records and known projects.
11. Preserve projectless-thread markers and existing conflicting assignments.
12. Restore compatible membership-host IDs and workspace-root hints without overwriting current values.
13. Create a timestamped backup of the JSON state files and a consistent SQLite snapshot.
14. Write the repaired JSON files and validate the result.
15. Attempt to restore the original JSON files if a write or post-write validation step fails.

The normal run displays a recovery plan and asks you to type YES before applying it.

## What it changes — and what it does not

| Item | Behaviour |
|---|---|
| .codex-global-state.json | Rebuilt/merged after explicit confirmation |
| .codex-global-state.json.bak | Updated to match the recovered state; original backed up first |
| state_5.sqlite | Opened read-only; live database is never modified |
| Project folders and source files | Never modified |
| Conversation rollout/history files | Never modified |
| Migration flags/checkpoints | Preserved; not reset |
| Projectless-thread markers | Preserved; not removed |
| Network requests / telemetry | None |

The tool writes only the two global-state JSON files during a successful repair. It creates a backup first under:

    %USERPROFILE%\\Desktop\\Codex-Recovery-Backups\\

If the Desktop folder is unavailable, it uses a Codex-Recovery-Backups folder inside the Codex data directory. The backup includes the original JSON files (when present), a consistent SQLite snapshot, and a manifest with file hashes.

**The backup contains private local metadata. Keep it private. Do not upload it to GitHub or share it in a public issue.**

## Before you start

### 1. Confirm that this tool matches your problem

Use it only if local Codex projects are missing from the Windows sidebar and the local Codex database is still present.

The default Codex data directory is:

    %USERPROFILE%\\.codex

It should contain at least:

- .codex-global-state.json
- state_5.sqlite

If either file is missing, stop. Do not create empty replacement files.

### 2. Check the requirements

- Windows 10 or Windows 11.
- Codex Desktop for Windows.
- PowerShell 5.1 or newer.
- Python 3.9 or newer.
- The Python launcher (py.exe) or python.exe available on PATH.

Check Python from PowerShell:

    py -3 --version

If py is not recognised, try:

    python --version

If neither command works, install Python 3.9 or newer from [python.org/downloads](https://www.python.org/downloads/). During installation, enable the Python launcher and/or the option to add Python to PATH. No third-party Python packages are needed.

### 3. Keep the files together

Download the ZIP asset from the repository's **Releases** page and extract it fully. Do not run the scripts from inside the ZIP preview.

Keep these files in the same extracted project folder:

- Recover-CodexProjects.ps1
- codex_project_recovery.py
- Restore-CodexBackup.ps1

The docs and tests folders should also remain in the extracted folder. Do not move the Python engine away from the PowerShell launcher.

## Exact step-by-step instructions

### Step 1 — Download the release

1. Open the [Releases page](https://github.com/CaptainVid/codex-local-project-recovery/releases).
2. Open the latest release.
3. Under **Assets**, download codex-local-project-recovery-v1.0.0.zip.
4. Save it locally, for example in your Downloads folder.

Only download release assets from the official repository URL above. If the repository owner publishes a newer version, read that version's release notes before using it.

### Step 2 — Extract the ZIP

1. In File Explorer, open the folder where you downloaded the ZIP.
2. Right-click the ZIP file.
3. Select **Extract All…**.
4. Choose a normal local folder, such as Downloads, and extract the files.
5. Open the extracted codex-local-project-recovery folder.
6. Confirm that Recover-CodexProjects.ps1 and codex_project_recovery.py are side by side.

Do not extract the tool into the .codex data directory.

### Step 3 — Save your work and fully quit Codex

1. Save any work currently open in Codex.
2. In Codex Desktop, select **File → Quit ChatGPT** (the exact menu wording may vary by app version).
3. Do not just close or minimise the window; the desktop app may continue running in the background.
4. Wait a few seconds.

The launcher checks for known Codex processes. If one is still running, it stops without applying a repair. It does not force-close the application.

### Step 4 — Open PowerShell in the extracted folder

Open the extracted codex-local-project-recovery folder in File Explorer.

On Windows 11, right-click inside the folder and choose **Open in Terminal**. Make sure the terminal is running PowerShell, not Command Prompt.

Alternatively, open PowerShell normally and change to the folder. Replace the example path with the actual folder location:

    cd "$env:USERPROFILE\\Downloads\\codex-local-project-recovery"

Run the tool as the same Windows user who owns the Codex data directory.

**Do not run PowerShell as Administrator unless you have a specific, understood reason.** Elevation is not normally required and can create file-ownership problems if used unnecessarily.

### Step 5 — Run a preview first (recommended)

Copy and run:

    powershell.exe -NoProfile -ExecutionPolicy Bypass -File ".\\Recover-CodexProjects.ps1" -PreviewOnly

The preview:

- checks the local database and its schema;
- lists the projects it can identify;
- shows whether their recorded root folders exist;
- reports the number of historical conversation assignments that appear recoverable;
- reports conflicts and missing folders;
- does **not** change Codex files;
- does **not** create a recovery backup.

Read the entire output before continuing.

A healthy preview should show:

- SQLite integrity: OK;
- all project identities resolved;
- a complete project order;
- no unresolved project-ID or account-snapshot ambiguity.

The exact project and conversation counts depend on the computer. They will not necessarily match the maintainer's 9-project / 37-assignment example.

If the output says SAFETY STOP, reports an unsupported schema, fails SQLite integrity, or cannot resolve project identities uniquely, **stop here**. Do not bypass the safety check or edit the JSON manually.

If any project root is marked MISSING, note that path. The tool can restore the sidebar entry, but it cannot recreate or locate a missing source folder.

### Step 6 — Apply the recovery

Only continue if you have reviewed the preview and it matches the projects you expect.

Run:

    powershell.exe -NoProfile -ExecutionPolicy Bypass -File ".\\Recover-CodexProjects.ps1"

The tool displays the proposed changes and asks:

    Proceed with the recovery? Type YES to continue:

Type exactly:

    YES

Press Enter.

Any other response cancels the operation.

Do not use the -Yes switch unless you have reviewed the code and deliberately want to skip the interactive confirmation.

### Step 7 — Verify the result in Codex

When the script reports RECOVERY APPLIED SUCCESSFULLY:

1. Note the backup folder path printed by the script.
2. Close the PowerShell window.
3. Reopen Codex Desktop.
4. Check the Projects sidebar.
5. Confirm that the expected project names are present.
6. Open several existing conversations and verify their contents and project associations.
7. Check any project previously reported as MISSING.

If a project entry is visible but its folder is missing, locate the original repository yourself and reopen it using Codex's normal project-opening workflow. This recovery tool does not move, rename, or recreate project folders.

The tool cannot verify the graphical sidebar while Codex is closed. You must perform this final visual check yourself.

## Preview and optional parameters

### Preview only

    powershell.exe -NoProfile -ExecutionPolicy Bypass -File ".\\Recover-CodexProjects.ps1" -PreviewOnly

### Use a custom Codex data directory

If your Codex data is not in %USERPROFILE%\\.codex, pass its path:

    powershell.exe -NoProfile -ExecutionPolicy Bypass -File ".\\Recover-CodexProjects.ps1" -CodexHome "D:\\MyCodexData" -PreviewOnly

After checking the preview, omit -PreviewOnly to apply the recovery.

### Show full project paths in the local report

By default, the report shows project-root folder names rather than full paths. To display full paths on your own computer:

    powershell.exe -NoProfile -ExecutionPolicy Bypass -File ".\\Recover-CodexProjects.ps1" -PreviewOnly -ShowPaths

Do not post this output publicly without redacting personal folder names.

## How to roll back

If Codex behaves unexpectedly after the repair, you can restore the original JSON state files from the backup.

1. Fully quit Codex using **File → Quit ChatGPT**.
2. Find the backup folder printed at the end of the recovery. By default, backups are stored under:

    %USERPROFILE%\\Desktop\\Codex-Recovery-Backups\\

3. Open PowerShell in the extracted tool folder.
4. Run the rollback helper, replacing the example timestamp with the actual backup folder name:

    powershell.exe -NoProfile -ExecutionPolicy Bypass -File ".\\Restore-CodexBackup.ps1" -BackupFolder "$env:USERPROFILE\\Desktop\\Codex-Recovery-Backups\\Codex-Recovery-YYYYMMDD-HHMMSS"

5. When prompted, type exactly YES.
6. Reopen Codex and check the previous state.

The rollback helper verifies the backup manifest and JSON before restoring. It also preserves the current post-recovery JSON files in a Codex-Pre-Rollback-* folder.

The rollback helper restores .codex-global-state.json and .codex-global-state.json.bak. It does not restore SQLite because the recovery tool does not modify the live SQLite database.

If the backup was stored inside the Codex data directory because the Desktop folder was unavailable, use the actual path printed by the recovery script instead of the example path above.

## Important risks and limitations

- **Undocumented internal format:** Codex can change its JSON or SQLite schema at any time. This release checks required tables and columns, but a passing check does not guarantee compatibility with every app build.
- **Account snapshot selection:** Multiple inactive account snapshots may exist. The tool scores their project/thread evidence and stops if equally strong snapshots disagree. Never manually substitute a snapshot to bypass this stop.
- **Project identity mapping:** The tool relies on a unique project_idempotency_keys.key associated with a projects.id record, corroborated by existing mappings when available. This is an internal implementation detail, not a public API.
- **Missing folders:** A project can reappear in the sidebar while its recorded source folder is unavailable. The tool does not recreate, move, rename, or search for project folders.
- **Incomplete conversation grouping:** Only explicit saved assignments that match known projects and existing thread records are restored. The tool does not guess from directory names or working-directory paths.
- **Private backup data:** Backups contain local project names, paths, and Codex metadata. Keep them private.
- **No universal guarantee:** This tool cannot repair every cause of missing projects, corrupted databases, deleted conversation files, or cloud/account synchronization problems.

### Automatic safety stops

The tool stops if:

- Codex is still running;
- required files or database tables are missing;
- SQLite integrity check fails;
- the database schema is not recognised;
- project identities cannot be resolved uniquely;
- equally plausible inactive snapshots contain different recovery data;
- existing mappings conflict;
- the global-state file changes during the recovery operation.

A safety stop is intentional. It is better to leave the data untouched than to guess.

## Privacy and data handling

- The tool runs locally on your computer.
- It does not make network requests or upload files.
- It does not collect telemetry.
- It opens the live SQLite database read-only.
- It does not modify project source files or conversation history.
- The release archive contains source code, documentation, and synthetic tests — not the maintainer's Codex database, personal project folders, conversation history, account tokens, or local backup files.

**Never share** .codex-global-state.json, .bak files, SQLite databases, auth.json, tokens, API keys, raw conversation files, or unredacted diagnostic output in a public issue.

## Troubleshooting

### “Codex is still running”

Quit Codex using **File → Quit ChatGPT** and wait for its background processes to exit. Do not force-kill processes unless you understand the consequences.

### “Python 3.9 or newer was not found”

Install Python 3.9+ from [python.org](https://www.python.org/downloads/). Reopen PowerShell after installation and check py -3 --version. No pip packages are needed.

### “The recovery engine is missing”

Keep Recover-CodexProjects.ps1 and codex_project_recovery.py in the same extracted folder. Do not run the launcher from inside the ZIP archive.

### “Unsupported Codex database schema”

This release does not recognise the installed Codex database layout. Do not remove schema checks. Report the Codex version and a redacted error summary in a GitHub issue.

### “Ambiguous inactive account snapshots” or “Project identity validation failed”

The tool cannot safely decide which account snapshot or project ID is correct. It stops without applying the recovery. Do not choose one manually. Report only the redacted summary.

### “Some project roots are missing”

The database contains a path that Windows cannot currently find. The sidebar entry may be restored, but you must locate the repository yourself.

### Projects appear but some conversations are not grouped

Only safely identifiable saved assignments are restored. Other conversations may remain accessible in Recents without a project association.

### Access denied or file changed during recovery

Confirm Codex is fully closed and run the tool as the same Windows user who owns the Codex data directory. Do not immediately rerun as Administrator. If the issue persists, stop and report the redacted error.

## Testing and compatibility status

The repository includes five synthetic Python tests covering project identity resolution, historical assignments, projectless markers, conflicting assignments, ambiguous snapshots, and missing roots.

For this release:

- Python syntax compilation: passed.
- Synthetic unit tests: 5 passed.
- The exact PowerShell launcher and end-to-end recovery have not yet been independently validated across Windows/Codex versions.
- The maintainer's earlier successful recovery (9 projects and 37 assignments) motivated this project but does not establish compatibility for every release.

The included GitHub Actions workflow runs the synthetic tests on Windows across supported Python versions when configured in the repository.

## Repository contents

- Recover-CodexProjects.ps1 — Windows launcher and process safety checks.
- codex_project_recovery.py — read-only SQLite analysis, recovery planning, backup, and guarded JSON repair.
- Restore-CodexBackup.ps1 — restores the original project-cache JSON files from a generated backup.
- tests/ — synthetic SQLite/state tests.
- docs/RECOVERY-GUIDE.md — additional recovery and troubleshooting guide.
- docs/KNOWN-LIMITATIONS.md — known limitations.
- SECURITY.md — security and privacy guidance.
- CONTRIBUTING.md — contribution and testing instructions.
- CHANGELOG.md — release history.
- LICENSE — MIT License.

## Support and bug reports

Before opening an issue:

1. Check existing issues.
2. Include your Windows version and Codex app version, if known.
3. Include only the redacted summary output.
4. State whether you ran preview mode or applied a recovery.

Never attach raw .codex files, SQLite databases, global-state JSON, account IDs, credentials, tokens, or conversation contents.

## Disclaimer

This project is independent and community-maintained. It is not affiliated with, endorsed by, or supported by OpenAI. “Codex” and “ChatGPT” are product names of their respective owner. Use this tool at your own risk and keep a verified backup before changing local application state.

## License and copyright

Copyright (c) 2026 Vid Kovačič.

This project is licensed under the MIT License. See [LICENSE](LICENSE) for the full license text.
