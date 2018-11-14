## Repositories

### Tôi muốn bắt đầu với một repository trên local

Cách khởi tạo thư mục hiện có dưới dạng một Git repository:

```sh
(my-folder) $ git init
```

### Bạn muốn sao chép một remote repository

Để sao chép một remote repository, cần copy url của repository, và chạy lệnh:

```sh
$ git clone [url]
```

Điều này sẽ lưu nó vào một thư mục có tên giống như remote repository's. Hãy chắc chắn rằng bạn có kết nối đến remote server mà bạn đang sao chép (điều này có nghĩa là đảm bảo bạn được kết nối với internet).
Để sao chép nó vào một thư mục có tên khác với tên repository mặc định:

```sh
$ git clone [url] name-of-new-folder
```

## Chỉnh sửa Commits

<a name="diff-last"></a>
### Tôi đã commit gì ?

Giả sử bạn chỉ thực hiện các thay đổi một cách mù quáng với `git commit -a` và bạn không chắc chắn nội dung thực sự commit mà bạn vừa tạo ra là gì. Bạn có thể hiển thị commit mới nhất trên con trỏ HEAD hiện tại của bạn với:

```sh
(master)$ git show
```

Or

```sh
$ git log -n1 -p
```

Nếu bạn muốn xem một tập tin tại một commit cụ thể, bạn cũng có thể làm điều này (trong đó `<commitid>` là commit mà bạn quan tâm):

```sh
$ git show <commitid>:filename
```

### Tôi đã viết sai vài thứ trong message của commit

Nếu bạn đã viết sai và commit chưa được đẩy lên, bạn có thể làm như sau để thay đổi message commit mà không thay đổi các thay đổi trong commit:

```sh
$ git commit --amend --only
```
Thao tác này sẽ mở trình soạn thảo văn bản mặc định của bạn, nơi bạn có thể chỉnh sửa message. Mặt khác, bạn có thể làm tất cả điều này trong một lệnh

```sh
$ git commit --amend --only -m 'xxxxxxx'
```

Nếu bạn đã đẩy message lên, bạn có thể chỉnh sửa commit và force push, nhưng điều đó không được khuyến khích.

<a name="commit-wrong-author"></a>
### Tôi đã commit với tên và email cấu hình sai

Nếu đó là một commit, hãy sửa đổi nó

```sh
$ git commit --amend --no-edit --author "New Authorname <authoremail@mydomain.com>"
```

Một cách khác là cấu hình chính xác tác giả của bạn trong `git config --global author. (name | email)` và sau đó sử dụng

```sh
$ git commit --amend --reset-author --no-edit
```

Nếu bạn cần thay đổi tất cả lịch sử, hãy xem trang với `git filter-branch`.

### Tôi muốn xóa một file khỏi commit trước đó

Để xóa các thay đổi đối với một file khỏi commit trước đó, hãy làm như sau:

```sh
$ git checkout HEAD^ myfile
$ git add myfile
$ git commit --amend --no-edit
```

Trong trường hợp file mới được thêm vào commit và bạn muốn xóa nó (từ Git), hãy thực hiện:

```sh
$ git rm --cached myfile
$ git commit --amend --no-edit
```

Điều này đăc biệt hữu ích khi bạn có một bản patch mở và bạn đã commit một file không cần thiết và force push để cập nhật bản patch trên remote. Tuỳ chọn `--no-edit` được sử dụng để giữ message cho commit hiện tại.

<a name="delete-pushed-commit"></a>
### I want to delete or remove my last commit

If you need to delete pushed commits, you can use the following. However, it will irreversibly change your history, and mess up the history of anyone else who had already pulled from the repository. In short, if you're not sure, you should never do this, ever.

```sh
$ git reset HEAD^ --hard
$ git push --force-with-lease [remote] [branch]
```

If you haven't pushed, to reset Git to the state it was in before you made your last commit (while keeping your staged changes):

```
(my-branch*)$ git reset --soft HEAD@{1}

```

