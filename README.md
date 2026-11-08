# Codex Local Project Recovery

An unofficial Windows utility to diagnose and safely recover missing Codex Desktop local projects when the local SQLite project catalog is intact.

## Package

The v1.0.0 source package is being prepared for publication. The repository will contain the PowerShell launcher, Python recovery engine, rollback helper, documentation, MIT license, tests, and GitHub Actions workflow.

## Important

This is an independent community utility. It is **not affiliated with, endorsed by, or supported by OpenAI**.

The tool edits Codex internal local state files, which are undocumented and may change between releases. Read the included documentation and risk notes before use. Always keep a backup. The tool is not guaranteed to work on every Codex Desktop version or every missing-project scenario.

## Maintainer's recovery report

The recovery approach helped restore all 9 local project entries and 37 saved conversation-to-project assignments on the maintainer's own Windows installation. This is one successful case, not a universal compatibility guarantee.

## License

MIT. See [LICENSE](LICENSE).
