# Automated Git Versioning Tool `git-auto-version`

The `git-auto-version` tool sets git's **version tags** and generates the
**ChangeLog** file, _**all automatically**_, only from the content of commit
messages.  You just need to write your commit messages in a syntax similar to
[conventional commits](https://www.conventionalcommits.org/).

## How does it work?

Each commit message should be prefixed with a _commit type_ like `fix`, `feat`,
`docs`, `chore`, etc., that describes the content and the purpose of the
commit.  For example:

```git
fix: Fixed bug with planes crashing
```

With `git-auto-version`, you can choose type names freely and you don't even
need to use the types unless you want to increase the version number.  Only the
type `chore` has a special meaning for the ChangeLog below.

### Version control

Version is a triple of numbers (_major_, _minor_, _patch_), for example,
`1.2.0`.  To increase the _patch_ version, you just put `!` after the type in
your commit message.  Similarly, to increase the _minor_ (and reset _patch_ to
zero), you use `!!`, and to increase the _major_, you use `!!!`.  For example,
if the current version is `1.2.23` and you commit changes with the message
```git
release!!: New version released
```
the version becomes `1.3.0`.  Git commits are automatically tagged with git's
version tags, like `v1.3.2`, after every commit.

### ChangeLog generation

The file `ChangeLog.md` is automatically generated and updated with every
commit.  Basically, every commit message becomes one line in the ChangeLog and
the commits are grouped by versions.  So, you should write your commit messages
with this in mind.

The change log is formatted in [MarkDown](https://www.markdownguide.org/) and
it might look like this:

> ## v0.0.2 (2024-08-10)
>
> * feat!: Support for wing flips [[details](https://github.com/cbboyan/solverpy/commit/249e817)]
> * refactor: Converted code to modules [[details](https://github.com/cbboyan/solverpy/commit/249e817)]
> * docs: Added documentation [[details](https://github.com/cbboyan/solverpy/commit/6c9ba77)]
>
> ## v0.0.1 (2024-08-09)
>
> * fix!: Fixed bug with planes crashing [[details](https://github.com/cbboyan/solverpy/commit/249e817)]
> * init: Initial version [[details](https://github.com/cbboyan/solverpy/commit/6c9ba77)]

If you want to hide some commit from the ChangeLog, use the commit type `chore`.

## Installation

1. Copy `gitautoversion.py` from this repository to the root of your git
   project.
2. Set up the git's `post-commit` hook by copying the file `post-commit` from
   this repository to `.git/hooks` and making it executable.

Optionally:

* If you want to automatically push the tags to the remote, set `followTags` to
  true as follows:
  ```bash
  $ git config push.followTags true
  ```

* If you want to adjust commits ignored for the ChangeLog, edit the variables
  `SKIP_TYPE` or `SKIP_MSG` at the top of `gitautoversion.py`.

## How is it implemented?

All the stuff is done in the git's `post-commit` hook.  First, `ChangeLog.md`
is updated based on the current commit message.  The commit is then updated
with the up-to-date ChangeLog using the `git commit --amend` command.  After
that, missing version tags are set.

A technical "tail-chasing" problem here is that you need to know the current
commit message to generate the ChangeLog.  Hence, you can not (at least easily)
generate the ChangeLog in the `pre-commit` hook because the message is not yet
known.  In later commit hooks, you can no longer update the content of the
commit since its hash is already computed.  Hence, we wait for `post-commit`
and _amend_ it.  Note that there is still a problem because amending the commit
changes its hash code, which is used in the ChangeLog as a GitHub link.  Hence,
the hash code of the last commit is excluded from the ChangeLog.  Luckily, the
GitHub link without any hash automatically opens the latest commit, so the
desired functionality is preserved (at least for GitHub).



