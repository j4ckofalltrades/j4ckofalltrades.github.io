---
title: "Talk: git config --make-git-work-for-you"
date: 2025-10-19T20:58:12+08:00
description: Uncommon (and interesting?) git configuration options that I use.
draft: false
tags:
  - git
  - gitconfig
  - talk
categories:
mastodonPostID:
---
This is an annotated version of the [git config --make-git-work-for-you](https://talks.jduabe.dev/git-config/) talk that I presented at the [Perth GitHub Meetup group]() where I shared the git configuration options that I use.

{{< toc >}}

## Configuring git

![Sample contents of the .gitconfig](https://res.cloudinary.com/j4ckofalltrades/image/upload/v1760879209/talks/git-config/slide-06_m2le9c.png)

![git configuration scopes (system, global, local)](https://res.cloudinary.com/j4ckofalltrades/image/upload/v1760879209/talks/git-config/slide-05_ftripo.png)

`git` looks at several (OS-specific) locations where it reads the configuration options:

- _system_ - config options that apply to **all** the users of an environment
- _global_ - config options that apply to all repositories for the **current** user (typically where most if not all your config options live)
- local - config options that apply to the local repository

If a config option is present in several locations then the **local** config wins e.g., `local > global > system`.

## Quality of life config options

### Autocorrect

![Enable autocorrect](https://res.cloudinary.com/j4ckofalltrades/image/upload/v1760879208/talks/git-config/slide-09_fh1lxx.png)

![git autocorrect example (using git branhc typo)](https://res.cloudinary.com/j4ckofalltrades/image/upload/v1760879208/talks/git-config/slide-10_eu0v4b.png)

By default, if you enter a command that does not exist git will just exit -- if autocorrect is enabled then `git` will search for the closest matching valid command and suggest that to you. If there are multiple possible matches then `git` lists out those commands and then exits.

There are other possible options other than `prompt` which I recommend, namely `immediate` which will automatically execute the matched command, and a delay in deciseconds e.g. setting the value to `10` means that if a possible match is found then it is automatically executed after 1 second.   

### Aliases

![Configuring git aliases with the alias config option](https://res.cloudinary.com/j4ckofalltrades/image/upload/v1760879209/talks/git-config/slide-11_lspqok.png)

![Sample gitconfig with aliases](https://res.cloudinary.com/j4ckofalltrades/image/upload/v1760879209/talks/git-config/slide-12_mahxbm.png)

![Invoking git aliases and interaction with autocorrect](https://res.cloudinary.com/j4ckofalltrades/image/upload/v1760879209/talks/git-config/slide-13_hcrgv3.png)

`git` also allows you to configure your own custom aliases for frequent commands that you use as well as calling external scripts with `!/path/to/external/script`.

The registered aliases behave as valid commands which means that autocorrect will also work for them. 

### Sorting

![Sorting git branches by commit date](https://res.cloudinary.com/j4ckofalltrades/image/upload/v1760879209/talks/git-config/slide-14_vxtzee.png)

By default, branches are sorted alphabetically. Sorting branches by committer date in descending order sets the output of `git branch` to show the branches with the most recent commit at the top, this is  useful for quickly checking which of your local branches are possibly stale and can be removed. 

![Sorting tags](https://res.cloudinary.com/j4ckofalltrades/image/upload/v1760879209/talks/git-config/slide-15_jyilhb.png)

![Sorting tags with version:refname](https://res.cloudinary.com/j4ckofalltrades/image/upload/v1760879210/talks/git-config/slide-16_yhfnk1.png)

Tags are generally used to mark releases and commonly follow the semantic versioning specification which are in the format `MAJOR.MINOR.PATCH` e.g. `1.0.0`.

By default, tags are treated as a string and are sorted accordingly which leads to unexpected results -- by specifying `version:refname` tags are treated as numeric values.    

### Better fetch, diff, commit, push

![git fetch prune settings](https://res.cloudinary.com/j4ckofalltrades/image/upload/v1760879210/talks/git-config/slide-17_sq4xne.png)

Deletes references to all branches and tags that do not exist on the remote repository, no local branches or tags are deleted.

![https://res.cloudinary.com/j4ckofalltrades/image/upload/v1760879210/talks/git-config/slide-20_hjcpgk.png](https://res.cloudinary.com/j4ckofalltrades/image/upload/v1760879210/talks/git-config/slide-20_hjcpgk.png)

`diff.colorMoved` displays moved lines in a different color from added/deleted lines while diff.renames detects if a file has been renamed.

`diff.mnemonicPrefix` instructs git to show a different prefix (defaults to `a/`, `b/`) depending on what is being compared e.g., `git diff` will use `i/` and `w/` as it compares `(i)`ndex and `(w)`ork tree.

There are 4 diff algorithms that are shipped with git: myers (default), minimal, patience, and histogram. This discussion in [StackOverflow](https://stackoverflow.com/questions/32365271/whats-the-difference-between-git-diff-patience-and-git-diff-histogram) has more details on the differences (ha!) of these algorithms.

![git verbose commit](https://res.cloudinary.com/j4ckofalltrades/image/upload/v1760879217/talks/git-config/slide-23_uqxerf.png)

This setting will append the `diff` output to the commit editor. 

![https://res.cloudinary.com/j4ckofalltrades/image/upload/v1760879210/talks/git-config/slide-18_pl7jbq.png](https://res.cloudinary.com/j4ckofalltrades/image/upload/v1760879210/talks/git-config/slide-18_pl7jbq.png)

Simplifies pushing to just git push in most cases.

`push.default = simple` will automatically push commits from your local branch to the branch with the same name on the remote. This is the default behavior since `v2.0`.

`push.followTags` will push tags together with commits without the need of specifying the `--tags` option. 

![git push.autoSetupRemote error message](https://res.cloudinary.com/j4ckofalltrades/image/upload/v1760879210/talks/git-config/slide-19_adhdn4.png)

`push.autoSetupRemote` will automatically create a branch in the remote repository if it does not already exist and setup a remote tracking branch on your local repository.

### Global gitignore

![git global gitignore](https://res.cloudinary.com/j4ckofalltrades/image/upload/v1760879218/talks/git-config/slide-26_njc7oa.png)

Similar to a `.gitignore` but is applied across all repositories, the configured entries here should be generic e.g., swap files, editor configuration, and other generated files.

## Workflow-specific config options

### External tooling: delta

![delta](https://res.cloudinary.com/j4ckofalltrades/image/upload/v1760879218/talks/git-config/slide-28_rcx8jh.png)

![https://res.cloudinary.com/j4ckofalltrades/image/upload/v1760879218/talks/git-config/slide-29_jqinjs.png](https://res.cloudinary.com/j4ckofalltrades/image/upload/v1760879218/talks/git-config/slide-29_jqinjs.png)

[delta](https://github.com/dandavison/delta) is a syntax-highlighting pager for git `diff`, `grep`, and `blame` output.
### Rebase

![git rebase settings](https://res.cloudinary.com/j4ckofalltrades/image/upload/v1760879227/talks/git-config/slide-34_v4m8vs.png)

`pull.rebase` tells git what to do when you run into conflicts when pulling changes.

`rebase.autoStash` automatically stashes your working tree changes (if there are any) when the _rebase_ operation starts and applies it after the operation ends.

`rebase.updateRefs` automatically force-update any branches that point to commits that are being rebased.

`rebase.autoSquash` used in combination with fixup commits allows you to amend a series of commits automatically to maintain a clean history.

#### Demo: Fixup commits with autosquash

{{< asciicast src="/images/annotated-slides/git-config/git-rebase-autosquash.cast" terminalFontFamily="'JetBrains Mono', monospace" theme="solarized-dark" terminalFontSize="13px" fit="none" idleTimeLimit="1" speed="1" poster="npt:0:01" >}}

Fixup commits mark a target commit where the _new_ commit will be squashed together, otherwise you'll have to manually move commits around when performing an interactive rebase.

### Reuse Recorded Resolution

![https://res.cloudinary.com/j4ckofalltrades/image/upload/v1760879228/talks/git-config/slide-35_ui09cy.png](https://res.cloudinary.com/j4ckofalltrades/image/upload/v1760879228/talks/git-config/slide-35_ui09cy.png)

This setting is useful if you're rebasing multiple commits and run into the same conflict again and again.

With `rerere.enabled` set, git will remember how to resolve a conflict and reuse it when it encounters the same conflict.

`rerere.autoUpdate` automatically stages the changes that resulted from the resolution.

### Signing commits and tags

![https://res.cloudinary.com/j4ckofalltrades/image/upload/v1760879228/talks/git-config/slide-36_enwv7r.png](https://res.cloudinary.com/j4ckofalltrades/image/upload/v1760879228/talks/git-config/slide-36_enwv7r.png)

![https://res.cloudinary.com/j4ckofalltrades/image/upload/v1760879228/talks/git-config/slide-37_jb3xak.png](https://res.cloudinary.com/j4ckofalltrades/image/upload/v1760879228/talks/git-config/slide-37_jb3xak.png)

You can use either `GPG` or `SSH` to sign tags and commits locally.

![https://res.cloudinary.com/j4ckofalltrades/image/upload/v1760879229/talks/git-config/slide-38_uuccgi.png](https://res.cloudinary.com/j4ckofalltrades/image/upload/v1760879229/talks/git-config/slide-38_uuccgi.png)

Forges like GitHub will display signed commits as _Verified_. Organizations may also require or enforce rules where unsigned commits are rejected.

### Conditional Includes

![https://res.cloudinary.com/j4ckofalltrades/image/upload/v1760879228/talks/git-config/slide-39_npegfr.png](https://res.cloudinary.com/j4ckofalltrades/image/upload/v1760879228/talks/git-config/slide-39_npegfr.png)

In cases where you want or need a separate set of configurations that apply to multiple repositories, `gitconfig` ships with an `includeIf` directive where you can conditionally _include_ git settings if certain conditions are met.

An example scenario is that you have personal and work repositories on the same machine and you want to associate commits made to these repositories to different emails.

## Resources & Takeaways

![https://res.cloudinary.com/j4ckofalltrades/image/upload/v1760879228/talks/git-config/slide-40_jcvxed.png](https://res.cloudinary.com/j4ckofalltrades/image/upload/v1760879228/talks/git-config/slide-40_jcvxed.png)

A list of recommended reading and resources:

- [Pro Git Book](https://git-scm.com/book/en/v2)
- [Julia Evans' blog](https://jvns.ca/categories/git/)
- [So You Think You Know Git?](https://speakerdeck.com/schacon/so-you-think-you-know-git)
- [git-sim](https://github.com/initialcommit-com/git-sim)
- [What's in my .gitconfig?](https://jduabe.dev/posts/2025/git-config/)

The best configuration is the one that works for you.