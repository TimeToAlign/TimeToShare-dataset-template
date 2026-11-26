# Contributing

The version of every release of this dataset is computed from the commit types, so
each commit must say what it did to the data.

**Install the hooks** in your clone before the first commit (README, "Set up on your
machine"):

```console
pre-commit install --hook-type pre-commit --hook-type prepare-commit-msg --hook-type commit-msg
```

They validate the staged manifests, write the commit type the staged change requires,
and refuse a message that understates it.

**Commit types** (README, "Commit types"): `feat` for added or edited data,
corrections included; `remove` with `!` for removals; `belief`, `label`, `graphic`,
`meta` and `move` for beliefs, names, drawing, application data and moved files;
`docs` for prose; `fix` for a fix to the repository's scripts; `build`, `chore`,
`ci`, `refactor`, `test` and `style` for code and configuration, never `feat`. The
scope names the manifest or part of the dataset.

**A removal** is written `remove(<scope>)!:`, and its footer names every removed ID:

```text
remove(mtd)!: drop the duplicate excerpt of theme 112

BREAKING CHANGE: removes #excerpt_112b, #region_7
```

Changing an ID is a removal and an addition, and so is renaming or moving a manifest
file. A removed ID is never given to another element.
