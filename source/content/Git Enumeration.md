---
tags: git
---
Dump the git repo from website:
(tool: https://github.com/arthaud/git-dumper)
`git-dumper http://website.com/.git ~/website`

First, change the current directory to the git repo

Display state of the Git working directory:
`git status`

Show commit history:
`git log`

Show differences between commits:
`git show 612ff5783cc5dbd1e0e008523dba83374a84aaf1`
- `612ff5783cc5dbd1e0e008523dba83374a84aaf1`: replace with commit hash of the commit you are interested in (obtained from `git log` command)

Check config file for username:
`cat .git/config`