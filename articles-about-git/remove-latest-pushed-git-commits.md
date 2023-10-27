
## Remove latest pushed Git-commits from remote repo

We’ll do an experiment on GitHub, but practically you can do this on other similar platforms as well. Our goal is to delete recently pushed commits from the remote repository branch.

Let’s assume we have a remote Git repository, and there we already have some commits pushed. Two developers maintaining the project:

- _Owen_ — the repository owner
- _Cole_ — the repository collaborator

In some moment of time project’s comment history on the **_master_** branch looks like this:

<img src="https://i.imgur.com/n9vBEJD.png" style="width:100%;">

_Fake page from GitHub commits history_

As we can see, the first five (starting from the bottom) commits were pushed by the owner, and the next four commits were pushed by the collaborator.

**Now the question:**

Is there a way to remove the latest commits from the **_master_** branch?

**Criterias:**

- **_master_** branch is the default branch of the project
(it could be any branch)
- no one can delete **_master_** branch
(we can’t delete the branch and recreate it anyway)
- last commits done by a collaborator of the project
(it could be one commit or more by one or more collaborators)

**Our goal:**

Our goal is to clean up all the latest commits (in our case, from _“Commit 6”_ to _”Commit 9"_) from the **_master_** branch.

**Solution steps:**

Let’s make sure, that we are on the **_master_** branch, and pull the remote state to the local machine:

```shell
git checkout master
git pull origin master
```

Let’s remove all the latest commits on the local machine, which we want to delete. We’ll point the hash-code of the commit, that would be the HEAD commit after all the deletion (the hash of _“Commit 5”_ in our case):

<img src="https://i.imgur.com/vxC9MkI.jpg" style="width:100%;">

_commits that will be removed are in red rectangle_

```shell
git reset --soft 9d24c70
git reset
git checkout .
```

Now we’ll create a new temporary branch and switch to it with current HEAD. We’ll remember to delete that right after the commits removal.

```shell
git checkout -b temp
```

Let’s make some small changes (something, anything safe) to that temporary branch, commit them, and push to remote:

```shell
git add .
git commit -m "small changes"
git push origin temp
```

**_Now, here’s the most important part of this article:_**
Switch back to the **_master_** branch, rebase the temporary branch on it, and push that to remote:

```shell
git checkout master
git rebase temp
git push origin master
```

At this moment we already have **_master_** branch cleaned from the latest unnecessary commits.

<img src="https://i.imgur.com/r0Qo5hF.jpg" style="width:100%;">

_master branch now is clean_

We can be sure about having the actual result like this:

```shell
git log --graph --oneline --decorate --all
```

Additionally, don’t forget to remove the temporary branch from the remote and from the local as well:

```shell
git push origin temp --delete
git branch -d temp
```

**That’s all.**

Here’s the link of the appropriate [**_GitHub Gist_**](https://gist.github.com/boolfalse/e6dad72276b2a3703fb53a1730314bd5) that you can check out as well.

***

If you liked this article, feel free to follow me here. 😇

To explore projects working with various modern technologies, you can follow me on [**GitHub**](https://github.com/boolfalse), where I actively publicize much of my work.

For more info, you can visit my website [**boolfalse.com**](https://boolfalse.com/)

Thank you !!!
