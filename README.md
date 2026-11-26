# TimeToShare dataset template

## What this template gives

A repository made from this template publishes one dataset described by
[TimeToShare](https://github.com/TimeToAlign/TimeToShare) manifests, together with
the tables the manifests name. Its releases are versioned by what happened to the
data since the last release: removing something makes a new major version, adding or
editing data a new minor version, and names, drawing, application data, moves and
beliefs a new patch version. The manifests carry no version of their own: the
`timetoshare` value in each one names the format version it is written in, and the
release tag of the repository (`v2.1.0`) names the state of all its data together.
The template holds configuration, this documentation and one sample manifest, and no
code: every tool comes from TimeToShare, at the version the configuration names.

| File | Does |
| --- | --- |
| `.pre-commit-config.yaml` | the git hooks that validate the manifests, write or suggest the commit type, and check it |
| `.github/workflows/validate.yml` | validates every manifest on each push to `main` and each pull request |
| `.github/workflows/release.yml` | checks the version step of the data, then runs release-please |
| `release-please-config.json`, `.release-please-manifest.json` | release-please's configuration and the current version |
| `.gitattributes`, `.gitignore` | line endings for YAML and JSON, Parquet as binary; files git ignores |
| `example/manifest.yaml` | a sample manifest, to be replaced |

## Create a dataset repository from it

1. On GitHub, click **Use this template**, then **Create a new repository**.
2. Name the repository after the dataset. One repository holds one dataset: its
   release tags version everything in it together, so two datasets in one repository
   could not be released apart.
3. Choose a licence for the data and add it as `LICENSE`. The template includes none,
   since data licences differ from dataset to dataset (Creative Commons licences are
   common for data; tables imported from elsewhere keep their own terms, which
   `FETCH.md` records, see [Laying out the data](#laying-out-the-data)).
4. Replace `example/manifest.yaml` with the dataset's own manifests. It is a sample
   only: one recording timeline in seconds with an excerpt as its child, one score
   timeline in quarters with a region, and the inline `@context` with the format's
   terms, which every manifest carries.
5. Keep TimeToShare pinned to one commit, the same full commit hash in three places.
   The template comes pinned already; to move to a newer TimeToShare, change all three:
   - `.pre-commit-config.yaml`: the `rev:` of the TimeToShare repository;
   - `.github/workflows/release.yml`: the hash after the `@` at the end of the
     `uses:` line of the job `check`;
   - `.github/workflows/release.yml`: the job's input `timetoshare-ref:`.

   `git ls-remote https://github.com/TimeToAlign/TimeToShare main` prints the latest
   commit of TimeToShare's `main`. Pin a commit of `main`, not a release tag, until a
   TimeToShare release carries the hooks and format version 0.2.0: the latest release,
   `v0.1.0`, has neither. For the same reason, do not run `pre-commit autoupdate`
   until then: it moves `rev:` to the latest release tag, which lacks the hooks.

   Pinning matters because the version of each release is decided by TimeToShare's
   step rules, once by the hooks on the machine of whoever commits and once by the
   check before a release is tagged. Both must run one known version of those rules:
   otherwise a commit the hooks accept can be refused at release time, or the reverse.
   `main` moves, and pre-commit keeps the environment it built for a `rev` without
   looking again, so two machines can hold different versions of `main` for months.
   Move all three places together, to the same new TimeToShare commit, in one commit
   of this repository. TimeToShare reads only the format version it was released for,
   so moving to a TimeToShare version of a newer format version comes with converting
   the manifests, in the same commit.
6. In the repository's **Settings → Actions → General → Workflow permissions**, allow
   GitHub Actions to create and approve pull requests, so that release-please can open
   its release pull request.
7. Before the first release, set up archiving, see [Archiving with a
   DOI](#archiving-with-a-doi).

## Set up on your machine

Install [pre-commit](https://pre-commit.com/) once:

```console
pipx install pre-commit
```

or `pip install pre-commit`. Then, in every clone of the repository:

```console
pre-commit install --hook-type pre-commit --hook-type prepare-commit-msg --hook-type commit-msg
```

pre-commit fetches the pinned TimeToShare and installs three hooks:

- **`timetoshare-validate`** (before the commit) validates every staged manifest and
  stops the commit when one is invalid.
- **`timetoshare-commit-type`** (before the message is edited) compares the staged
  manifests and tables with the last commit and finds what kind of change they hold.
  For additions, edits and removals it writes the type itself, when the message has
  none yet:
  `feat(<scope>): ` for an addition or an edit, and for a removal
  `remove(<scope>)!: ` with the footer `BREAKING CHANGE: removes …` naming the
  removed IDs. You write the subject only. Staging timelines added to
  `bpsd/op106/manifest.yaml` and running

  ```console
  git commit -m "add the BPSD recordings"
  ```

  commits `feat(op106): add the BPSD recordings`.

  For the smaller kinds (beliefs, names, drawing, application data, moved files) it
  writes nothing and suggests a type instead. When you write the message in the
  editor, the suggestion stands there as a comment line, which git removes from the
  message; when you give the message with `-m`, the hook prints the suggestion in the
  terminal. Staging a renamed timeline in `bpsd/op106/manifest.yaml` and running

  ```console
  git commit -m "name the movements"
  ```

  prints the suggestion `label(op106): …` and commits `name the movements` as you
  wrote it. Add the type if you want the change to show in the changelog
  (`git commit -m "label(op106): name the movements"`), or leave the commit without
  one.

  The scope is found the same way in both cases. When the change lies in one
  manifest, it is the name of the folder holding it if the file is called `manifest`
  with any extension (`bpsd/op106/manifest.yaml` gives `op106`), and the file's name
  without its extension otherwise; when it lies in several manifests under one top
  folder, it is that folder (`bpsd/op106/manifest.yaml` and
  `bpsd/op109/manifest.yaml` give `bpsd`); otherwise there is none. A type you write
  yourself is kept as it is.

  When a commit holds several kinds of change, say a new timeline and a renamed one,
  the hook suggests making one commit for each. Small commits, one change at a time,
  keep the history readable and give every change its own line in the changelog.
  Merges, amends, squashes and commits that change no data are left to you.
- **`timetoshare-commit-check`** (on the final message) refuses the commit only when
  the message would hide what the data change does:
  - a type written by hand that hides an addition, an edit or a removal (a `label`
    commit that adds a timeline, say);
  - a removal not typed `remove(…)!` (a `feat` that removes, a `remove` without `!`);
  - an ID given to another element;
  - a type with no subject after it;
  - a message without a type for a change that adds, edits or removes, which happens
    only when the first hook did not run (a commit made without the hooks installed
    in that clone, for instance).

  Everything else is a warning, and the commit goes through: a missing type on a
  belief, a name, drawing, application data or a move; a type that names a larger
  step than the change; a `BREAKING CHANGE` footer that misses a removed ID or stands
  on a commit that removes nothing.

## Laying out the data

Manifests may lie anywhere in the repository; by convention each piece has its own
folder, `<part>/<piece>/manifest.yaml`. Every table lies beside the manifest whose
`resources` name it. Write manifests in canonical YAML (TimeToShare's `dumps`) so
that diffs show only what changed.

A manifest's path is part of its address: everyone citing one of its elements uses
it. Renaming or moving a manifest file therefore counts as removing that manifest and
adding another, which makes a new major version. Choose the layout before the first
release.

Files imported from elsewhere (recordings' metadata, tables, scores) are recorded in a
`FETCH.md` next to them: the source (URL or citation), the date of the download, the
licence, and a checksum of each file (`sha256sum`).

## Commit types

release-please computes each version from the
[Conventional Commits](https://www.conventionalcommits.org/) types of the commits since
the last release: `feat` makes a minor step, `!` after the type or scope a major step,
every other type a patch. A commit without a type does not appear in the changelog
and raises no version: release-please reads typed commits only. A dataset repository
gives these types data meanings:

| Type | Instead of | Step | Changelog section | For |
|---|---|---|---|---|
| `feat` | `feat`; `fix` for a correction | minor | Added or changed | adding an element, a member, a pair, a resource, a MatchClaim, a row, a manifest; editing what an element states, corrections included. The subject starts with its verb: `feat(op106): add the BPSD recordings`, `feat(op106): correct the offset of movement 2` |
| `remove` with `!` | `feat!` | major (minor below 1.0.0) | Removed, and the removed IDs under "⚠ BREAKING CHANGES" | removing anything with an ID, a member, a pair, a MatchClaim, a row, a manifest, or changing an ID; the footer `BREAKING CHANGE: removes #clt_3, #region_7` names what goes. `remove(mtd)!: drop the duplicate excerpt of theme 112` |
| `belief` | `fix` | patch | Beliefs | recording a belief: confirming, rejecting, finding unlikely. `belief(op106): reject MatchClaim mc_3kq9x0m2ztb7a, off by one beat` |
| `label` | `chore` | patch | Names | the `name` of the manifest or of any element |
| `graphic` | `style` | patch | Drawing | `graphical`: styling rules, orders, display words, elisions |
| `meta` | `chore` | patch | Application data | `meta` |
| `move` | `refactor` | patch | Moved | a descriptor's `url` changed for the same bytes. Moving a manifest file changes its address and is a removal |
| `docs` | `docs` | patch | Documentation | README, FETCH.md and other prose |
| `fix` | `fix` | patch | Scripts | a fix to the repository's own scripts or loaders |
| `chore`, `build`, `ci`, `refactor`, `test`, `style` | the same | none of their own (hidden) | — | everything else about the repository's code |

The hooks write `feat` and `remove` with `!` themselves, since a change of that kind
must be typed. They only suggest `belief`, `label`, `graphic`, `meta` and `move`: you
may add the suggested type, which gives the change a line in the changelog and a
patch release, or leave the commit untyped. The other types are yours to choose.

The scope names the manifest or the part of the dataset (`op106`, `bpsd`, `mtd`). A
commit touching data and code takes the type of its data. Code (scripts, notebooks,
loaders) is committed as `build`, `chore` or `refactor` (`fix` for a fix to it), never
as `feat`: `feat` announces new data.

## What each step means

A release compares the tree of the last release tag with the new one, every manifest
and every table together, by value, elements matched by their IDs at any depth. The
full rules are in TimeToShare's README, section "Versions of a dataset".

- **Major: removal.** Removing anything with an ID (a timeline, a child, a group, a
  map, a region, a flow-control element, a resource, a named timestamp, an anchor and
  the rest), a group member, a statement pair, a MatchClaim, a row of a table, or a
  manifest file. Changing an ID is a removal and an addition, so it is major too.
- **Minor: additions and edits.** Adding any of these, and editing what an element
  states (a unit, a length, an offset, a map's numbers, a coordinate, a statement's
  value), corrections included. A major version promises that every cited ID still
  exists, not that every coordinate keeps its meaning: whoever needs to reproduce a
  result exactly cites the full release.
- **Patch: names, drawing, application data, moves and beliefs.** A `name`, styling
  and elisions in `graphical`, `meta`, a resource's `url` changed for the same bytes,
  and a belief, a rejection included.
- **No step:** another serialization or form of the same value, comments, and a
  conversion to a later TimeToShare format version that keeps every element.
- **Refused:** giving an ID to another element, whether of another type or after its
  own element was removed in an earlier release. An ID that was published stays
  bound to what it named.

## Releases

On every push to `main`, the workflow `release.yml` first runs the check, then
release-please. release-please keeps one release pull request open, titled with the
next version, which updates `CHANGELOG.md` and `.release-please-manifest.json`; each
further push updates it. Merging it publishes the release: release-please tags the
merge commit `vX.Y.Z` and creates the GitHub release with the changelog entry as its
notes.

**The check** treats a push as a release when the version in
`.release-please-manifest.json` is not `0.0.0` and has no tag `v<version>` yet: that is
the push merging a release pull request, which raised the version. On that push it
compares the data at the latest version tag (before the first release, an empty
repository) with the pushed state, Git LFS files included, and fails, so that
release-please tags nothing, when

- the version is lower than the change of the data requires (a removal under a minor
  version, say), naming the IDs;
- an ID is reused: it names another element than in an earlier release;
- a change cannot be determined, such as a table that could not be compared.

The remedy is a commit whose type says what happened, or a commit with a
`Release-As: X.Y.Z` footer; a change that could not be determined is made readable
first (a table that parses, an LFS file that is pushed). On every other push, the
check writes the step of the pending release into the job summary and fails nothing.

**Drafts.** Until the dataset is first published, its releases are `0.y.z`: a removal
raises the middle number (`0.3.2` to `0.4.0`), an addition or edit the last (`0.3.2`
to `0.3.3`), and every other change the last as well.

**The first release is `0.1.0`.** `.release-please-manifest.json` starts at `0.0.0`,
which stands for no release, and `release-please-config.json` sets `initial-version`
to `0.1.0`, so the first release pull request proposes `0.1.0`. Without that setting
release-please would propose `1.0.0`, which would announce the dataset as published
before anyone decided so; `0.1.0` keeps the first releases drafts, compared by the
check with the empty repository, where all their data counts as added.

**The first publication** is `1.0.0`, set by a person, with an empty commit whose
message carries the footer:

```console
git commit --allow-empty -m "chore: publish the dataset" -m "Release-As: 1.0.0"
```

release-please then proposes `1.0.0`; from there on, removals raise the first number.

**Never move or delete a tag.** Citations point at tags; a tag that moves changes what
they cite. A mistake in a release is corrected by the next one.

**Citing a release**: name the repository and the tag (`<owner>/<repository>
v1.2.0`), or the release's version DOI, see below.

## Archiving with a DOI

Before the first release, sign in to [Zenodo](https://zenodo.org/) with the GitHub
account that owns the repository, open **GitHub** in the account settings, and switch
the repository on. Every GitHub release published after that is archived by Zenodo and
gets a version DOI of its own, and Zenodo mints one further DOI that stands for all
versions and resolves to the newest. Releases made before the switch are not archived.
Zenodo takes the record's metadata from a `CITATION.cff` or `.zenodo.json` file in the
repository, where there is one.

## Referring to this dataset from elsewhere

Every element of a manifest has an ID, written `#…` in the file, and a global address:
the manifest's URL with the ID as its fragment. Pin the address to a release by using
the URL at a release tag,

```text
https://raw.githubusercontent.com/<owner>/<repository>/v1.2.0/bpsd/op106/manifest.yaml#clt_3
```

or cite the release's version DOI together with the manifest's path and the ID. A
reference that names a release keeps resolving within its major version: the cited ID
exists in every later `1.x.y` release, so the reference may be moved to the newest one
of them; only a new major version can remove it.
