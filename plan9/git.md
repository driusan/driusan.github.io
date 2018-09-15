# Using dgit as a git client under Plan 9

In order to use dgit as a git client under Plan 9, you'll need go 1.10
or higher.  You can download a go binary at http://9legacy.org/download.html
if you don't already have it installed.

Since `go get` requires git and dgit is written in Go, you need a way
to bootstrap the download/installation of dgit.  You can use the
9legacy rc script which fakes the git command line just enough for Go
(if you're not doing any development) at http://9legacy.org/9legacy/tools/git
for this.

```
hget http://9legacy.org/9legacy/tools/git > $home/bin/rc/git
chmod +x $home/bin/rc/git
```

You should then be able to `go get github.com/driusan/dgit` which will
install dgit into `$GOPATH/bin/dgit` (Note: `$GOPATH` defaults to
`$home/go` if not set.) You may want to `bind -qa $GOPATH/bin /bin` to
ensure it (and other Go programs) are in your path.

At this point you can use the `dgit` command (which should match the
git command line, if not file a bug report describing what
subcommand/option you tried to use) for basic git
init/add/merge/commit usage.  It's not mature enough to replace `git`
as a daily driver on other platforms and may eat your dog if you try,
but it should be mature enough to do basic development under Plan 9.

## Go

If you want `go get` to use dgit, you'll need to make sure it's
available under the name "git", not "dgit".  You can bind it using
`bind $GOPATH/dgit /bin/git` (adjust `$GOPATH` as necessary) or `bind
/bin/dgit /bin/git` if your `$GOPATH` in your path.  You'll probably
want to make sure you bind it in `$home/lib/profile`.

## Significant Limitations

While dgit should mostly be useable for git, the following limitations
apply that are worth noting.

In general limitations, only the https transport is currently
supported.  Merge conflicts are poorly tested and likely to be buggy.
Some subcommands or options may be missing.

For the purposes of Go the `git submodule` subcommand (used by
vendoring) is missing.  `git show-ref` (used by Go modules to find
tagged versions available) is also currently missing.

For the purposes of GitHub collabourations, the git (and hence, dgit)
command line does not have a mechanism for sending pull requests.
dgit will likely add one in the future as an extension to git.
Unfortunately currently the easiest way is to just use your phone or
tablet or another computer to click the "Send Pull Request" button in
the GitHub interface.  You may also want to look into
[https://github.com/sirnewton01/ghfs](ghfs) for a 9p interface to the
GitHub API (but ghfs also doesn't currently support sending PRs.)



