---
title: "Autocomplete for git's plumbing commands"
date: 2025-10-19T21:15:30+08:00
description: "Enabling autocomplete for git's plumbing commands."
draft: false
tags:
  - "til"
  - "git"
categories:
series:
asciinema: false
mastodonPostID:
---

By default, git's completion only shows "porcelain" commands such as `add`, `commit`, etc., and hides the low-level
"plumbing" commands like `read-tree`.

```bash
$ git re<TAB>
rebase         remote         replace        reset          revert
reflog         repack         request-pull   restore
```

This behavior can be modified by setting the `GIT_COMPLETION_SHOW_ALL_COMMANDS=1` environment variable so that plumbing commands are also shown, which is documented in its [completion script](https://github.com/git/git/blob/master/contrib/completion/git-completion.bash#L59-L82).

```
GIT_COMPLETION_SHOW_ALL_COMMANDS

  When set to "1" suggest all commands, including plumbing commands
  which are hidden by default (e.g. "cat-file" on "git ca<TAB>").
```

```bash
$ export GIT_COMPLETION_SHOW_ALL_COMMANDS=1
$ git re<TAB>
read-tree      receive-pack   remote         remote-fd      replace        rerere         restore        rev-list
rebase         reflog         remote-ext     repack         request-pull   reset          revert         rev-parse
```
