# claude-skills

Public agent skills repo: `FloyyMP/claude-skills`, published via skills.sh. **The only home for every custom skill the owner creates**, on every machine and VPS. One branch: `main`.

## Layout
- `skills/<name>/SKILL.md` — one folder per skill (skills.sh / `npx skills` standard), plus optional `references/` and `scripts/`.
- Scripts ship in pairs where it matters: `.ps1` for Windows, `.sh`/`.py` for Linux. The `.sh`/`.py` is the source of truth; any change to it updates the `.ps1` twin in the same commit. No local pwsh? Test with a portable one: the `linux-x64.tar.gz` from PowerShell's GitHub releases, extracted to `/tmp`.
- Line endings are LF (`.gitattributes`).
- Skills are public: no personal names, home-folder paths or emails in them. Say "the user". The `FloyyMP` GitHub handle stays only where it is the install path.
- Project-specific skills stay in their own project repo (`<project>/.claude/skills/`); everything general-purpose lives here.
- Installing copies a skill folder verbatim into `~/.claude/skills/`, so a skill must run in place from there: no repo-relative paths, no build step, no install of its own.

## Verification

There is no build and no test suite. Python is stdlib-only — no `requirements.txt`, no `pyproject.toml`, no dependency to install; that is what keeps every script runnable in place out of `~/.claude/skills/`. Run Python with `-I` so a stray `json.py`/`re.py` in a skill folder can't shadow the stdlib.

- `python3 -I skills/find-tooling/scripts/search.py --selftest` — the only real test; covers the pure matching/scoring logic. Must print `selftest ok`.
- `<script> -h` / `--help` — the `.py` scripts print argparse help, `run-gates.sh` and `prove-clean.sh` print their own header comment. Run it after touching a header, flag or usage line. The exit code is **not** uniform: argparse and `run-gates.sh -h` exit 0, `prove-clean.sh -h` exits 2 (same as its usage error). Bare `bash run-gates.sh` with no `--repo` exits 3, its nothing-runnable code.
- `.ps1` twins: compare by reading against the `.sh`/`.py` (or a portable pwsh from `/tmp`, see Layout). Nothing here exercises them.

Each skill's `## Steps` is the checklist for edits to that skill; the scripts are only part of it.

## Script contract

The scripts are a skill's API, so their CLI is a public interface that installed copies already call:

- Exit codes are documented per script and branched on by the skills — `prove-clean` 0 clean / 1 open / 2 usage, `run-gates` 3 nothing-runnable, `session-facts` 1 transcript missing. Never renumber them to "tidy up".
- Scripts documented as read-only stay read-only; `prove-clean` and `memory-index-check` run at close-out, before commits land.
- `session-facts.py` reads the on-disk transcript as the source of truth and treats the conversation as a supplement. Don't make anything else depend on recall.
- `CLAUDE_CONFIG_DIR` overrides `~/.claude` wherever a config path is resolved (`search.py` marketplace cache, the `find-tooling` skill dir). New code resolving a config path should honour it too.

## Skill frontmatter

- `disable-model-invocation: true` — manual-only. Any skill with side effects (commit, push, install, delete, zip) gets it, and its description opens with "Manual-only".
- `allowed-tools:` — narrow it when the skill must not write; `automation-advisor` is read-only until the user picks IDs.
- Descriptions load every session and are pure trigger text: say what it does *and* when to use it, and name the sibling that covers the adjacent case (`automation-advisor` ↔ `find-tooling`, `review-session` vs `complete-session`).

## Versioning
Each `SKILL.md` carries a `version:` field (semver). Bump it on every edit: patch for fixes/wording, minor for new behaviour, major for breaking changes.

## Workflow (same on every machine)
Each machine has one clone of this repo. `~/.claude/skills/<name>/` holds **installed copies** made by `npx skills add` (alongside third-party skills), so it is never a git repo and never edited by hand.

1. `git pull` first. Skills are edited from several machines; skipping the pull is how history diverged on 2026-10-08.
2. Edit the skill here. A new skill starts here too, never directly in `~/.claude/skills/`.
3. Commit and push to `main` straight away.
4. Reinstall: `npx skills add FloyyMP/claude-skills --agent claude-code -g -y --skill '*'` (`/sync-skills` runs steps 1, 3 and 4).
5. When adding a skill, add a row to the README table and the name to the GitHub repo description: `gh repo edit FloyyMP/claude-skills --description "Claude Code skills: <comma-separated names>"`.
6. When removing a skill, also remove its installed copies on every machine: `npx skills remove <name> -g -y` (cleans every agent's copy). Reinstalling doesn't remove them.

## skills.sh listing
- There's no submit step and no crawler. A skill appears on skills.sh only after it's been installed through `npx skills add` (install telemetry). Step 4 above is what lists a new skill.
- Check the listing: `https://www.skills.sh/FloyyMP/claude-skills/<name>`. "isn't available in this repository" means it hasn't been installed via the CLI yet.
- `npx skills add FloyyMP/claude-skills -l` lists the skills the CLI detects without installing them.
