# Codex Local Project Recovery

An unofficial Windows utility to diagnose and recover missing local projects in the Codex Desktop sidebar when the local SQLite project catalog is still intact.

> This is an independent community project. It is not affiliated with, endorsed by, or supported by OpenAI. Codex's local storage format is undocumented and may change without notice.

- Windows 10/11
- PowerShell 5.1+
- Python 3.9+
- Python standard library only; no pip packages required
- MIT License

## Intended use

Use this tool only when local projects have disappeared from the Codex Desktop Projects sidebar, but the local SQLite database still exists, passes its integrity check, and contains the project records. This is not a general Codex repair utility. It cannot recover deleted source folders, deleted conversation contents, cloud account data, or a corrupted database.

## Maintainer's recovery report

The issue that motivated this project occurred on the maintainer's Windows computer. A previous recovery restored 9 local project entries and 37 saved conversation-to-project assignments. Two recorded project-root paths were missing; those folders were not recreated or moved. This is one successful user report, not a guarantee for other Codex versions or installations.

The repository includes five synthetic Python tests. The exact PowerShell launcher and end-to-end recovery have not been independently validated across all Windows/Codex versions.

## What the tool does

1. Detects CODEX_HOME or defaults to the current user's .codex folder.
2. Refuses to continue while known Codex Desktop processes are running.
3. Checks Python and required SQLite tables/columns.
4. Opens state_5.sqlite read-only and runs SQLite integrity validation.
5. Inventories projects, roots, idempotency keys, and thread records.
6. Resolves project IDs only when evidence is unambiguous.
7. Inspects inactive account snapshots and stops if equally plausible snapshots disagree.
8. Rebuilds/merges the local project catalog and project order.
9. Restores only saved conversation assignments matching existing threads and known projects.
10. Preserves projectless-thread markers and conflicting active assignments.
11. Creates a timestamped backup before writing the repaired JSON state.
12. Validates the result and attempts to roll back JSON files if a write/validation step fails.

## What changes

| Item | Behaviour |
|---|---|
| .codex-global-state.json | Rebuilt/merged after confirmation |
| .codex-global-state.json.bak | Updated to match recovered state; original backed up |
| state_5.sqlite | Read-only; live database never modified |
| Project folders/source files | Never modified |
| Conversation history files | Never modified |
| Network/telemetry | None |

Backups are created under the user's Desktop in Codex-Recovery-Backups (or inside the Codex data directory if Desktop is unavailable). They contain private project names, paths, and Codex metadata. Never publish or share backup folders.

## Requirements

- Windows 10 or Windows 11.
- Codex Desktop for Windows.
- PowerShell 5.1 or newer.
- Python 3.9 or newer, available as py.exe or python.exe.
- The Codex data directory must contain .codex-global-state.json and state_5.sqlite.

Check Python in PowerShell:

    py -3 --version

If py is not recognised, try python --version. Install Python from https://www.python.org/downloads/ if needed. No third-party packages are required.

## Exact Windows instructions

### 1. Download and extract

1. Open the GitHub Releases page: https://github.com/CaptainVid/codex-local-project-recovery/releases
2. Open the release you want to use.
3. Download its ZIP asset under Assets.
4. In File Explorer, right-click the ZIP and select Extract All.
5. Extract it to a normal folder, for example Downloads\codex-local-project-recovery.
6. Open the extracted folder.
7. Keep Recover-CodexProjects.ps1, codex_project_recovery.py, and Restore-CodexBackup.ps1 together. Do not run scripts from inside the ZIP preview or extract the tool into the .codex directory.

### 2. Save work and quit Codex completely

1. Save any open work.
2. In Codex Desktop, select File → Quit ChatGPT (menu wording may vary).
3. Do not merely close or minimise the window; background processes may remain running.
4. Wait a few seconds.

The launcher checks for known Codex processes and stops if any remain. It does not force-close the app.

### 3. Open PowerShell in the extracted folder

In File Explorer, open the extracted folder, right-click inside it, and choose Open in Terminal. Ensure the terminal is PowerShell, not Command Prompt.

Alternatively, open PowerShell and change to the folder (adjust the path if necessary):

    cd "$env:USERPROFILE\Downloads\codex-local-project-recovery"

Run as the same Windows user who owns the Codex data directory. Do not run as Administrator unless you have a specific reason.

### 4. Run a preview first (recommended)

    powershell.exe -NoProfile -ExecutionPolicy Bypass -File ".\Recover-CodexProjects.ps1" -PreviewOnly

Preview mode checks the database and schema, lists identifiable projects and whether their root folders exist, reports recoverable assignments and conflicts, and does not change Codex files or create a recovery backup.

Read the entire output. A healthy preview should report SQLite integrity as OK, resolve all project identities, and construct a complete project order.

If you see SAFETY STOP, an unsupported schema, failed integrity check, or ambiguous project identities/snapshots, stop. Do not bypass checks or manually edit JSON.

A root marked MISSING means Windows cannot find its recorded folder. The tool may restore the sidebar entry, but it cannot recreate or locate that folder.