This only works if you haven't pushed. If you have pushed, the only truly safe thing to do is `git revert SHAofBadCommit`. That will create a new commit that undoes all the previous commit's changes. Or, if the branch you pushed to is rebase-safe (ie. other devs aren't expected to pull from it), you can just use `git push --force-with-lease`. For more, see [the above section](#deleteremove-last-pushed-commit).

<a name="delete-any-commit"></a>
### Delete/remove arbitrary commit

The same warning applies as above. Never do this if possible.

```sh
$ git rebase --onto SHA1_OF_BAD_COMMIT^ SHA1_OF_BAD_COMMIT
$ git push --force-with-lease [remote] [branch]
```

Or do an [interactive rebase](#interactive-rebase) and remove the line(s) corresponding to commit(s) you want to see removed.

<a name="#force-push"></a>
### I tried to push my amended commit to a remote, but I got an error message

```sh
To https://github.com/yourusername/repo.git
! [rejected]        mybranch -> mybranch (non-fast-forward)
error: failed to push some refs to 'https://github.com/tanay1337/webmaker.org.git'
hint: Updates were rejected because the tip of your current branch is behind
hint: its remote counterpart. Integrate the remote changes (e.g.
hint: 'git pull ...') before pushing again.
hint: See the 'Note about fast-forwards' in 'git push --help' for details.
```

Note that, as with rebasing (see below), amending **replaces the old commit with a new one**, so you must force push (`--force-with-lease`) your changes if you have already pushed the pre-amended commit to your remote. Be careful when you do this &ndash; *always* make sure you specify a branch!

```sh
(my-branch)$ git push origin mybranch --force-with-lease
```

In general, **avoid force pushing**. It is best to create and push a new commit rather than force-pushing the amended commit as it will cause conflicts in the source history for any other developer who has interacted with the branch in question or any child branches. `--force-with-lease` will still fail, if someone else was also working on the same branch as you, and your push would overwrite those changes.

If you are *absolutely* sure that nobody is working on the same branch or you want to update the tip of the branch *unconditionally*, you can use `--force` (`-f`), but this should be avoided in general.

<a href="undo-git-reset-hard"></a>
### I accidentally did a hard reset, and I want my changes back

If you accidentally do `git reset --hard`, you can normally still get your commit back, as git keeps a log of everything for a few days.

Note: This is only valid if your work is backed up, i.e., either committed or stashed. `git reset --hard` _will remove_ uncommitted modifications, so use it with caution. (A safer option is `git reset --keep`.)

```sh
(master)$ git reflog
```

You'll see a list of your past commits, and a commit for the reset. Choose the SHA of the commit you want to return to, and reset again:

```sh
(master)$ git reset --hard SHA1234
```

And you should be good to go.

<a href="undo-a-commit-merge"></a>
### I accidentally committed and pushed a merge

If you accidentally merged a feature branch to the main development branch before it was ready to be merged, you can still undo the merge. But there's a catch: A merge commit has more than one parent (usually two).

The command to use
```sh
(feature-branch)$ git revert -m 1 <commit>
```
where the -m 1 option says to select parent number 1 (the branch into which the merge was made) as the parent to revert to.

Note: the parent number is not a commit identifier. Rather, a merge commit has a line `Merge: 8e2ce2d 86ac2e7`. The parent number is the 1-based index of the desired parent on this line, the first identifier is number 1, the second is number 2, and so on.

<a href="undo-sensitive-commit-push"></a>
### I accidentally committed and pushed files containing sensitive data

If you accidentally pushed files containing sensitive data (passwords, keys, etc.), you can amend the previous commit. Keep in mind that once you have pushed a commit, you should consider any data it contains to be compromised. These steps can remove the sensitive data from your public repo or your local copy, but you **cannot** remove the sensitive data from other people's pulled copies. If you committed a password, **change it immediately**. If you committed a key, **re-generate it immediately**. Amending the pushed commit is not enough, since anyone could have pulled the original commit containing your sensitive data in the meantime. 

If you edit the file and remove the sensitive data, then run
```sh
(feature-branch)$ git add edited_file
(feature-branch)$ git commit --amend --no-edit
(feature-branch)$ git push --force-with-lease origin [branch]
```

If you want to remove an entire file (but keep it locally), then run
```sh
(feature-branch)$ git rm --cached sensitive_file
echo sensitive_file >> .gitignore
(feature-branch)$ git add .gitignore
(feature-branch)$ git commit --amend --no-edit
(feature-branch)$ git push --force-with-lease origin [branch]
```
Alternatively store your sensitive data in local environment variables.

If you want to completely remove an entire file (and not keep it locally), then run
```sh
(feature-branch)$ git rm sensitive_file
(feature-branch)$ git commit --amend --no-edit
(feature-branch)$ git push --force-with-lease origin [branch]
```

If you have made other commits in the meantime (i.e. the sensitive data is in a commit before the previous commit), you will have to rebase. 

## Staging

<a href="#i-need-to-add-staged-changes-to-the-previous-commit"></a>
### I need to add staged changes to the previous commit

```sh
(my-branch*)$ git commit --amend
```

If you already know you don't want to change the commit message, you can tell git to reuse the commit message:

```sh
(my-branch*)$ git commit --amend -C HEAD
```


<a name="commit-partial-new-file"></a>
### I want to stage part of a new file, but not the whole file

Normally, if you want to stage part of a file, you run this:

```sh
$ git add --patch filename.x
```

`-p` will work for short. This will open interactive mode. You would be able to use the `s` option to split the commit - however, if the file is new, you will not have this option. To add a new file, do this:

```sh
$ git add -N filename.x
```

Then, you will need to use the `e` option to manually choose which lines to add. Running `git diff --cached` or
`git diff --staged` will show you which lines you have staged compared to which are still saved locally.

<a href="stage-in-two-commits"></a>
### I want to add changes in one file to two different commits

`git add` will add the entire file to a commit. `git add -p` will allow to interactively select which changes you want to add.

<a href="unstaging-edits-and-staging-the-unstaged"></a>
### I want to stage my unstaged edits, and unstage my staged edits

This is tricky. The best I figure is that you should stash your unstaged edits. Then, reset. After that, pop your stashed edits back, and add them.

```sh
$ git stash -k
$ git reset --hard
$ git stash pop
$ git add -A
```

## Unstaged Edits

<a href="move-unstaged-edits-to-new-branch"></a>
### I want to move my unstaged edits to a new branch

```sh
$ git checkout -b my-branch
```

<a href="move-unstaged-edits-to-old-branch"></a>
### I want to move my unstaged edits to a different, existing branch

```sh
$ git stash
$ git checkout my-branch
$ git stash pop
```

<a href="i-want-to-discard-my-local-uncommitted-changes"></a>
### I want to discard my local uncommitted changes (staged and unstaged)

If you want to discard all your local staged and unstaged changes, you can do this:

```sh
(my-branch)$ git reset --hard
# or
(master)$ git checkout -f
```

This will unstage all files you might have staged with `git add`:

```sh
$ git reset
```

This will revert all local uncommitted changes (should be executed in repo root):

```sh
$ git checkout .
```

You can also revert uncommitted changes to a particular file or directory:

```sh
$ git checkout [some_dir|file.txt]
```

Yet another way to revert all uncommitted changes (longer to type, but works from any subdirectory):

```sh
$ git reset --hard HEAD
```

This will remove all local untracked files, so only files tracked by Git remain:

```sh
$ git clean -fd
```

`-x` will also remove all ignored files.

### I want to discard specific unstaged changes

When you want to get rid of some, but not all changes in your working copy.

Checkout undesired changes, keep good changes.

```sh
$ git checkout -p
# Answer y to all of the snippets you want to drop
```

Another strategy involves using `stash`. Stash all the good changes, reset working copy, and reapply good changes.

```sh
$ git stash -p
# Select all of the snippets you want to save
$ git reset --hard
$ git stash pop
```

Alternatively, stash your undesired changes, and then drop stash.

```sh
$ git stash -p
# Select all of the snippets you don't want to save
$ git stash drop
```

### I want to discard specific unstaged files

When you want to get rid of one specific file in your working copy.

```sh
$ git checkout myFile
```

Alternatively, to discard multiple files in your working copy, list them all.

```sh
$ git checkout myFirstFile mySecondFile
```

### I want to discard only my unstaged local changes

When you want to get rid of all of your unstaged local uncommitted changes

```sh
$ git checkout .
```
<a href="i-want-to-discard-all-my-untracked-files"></a>
### I want to discard all of my untracked files

When you want to get rid of all of your untracked files

```sh
$ git clean -f
```

<a href="I-want-to-unstage-specific-staged-file"></a>
### I want to unstage a specific staged file

Sometimes we have one or more files that accidentally ended up being staged, and these files have not been committed before. To unstage them:

```sh
$ git reset -- <filename>
```

This results in unstaging the file and make it look like it's untracked.

## Branches

### I want to list all branches

List local branches

```sh
$ git branch
```

List remote branches

```sh
$ git branch -r
```

List all branches (both local and remote)

```sh
$ git branch -a
```

<a name="create-branch-from-commit"></a>
### Create a branch from a commit
```sh
$ git checkout -b <branch> <SHA1_OF_COMMIT>
```

<a name="pull-wrong-branch"></a>
### I pulled from/into the wrong branch

This is another chance to use `git reflog` to see where your HEAD pointed before the bad pull.

```sh
(master)$ git reflog
ab7555f HEAD@{0}: pull origin wrong-branch: Fast-forward
c5bc55a HEAD@{1}: checkout: checkout message goes here
```

Simply reset your branch back to the desired commit:

```sh
$ git reset --hard c5bc55a
```

Done.

<a href="discard-local-commits"></a>
### I want to discard local commits so my branch is the same as one on the server

Confirm that you haven't pushed your changes to the server.

`git status` should show how many commits you are ahead of origin:

```sh
(my-branch)$ git status
# On branch my-branch
# Your branch is ahead of 'origin/my-branch' by 2 commits.
#   (use "git push" to publish your local commits)
#
```

One way of resetting to match origin (to have the same as what is on the remote) is to do this:

```sh
(master)$ git reset --hard origin/my-branch
```

<a name="commit-wrong-branch"></a>
### I committed to master instead of a new branch

Create the new branch while remaining on master:

```sh
(master)$ git branch my-branch
```

Reset the branch master to the previous commit:

```sh
(master)$ git reset --hard HEAD^
```

`HEAD^` is short for `HEAD^1`. This stands for the first parent of `HEAD`, similarly `HEAD^2` stands for the second parent of the commit (merges can have 2 parents).

Note that `HEAD^2` is **not** the same as `HEAD~2` (see [this link](http://www.paulboxley.com/blog/2011/06/git-caret-and-tilde) for more information).

Alternatively, if you don't want to use `HEAD^`, find out what the commit hash you want to set your master branch to (`git log` should do the trick). Then reset to that hash. `git push` will make sure that this change is reflected on your remote.

For example, if the hash of the commit that your master branch is supposed to be at is `a13b85e`:

```sh
(master)$ git reset --hard a13b85e
HEAD is now at a13b85e
```

Checkout the new branch to continue working:

```sh
(master)$ git checkout my-branch
```

<a name="keep-whole-file"></a>
### I want to keep the whole file from another ref-ish

Say you have a working spike (see note), with hundreds of changes. Everything is working. Now, you commit into another branch to save that work:

```sh
(solution)$ git add -A && git commit -m "Adding all changes from this spike into one big commit."
```

When you want to put it into a branch (maybe feature, maybe `develop`), you're interested in keeping whole files. You want to split your big commit into smaller ones.

Say you have:

  * branch `solution`, with the solution to your spike. One ahead of `develop`.
  * branch `develop`, where you want to add your changes.

You can solve it bringing the contents to your branch:

```sh
(develop)$ git checkout solution -- file1.txt
```

This will get the contents of that file in branch `solution` to your branch `develop`:

```sh
# On branch develop
# Your branch is up-to-date with 'origin/develop'.
# Changes to be committed:
#  (use "git reset HEAD <file>..." to unstage)
#
#        modified:   file1.txt
```

Then, commit as usual.

Note: Spike solutions are made to analyze or solve the problem. These solutions are used for estimation and discarded once everyone gets clear visualization of the problem. ~ [Wikipedia](https://en.wikipedia.org/wiki/Extreme_programming_practices).

<a name="cherry-pick"></a>
### I made several commits on a single branch that should be on different branches

Say you are on your master branch. Running `git log`, you see you have made two commits:

```sh
(master)$ git log

commit e3851e817c451cc36f2e6f3049db528415e3c114
Author: Alex Lee <alexlee@example.com>
Date:   Tue Jul 22 15:39:27 2014 -0400

    Bug #21 - Added CSRF protection

commit 5ea51731d150f7ddc4a365437931cd8be3bf3131
Author: Alex Lee <alexlee@example.com>
Date:   Tue Jul 22 15:39:12 2014 -0400

    Bug #14 - Fixed spacing on title

commit a13b85e984171c6e2a1729bb061994525f626d14
Author: Aki Rose <akirose@example.com>
Date:   Tue Jul 21 01:12:48 2014 -0400

    First commit
```

Let's take note of our commit hashes for each bug (`e3851e8` for #21, `5ea5173` for #14).

First, let's reset our master branch to the correct commit (`a13b85e`):

```sh
(master)$ git reset --hard a13b85e
HEAD is now at a13b85e
```

Now, we can create a fresh branch for our bug #21:

```sh
(master)$ git checkout -b 21
(21)$
```

Now, let's *cherry-pick* the commit for bug #21 on top of our branch. That means we will be applying that commit, and only that commit, directly on top of whatever our head is at.

```sh
(21)$ git cherry-pick e3851e8
```

At this point, there is a possibility there might be conflicts. See the [**There were conflicts**](#merge-conflict) section in the [interactive rebasing section above](#interactive-rebase) for how to resolve conflicts.

Now let's create a new branch for bug #14, also based on master

```sh
(21)$ git checkout master
(master)$ git checkout -b 14
(14)$
```

And finally, let's cherry-pick the commit for bug #14:

```sh
(14)$ git cherry-pick 5ea5173
```

<a name="delete-stale-local-branches"></a>
### I want to delete local branches that were deleted upstream
Once you merge a pull request on GitHub, it gives you the option to delete the merged branch in your fork. If you aren't planning to keep working on the branch, it's cleaner to delete the local copies of the branch so you don't end up cluttering up your working checkout with a lot of stale branches.

```sh
$ git fetch -p upstream
```

where, `upstream` is the remote you want to fetch from.

<a name='restore-a-deleted-branch'></a>
### I accidentally deleted my branch

If you're regularly pushing to remote, you should be safe most of the time. But still sometimes you may end up deleting your branches. Let's say we create a branch and create a new file:

```sh
(master)$ git checkout -b my-branch
(my-branch)$ git branch
(my-branch)$ touch foo.txt
(my-branch)$ ls
README.md foo.txt
```

Let's add it and commit.

```sh
(my-branch)$ git add .
(my-branch)$ git commit -m 'foo.txt added'
(my-branch)$ foo.txt added
 1 files changed, 1 insertions(+)
 create mode 100644 foo.txt
(my-branch)$ git log

commit 4e3cd85a670ced7cc17a2b5d8d3d809ac88d5012
Author: siemiatj <siemiatj@example.com>
Date:   Wed Jul 30 00:34:10 2014 +0200

    foo.txt added

commit 69204cdf0acbab201619d95ad8295928e7f411d5
Author: Kate Hudson <katehudson@example.com>
Date:   Tue Jul 29 13:14:46 2014 -0400

    Fixes #6: Force pushing after amending commits
```

Now we're switching back to master and 'accidentally' removing our branch.

```sh
(my-branch)$ git checkout master
Switched to branch 'master'
Your branch is up-to-date with 'origin/master'.
(master)$ git branch -D my-branch
Deleted branch my-branch (was 4e3cd85).
(master)$ echo oh noes, deleted my branch!
oh noes, deleted my branch!
```

At this point you should get familiar with 'reflog', an upgraded logger. It stores the history of all the action in the repo.

```
(master)$ git reflog
69204cd HEAD@{0}: checkout: moving from my-branch to master
4e3cd85 HEAD@{1}: commit: foo.txt added
69204cd HEAD@{2}: checkout: moving from master to my-branch
```

As you can see we have commit hash from our deleted branch. Let's see if we can restore our deleted branch.

```sh
(master)$ git checkout -b my-branch-help
Switched to a new branch 'my-branch-help'
(my-branch-help)$ git reset --hard 4e3cd85
HEAD is now at 4e3cd85 foo.txt added
(my-branch-help)$ ls
README.md foo.txt
```

Voila! We got our removed file back. `git reflog` is also useful when rebasing goes terribly wrong.

### I want to delete a branch

To delete a remote branch:

```sh
(master)$ git push origin --delete my-branch
```

You can also do:

```sh
(master)$ git push origin :my-branch
```

To delete a local branch:

```sh
(master)$ git branch -d my-branch
```

To delete a local branch that *has not* been merged to the current branch or an upstream:

```sh
(master)$ git branch -D my-branch
```

### I want to delete multiple branches

Say you want to delete all branches that start with `fix/`:

```sh
(master)$ git branch | grep 'fix/' | xargs git branch -d
```

### I want to rename a branch

To rename the current (local) branch:

```sh
(master)$ git branch -m new-name
```

To rename a different (local) branch:

```sh
(master)$ git branch -m old-name new-name
```

<a name="i-want-to-checkout-to-a-remote-branch-that-someone-else-is-working-on"></a>
### I want to checkout to a remote branch that someone else is working on

First, fetch all branches from remote:

```sh
(master)$ git fetch --all
```

Say you want to checkout to `daves` from the remote.

```sh
(master)$ git checkout --track origin/daves
Branch daves set up to track remote branch daves from origin.
Switched to a new branch 'daves'
```

(`--track` is shorthand for `git checkout -b [branch] [remotename]/[branch]`)

This will give you a local copy of the branch `daves`, and any update that has been pushed will also show up remotely.

### I want to create a new remote branch from current local one

```sh
$ git push <remote> HEAD
```

If you would also like to set that remote branch as upstream for the current one, use the following instead:

```sh
$ git push -u <remote> HEAD
```

With the `upstream` mode and the `simple` (default in Git 2.0) mode of the `push.default` config, the following command will push the current branch with regards to the remote branch that has been registered previously with `-u`:

```sh
$ git push
```

The behavior of the other modes of `git push` is described in the [doc of `push.default`](https://git-scm.com/docs/git-config#git-config-pushdefault).

### I want to set a remote branch as the upstream for a local branch

You can set a remote branch as the upstream for the current local branch using:

```sh
$ git branch --set-upstream-to [remotename]/[branch]
# or, using the shorthand:
$ git branch -u [remotename]/[branch]
```

To set the upstream remote branch for another local branch:

```sh
$ git branch -u [remotename]/[branch] [local-branch]
```

<a name="i-want-to-set-my-HEAD-to-track-the-default-remote-branch"></a>
### I want to set my HEAD to track the default remote branch

By checking your remote branches, you can see which remote branch your HEAD is tracking. In some cases, this is not the desired branch.

```sh
$ git branch -r
  origin/HEAD -> origin/gh-pages
  origin/master
```

To change `origin/HEAD` to track `origin/master`, you can run this command:

```sh
$ git remote set-head origin --auto
origin/HEAD set to master
```

### I made changes on the wrong branch

You've made uncommitted changes and realise you're on the wrong branch. Stash changes and apply them to the branch you want:

```sh
(wrong_branch)$ git stash
(wrong_branch)$ git checkout <correct_branch>
(correct_branch)$ git stash apply
```

## Rebasing and Merging

<a name="undo-rebase"></a>
### I want to undo rebase/merge

You may have merged or rebased your current branch with a wrong branch, or you can't figure it out or finish the rebase/merge process. Git saves the original HEAD pointer in a variable called ORIG_HEAD before doing dangerous operations, so it is simple to recover your branch at the state before the rebase/merge.

```sh
(my-branch)$ git reset --hard ORIG_HEAD
```

<a name="force-push-rebase"></a>
### I rebased, but I don't want to force push

Unfortunately, you have to force push, if you want those changes to be reflected on the remote branch. This is because you have changed the history. The remote branch won't accept changes unless you force push. This is one of the main reasons many people use a merge workflow, instead of a rebasing workflow - large teams can get into trouble with developers force pushing. Use this with caution. A safer way to use rebase is not to reflect your changes on the remote branch at all, and instead to do the following:

```sh
(master)$ git checkout my-branch
(my-branch)$ git rebase -i master
(my-branch)$ git checkout master
(master)$ git merge --ff-only my-branch
```

For more, see [this SO thread](https://stackoverflow.com/questions/11058312/how-can-i-use-git-rebase-without-requiring-a-forced-push).

<a name="interactive-rebase"></a>
### I need to combine commits

Let's suppose you are working in a branch that is/will become a pull-request against `master`. In the simplest case when all you want to do is to combine *all* commits into a single one and you don't care about commit timestamps, you can reset and recommit. Make sure the master branch is up to date and all your changes committed, then:

```sh
(my-branch)$ git reset --soft master
(my-branch)$ git commit -am "New awesome feature"
```

If you want more control, and also to preserve timestamps, you need to do something called an interactive rebase:

```sh
(my-branch)$ git rebase -i master
```

If you aren't working against another branch you'll have to rebase relative to your `HEAD`. If you want to squash the last 2 commits, for example, you'll have to rebase against `HEAD~2`. For the last 3, `HEAD~3`, etc.

```sh
(master)$ git rebase -i HEAD~2
```

After you run the interactive rebase command, you will see something like this in your  text editor:

```vim
pick a9c8a1d Some refactoring
pick 01b2fd8 New awesome feature
pick b729ad5 fixup
pick e3851e8 another fix

# Rebase 8074d12..b729ad5 onto 8074d12
#
# Commands:
#  p, pick = use commit
#  r, reword = use commit, but edit the commit message
#  e, edit = use commit, but stop for amending
#  s, squash = use commit, but meld into previous commit
#  f, fixup = like "squash", but discard this commit's log message
#  x, exec = run command (the rest of the line) using shell
#
# These lines can be re-ordered; they are executed from top to bottom.
#
# If you remove a line here THAT COMMIT WILL BE LOST.
#
# However, if you remove everything, the rebase will be aborted.
#
# Note that empty commits are commented out
```

All the lines beginning with a `#` are comments, they won't affect your rebase.

Then you replace `pick` commands with any in the list above, and you can also remove commits by removing corresponding lines.

For example, if you want to **leave the oldest (first) commit alone and combine all the following commits with the second oldest**, you should edit the letter next to each commit except the first and the second to say `f`:

```vim
pick a9c8a1d Some refactoring
pick 01b2fd8 New awesome feature
f b729ad5 fixup
f e3851e8 another fix
```

If you want to combine these commits **and rename the commit**, you should additionally add an `r` next to the second commit or simply use `s` instead of `f`:

```vim
pick a9c8a1d Some refactoring
pick 01b2fd8 New awesome feature
s b729ad5 fixup
s e3851e8 another fix
```

You can then rename the commit in the next text prompt that pops up.

```vim
Newer, awesomer features

# Please enter the commit message for your changes. Lines starting
# with '#' will be ignored, and an empty message aborts the commit.
# rebase in progress; onto 8074d12
# You are currently editing a commit while rebasing branch 'master' on '8074d12'.
#
# Changes to be committed:
#   modified:   README.md
#

```

If everything is successful, you should see something like this:

```sh
(master)$ Successfully rebased and updated refs/heads/master.
```

#### Safe merging strategy
`--no-commit` performs the merge but pretends the merge failed and does not autocommit, giving the user a chance to inspect and further tweak the merge result before committing. `no-ff` maintains evidence that a feature branch once existed, keeping project history consistent.

```sh
(master)$ git merge --no-ff --no-commit my-branch
```

#### I need to merge a branch into a single commit

```sh
(master)$ git merge --squash my-branch
```

<a name="rebase-unpushed-commits"></a>
#### I want to combine only unpushed commits

Sometimes you have several work in progress commits that you want to combine before you push them upstream. You don't want to accidentally combine any commits that have already been pushed upstream because someone else may have already made commits that reference them.

```sh
(master)$ git rebase -i @{u}
```

This will do an interactive rebase that lists only the commits that you haven't already pushed, so it will be safe to reorder/fix/squash anything in the list.

#### I need to abort the merge

Sometimes the merge can produce problems in certain files, in those cases we can use the option `abort` to abort the current conflict resolution process, and try to reconstruct the pre-merge state.

```sh
(my-branch)$ git merge --abort
```

This command is available since Git version >= 1.7.4

### I need to update the parent commit of my branch

Say I have a master branch, a feature-1 branch branched from master, and a feature-2 branch branched off of feature-1. If I make a commit to feature-1, then the parent commit of feature-2 is no longer accurate (it should be the head of feature-1, since we branched off of it). We can fix this with `git rebase --onto`.

```sh
(feature-2)$ git rebase --onto feature-1 <the first commit in your feature-2 branch that you don't want to bring along> feature-2
```

This helps in sticky scenarios where you might have a feature built on another feature that hasn't been merged yet, and a bugfix on the feature-1 branch needs to be reflected in your feature-2 branch.

### Check if all commits on a branch are merged

To check if all commits on a branch are merged into another branch, you should diff between the heads (or any commits) of those branches:

```sh
(master)$ git log --graph --left-right --cherry-pick --oneline HEAD...feature/120-on-scroll
```

This will tell you if any commits are in one but not the other, and will give you a list of any nonshared between the branches. Another option is to do this:

```sh
(master)$ git log master ^feature/120-on-scroll --no-merges
```

### Possible issues with interactive rebases

<a name="noop"></a>
#### The rebase editing screen says 'noop'

If you're seeing this:
```
noop
```

That means you are trying to rebase against a branch that is at an identical commit, or is *ahead* of your current branch. You can try:

* making sure your master branch is where it should be
* rebase against `HEAD~2` or earlier instead

<a name="merge-conflict"></a>
#### There were conflicts

If you are unable to successfully complete the rebase, you may have to resolve conflicts.

First run `git status` to see which files have conflicts in them:

```sh
(my-branch)$ git status
On branch my-branch
Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git checkout -- <file>..." to discard changes in working directory)

  both modified:   README.md
```

In this example, `README.md` has conflicts. Open that file and look for the following:

```vim
   <<<<<<< HEAD
   some code
   =========
   some code
   >>>>>>> new-commit
```

You will need to resolve the differences between the code that was added in your new commit (in the example, everything from the middle line to `new-commit`) and your `HEAD`.

If you want to keep one branch's version of the code, you can use `--ours` or `--theirs`:

```sh
(master*)$ git checkout --ours README.md
```

- When *merging*, use `--ours` to keep changes from the local branch, or `--theirs` to keep changes from the other branch.
- When *rebasing*, use `--theirs` to keep changes from the local branch, or `--ours` to keep changes from the other branch. For an explanation of this swap, see [this note in the Git documentation](https://git-scm.com/docs/git-rebase#git-rebase---merge).

If the merges are more complicated, you can use a visual diff editor:

```sh
(master*)$ git mergetool -t opendiff
```

After you have resolved all conflicts and tested your code, `git add` the files you have changed, and then continue the rebase with `git rebase --continue`

```sh
(my-branch)$ git add README.md
(my-branch)$ git rebase --continue
```

If after resolving all the conflicts you end up with an identical tree to what it was before the commit, you need to `git rebase --skip` instead.

If at any time you want to stop the entire rebase and go back to the original state of your branch, you can do so:

```sh
(my-branch)$ git rebase --abort
```

<a name="stashing"></a>
## Stash

### Stash all edits

To stash all the edits in your working directory

```sh
$ git stash
```

If you also want to stash untracked files, use `-u` option.

```sh
$ git stash -u
```

### Stash specific files

To stash only one file from your working directory

```sh
$ git stash push working-directory-path/filename.ext
```

To stash multiple files from your working directory

```sh
$ git stash push working-directory-path/filename1.ext working-directory-path/filename2.ext
```

<a name="stash-msg"></a>
### Stash with message

```sh
$ git stash save <message>
```

<a name="stash-apply-specific"></a>
### Apply a specific stash from list

First check your list of stashes with message using

```sh
$ git stash list
```

Then apply a specific stash from the list using

```sh
$ git stash apply "stash@{n}"
```

Here, 'n' indicates the position of the stash in the stack. The topmost stash will be position 0.

## Finding

### I want to find a string in any commit

To find a certain string which was introduced in any commit, you can use the following structure:

```sh
$ git log -S "string to find"
```

Commons parameters:

* `--source` means to show the ref name given on the command line by which each commit was reached.

* `--all` means to start from every branch.

* `--reverse` prints in reverse order, it means that will show the first commit that made the change.

<a name="i-want-to-find-by-author-committer"></a>
### I want to find by author/committer

To find all commits by author/committer you can use:

```sh
$ git log --author=<name or email>
$ git log --committer=<name or email>
```

Keep in mind that author and committer are not the same. The `--author` is the person who originally wrote the code; on the other hand, the `--committer`, is the person who committed the code on behalf of the original author.

### I want to list commits containing specific files

To find all commits containing a specific file you can use:

```sh
$ git log -- <path to file>
```

You would usually specify an exact path, but you may also use wild cards in the path and file name:

```sh
$ git log -- **/*.js
```

While using wildcards, it's useful to inform `--name-status` to see the list of committed files:

```sh
$ git log --name-status -- **/*.js
```

### Find a tag where a commit is referenced

To find all tags containing a specific commit:

```sh
$ git tag --contains <commitid>
```

## Submodules

<a name="clone-submodules"></a>
### Clone all submodules

```sh
$ git clone --recursive git://github.com/foo/bar.git
```

If already cloned:

```sh
$ git submodule update --init --recursive
```

<a name="delete-submodule"></a>
### Remove a submodule

Creating a submodule is pretty straight-forward, but deleting them less so. The commands you need are:

```sh
$ git submodule deinit submodulename
$ git rm submodulename
$ git rm --cached submodulename
$ rm -rf .git/modules/submodulename
```

## Miscellaneous Objects

### Restore a deleted file

First find the commit when the file last existed:

```sh
$ git rev-list -n 1 HEAD -- filename
```

Then checkout that file:

```
git checkout deletingcommitid^ -- filename
```

### Delete tag

```sh
$ git tag -d <tag_name>
$ git push <remote> :refs/tags/<tag_name>
```

<a name="recover-tag"></a>
### Recover a deleted tag

If you want to recover a tag that was already deleted, you can do so by following these steps: First, you need to find the unreachable tag:

```sh
$ git fsck --unreachable | grep tag
```

Make a note of the tag's hash. Then, restore the deleted tag with following, making use of [`git update-ref`](https://git-scm.com/docs/git-update-ref):

```sh
$ git update-ref refs/tags/<tag_name> <hash>
```

Your tag should now have been restored.

### Deleted Patch

If someone has sent you a pull request on GitHub, but then deleted their original fork, you will be unable to clone their repository or to use `git am` as the [.diff, .patch](https://github.com/blog/967-github-secrets) urls become unavailable. But you can checkout the PR itself using [GitHub's special refs](https://gist.github.com/piscisaureus/3342247). To fetch the content of PR#1 into a new branch called pr_1:

```sh
$ git fetch origin refs/pull/1/head:pr_1
From github.com:foo/bar
 * [new ref]         refs/pull/1/head -> pr_1
```

### Exporting a repository as a Zip file

```sh
$ git archive --format zip --output /full/path/to/zipfile.zip master
```
### Push a branch and a tag that have the same name

If there is a tag on a remote repository that has the same name as a branch you will get the following error when trying to push that branch with a standard `$ git push <remote> <branch>` command.

```sh
$ git push origin <branch>
error: dst refspec same matches more than one.
error: failed to push some refs to '<git server>'
```

Fix this by specifying you want to push the head reference.

```sh
$ git push origin refs/heads/<branch-name>
```

If you want to push a tag to a remote repository that has the same name as a branch, you can use a similar command.

```sh
$ git push origin refs/tags/<tag-name>
```

## Tracking Files

<a href="i-want-to-change-a-file-names-capitalization-without-changing-the-contents-of-the-file"></a>
### I want to change a file name's capitalization, without changing the contents of the file

```sh
(master)$ git mv --force myfile MyFile
```

### I want to overwrite local files when doing a git pull

```sh
(master)$ git fetch --all
(master)$ git reset --hard origin/master
```

<a href="remove-from-git"></a>
### I want to remove a file from Git but keep the file

```sh
(master)$ git rm --cached log.txt
```

### I want to revert a file to a specific revision

Assuming the hash of the commit you want is c5f567:

```sh
(master)$ git checkout c5f567 -- file1/to/restore file2/to/restore
```

If you want to revert to changes made just 1 commit before c5f567, pass the commit hash as c5f567~1:

```sh
(master)$ git checkout c5f567~1 -- file1/to/restore file2/to/restore
```

### I want to list changes of a specific file between commits or branches

Assuming you want to compare last commit with file from commit c5f567:

```sh
$ git diff HEAD:path_to_file/file c5f567:path_to_file/file
```

Same goes for branches:

```sh
$ git diff master:path_to_file/file staging:path_to_file/file
```

### I want Git to ignore changes to a specific file

This works great for config templates or other files that require locally adding credentials that shouldn't be committed.

```sh
$ git update-index --assume-unchanged file-to-ignore
```

Note that this does *not* remove the file from source control - it is only ignored locally. To undo this and tell Git to notice changes again, this clears the ignore flag:

```sh
$ git update-index --no-assume-unchanged file-to-stop-ignoring
```

## Configuration

### I want to add aliases for some Git commands

On OS X and Linux, your git configuration file is stored in ```~/.gitconfig```.  I've added some example aliases I use as shortcuts (and some of my common typos) in the ```[alias]``` section as shown below:

```vim
[alias]
    a = add
    amend = commit --amend
    c = commit
    ca = commit --amend
    ci = commit -a
    co = checkout
    d = diff
    dc = diff --changed
    ds = diff --staged
    extend = commit --amend -C HEAD
    f = fetch
    loll = log --graph --decorate --pretty=oneline --abbrev-commit
    m = merge
    one = log --pretty=oneline
    outstanding = rebase -i @{u}
    reword = commit --amend --only
    s = status
    unpushed = log @{u}
    wc = whatchanged
    wip = rebase -i @{u}
    zap = fetch -p
```

### I want to add an empty directory to my repository

You can’t! Git doesn’t support this, but there’s a hack. You can create a .gitignore file in the directory with the following contents:

```
 # Ignore everything in this directory
 *
 # Except this file
 !.gitignore
```

Another common convention is to make an empty file in the folder, titled .gitkeep.

```sh
$ mkdir mydir
$ touch mydir/.gitkeep
```

You can also name the file as just .keep , in which case the second line above would be ```touch mydir/.keep```

### I want to cache a username and password for a repository

You might have a repository that requires authentication.  In which case you can cache a username and password so you don't have to enter it on every push / pull. Credential helper can do this for you.

```sh
$ git config --global credential.helper cache
# Set git to use the credential memory cache
```

```sh
$ git config --global credential.helper 'cache --timeout=3600'
# Set the cache to timeout after 1 hour (setting is in seconds)
```

### I want to make Git ignore permissions and filemode changes

```sh
$ git config core.fileMode false
```

If you want to make this the default behaviour for logged-in users, then use:

```sh
$ git config --global core.fileMode false
```

### I want to set a global user

To configure user information used across all local repositories, and to set a name that is identifiable for credit when review version history:

```sh
$ git config --global user.name “[firstname lastname]”
```

To set an email address that will be associated with each history marker:

```sh
git config --global user.email “[valid-email]”
```

### I want to add command line coloring for Git

To set automatic command line coloring for Git for easy reviewing:

```sh
$ git config --global color.ui auto
```

## I've no idea what I did wrong

So, you're screwed - you `reset` something, or you merged the wrong branch, or you force pushed and now you can't find your commits. You know, at some point, you were doing alright, and you want to go back to some state you were at.

This is what `git reflog` is for. `reflog` keeps track of any changes to the tip of a branch, even if that tip isn't referenced by a branch or a tag. Basically, every time HEAD changes, a new entry is added to the reflog. This only works for local repositories, sadly, and it only tracks movements (not changes to a file that weren't recorded anywhere, for instance).

```sh
(master)$ git reflog
0a2e358 HEAD@{0}: reset: moving to HEAD~2
0254ea7 HEAD@{1}: checkout: moving from 2.2 to master
c10f740 HEAD@{2}: checkout: moving from master to 2.2
```

The reflog above shows a checkout from master to the 2.2 branch and back. From there, there's a hard reset to an older commit. The latest activity is represented at the top labeled `HEAD@{0}`.

If it turns out that you accidentally moved back, the reflog will contain the commit master pointed to (0254ea7) before you accidentally dropped 2 commits.

```sh
$ git reset --hard 0254ea7
```

Using `git reset` it is then possible to change master back to the commit it was before. This provides a safety net in case history was accidentally changed.

(copied and edited from [Source](https://www.atlassian.com/git/tutorials/rewriting-history/git-reflog)).

# Other Resources
