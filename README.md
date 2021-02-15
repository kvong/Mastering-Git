## Basic of git

What is a commit?
- A commit is a record of changes made to the previous version of the file(s).
- We can think of commits as a snapshot.
- Commits are lightweight and it doesn't simply copy the entire repository.

What is a branch?
- Branches are simply pointers to a specific commit.
- Create a branch on the current HEAD by running `git branch feature`. After that, a branch will be created on the current commit the HEAD is on.
- Running `git branch` will not change anything. In fact, the branch created is simply what is already there.

What if you branch when there are uncommitted changes?
- The branch will have the uncommitted changes. Creation of a branch can be viewed as a simple declaration.
	
Merging in Git:
- When we want to combine the work of two branches ( the branch we are on and another branch ) we run `git merge <target_branch>`
- The merge will result in a new commit and the current branch will be pointing to it.
- Note that the merge will not update the target branch with the merged main branch.
	- To update the target branch do `git checkout <t-branch> && git merge main`.

What happens when there is a merge conflict?
1. Fast-forward merge: No conflicting changes. Current branch simply update to match the target branch. 
2. Auto-merge: Changes were made on both branch but they were non-conflicting changes.
3. Three-way merge: Conflicting changes that git could not resolve with auto-merge. This will cancel the merge, ask the user to view the conflicted files by running `git status`. Once all conflicts are resolved `git commit -a` will create a new commit with the resolved changes, which the current branch will point to.

Rebase in Git:
- Syntax: `git rebase <base> <target>`
	- The commit of the `target` appended to the `base`.
- Another way of combining work between branches. 
- Rebasing essentially takes a set of commits, "copies" them, and put them somewhere.
- Advantage of rebasing is that it can used to create a linear sequence of commits. Cleaning up the `git log`.
- We can copy our branch to a target branch by running `git rebase <target-branch>`.
- This will create a copy of our current commit on our current branch and place it on the target branch to be updated there.
- Merge conflicts are resolve the same way as merge.
- After rebasing, the current branch will be the most updated.
- To update the target branch, `git checkout t-branch && git rebase`.

Detached HEAD State:
- The HEAD is usually pointed at the more recent comment of the checked out branch. We have a detached HEAD when the HEAD is not pointed at the most recent commit of the branch but an older commit.

What happen if we didn't commit before rebase or merge?
- When combining work, git will look at the current state of the files, not the commit. Thus it does not matter if we had committed or not. Git will still warn us about it and cancel the merge/rebase.

Relative References:
- Instead of typing out the commit ID we can use the relative reference to checkout.
	- <b>^</b> Moving backward one commit ( to the parent ).
		- Usage: `git checkout HEAD^`
            - Checkout the parent of HEAD.
        - Usage: `git checkout HEAD^2`
            - Checkout the second parent of HEAD.
            - A commit will have multiple parents if a merge has occurred.
	- <b>`~<number>`</b> Moving backward `<number>` commits
		- Usage: `git checkout HEAD~2`
		- Usage: `git checkout HEAD~`
            - Similar to `git checkout HEAD~1`

Branch Forcing
- We can force a branch to a commit by running `git branch -f main HEAD~3`

Reversing Changes in Git
- `git reset` will revert a commit back. How far back depends on the argument given. Example: `git reset HEAD~1`. It is as if the commit had never been made in the first place.
- Reset is great for local branches, but this way of resetting does not work on remote branches. 
- `For resetting remote branches shared by others; git revert` will create a new commit undoing the current commit.  From here you can push out the resetted code.

