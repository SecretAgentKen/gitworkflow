# Git Quick Reference

## Create a work directory (svn co)
	git clone git@url:path/<project>

This will create a `project` directory in your current directory pointing at `master`

## Checkout (and track) a branch (svn sw)

	git checkout -t origin/<branchname>

This will switch to `<branchname>` and also start tracking it so you can push and pull. For example `git checkout -t origin/scratch/foo` would create a local `scratch/foo` branch tracking `origin/scratch/foo`

## See what branch you are on and what files have changed (svn status/info)

	git status

## Add new files or remove files for a commit (svn add/rm)

	git add <file or dir>
	git rm <file or dir>

## See changes you've made (svn diff)

	git diff
	git difftool

## Drop the uncommitted changes made to one particular file (svn checkout -- filename)

	git checkout -- <filename>

## Commit added/removed files (svn ci)(LOCAL ONLY)

	git commit

## Commit everything I've changed that git already knows about (svn ci)(LOCAL ONLY)

	git commit -a

## Push you commits to the remote server (origin) so others can see them (svn ci)

	git push

## Pull other developer changes into your workspace (svn update)

	git pull --rebase
See note about rebase

## Resolve the merge conflicts from a merge or pull
First, see what instructions git tells you. Typically, if you do a `--rebase`, it wants you to do a `git rebase --continue`. If it's a normal merge, it's `git add` and `git commit`.
You have two options: Edit in the file itself or use a merge tool. I HIGHLY recommend installing p4merge which is part of the Perforce Client Tools. Follow the instructions here: https://gist.github.com/1510148.  Then, you can:

	git mergetool
	git commit

Also read this dealing with merging different branches: http://tech.novapost.fr/merging-the-right-way-en.html

And when working on a single branch with other people, please `--rebase` like so: http://stevenharman.net/git-pull-with-automatic-rebase

If you edit the file yourself, it's typical diff format:

	<<<<< HEAD:filename
	some code
	=========
	conflict code
	>>>>> branch:filename

Edit the file so it's correct and save. Then do the `git add/commit` or `rebase` as instructed.

## Create a branch based on the current one (local only)

	git branch <branchname>
	git checkout <branchname>

or as one command

	git checkout -b <branchname>

## Switch back to an existing branch (svn sw)

	git checkout <branchname>

## Merge changes from one branch into the current one
For example, assume we are on `master` and we're merging `bug-fix` into it.

	git merge bug-fix
	git commit

OR merge while squashing the history so it only shows as a single commit. This is DANGEROUS! Do not use squash for a branch you intend to keep. ie. delete `bug-fix` after you've merged it and it shouldn't be something that's been pushed previously.

	git merge --squash bug-fix
	git commit

# HELP!!

## I want to make a branch I can share with other people

	git branch dev/somename
	git push origin dev/somename
	git branch --set-upstream dev/somename origin/dev/somename

The other user would now run:

	git checkout -t origin/dev/somename

This creates a local branch of `dev/somename` for them.

The `push` and `branch --set-upstream` can be combined into a single command as well:

	git push -u origin dev/somename

## I've been working in the wrong branch but I haven't committed yet

	git stash
	git checkout <branch>
	git stash pop

## I've been working the wrong branch but I've committed some stuff

	git stash
	git checkout <branch>
	git merge <otherbranch>
	git commit
	git stash pop

## I made a branch I don't want anymore

	git branch -d <branchname>

## I can't push or pull my branch from origin, it keeps telling me it's not tracking
IFF this is a branch that should be shared, ie. master, a dev branch, etc.

	git branch --set-upstream <branchname> origin/<branchname>

## I don't know if my branch is tracking or not

	git remote show origin

This will show all local and remote branches and if they're tracking locally

## When I push, it keeps complaining about master, release/X, or other branches i'm not pushing or care about

	git config --global push.default simple

Push can push all changes or just the ones on the branch you're currently on. This updates your .gitconfig to do just the one you're on.

## git log is useless, I can't see a real history
Add the following to your .gitconfig in your home directory under `[alias]`:

	lg = log --graph --pretty=format:'%Cred%h%Creset -%C(yellow)%d%Creset %s%Cgreen(%cr) %C(cyan)<%an>%Creset' --abbrev-commit --date=relative

Now, if you do a `git lg`, you'll get a tree view, pretty log history

Other options include:
	ls = log --pretty=format:"%C(yellow)%h%Cred%d\\ %Creset%s%Cblue\\ [%cn]" --decorate
	ll = log --pretty=format:"%C(yellow)%h%Cred%d\\ %Creset%s%Cblue\\ [%cn]" --decorate --numstat

## I've made a bunch of uncommitted changes that I don't want. Get back to my last commit
DANGEROUS! Will change the contents of your working directory!

	git reset --hard

## I've screwed up my develop branch with commits and just want to go back to what's on the origin. I don't care if I lose what I've done
DANGEROUS! Can lose commits!

	git rebase origin

## I keep forgetting which branch I'm on
Create a `.profile` or `.bash_profile` file in your home directory with the following contents:

	c_reset='\[\e[0m\]'
	c_git_clean='\[\e[1;36m\]'
	c_git_dirty='\[\e[1;35m\]'

	git_prompt()
	{
		if ! git rev-parse --git-dir > /dev/null 2>&1; then
			return 0
		fi

		git_branch=$(git branch 2>/dev/null | sed -n '/^\*/s/^\* //p')

		if git diff --quiet 2>/dev/null >&2; then
			git_color="$c_git_clean"
		else
			git_color="$c_git_dirty"
		fi

		echo " [$git_color$git_branch${c_reset}]"
	}

	PROMPT_COMMAND='PS1="\h:\W \u$(git_prompt)\% "'

This will change:

	myhost:myfolder myusername$

To (with clean/dirty colors)

	myhost:myfolder myusername [branchname]$
