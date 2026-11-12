# nk-indent-guard

An agent skill for [Claude Code](https://code.claude.com) and [OpenAI Codex](https://developers.openai.com/codex). Stop a one-line edit to a JSON or YAML data file from re-indenting the whole file and burying the real change in a 400-line diff.

**What you get** (a real run): a data file that a one-line edit re-indented is named, with the old and the new indentation, before it reaches the commit.

```text
$ python3 scripts/indent_guard.py
✘ .claude-plugin/plugin.json: 2 spaces → 1 space  (whole file re-indented; rewrite it with the original unit)
✘ 1 re-indented, 1 unchanged, 0 new (vs HEAD, ext .json,.yaml,.yml)
```

![nk-indent-guard](https://raw.githubusercontent.com/NickkkLian/nickkk-skills/main/gallery/social/nk-indent-guard.png)

Part of [nickkk-skills](https://github.com/NickkkLian/nickkk-skills): 14 agent skills that work with numbers, check an agent's work, guard a repository, and write and present.

## Try it

Clone and run the example (it re-indents one file inside the clone, runs the check, then puts the file back):

```bash
git clone https://github.com/NickkkLian/nk-indent-guard && cd nk-indent-guard
python3 -c "import json; p = '.claude-plugin/plugin.json'; json.dump(json.load(open(p)), open(p, 'w'), indent=1)"
python3 scripts/indent_guard.py
git checkout .claude-plugin/plugin.json
```

None of this installs anything or changes `~/.claude`.

The `indent_guard.py` line prints the block at the top of this page; its last line is the one below, and its exit code is 1 (non-zero on purpose: it found something).

```text
✘ 1 re-indented, 1 unchanged, 0 new (vs HEAD, ext .json,.yaml,.yml)
```

![nk-indent-guard demo: before and after](https://raw.githubusercontent.com/NickkkLian/nickkk-skills/main/gallery/nk-indent-guard.gif)

The demo above is a rendering of an earlier run and cuts its longest lines short; the block at the top of this page is a full run of this version.

## What it does

- Compares the indentation unit of every tracked JSON/YAML file with the last committed version.
- Names the file and the change (`2 spaces → 1 space`, `pretty → single line`, `spaces → tabs`).
- Shows how to write files back with their own unit in Python and Node.
- Runs on every commit in one of two ways: through the pre-commit framework (the repository carries the `.pre-commit-hooks.yaml` that framework reads; four lines in your config) or as a plain git hook.

The full procedure, the boundaries and where the rules came from are in [SKILL.md](SKILL.md).

## How it works

1. After editing, before committing: `python3 scripts/indent_guard.py` (whole repo) or `python3 scripts/indent_guard.py data/` (a subtree); `--ref` compares against another revision, `--ext` changes the extensions.
2. A red line names the file and the change of unit (`2 spaces → 1 space`, `4 spaces → tab`, `2 spaces → none` for pretty → single line).
3. New files have no baseline; they are listed, not judged.
4. To make it automatic, use one of the two hooks in `references/pre-commit.md`.

## Use it from pre-commit

If your repository already uses the [pre-commit](https://pre-commit.com) framework, add this to its `.pre-commit-config.yaml`:

```yaml
repos:
  - repo: https://github.com/NickkkLian/nk-indent-guard
    rev: v0.1.5
    hooks:
      - id: indent-guard
```

`rev` takes a version tag, as in the block above, or a commit id. `git ls-remote https://github.com/NickkkLian/nk-indent-guard`
lists this repository's tags and the id of its newest commit. The hook is the script itself (`language: script` in
[.pre-commit-hooks.yaml](.pre-commit-hooks.yaml)): it needs `python3` (3.9 or newer) and git on the machine, and the framework installs
nothing for it. It is given the staged `.json`, `.yaml` and `.yml` files. A repository without the framework uses the plain git hook in
[references/pre-commit.md](references/pre-commit.md).

## Why it is built this way

**The idea.** A rewritten data file passes every validator and destroys the diff. Writing "keep the indent" into a memory did not stop it happening a third time; a machine check did.

**Where it came from.** Own practice, 2026-08: the same mistake three times in one month across two repositories (a devlog rewritten with `indent=1`, a fix script with `indent=1`, a sources file two weeks later).

**Evidence.** What was broken on purpose to show that the self-tests can fail is under [Verify](#verify); what was run end to end, and in which agent, is under [Compatibility](#compatibility).

## Install

Pick one of four ways: three for Claude Code, one for OpenAI Codex. Skills load when a session starts, so open a **new** session after installing.

### 1 · Terminal, one command

```bash
git clone https://github.com/NickkkLian/nk-indent-guard ~/.claude/skills/nk-indent-guard
```

1. Run the command above (for one project only, clone into `.claude/skills/nk-indent-guard` inside that project).
2. Start a new Claude Code session.
3. Check it loaded: type `/nk-indent-guard` — it appears in the slash-command menu. Or just ask for the task; the skill triggers on its own.

### 2 · Claude Code in a terminal session (plugin)

The plugin route goes through the [nickkk-skills](https://github.com/NickkkLian/nickkk-skills) marketplace. Add it once; after that each skill is one command.

```
/plugin marketplace add NickkkLian/nickkk-skills
/plugin install nk-indent-guard@nickkk-skills
```

1. In a Claude Code session, run the first line (once per machine).
2. Run the second line.
3. Start a new session (or run `/reload-plugins`). The skill shows up as `nk-indent-guard:nk-indent-guard`.

Without opening a session, the same two steps work from a shell: `claude plugin marketplace add NickkkLian/nickkk-skills` then `claude plugin install nk-indent-guard@nickkk-skills`.

### 3 · Claude desktop app (Code tab)

**Add the marketplace first — Discover only searches marketplaces you have already added.**

<img src="https://raw.githubusercontent.com/NickkkLian/nickkk-skills/main/gallery/panel-route/panel-route.gif" alt="Adding the marketplace and installing a skill in the desktop app" width="640">

<sub>Recorded on 2026-09-16, when the marketplace listed ten skills, all at version 0.1.0; it lists more now. The repository list in this recording shows the recorder's own repositories because a GitHub account is connected; yours will show yours. Type the full name as in step 4.</sub>

1. In the chat box, type `/plugin marketplace` and press Enter (or open **Settings → Customize → Plugins**). The **Plugins** panel opens.
   <br><img src="https://raw.githubusercontent.com/NickkkLian/nickkk-skills/main/gallery/panel-route/step1-type-plugin-marketplace.png" alt="/plugin marketplace typed in the chat box" width="480">
2. Top right, open **Add ▾** and choose **Add marketplace**.
   <br><img src="https://raw.githubusercontent.com/NickkkLian/nickkk-skills/main/gallery/panel-route/step2-add-menu.png" alt="The Add menu with Add marketplace" width="480">
3. Choose **Add from a repository**.
   <br><img src="https://raw.githubusercontent.com/NickkkLian/nickkk-skills/main/gallery/panel-route/step3-add-from-repository.png" alt="Add marketplace dialog: Add from a repository" width="480">
4. In **URL**, type the full `NickkkLian/nickkk-skills`. At the bottom of the list choose the row **Use "NickkkLian/nickkk-skills"**, then press **Sync**.
   <br><img src="https://raw.githubusercontent.com/NickkkLian/nickkk-skills/main/gallery/panel-route/step4-url-then-sync.png" alt="URL filled in, Sync button" width="480">
5. You land on **Discover**, filtered to the new marketplace (**Filter · 1**). Find **Nk indent guard** and press **Add**. Installed ones show **✓ Added**.
   <br><img src="https://raw.githubusercontent.com/NickkkLian/nickkk-skills/main/gallery/panel-route/step5-discover-add.png" alt="Discover list with Added and Add buttons" width="480">
6. Close the panel and start a new session.

To try it for one session without installing anything: `claude --plugin-dir ./nk-indent-guard` from a clone.

### 4 · OpenAI Codex CLI

```bash
git clone https://github.com/NickkkLian/nk-indent-guard.git ~/.agents/skills/nk-indent-guard
```

1. Run the command above (for one project only, clone into `.agents/skills/nk-indent-guard` inside that project).
2. Start a new Codex session.
3. Check it loaded, without spending a model call: `codex debug prompt-input | grep -o -- '- nk-indent-guard[a-z0-9:-]*' | sort -u` prints `- nk-indent-guard:nk-indent-guard:`. Codex adds the `nk-indent-guard:` prefix because this repository also carries a Claude Code plugin manifest. Ask for the task and the skill triggers on its own, or type `$` and pick it from the list.

## Compatibility

| Agent | Tested | What was checked |
|---|---|---|
| Claude Code (CLI 2.1.173, macOS) | yes | In a fresh project with an isolated Claude config, inside a macOS sandbox that blocked reading the tester's ~/.claude folder (settings, session history, memory), Desktop, Documents and Downloads, SSH keys and git identity, a plain request that never names the skill triggered it and it ran its bundled script. The route 2 plugin commands were also run from a shell with an isolated config: marketplace add, install, list. |
| OpenAI Codex CLI (0.154.0-alpha.6.2, gpt-5.6-sol, low reasoning, macOS) | yes | Copied into `~/.agents/skills` of a temporary home (the folder route 4 clones into), in a fresh project, without the user's Codex config. From a plain request that never names the skill, Codex read SKILL.md, ran `scripts/indent_guard.py` on the re-indented file, restored the original indentation and left a one-line diff. |
| Cursor, Gemini CLI | no | Not tested. Their documentation says both read `~/.agents/skills`, the folder route 4 clones into; Gemini CLI asks before it activates a skill. |

In this skill's Codex run, every call into the skill folder's scripts/ used that folder's absolute path. Route 4 was checked for this repository: cloned from GitHub into a temporary home's `~/.agents/skills`, it was listed by the step 3 command. This skill's frontmatter uses only name, description, license and metadata.

## Verify

```bash
python3 scripts/indent_guard.py --selftest
```

The self-test prints:

```text
indent_guard selftest · 16/16 passed
```

The run at the top of this page is nk-indent-guard 0.1.5, copied from the terminal on 2026-10-14. The demo GIF also shows "1 spaces", which 0.1.1 printed; 0.1.2 prints "1 space".

Standard library only, Python 3.9+, and git. On 2026-10-14 every self-test above passed, and
a break-check script broke each script on purpose in a sandbox copy:

- `indent_guard.py`: 3 lines broken one at a time; each turned the self-test red without a traceback.

The unmutated control stayed green every time. Only lines that record a finding, raise, or return a failing exit code
were broken (the tool's pattern, or the hand-written list); a line number refers to the script as shipped in this version.
This shows those lines are covered. It does not show that nothing else can fail.

How far this was checked. On 2026-10-09 the manifest was read against the framework's documentation and its manifest schema, and the
script was started by a hand-written git hook the way that documentation describes for this hook type (the file itself, from the
repository root, with the staged data files as arguments): a one-line edit was committed, a re-indented file was refused under
`git commit -a`. On 2026-10-10 it was run under pre-commit 4.6.2 on macOS 15.7 (Apple silicon, git 2.39.3, Python 3.13), with the
framework cloning this repository at its 0.1.3 commit from a local copy: `git commit` passed a one-line edit, was refused for a
re-indented JSON file and again under `git commit -a` for a re-indented JSON and YAML file, and `pre-commit run --all-files` failed
while the re-indented file was there and passed once it was put back. The script and `.pre-commit-hooks.yaml` are unchanged in 0.1.4
and 0.1.5. Later on 2026-10-10, after 0.1.4 was released, it was run again the same way with the framework cloning
`https://github.com/NickkkLian/nk-indent-guard` at tag `v0.1.4` (the configuration block under [Use it from pre-commit](#use-it-from-pre-commit), as the 0.1.4 README printed it): `git commit` passed a
one-line edit, skipped a commit that touched no data file and was refused for a re-indented JSON file, and `pre-commit run --all-files`
failed while that file was there and passed once it was put back. It was not run on Linux or on Windows, or under another pre-commit
version.

## Limits

- It does not validate the content; pair it with your schema check.
- It reads the first 200 lines to find the unit; a file whose first indented line is inconsistent with the rest is judged by that first line.
- Key order changes, trailing-newline changes and reformatting inside a line are invisible to it — those also inflate diffs, but they are a different guard.
- The pre-commit framework entry was run under pre-commit 4.6.2 on macOS 15.7 on 2026-10-10, with the hook repository cloned from a local copy: it refused a re-indented file and passed a one-line edit. The same evening, after 0.1.4 was released, it was run again with the framework cloning the GitHub address at tag v0.1.4: it passed a one-line edit and refused a re-indented JSON file. It was not run on Linux or on Windows.

## License

MIT. Read a script before letting it run in your environment.
