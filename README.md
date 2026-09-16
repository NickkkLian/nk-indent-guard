# nk-indent-guard

A [Claude Code](https://code.claude.com) skill. Stop a one-line edit to a JSON or YAML data file from re-indenting the whole file and burying the real change in a 400-line diff.

Part of [nickkk-skills](https://github.com/NickkkLian/nickkk-skills) — skills that stop an AI coding agent's
"done, tested, safe" from being taken on faith.

## What it does

- Compares the indentation unit of every tracked JSON/YAML file with the last committed version.
- Names the file and the change (`2 spaces → 1 space`, `pretty → single line`, `spaces → tabs`).
- Shows how to write files back with their own unit in Python and Node, and a pre-commit snippet.

The full procedure, the boundaries and where the rules came from are in [SKILL.md](SKILL.md).

## Install

Copy the folder into your skills directory (the skill is the repository root):

```bash
git clone https://github.com/NickkkLian/nk-indent-guard ~/.claude/skills/nk-indent-guard
```

or inside one project: `git clone … .claude/skills/nk-indent-guard`.

As a plugin, through the marketplace in the index repository:

```
/plugin marketplace add NickkkLian/nickkk-skills
/plugin install nk-indent-guard@nickkk-skills
```

To try it for one session without installing: `claude --plugin-dir ./nk-indent-guard`.

## Verify

```bash
python3 scripts/indent_guard.py --selftest
```

Standard library only, Python 3.9+. Before publishing, the guarded lines of each script were
mutated one at a time in a sandbox copy and the self-test was confirmed to go red on the named
assertion, without a traceback; the unmutated control stayed green.

## License

MIT. Read a script before letting it run in your environment.