Moving commits around with `git cherry-pick`
- We can use `git cherry-pick to select commits from other branches and put in on the current branch. 
- Syntax: `git cherry-pick Cx Cy Cz ...`
- Commits will be placed onto the current branch.
- This will handle 3-way merge conflict as well.
- No conflict will simply move the head to the newly added commit.

Moving Commits around with `git rebase -i`
- An easier way to move around commits. Interactive rebase does not require the user to remember the commit they want to move.
- For example: `git rebase -i HEAD~4` will take the 4 commits back prior to the commit HEAD is pointed at, then a text editor will open to allow the user to structure the 4 commits accordingly.
- This does not rearrange the current instance of the commits but simply copy over and then rearrange in a different instance.
- Usage: `git rebase -i <t-branch> <r-branch>` -- will take `r-branch` from the split point between `t-branch` and `r-branch`, and move it to `<t-branch>`. `r-branch` is optional, if not provided, this will be HEAD.


A case for <b>interactive rebase</b> and <b>cherry-picking</b>:
- Suppose you are on a branch where several commits that are simply debugging print statements, and the latest commit is a the refactorization of the code. Instead of doing a fast-forward merge, where the resulting `git log` is filled with the unnecessary debugging commits. We can use <b>interactive rebase</b> or <b>cherry-picking</b> to take the refactored commit and have the MAIN branch merge to that single commit. This makes for a nicer looking `git log`.

Amending an older commit:
- Rebase approach: Suppose we have an older commit we wish to change. For example, we are on branch bugFix, and we want to modify commit C4 but our HEAD is on commit C5. Before amending C4, we must perform an interactive rebase of C4 and C5. We will make it that C4 is a head of C5. Then, change the content of C4 and commit with `git commit -amend`. Once C4 is updated, perform the interactive rebase again to reorder the commits in the correct order ( C4 then C5 ). Through out this process, if there is a merge conflict, it will need to be handled then before the rebase is completed. Finally with the new C4 then C5, we can perform a fast-forward merge. 
- Cherry-Pick approach: Suppose we have an older commit we wish to change. For example, we are on branch bugFix, and we want to modify C5 but our HEAD is on commit C5. While we can use rebase to get the job done. Cherry-pick offer a much easier solution. Go to the node before C4, `git cherry-pick C4` to get a copy of C4, amend using `git commit --amend`. Now with the amended commit, we can run `git cherry-pick C5` to be placed after the amended C4. Any conflicts must be resolve while cherry-picking. If no conflict, it will be a fast-forward merge.

Git tags can be used to mark certain commits. Set a tag using `git tag v1 Cx`. Leaving out the commit will put the tag on the HEAD. You cannot checkout a tag and perform work as tags only represents an anchor on a commit tree.

## Git Remotes

Git Clone - download a remote git repository to the local machine.
- The clone repository contains branches that are a reflection of the remote repository. These branches are called remote branches, and they changes pushed from these the local branch will update the remote repository.
- When we clone a repository, for all the branches that exists in that repository at that time, there will be a mirrored branch. For example, there will be a master branch along with a origin/master branch.
- Example workflow: You want to work on a project so you'll run `git clone`. After you have added some work, run `git add` and then `git commit`. To update the change on to the repository run `git push`
	- Note: While origin/main branch is seen when we run `git log` we cannot actually work with it. Any branch with origin/ means that it's where the remote repository is. While we can still checkout any of the origin/* branch, but commitng it will put us in a detached head state.

Git Fetch - Update the remote branch ( often named the origin/ branch ) to reflect the current remote repository. 
- Note: This does not change your local branch.
- Note: This will update all remote branches.

Git Pull - download and update the local repository to match the remote repository.
- Running `git pull` is a combination of `git fetch && git merge`.
- Note: The merge is occuring between the local origin/ and the local branch
- Example: Suppose on a local repository, you have your main branch on commit CX, and your main remote branch is on commit CY. Running `git pull` will, first download CY to the origin branch on the local repository, then it will run a `git merge` between the `main` and `origin/main`. 

Git Push - Update the remote repository along with the `origin/`.


Diverged Work - How do I update my work when the remote repository has been updated?
- In cases where the work our commit is working off of is updated between we finish the work. We can run `git fetch`, then `git rebase` our commit on to the `origin/` branch, then run `git push`.
	- Short hand for this is to do `git pull --rebase; git push`.
- Using `git merge`: By running `git fetch; git merge origin/main; git push` we will perform a merge similar to the example above. However the difference is that the remote will have the merge as well. Unlike rebase, where the remote does not change to reflect our local repository.
	- Short hand for this is to do `git pull; git push`.
	
Master is locked and is asking for a pull request.
- Solution: 
	1. Reset the main branch to be in sync as the remote by running `git branch -f main origin/main`.
	2. Treat your changes as a new feature by running `git branch -f feature`.
	3. Run `git push` on the `feature` branch to update the remote with a new branch that is ahead of `main`.
	
Rebase vs Merge:
- Rebase will reorder the commit tree but keep the tree straight.
- Merge will preserve the order of the commit history but will lead to a messy tree.

Remote Tracking - Forming a connection to the origin/<branch>.
- When we do a git clone. Git will automatically set up remote tracking from `main` to `origin/main`.
- We can do this too using `git checkout`
	- Running `git checkout -b newMain origin/main` will create a branch named `newMain` and sets it to track the `origin/main` branch. Thus, the next time we run a `git pull` the remote will download, and update to `newMain`.
	- Another way is to run `git branch -u origin/main mainBranch`.
		- If mainBranch is currently checkout run `git branch -u origin/main`.
	
Git push parameters.
- Syntax: `git push <remote> <branch>`
	- Update the `<remote>` with the name `<branch>` to match the local `<branch>`.
- We can also push from another branch that is not being tracked by the remote.
	- For example, what if we want to push `bugFix` on to the `main` remote?
	- Running `git push origin <source>:<destination>` will do the trick!
		- This is telling git to update `origin/<destination>` with the local `<source>` branch.
		- Note: `<source>` does not have to be a branch it could be a reference as well ( HEAD~1 ).
		- Note: `<destination>` does not have to be an existing branch. If it doesn't exist, git will simply create it, on both the remote and the local.
		- Note: `<source>:<destination>` is called 'colon refspec'.
Git fetch parameters.
- Syntax: `git fetch <remote> <branch>`
	- Download the `<remote>` branch named `<cranch>` to the local `<branch>`.
- Git fetch behaves in the opposite way that `git pull` does.
	- For example: `git fetch origin foo~1:bar` will look for `foo~1` on the `origin` remote and download it to `bar`.
	- We don't usually do this because, incase we have some work on `bar`, fetching on to it will overwrite it. It would be better to run `git fetch origin foo~1` leaving the destination blank so that the `origin/foo` branch get updated instead, then running `git rebase origin/foo foo` to handle any conflicts.
	- Note: Similarly to when we push, when we fetch, if the branch to download to does not exist, it will be created.
- Note: Running `git fetch` with no argument will download all the commits form the remote on to the remote branches.
		
	
Colon refspec: Above we can see that providing a `<source>:<destination>` tells git to do something with the branch. A neat trick arises when we provide nothing for `<source>` parameter.
- Running `git fetch origin :<branch>` will create a new branch named `<branch>`. This makes sense because we are essentially telling git to pull nothing on to the branch `<branch>`. With nothing to pull a branch is simply crated with nothing added to it.
- Running `git push origin :<destination>` will delete the remote branch named `<destination>`. Meaning that we are uploading nothing. Uploading nothing to a branch will delete it as it is now "nothing".
	- Note: This deletion deletes both the remote and the local `<destination>` branch.
	
Git pull parameters
- Remember that `git pull` is the same as `git fetch; git merge`. Breaking down these commands:
	- `git pull origin foo` = `git fetch origin foo; git merge origin/foo`.
		- Fetching the origin will update the `origin/foo` branch, then we can merge with the actual `foo` branch to apply our changes.
	- `git pull origin bar~1:bugFix` = `git fetch origin bar~1:bugFix; git merge bugFix`.
		- Fetching the remote `origin` commit location `bar~1` and downloading it to bugFix, then merging the current checkout branch with the newly downloaded `bugFix`.
- Note: The same rule apply, if the destination to be downloaded to does not exist, it will be created.
