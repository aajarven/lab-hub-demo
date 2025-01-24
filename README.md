# lab-hub demo

NB: this description assumes that you have git alias `graph` that does `log --all --graph --decorate --oneline`.

## Creating the repo and initial commits to each remote (GitHub and GitLab)

I created a new repo into gitlab (could have been github equally easily, a git repo is a git repo) and cloned it:
```
vagrant@kpdev:~/scratch$ git clone ssh://git@gitlab.ci.csc.fi:10022/ajarven/lab-hub-demo.git
...
vagrant@kpdev:~/scratch/lab-hub-demo$ git graph
* 33d7d0b (HEAD -> main, origin/main, origin/HEAD) Initial commit
vagrant@kpdev:~/scratch/lab-hub-demo$ git branch -a
* main
  remotes/origin/HEAD -> origin/main
  remotes/origin/main
```

I also added a couple of commits so there's a bit more history:
```
vagrant@kpdev:~/scratch/lab-hub-demo$ git graph
* 70f4c00 (HEAD -> main) ignore temporary files from vim
* a617f44 remove extra stuff from readme
* 33d7d0b (origin/main, origin/HEAD) Initial commit
```

Now if I want to have the same repo be present in github.com, I can just create an empty repo there (does not need to have the same name or anything, but for this demo it will). This needs to be done without any readme, license or such files automatically committed. Github automatically suggests I create some initial commits and branches (already done) and then run `git remote add origin git@github.com:aajarven/lab-hub-demo.git`. I can't do that exactly though, as remote name `origin` is already reserved for GitLab, at least not without changing the gitlab-pointing remote reference to another name. Instead let's say that we want to publish the code in GitHub, so we'll name the new remote `publish`:
```
vagrant@kpdev:~/scratch/lab-hub-demo$ git remote add publish git@github.com:aajarven/lab-hub-demo.git
```
Now we see that we have two remotes:
```
vagrant@kpdev:~/scratch/lab-hub-demo$ git remote -v
origin	ssh://git@gitlab.ci.csc.fi:10022/ajarven/lab-hub-demo.git (fetch)
origin	ssh://git@gitlab.ci.csc.fi:10022/ajarven/lab-hub-demo.git (push)
publish	git@github.com:aajarven/lab-hub-demo.git (fetch)
publish	git@github.com:aajarven/lab-hub-demo.git (push)
```

Now I can create and push branches to either of the remotes. Let's say that we first push the newly created commits to gitlab and then create a new branch to represent the corresponding branch on GitHub:
```
vagrant@kpdev:~/scratch/lab-hub-demo$ git status
On branch main
Your branch is ahead of 'origin/main' by 2 commits.
  (use "git push" to publish your local commits)

nothing to commit, working tree clean
vagrant@kpdev:~/scratch/lab-hub-demo$ git push
...
To ssh://gitlab.ci.csc.fi:10022/ajarven/lab-hub-demo.git
   33d7d0b..70f4c00  main -> main
vagrant@kpdev:~/scratch/lab-hub-demo$ git checkout -b publish/main
Switched to a new branch 'publish/main'
```
We can't immediately push this new branch though, because git doesn't know where to:
```
vagrant@kpdev:~/scratch/lab-hub-demo$ git push
fatal: The current branch publish/main has no upstream branch.
To push the current branch and set the remote as upstream, use

    git push --set-upstream origin publish/main
```
and the suggested remote is not the one we want. But let's fix the suggested command and run it:
```
vagrant@kpdev:~/scratch/lab-hub-demo$ git push --set-upstream publish  publish/main
...
To github.com:aajarven/lab-hub-demo.git
 * [new branch]      publish/main -> publish/main
Branch 'publish/main' set up to track remote branch 'publish/main' from 'publish'.
```

Now I can visit either https://github.com/aajarven/lab-hub-demo or https://gitlab.ci.csc.fi/ajarven/lab-hub-demo and see the same content.


## Software development

If we had pipelines runnin in GitLab, we'd likely want our process to start by creating a branch to be pushed on the GitLab side and do our development actions there:
```
vagrant@kpdev:~/scratch/lab-hub-demo$ git checkout -b awesome-feature
Switched to a new branch 'awesome-feature'
...
vagrant@kpdev:~/scratch/lab-hub-demo$ git graph
* fd4972b (HEAD -> awesome-feature) Make the feature awesomer
* 3783ae0 Add an awesome feature
* 70f4c00 (publish/publish/main, origin/main, origin/HEAD, publish/main, main) ignore temporary files from vim
* a617f44 remove extra stuff from readme
* 33d7d0b Initial commit
vagrant@kpdev:~/scratch/lab-hub-demo$ git push --set-upstream origin awesome-feature
...
To ssh://gitlab.ci.csc.fi:10022/ajarven/lab-hub-demo.git
 * [new branch]      awesome-feature -> awesome-feature
Branch 'awesome-feature' set up to track remote branch 'awesome-feature' from 'origin'.
```
Now the CI/CD pipelines would run, we would do our code reviews etc. Nothing out of ordinary this far. Then the merge request would be merged at gitlab (I'll do that by hand on command line without handling a MR):
```
vagrant@kpdev:~/scratch/lab-hub-demo$ git checkout main
vagrant@kpdev:~/scratch/lab-hub-demo$ git merge --ff-only awesome-feature
vagrant@kpdev:~/scratch/lab-hub-demo$ git push
vagrant@kpdev:~/scratch/lab-hub-demo$ git graph
* fd4972b (HEAD -> main, origin/main, origin/awesome-feature, origin/HEAD, awesome-feature) Make the feature awesomer
* 3783ae0 Add an awesome feature
* 70f4c00 (publish/publish/main, publish/main) ignore temporary files from vim
* a617f44 remove extra stuff from readme
* 33d7d0b Initial commit
```

## Publishing changes to GitHub
Now that we want to make the changes available to the public, we can just make sure that the GitHub remote branch is where we want it to be, and then push:
```
vagrant@kpdev:~/scratch/lab-hub-demo$ git checkout publish/main
vagrant@kpdev:~/scratch/lab-hub-demo$ git merge --ff-only main
vagrant@kpdev:~/scratch/lab-hub-demo$ git graph
* fd4972b (HEAD -> publish/main, origin/main, origin/awesome-feature, origin/HEAD, main, awesome-feature) Make the feature awesomer
* 3783ae0 Add an awesome feature
* 70f4c00 (publish/publish/main) ignore temporary files from vim
* a617f44 remove extra stuff from readme
* 33d7d0b Initial commit
vagrant@kpdev:~/scratch/lab-hub-demo$ git push
Enter passphrase for key '/home/vagrant/.ssh/id_ed25519_github':
...
To github.com:aajarven/lab-hub-demo.git
   70f4c00..fd4972b  publish/main -> publish/main
```

And now the changes are visible in GitHub too.
