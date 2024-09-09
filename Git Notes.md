## Git is Content-Addressible Storage

Git is a CAS, a kind of database, with a VCS on top of it. The CAS commands are _plumbing_ commands that deal with this underlying database. The usual `git {add,rm,commit}` commands are compound _porcelain_ commands that build on top of these lower level commands.

This means that you can actually use the plumbing commands to manually add, remove, commit things. The "Git Internals" section of the handbook has great examples.

But I used `git cat-file` to really investigate a lot of things. `git cat-file -p` will determine the type of object before pretty catting it.

There are four Types of Objects: `blob`, `tree`, `tag`, and `commit`.

## Areas

There are three areas: **Working**, **Staging**, and **Repository**.

**Working -> Staging**

* When you do `git diff` you are comparing Working to Staging.
* When you `git add` you are adding stuff from the Working to Staging.
* `git reset` does the reverse: moves changes back to Working.

**Staging -> Respository**

* `git diff --staged` compares Staging to Repository
* When you `git commit` you are adding stuff from the Staging to Repository.

When you `git rm` you are 'adding' the deletion from the Working to the Staging area. This is a small head-scratcher until you see the `tree` object and note that the file(s) you just deleted are no longer referenced. That's ultimately what happens. The file blob is not deleted; it's part of `git` forever (until you use some tool to parse history and remove it manually.)

## Other Trivia

Everything git writes is just a blob. There's no binary/text types.

The reflog does not contain all commits.

## Snippets

### List the Largest Files in a Repo

```bash
# Here's one way. It's supposed to be *BLAZINGLY FAST*
git rev-list --objects --all |
  git cat-file --batch-check='%(objecttype) %(objectname) %(objectsize) %(rest)' |
  sed -n 's/^blob //p' |
  sort --numeric-sort --key=2 |
  cut -c 1-12,41- |
  $(command -v gnumfmt || echo numfmt) --field=2 --to=iec-i --suffix=B --padding=7 --round=nearest

# Here's another. It's simpler to read.
git rev-list --objects --all \
  | grep "$(git verify-pack -v .git/objects/pack/*.idx \
           | sort -k 3 -n \
           | tail -10 \
           | awk '{print$1}')"
```

## References

* [Advanced Git: Graphs, Hashes, and Compression, Oh My!](https://www.youtube.com/watch?v=ig5E8CcdM9g)

