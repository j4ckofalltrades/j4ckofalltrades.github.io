---
title: What's in my .gitconfig?
date: 2025-03-15T12:21:19+08:00
description: Diving into my current git configuration.
draft: false
categories:
tags:
  - git
  - gitconfig
series:
images:
featured_image:
asciinema: true
mastodonPostID:
---
With git recently celebrating its 20th birthday, this seemed like a good time to share and review my current git configuration.

Most of the settings here are things that I've picked up from years of ~~messing up rebases~~ using git, I also highly recommend reading GitButler's blog post on [How Core Git Developers Configure Git](https://blog.gitbutler.com/how-git-core-devs-configure-git/) where I ~~stole~~ copied a couple of interesting and useful settings.

Read on for a section by section breakdown or just skip ahead to the full [gitconfig](#my-gitconfig).

{{< toc >}}

## Autocorrect

Typos in your git commands? The following setting will search for the closest matching valid command and suggest that to you.

```bash
git config --global help.autocorrect prompt
```

A prompt will only be shown if there is only one command that's a significantly close match, otherwise it will exit and list all the commands that are close matches.

```bash
$ git branhc
WARNING: You called a Git command named 'branhc', which does not exist.
Run 'branch' instead? (y/N)y
* main

$ git com
git: 'com' is not a git command. See 'git --help'.

The most similar commands are
        commit
        column
```

## Aliases

Git allows you to configure your own custom commands and to run external commands.

```bash
git config --global alias.lg "log --oneline --graph --abbrev-commit --date=relative"
```

Alternatively you can edit your `.gitconfig` file directly. If you have any external scripts, you can call them from git with `!/path/to/your/script`.

```text
[alias]
  bbr = !better-git-branch.sh
  br = branch
  cm = commit
  di = diff
  dis = diff --staged
  lg = log --oneline --graph --abbrev-commit --date=relative
  rs = restore
  rss = restore --staged
  st = status -sb
  sh = stash
  sw = switch
```

When your aliases are registered in git, they behave as valid git commands which means they also benefit from git's autocompletion and autocorrect.

```bash
# tab completion works for all aliases
$ git rs
rs    rss

# if autocorrect is enabled
$ git cx
WARNING: You called a Git command named 'cx', which does not exist.
Run 'cm' instead? (y/N)y
On branch main
nothing to commit, working tree clean
```

## Sorting

Sort branches by the most recent commit date.

```bash
git config --global branch.sort -committerdate
```

```bash
$ git branch
  feat-3
  feat-2
  bugfix-1
* main
```

This setting treats version numbers as integers instead of the full tag name as a string.

```bash
git config --global tag.sort version:refname
```

```bash
# default "sort" for tags
$ git tag
1.0.0
1.0.1000
1.0.101
1.0.2

# when tag.sort is set to version:refname
$ git tag
1.0.0
1.0.2
1.0.101
1.0.1000
```

## Fetch

Deletes references to all branches and tags that do not exist on the remote repository. This will not delete local branches or tags. 

```bash
git config --global fetch.prune true
git config --global fetch.pruneTags true
```

## Push

Simplifies pushing to just `git push` in most cases.

```bash
git config --global push.default simple
git config --global push.autoSetupRemote true
git config --global push.followTags true
```

The `push.default = simple` setting pushes the current branch to the branch with the same name on the remote (default since v2.0), while `push.autoSetupRemote = true` gets rid of the following error which you might be familiar with:

```bash
$ git push
fatal: The current branch my-branch-name has no upstream branch.
To push the current branch and set the remote as upstream, use

    git push --set-upstream origin my-branch-name
```

`push.followTags = true` automatically pushes local tags that aren't on the remote, otherwise you need to explicitly use the `--tags` flag.

## Diff

```bash
git config --global diff.colorMoved plain
git config --global diff.mnemonicPrefix true
git config --global diff.renames true
git config --global diff.algorithm histogram
```

`diff.colorMoved` displays moved lines in a different color from added/deleted lines while  `diff.renames` detects if a file has been renamed.

`diff.mnemonicPrefix` instructs git to show a different prefix (defaults to `a/`, `b/`) depending on what is being compared e.g. `git diff` will use `i/` and `w/` as it compares (i)ndex and (w)ork tree. Refer to the [docs](https://git-scm.com/docs/git-config#Documentation/git-config.txt-codediffmnemonicPrefixcode) for the full list.

There are 4 diff algorithms that are shipped with git: `myers` (default), `minimal`, `patience`, and `histogram`.

Here's an example of two diffs using different algorithms, the first one using the default `myers` algorithm and the other one using `histogram`.

```diff
void func1() {
     x += 1
+}
+
+void functhreehalves() {
+    x += 1.5
}
 
void func2() {
     x += 2
}
```

```diff
void func1() {
     x += 1
}

+void functhreehalves() {
+    x += 1.5
+}
+
void func2() {
     x += 2
}
```

## Editor

> Q: What is the best text editor?
>
> A: The correct answer is vi.

This setting tells git what editor to use editing commit and tag messages.

```bash
git config --global core.editor vi
```

## Global gitignore

Effectively a `.gitignore` that applies to all of your repositories.

```bash
git config --global core.excludesfile ~/.gitignore-global
```

```ini
.DS_Store
*.orig
*.swp

.idea/
.vscode/
```

## External tooling: delta

_delta_ is a syntax-highlighting pager for git, diff, grep, and blame output.

```bash
git config --global core.pager delta
git config --global interactive.diffFilter 'delta --color-only'
git config --global merge.conflictStyle zdiff3
```

The configuration I use shows a side-by-side diff view (instead of the default unified diff) for the Solarized Dark colorscheme.

delta has a lot of features and is very customizable, check out the [docs](https://dandavison.github.io/delta/) for all available configuration options and sample usages.

```bash
[delta]
	navigate = true  # use n and N to move between diff sections
    dark = true      # or light = true, or omit for auto-detection
	features = side-by-side unobtrusive-line-numbers decorations
	whitespace-error-style = 22 reverse
	syntax-theme = Solarized (dark)
	plus-style = "syntax #003800"
	minus-style = "syntax #3f0001"

[delta "unobtrusive-line-numbers"]
	line-numbers = true
	line-numbers-minus-style = "#444444"
	line-numbers-zero-style = "#444444"
	line-numbers-plus-style = "#444444"
	line-numbers-left-format = "{nm:>4}┊"
	line-numbers-right-format = "{np:>4}│"
	line-numbers-left-style = blue
	line-numbers-right-style = blue

[delta "decorations"]
	commit-decoration-style = bold yellow box ul
	file-style = bold yellow ul
	file-decoration-style = none
	hunk-header-decoration-style = yellow box
```

Side-by-side diff output:

![git-delta side-by-side](https://res.cloudinary.com/j4ckofalltrades/image/upload/f_auto,q_auto/v1/blog/git-delta-side-by-side)

## Rebase

Some quality of life changes if you use _rebase_ as your merge strategy.

```bash
git config --global pull.rebase true
git config --global rebase.autoSquash true
git config --global rebase.autoStash true
git config --global rebase.updateRefs true
```

`pull.rebase` tells git what to do when you run into conflicts when pulling changes.

`rebase.autoStash` automatically stashes your working tree changes (if there are any) when the _rebase_ operation starts and applies it after the operation ends.

`rebase.updateRefs` automatically force-update any branches that point to commits that are being rebased.

`rebase.autoSquash` used in combination with fixup commits allows you to amend a series of commits automatically to maintain a clean history. I recommend reading GitButler's autosquash explainer on their [blog](https://blog.gitbutler.com/git-autosquash/). 

## Reuse Recorded Resolution

This setting is useful if you're rebasing multiple commits and run into the same conflict again and again.

With `rerere.enabled` set,  git will remember how to resolve a conflict and _reuse_ it when it encounters the same conflict.  `rerere.autoUpdate` automatically stages the changes that resulted from the resolution.

```bash
git config --global rerere.enabled true
git config --global rerere.autoUpdate true
```

## Signing commits and tags

You can use GPG or SSH to sign tags and commits locally.

```bash
# tell git about your signing key
git config --global user.signingKey 49CC7B953A9FA12D
# sign commits
git config --global commit.gpgSign true
# sign tags
git config --global commit.gpgSign true
```

Alternatively, you can also sign commits and tags with a SSH key.

```bash
git config --global gpg.format ssh
git config --global user.signingkey /path/to/.ssh/key.pub
```

Git hosting providers e.g. GitHub will then mark signed commits (or tags) that are verifiable as _Verified_.

![GitHub verified commit](https://res.cloudinary.com/j4ckofalltrades/image/upload/f_auto,q_auto/v1/blog/gh-verified-commit)

## Multiple profiles

Imagine the following scenario, you have repositories that you want to associate with your work email and your personal email for the other ones.

```bash
$ tree
.
├── .gitconfig
├── .gitconfig-oss
├── .gitconfig-work
├── oss
│   ├── oss-repo-1
│   └── oss-repo-2
└── work
    └── work-repo-1
```

git provides a facility to conditionally load config files that meet a certain condition.

In the example below, a different user email and signingKey is used depending on the location of the current repository. This is done with git config's _conditional includes_ and the _gitdir_ property.

```bash
# .gitconfig
[includeIf "gitdir:~/oss/"]
	path = ~/.gitconfig-oss

[includeIf "gitdir:~/work/"]
	path = ~/.gitconfig-work
```

```bash
# .gitconfig-oss
[user]
	email = my-oss-email.com
	signingKey = ABC123DEF
```

```bash
# .gitconfig-work
[user]
	email = my-work-email.com
	signingKey = 101XYZ101
```

## My gitconfig

```bash
[user]
	name = Jordan Duabe

[includeIf "gitdir:~/oss/"]
	path = ~/.gitconfig-oss

[includeIf "gitdir:~/work/"]
	path = ~/.gitconfig-work

[init]
	defaultBranch = main

[help]
	autoCorrect = prompt

[alias]
	bbr = !better-git-branch.sh
	br = branch
	cm = commit
	di = diff
	dis = diff --staged
	lg = log --oneline --graph --abbrev-commit --date=relative
	rs = restore
	rss = restore --staged
	st = status --short
	sh = stash
	sw = switch

[core]
	editor = vi
	pager = delta

[merge]
	conflictstyle = zdiff3

[delta]
	features = side-by-side unobtrusive-line-numbers decorations
	whitespace-error-style = 22 reverse
	syntax-theme = Solarized (dark)
	plus-style = "syntax #003800"
	minus-style = "syntax #3f0001"
	navigate = true

[delta "unobtrusive-line-numbers"]
	line-numbers = true
	line-numbers-minus-style = "#444444"
	line-numbers-zero-style = "#444444"
	line-numbers-plus-style = "#444444"
	line-numbers-left-format = "{nm:>4}┊"
	line-numbers-right-format = "{np:>4}│"
	line-numbers-left-style = blue
	line-numbers-right-style = blue

[delta "decorations"]
	commit-decoration-style = bold yellow box ul
	file-style = bold yellow ul
	file-decoration-style = none
	hunk-header-decoration-style = yellow box

[pull]
	rebase = true

[rebase]
	autoSquash = true
	autoStash = true
	updateRefs = true

[interactive]
	diffFilter = delta --color-only

[diff]
	algorithm = histogram
	colorMoved = default
	mnemonicPrefix = true
	renames = true

[branch]
	sort = -committerdate

[commit]
	gpgSign = true

[tag]
	sort = version:refname
	gpgSign = true

[rerere]
	enabled = true
	autoUpdate = true

[fetch]
	prune = true
	pruneTags = true

[push]
	simple = true
	autoSetupRemote = true
	followTags = true

[filter "lfs"]
	smudge = git-lfs smudge -- %f
	process = git-lfs filter-process
	required = true
	clean = git-lfs clean -- %f
```

Try out and experiment with any of these settings and see what fits with your workflow.

Remember, the best configuration is the one that works for you.
