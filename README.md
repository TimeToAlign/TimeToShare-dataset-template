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
| `.pre-commit-config.yaml` | the git hooks that validate the manifests and write and check the commit type |
| `.github/workflows/validate.yml` | validates every manifest on each push and pull request |
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
   only: one recording timeline in seconds with an excerpt as its child, and one
   score timeline in quarters with a region.
5. Pin TimeToShare, in three places, to one TimeToShare release tag (`v0.3.0`) or
   commit hash:
   - `.pre-commit-config.yaml`: the `rev:` of the TimeToShare repository;
   - `.github/workflows/release.yml`: the `@main` at the end of the `uses:` line of the
     job `check`, and its input `timetoshare-ref:`.

   Each carries the comment `# pin to a TimeToShare release tag or commit`. The
   releases are listed at <https://github.com/TimeToAlign/TimeToShare/releases>;
   `pre-commit autoupdate --freeze --repo https://github.com/TimeToAlign/TimeToShare`
   writes the commit of the latest release into `.pre-commit-config.yaml`, after
   which the same hash goes into both places in `release.yml`.

   Pinning matters because the version of each release is decided by TimeToShare's
   step rules, once by the hooks on the machine of whoever commits and once by the
   check before a release is tagged. Both must run one known version of those rules:
   otherwise a commit the hooks accept can be refused at release time, or the reverse.
   `main` moves, and pre-commit keeps the environment it built for a `rev` without
   looking again, so two machines can hold different versions of `main` for months.
   Move all three places together, in one commit. TimeToShare reads only the format
   version it was released for, so a TimeToShare release of a newer format version
   comes with converting the manifests, in the same commit.
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
  manifests and tables with the last commit and writes the header the change
  requires: the type of the highest step, a scope, and for a removal `!` and the
  footer `BREAKING CHANGE: removes …` naming the removed IDs. You write the subject
  only. Staging timelines added to `bpsd/op106/manifest.yaml` and
  `bpsd/op109/manifest.yaml` and running

  ```console
  git commit -m "add the BPSD recordings"
  ```

  commits `feat(bpsd): add the BPSD recordings`. The scope is the manifest file's
  name without its extension when the change lies in one manifest, the top folder
  when it lies in several manifests under one, and absent otherwise; where every
  manifest is called `manifest.yaml`, a change to one of them gets the scope
  `manifest`, so name the piece by typing the header yourself
  (`git commit -m "feat(op106): add the BPSD recordings"`). A header you type is kept
  as it is. When the commit holds smaller changes beside the highest one, the hook
  counts them in a line `also: …` (`also: 2 belief, 1 label`); splitting the commit
  keeps the changelog finer. Merges, amends, squashes and commits that change no data
  are left to you.
- **`timetoshare-commit-check`** (on the final message) refuses a message whose type
  gives a lower step than the staged change (a `label` commit that adds a timeline, a
  removal written as anything but `remove` with `!`, a `remove` without `!`), a header
  without a subject, and an ID given to an element of another type. It warns where the
  type gives a higher step than the change, and where the `BREAKING CHANGE` footer
  misses a removed ID or stands on a commit that removes nothing.

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
every other type a patch. A dataset repository gives these types data meanings:

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
- **No step:** another serialization or form of the same value, comments.
- **Refused:** giving an ID to another element, whether of another type or after its
  own element was removed in an earlier release. An ID that was published stays
  bound to what it named.

## Releases

On every push to `main`, the workflow `release.yml` first runs the check, then
release-please. release-please keeps one release pull request open, titled with the
next version, which updates `CHANGELOG.md` and `.release-please-manifest.json`; each
further push updates it. Merging it publishes the release: release-please tags the
merge commit `vX.Y.Z` and creates the GitHub release with the changelog entry as its
notes. On the push that merges a release pull request, the check compares the data at
the last release tag with the merged state and fails when the proposed version is
lower than the data's change requires (a removal under a minor version, say),
naming the IDs; release-please then tags nothing. The remedy is a commit whose type
says what happened, or a commit with a `Release-As: X.Y.Z` footer. On every other
push, the check writes the step of the pending release into the job summary.

**Drafts.** Until the dataset is first published, its releases are `0.y.z`, starting
at `0.1.0`: a removal raises the middle number (`0.3.2` to `0.4.0`), an addition or
edit the last (`0.3.2` to `0.3.3`), and every other change the last as well.

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
