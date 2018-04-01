---
layout: post
title: "Source Code Management Tool - Git"
date:   2018-04-01 21:51:30 +0530
categories: 
---
Quick Notes of using git and github.

## List Config
To list configuration details,
```bash
	git config --list
```

## Set Config ( global ) 
To set configuration parameters globally ie. to use for all repositories.
```bash
	git config --global user.name "<username>"
	git config --global user.email "<email address>"
```
To set configuration parameters for a specific repository,
```bash
	git config user.name "<username>"
	git config user.email "<email address>"
```

## Get local copy
To get local copy of a repository from distributed server, 
```bash
	git clone <github-url>
```

## Publish changes

There three phases in git.

### Staging area ( buffer )
`git add` command is used to add the modified or new files/folders to the staging area. Note that `git add` will not give any output
```bash
	git add <filename>
	git add .    # add current directory changes
	git add -A   # add all the directories. 
	git status
```

### Local Repo:
 `git commit` is used to add to local repository. While commiting the files, SHA-1 key will be generated for every commit.
```bash
	git commit  			# this will open an editor to give commit message
	git commit -m "reason"	# -m used to give commit message in CLI itself
	git status		# will give info about files are in statging area,local repo, remote repo
				# Untracked files - which are yet to add to staging
				# Changes to be committed - which are in staging area
	git log
```

### Remote Repo
`git push` is used to push the changes to the distributed repository.
```bash
	git push 
	git push -u origin master
```

To get remote repo details,
```bash
	git remote -v
```

To rename remote repo name,
```bash
	git remote rename origin destination
	git remote -v
```

## Some git repo stuff
To initialize git repo locally,
```bash
	git init 
```
To add remote repo for local repo,
```bash
	git remote add origin <url>
```
To push the current branch and set the remote as upstream,
```bash
	git push --set-upstream origin master
```
To pull the recent content from remote repo,
```bash
	git pull origin master
```

## Advance git commands
To remove Untracked files,
```
	git clean -f 	# clean the files. -f option for forceful
	git clean -fd	# clean file and directory. -f option for forceful
```
To remove files which are in Staging area,
```
	git reset --hard 	# changes made after the commit are discarded. It resets index and working tree.
```
To get previous revision,
```
	git checkout <filename>	
```

To get back past commit changes,
```
	git revert <commit-id-taken-from-log>
```

To push the changes forcefully,
```
	git push -f 
```

To commit recent changes in a file which are already committed
```
	git commit --amend 
```

If we don't want file ( which had some modifications) and keep it for sometime, we can make use of stash command. The stash content will be stored in stACKing area
```
	git stash
	git staus
	git stash -p 	# dsiplay the chunks of changes which are stashed
	git stash list
	git stash apply
```

Other useful reset commands,
```
	git reset --hard 	# changes made after the commit are discarded.
	git reset --soft 	# changes made after the commit are moved to "staged for commit" stage.
				# you can run git commit command to add files back to repo.
	git reset 		# changes made after commit are moved to "not yet staged for commit" stage
```

## To display log
`git` supports both text based and graph based log histroy,

``` 
	git log 					# shows the log information
	git log --graph --oneline --decorate --all 	# display bit of graph
```

To display the list of commits to be push to remote repo,
```
	git checrry -v
```

## Branching Mechanism

```
	git branch		# display the branch in use
	git branch -a		# display list of branches 
	git merge  		# merge two branches
	git checkout develop	# switch the branch to develop
	git checkout -b develop master	# create a branch'develop' from 'master' branch
```

When you are trying to merge from develop to master branch as below
develop -> master
source -> destination

```
	git checkout master
	git merge develop
```

To create an alias,
```
	git config --global alias.XZ
```

To generate some reports of repo changes, history and logs
```
	git for-each-ref 
```

To see the contents from other branch
```
	git show some-branch:file.js
```

To list files with status,
```
	git status -s
```

To select part of changes
```
	git add -p
```

To pick some changes from a commit and apply it here
```
	git cherry-pick <commitSHA>
```

Pull and merge can be done together using rebase
```
	git rebase
```

```
git log options,
	- author="..."
	- name-only
	- oneline
	- graph
	- reverse
	- after
	- before
```

