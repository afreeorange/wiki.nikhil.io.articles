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

## Large File Storage (LFS)

Initialize with

```bash
git lfs install
```

Then add a `.gitattributes` file:

```bash
# Image files
*.bmp filter=lfs diff=lfs merge=lfs -text
*.gif filter=lfs diff=lfs merge=lfs -text
*.jpeg filter=lfs diff=lfs merge=lfs -text
*.jpg filter=lfs diff=lfs merge=lfs -text
*.png filter=lfs diff=lfs merge=lfs -text
*.tiff filter=lfs diff=lfs merge=lfs -text
*.webp filter=lfs diff=lfs merge=lfs -text

# Audio files
*.aac filter=lfs diff=lfs merge=lfs -text
*.flac filter=lfs diff=lfs merge=lfs -text
*.mp3 filter=lfs diff=lfs merge=lfs -text
*.wav filter=lfs diff=lfs merge=lfs -text

# Video files
*.avi filter=lfs diff=lfs merge=lfs -text
*.mkv filter=lfs diff=lfs merge=lfs -text
*.mov filter=lfs diff=lfs merge=lfs -text
*.mp4 filter=lfs diff=lfs merge=lfs -text

# Document files
*.doc filter=lfs diff=lfs merge=lfs -text
*.docx filter=lfs diff=lfs merge=lfs -text
*.pdf filter=lfs diff=lfs merge=lfs -text
*.ppt filter=lfs diff=lfs merge=lfs -text
*.pptx filter=lfs diff=lfs merge=lfs -text

# Archive files
*.zip filter=lfs diff=lfs merge=lfs -text
*.tar filter=lfs diff=lfs merge=lfs -text
*.gz filter=lfs diff=lfs merge=lfs -text
*.rar filter=lfs diff=lfs merge=lfs -text

# Font files
*.woff filter=lfs diff=lfs merge=lfs -text
*.woff2 filter=lfs diff=lfs merge=lfs -text
*.eot filter=lfs diff=lfs merge=lfs -text

# Other binary files
*.exe filter=lfs diff=lfs merge=lfs -text
*.dll filter=lfs diff=lfs merge=lfs -text
*.bin filter=lfs diff=lfs merge=lfs -text
*.iso filter=lfs diff=lfs merge=lfs -text
```

Migrate existing repos with

```bash
git lfs migrate import --everything --include="*.bmp,*.gif,*.jpeg,*.jpg,*.png,*.tiff,*.webp,*.aac,*.flac,*.mp3,*.wav,*.avi,*.mkv,*.mov,*.mp4,*.doc,*.docx,*.pdf,*.ppt,*.pptx,*.zip,*.tar,*.gz,*.rar,*.woff,*.woff2,*.eot,*.exe,*.dll,*.bin,*.iso"
```

Now the `.git` folder _might_ be larger than before, which _sort of_ beats the whole point. This might be because:

- When you migrate files to LFS, Git keeps the original history alongside the new LFS pointers until garbage collection runs. You essentially have both versions temporarily stored.
- Git LFS maintains its own metadata structures in the `.git/lfs` directory.
- The migration process creates entries in Git's reflog, which tracks reference changes.

You can run Garbage Collection with

```bash
git gc --aggressive --prune=now
```

This has worked for me pretty well.

👉 To _completely_ remove old binary files from history (!!! REWRITES HISTORY !!!)

```bash
git filter-branch --force --index-filter "git rm --cached --ignore-unmatch path/to/files/*" --prune-empty --tag-name-filter cat -- --all
git reflog expire --expire=now --all
git gc --aggressive --prune=now
```

I'd make a backup.

## References

* [Advanced Git: Graphs, Hashes, and Compression, Oh My!](https://www.youtube.com/watch?v=ig5E8CcdM9g)