### 5. Apply the recovery

Only continue after reviewing the preview. Run:

    powershell.exe -NoProfile -ExecutionPolicy Bypass -File ".\Recover-CodexProjects.ps1"

Review the proposed changes. At the prompt, type exactly YES and press Enter. Any other response cancels. Do not use the -Yes switch unless you have reviewed the code and intentionally want to skip interactive confirmation.

### 6. Verify in Codex

1. Note the backup folder path printed by the tool.
2. Reopen Codex Desktop.
3. Check the Projects sidebar.
4. Open several existing conversations and verify their contents and project associations.
5. Check projects whose root paths were reported as missing.

The tool cannot inspect the graphical sidebar while Codex is closed; verify it after reopening the app.

## Optional commands

Custom Codex data directory (preview):

    powershell.exe -NoProfile -ExecutionPolicy Bypass -File ".\Recover-CodexProjects.ps1" -CodexHome "D:\MyCodexData" -PreviewOnly

Show full project paths in a local preview report:

    powershell.exe -NoProfile -ExecutionPolicy Bypass -File ".\Recover-CodexProjects.ps1" -PreviewOnly -ShowPaths

Do not post output containing personal folder names publicly.

## Rollback instructions

If Codex behaves unexpectedly after recovery:

1. Quit Codex completely.
2. Locate the backup folder printed by the recovery tool. The default is under the user's Desktop in Codex-Recovery-Backups.
3. Open PowerShell in the extracted tool folder.
4. Run the command below, replacing the example timestamp with the actual backup folder name:

    powershell.exe -NoProfile -ExecutionPolicy Bypass -File ".\Restore-CodexBackup.ps1" -BackupFolder "$env:USERPROFILE\Desktop\Codex-Recovery-Backups\Codex-Recovery-YYYYMMDD-HHMMSS"

5. Type exactly YES when prompted.
6. Reopen Codex and verify the previous state.

The rollback helper checks the backup manifest and JSON. It preserves current post-recovery JSON files in a Codex-Pre-Rollback folder. It restores the two global-state JSON files only; SQLite is not restored because the recovery tool does not modify the live database. If the backup is elsewhere, use the actual path printed by the tool.

## Risks and limitations

- Undocumented format: Codex may change its internal JSON or SQLite schema. Schema checks do not guarantee compatibility with every release.
- Snapshot selection: the tool stops rather than guess if equally plausible inactive account snapshots disagree.
- Project ID mapping: it relies on a unique project_idempotency_keys key associated with a projects.id row, corroborated by existing mappings when available. This is an internal detail, not a public API.
- Missing folders: the tool does not recreate, move, rename, or search for project folders.
- Conversation grouping: only explicit saved assignments matching known projects and existing threads are restored.
- Private backups: backup files contain local metadata and must remain private.
- No universal guarantee: this cannot fix every cause of missing projects, corrupted databases, deleted conversations, or cloud/account sync problems.

The tool stops if Codex is running, required files/tables are missing, SQLite integrity fails, the schema is unknown, project identities cannot be resolved uniquely, competing snapshots disagree, mappings conflict, or the global-state file changes during recovery.

## Privacy and data handling

The tool runs locally. It makes no network requests, uploads, or telemetry. It opens the live SQLite database read-only and does not modify source folders or conversation history.

Never publish .codex-global-state.json, .bak files, SQLite databases, auth.json, API keys, tokens, raw conversation files, or unredacted diagnostic output.

## Testing and compatibility status

The repository includes five synthetic Python tests. Run them with:

    py -3 -m unittest discover -s tests -v

Python syntax compilation and all five synthetic tests passed during preparation. The exact PowerShell launcher and end-to-end recovery have not been independently validated across Windows/Codex versions. The maintainer's earlier successful recovery (9 projects and 37 assignments) motivated this project but is not a compatibility guarantee.

## Repository contents

- Recover-CodexProjects.ps1 — Windows launcher and process safety checks.
- codex_project_recovery.py — SQLite analysis, recovery planning, backup, and guarded JSON repair.
- Restore-CodexBackup.ps1 — restores original project-cache JSON files.
- tests/ — synthetic SQLite/state tests.
- docs/ — recovery guide and known limitations.
- SECURITY.md — security and privacy guidance.
- CONTRIBUTING.md — contribution instructions.
- CHANGELOG.md — release history.
- LICENSE — MIT License.

## Bug reports

Check existing issues first. Include your Windows version, Codex app version (if known), and a redacted summary. State whether you ran preview mode or applied recovery.

Never attach raw .codex files, databases, global-state JSON, account IDs, credentials, tokens, or conversation contents.

## Disclaimer

This project is independent and community-maintained. It is not affiliated with, endorsed by, or supported by OpenAI. Codex and ChatGPT are product names of their respective owner. Use this tool at your own risk and keep a verified backup before changing local application state.

## License and copyright

Copyright (c) 2026 Vid Kovačič.

This project is licensed under the MIT License. See LICENSE for the full license text.
