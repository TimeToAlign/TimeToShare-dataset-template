# Contributing

The version of every release of this dataset is computed from the commit types, so
each commit must say what it did to the data.

**Install the hooks** in your clone before the first commit (README, "Set up on your
machine"):

```console
pre-commit install --hook-type pre-commit --hook-type prepare-commit-msg --hook-type commit-msg
```

They validate the staged manifests, write the commit type for an addition, an edit
or a removal, suggest one for a belief, a name, drawing, application data or a moved
file (`Suggested type "label(op106):", since this commit changes names only`), and
refuse a message that hides an addition, an edit or a removal. With `-m`, the check
also prints a warning naming the suggested type. A suggested type may be added or
left out: a commit without a type is kept, but does not appear in the changelog and
raises no version.

**One kind of change per commit.** Make small commits, one change at a time: add the
new recordings in one commit, rename the movements in the next. Each commit then gets
the right type and its own line in the changelog, and the history says what happened
when. When a commit mixes several kinds of change, the hooks suggest splitting it,
also when you typed the type yourself.

**Commit types** (README, "Commit types"): `feat` for added or edited data,
corrections included; `remove` with `!` for removals; `belief`, `label`, `graphic`,
`meta` and `move` for beliefs, names, drawing, application data and moved files;
`docs` for prose; `fix` for a fix to the repository's scripts; `build`, `chore`,
`ci`, `refactor`, `test` and `style` for code and configuration, never `feat`. The
scope names the manifest or part of the dataset that changed, never the dataset
itself; a change to the whole dataset has no scope.

**A removal** is written `remove(<scope>)!:`, with the `!`, and its footer names
every removed ID. The hooks write both when the message has no type yet; you add the
subject. Written by hand, it looks like this:

```text
remove(mtd)!: drop the duplicate excerpt of theme 112

BREAKING CHANGE: removes #excerpt_112b, #region_7
```

Changing an ID is a removal and an addition, and so is renaming or moving a manifest
file. A removed ID is never given to another element.
