# git-learning
![](resources/icons/git.png)

----
# Table of Contents
1. [Setting your environment](#settings)
2. [Working locally](#local)
   1. [Starting with git](#starting)
   2. [Tracking files](#tracking-files)
   3. [Log & alias](#log-alias)
   4. [HEAD, relative references & branch creation and repositioning](#head-branching)
   5. [Stashing](#stashing)
   6. [Rollback changes](#rollback)
   7. [Amending](#amending)
   8. [Cherry picking](#cherry-picking)
   9. [Branch integration (merge & rebase)](#merge-rebase)
3. [Working with remotes](#remotes)
   2. [Working with others 1 (fetching)](#fetching)
   1. [Working with others 2 (pull-push)](#pull-push)
   3. [Force push](#force-push)
   4. [Tracking your branches](#tracking-branches)
   5. [Pull Requests (PRs)](#prs)
----
## 0. Cheatsheet
We use some [mermaid] graphs, only visible in GitHub (you will see only code in local)
<!--https://mermaid-js.github.io/mermaid-live-editor-->
```mermaid
sequenceDiagram
    participant w as Working directory
    participant s as Staging area
    participant l as Local repository
    participant r as Remote repository
    w->>s: git add
    s->>l: git commit
    l->>l: git tag
    rect rgb(191, 223, 255)
    l->>l: git branch
    r->>l: git branch
    end
    l->>r: git push
    r->>l: git fetch
    rect rgb(191, 223, 255)
    l->>w: git checkout
    r->>w: git checkout
    end
    rect rgb(191, 223, 255)
    l->>w: git reset
    r->>w: git reset
    end
    # Note over w, r: You can also create a branch, checkout or reset from remote
```

## 1. Setting your environment<a name="settings"></a>
![](resources/icons/docker.png)

We will use [docker],[^docker] so we can have the same isolated environment:
```bash
cd docker/
docker build -t training/git .
docker run -it -d -v $(pwd)/git-volume/:/root/projects/ --name learning-git training/git:latest
docker exec -it learning-git bash
```
Now, inside your docker, got to **projects/local** folder: `cd projects/local`

----

## 2. Working locally<a name="local"></a>

### 2.1 Starting with git<a name="starting"></a>
![](resources/icons/config.png)

Let's see [git config] command
<details>
<summary>Command log</summary>

```bash
ls -l
# We should have an empty folder, create a new one if you want
git config --global init.defaultBranch main
git config --global user.email eduardo.ruiz@hablapps.com
git config --global user.name "Eduardo Ruiz Alonso"

git config --list

git init
echo -e "# This is my git project\n" > README.md
echo -e "Cheatsheet: https://devhints.io/git-log-format\n\n" >> README.md
git add .
git commit -m "Initial commit"

git config user.email eduardo.ruiz@hablapps.com
git config user.name "Eduardo Ruiz"

git config --list --show-origin
```
</details>

### 2.2 Tracking files<a name="tracking-files"></a>
![](resources/icons/track.png)
<details>
<summary>Command log</summary>

```bash
echo "this is a test" > test.txt
git status
echo "-----" >> README.md
git add test.txt
git status

git commit -a -m "commit 2"   ## -a ==> ONLY TRACKED FILES
git status

touch .gitignore
touch .testing
git status
git add .gitignore
echo ".*" >> .gitignore
git status
mkdir secrets
touch secrets/secret1.txt
touch secrets/secret2.txt
git status
git status -u
git status -u -s
echo "secrets/*" >> .gitignore
git status
git commit -am "gitignore"
```
</details>

What if I am already tracking a file I do not want to track?
<details>
<summary>Command log</summary>

```bash
touch script.sh
git add .
git commit -m "add script"
echo "*.sh" >> .gitignore
git commit -am "ignore sh"
echo "#bash" >> edu.sh
git status
git checkout .
git rm --cached edu.sh
git status
ls -l
git commit -m "untrack script.sh"
echo "#bash" >> edu.sh
git status
```
</details>

### 2.3 Log & alias<a name="log-alias"></a>
<details>
<summary>Command log</summary>

st=status -sb
recent = branch --format='%(HEAD) %(color:yellow)%(refname:short)%(color:reset) - %(contents:subject) %(color:green)(%(committerdate:relative)) [%(authorname)]' --sort=-committerdate
stash-unstaged=stash save --keep-index

```bash
echo "Something" >> README.md
git add .
git config --global alias.unstage 'reset --'
git config --global alias.discard 'checkout --'
git config --global alias.untrack 'rm --cached'
git config --global alias.stats 'status -s'

git stats
git unstage README.md
git stats
git discard README.md
git stats

git log
git log -n 1
git config --global alias.logtree "log --graph --abbrev-commit --decorate --format=format:'%C(bold blue)%h%C(reset) - %C(bold cyan)(%ar|%cr)%C(reset) %C(bold yellow)%d%C(reset) %C(white)%s%C(reset) %C(dim green)- %an%C(reset)' --all"
git logtree
```
</details>

### 2.4 HEAD, relative references & branch creation and repositioning<a name="head-branching"></a>
![](resources/icons/branch.png)
<details>
<summary>Command log</summary>

git diff HEAD^

```bash
git branch branch1
git branch
git checkout branch1
git branch
git checkout -b branch2
git branch
git logtree

echo "commit 3" >> README.md
git add .
git commit -am "commit 3"
git checkout branch1

echo "commit 4" >> tests.txt
git add .
git commit -am "commit 4"
git merge branch2
git logtree

git branch ref
git branch -f main ref
git logtree
git checkout main
git branch -f ref HEAD~1
git branch -f refParen1 main^1
git branch -f refParen2 main^2
git logtree

git branch newBranch commit_hash
git logtree

git tag v0.0.2
git tag v0.0.1 main~3
git tag
git branch release/v0.0.1 v0.0.1
git logtree
```
</details>

### 2.5 Stashing<a name="stashing"></a>
![](resources/icons/stash.png)
<details>
<summary>Command log</summary>

```bash
git checkout -b feature/myFeature
echo "some changes" >> README.md
git checkout main
git commit -am "main changes"
git checkout feature/myFeature
echo "other changes" >> README.md
git checkout main           # <----this will fail
git stats
git stash
git stash list
git stats
git checkout main

git checkout feature/myFeature
git stash pop stash@{0}
git stash list
git stash
git stash apply stash@{0}
git stash list
git stash -m "WIP feature my stash"
git stash list
git stash show stash@{0}
git stash show -p
```
</details>

### 2.6 Rollback changes<a name="rollback"></a>
![](resources/icons/rollback.png)
<details>
<summary>Command log</summary>

```bash
echo -e "\n\nthis will not fail\n" >> README.md
git commit -am "this will not fail"
echo -e "\n\nbut this one will\n" >> README.md
git commit -am "but this one will"
git logtree
git revert HEAD
git revert <COMMIT_ID HEAD - 2>
git logtree
git checkout main
git revert <COMMIT_ID HEAD - 2>
#### git revert (--continue | --skip | --abort | --quit)
git checkout feature/myFeature
git logtree
git reset --soft HEAD~3
git logtree
git status -vv
git commit -am "this too 2"
git reset --mixed HEAD~1
git logtree
git status -vv
git commit -am "this too 3"
git reset --hard HEAD~1
git logtree
git status -vv
git reset --hard main
git logtree
git reset --hard <COMMIT ID DELETED BRANCH>
git logtree
```
</details>

```mermaid
stateDiagram-v2
    state Revert {
        direction BT
        [*] --> +A
        +A --> +B
        +B --> +C
        +C --> A
        +A --> A: Undo changes\n(new commit)
    }
    state Reset {
        direction BT
        [*] --> Z
        Z --> Y
        Y --> W
        W --> Z: Go back in history\nremoving Y and W
    }
```

```mermaid
sequenceDiagram
    participant w as Working directory
    participant s as Staging area
    participant r as Local/Remote repository
    rect rgb(191, 223, 255)
    note over w,r: git reset --soft
    r->>s: download/delete + stage
    s->>s: remain
    w->>w: remain
    end
    rect rgb(191, 223, 255)
    note over w,r: git reset --mixed
    r->>w: download/delete + unstage
    s->>w: unstage
    w->>w: remain
    end
    rect rgb(191, 223, 255)
    note over w,r: git reset --hard
    r->>w: download/delete
    s--xw: unstage + delete
    w--xw: delete
    note right of w: untracked files will remain
    end
```

### 2.7 Amending<a name="amending"></a>
![](resources/icons/amend.png)
<details>
<summary>Command log</summary>

```bash
echo -e "\nDis is som text\n" >> README.md
git commit -am "Fixing readme"
git logtree
head -n -3 README.md > README.md
echo -e "\nThis is some text\n" >> README.md
git logtree
git commit -am "Fixing readme" --amend
git logtree

git config user.name "Mr. X"
git commit -am "Fixing readme 2" --amend
git log --pretty=fuller -n 2
```
</details>

### 2.8 Cherry picking<a name="cherry-picking"></a>
![](resources/icons/cherrypick.png)
<details>
<summary>Command log</summary>

```bash
git checkout -b feature/myOtherFeature main
git logtree
echo -e "MyClass()\n" > MyClass.scala
git add .
git commit -m "Add class"
echo "Utilities()" > utils.scala
git add utils.scala
git commit -m "add utils"

git checkout -b feature/myOtherFeature2 main
git logtree
echo -e "MyClass2()\n" > MyClass2.scala
git add MyClass2.scala
git commit -m "[+] MyClass2"
git logtree
##### echo -e "MyClass2() extends Utilities\n" > MyClass2.scala

git cherry-pick <COMMIT ID OTHER BRANCH>
#### git cherry-pick (--continue | --skip | --abort | --quit)
git logtree
git log --pretty=fuller -n 2
git config user.name "Eduardo Ruiz"
echo -e "MyClass2() extends Utilities\n" > MyClass2.scala
git commit -am "extends"
```
</details>

```mermaid
flowchart BT
    subgraph master
    A --> B
    end
    B --> f21 & f11
    f21 --> f21'
    subgraph feature1
    f11-->f21'
    end
    subgraph feature2
    f21-->f22
    end
```

### 2.9 Branch integration (merge & rebase)<a name="merge-rebase"></a>
![](resources/icons/branch_compare.png)
<details>
<summary>Command log</summary>

```bash
git checkout feature/myOtherFeature
echo -e "some code here...\n" >> MyClass.scala
git commit -am "some code"
git branch backup/feature/myOtherFeature feature/myOtherFeature
git checkout main
ls -l
git merge --no-ff feature/myOtherFeature
#### git merge (--continue | --skip | --abort | --quit)
git logtree
ls -l
git reset --hard HEAD~1
git logtree

git merge --squash feature/myOtherFeature
git commit -m "merge feature/myOtherFeature -> main"
git logtree
ls -l

git reset --hard HEAD~1
git checkout feature/myOtherFeature2
git rebase main
#### git rebase (--continue | --skip | --abort | --quit)
git logtree				### WATCH! deleted commit
git rebase -i main		### PICK + SQUASH
git logtree
```
</details>

Initial
```mermaid
gitGraph:
options
{
    "nodeSpacing": 100,
    "nodeRadius": 5
}
end
commit
branch newbranch
checkout newbranch
commit
commit
checkout master
commit
commit
```

Merge:
```mermaid
gitGraph:
options
{
    "nodeSpacing": 100,
    "nodeRadius": 5
}
end
commit
branch newbranch
checkout newbranch
commit
commit
checkout master
commit
commit
checkout newbranch
merge master
commit
```

```mermaid
gitGraph:
options
{
    "nodeSpacing": 100,
    "nodeRadius": 5
}
end
commit
commit
commit
branch newbranch
checkout newbranch
commit
commit
commit
```

----
----
## 3. Working with remotes<a name="remotes"></a>
![](resources/icons/github.png)![](resources/icons/gitlab.png)![](resources/icons/bitbucket.png)
In this section we will see how to work in a team using git.

### 3.1 Cloning a remote repository
ℹ️⚡ℹ️
```bash
git clone [-v | --verbose] [-l | --local]
          [--depth <depth>] [--[no-]single-branch] [--no-tags]
          [-o | --origin <name>] [-b | --branch <name>]
          [--] <repository> [<directory>]
```
The [git clone] command copies a repository into a new directory.
This is commonly used at the beginning of a project or at CI/CD[^CI/CD] pipelines.

We will also take a look at [git remote] command, which let us manage links between local and remote repositories.

ℹ️⚡ℹ️
```bash
git remote [-v | --verbose]
git remote add [-t <branch>] [-f] [--[no-]tags] <name> <URL>
git remote rename <old> <new>
git remote remove <name>
git remote set-url --add [--push] <name> <newurl>
git remote set-url --delete [--push] <name> <URL>
git remote [-v | --verbose] show [-n] <name>...
```

Let's clone this very same repo and see the firsts commands we can use while working int a team.

```bash
cd ~/projects
git clone https://github.com/eruizalo/git-learning.git
cd git-learning
# How can I see where I cloned this repo?
git remote
# Let's see a bit more
git remote show origin
```

> ⚠️⚠️ **GitHub support for password authentication was removed on August 13, 2021.**:
> Let's create a Personal Token Access (PAT) following the [official documentation](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/creating-a-personal-access-token)

### 3.1 Working with others 1 (fetching)<a name="fetching"></a>
ℹ️⚡ℹ️
```bash
git fetch [--all] [-v | --verbose]
         [--no-commit] [-e | --edit] [--ff-only]
         [--squash] [--autostash]
         [<repository> [<refspec>...]]
```
```bash
# Speaker will create a new branch & push a new commit
git branch
git branch -a
```



```bash
backup = !f() { git branch --verbose backup/$(git branch --show-current)/$(date +'%Y%m%d_%H%M%S') && echo Backup created: backup/$(git branch --show-current)/$(date +'%Y%m%d_%H%M%S');}; f
git checkout -b backup/$(git rev-parse --abbrev-ref HEAD)
```

> 🎁♻️ **_Cool alias:_**  `git config --global alias.sync = fetch origin main:main`


### 3.2 Working with others 2 (pull-push)<a name="pull-push"></a>
ℹ️⚡ℹ️
```bash
git pull [--tags] [-v | --verbose]
         [--no-commit] [-e | --edit] [--ff-only]
         [--squash] [--autostash]
         [<repository> [<refspec>...]]
```

The _[git pull]_ command is used to sync and download content from a remote repository.
Under the hood, `git pull` is a making the following steps for you:
1. _[git fetch]_: Sync local & remote repo histories scoped to the local branch that `HEAD` is pointed at
2. _[git merge]_ / _[git rebase]_: Git will reconcile the diverging branches (local/remote), if needed

So let's say you have cloned some time ago a repo, you are at main but your team have been working on this repo, but you didn't.
Your main branch will be behind several commits. If you perform a `git pull` command it will fetch + rebase your main branch:
```mermaid
flowchart LR
   A --- B[B,\n main]
   B --- C
   C --- E
   E --- F[F,\n origin/main]
```
After:
```mermaid
flowchart LR
   A --- B
   B --- C
   C --- E
   E --- F[F,\n origin/main,\n main]
   F -.-> |pull|F
```

But, what would have happened if you have been working over your main branch, or you are on another branch, and you want to "sync" it with main
```mermaid
flowchart LR
   A --- B
   B --- C
   B --- D[D,\n main]
   C --- E
   E --- F[F,\n origin/main]
   D --- H
   F -.-> |pull|H
```

<details>
<summary>Command log</summary>

```bash
# --> Download develop changes only if your develop hasn't changed
git pull --ff-only origin main
```
</details>

ℹ️⚡ℹ️
```bash
git push [--tags] [--porcelain] [-v | --verbose]
         [-f | --force] [-d | --delete]
         [<repository> [<refspec>...]]
```


### 3.3 Multiple remotes (forking)<a name="forking"></a>

### 3.4 Force push<a name="force-push"></a>

### 3.5 Tracking your branches<a name="tracking-branches"></a>
```bash
# rename a pushed branch
# push branch 1 (local) to branch 2 (remote)
```

### 3.6 Pull Requests (PRs)<a name="prs"></a>



[^docker]: Docker is an open platform for developing, shipping, and running applications, letting us build an isolated environment, like a virtual machine. This way, all of us will have the same brand-new environment, where we can run our command safely.
[^CI/CD]: Continuous integration (CI) and continuous delivery (CD)

[docker]: https://docs.docker.com/get-docker/
[mermaid]: https://mermaid-js.github.io/

[git config]: https://git-scm.com/docs/git-config
[git init]: https://git-scm.com/docs/git-init
[git commit]: https://git-scm.com/docs/git-commit
[git add]: https://git-scm.com/docs/git-add
[git status]: https://git-scm.com/docs/git-status
[git checkout]: https://git-scm.com/docs/git-checkout
[git rm]: https://git-scm.com/docs/git-rm
[git config alias]: https://git-scm.com/book/en/v2/Git-Basics-Git-Aliases
[git reset]: https://git-scm.com/docs/git-reset
[git diff]: https://git-scm.com/docs/git-diff
[git log]: https://git-scm.com/docs/git-log
[git branch]: https://git-scm.com/docs/git-branch
[git merge]: https://git-scm.com/docs/git-merge
[git tag]: https://git-scm.com/docs/git-tag
[git stash]: https://git-scm.com/docs/git-stash
[git revert]: https://git-scm.com/docs/git-revert
[git cherry-pick]: https://git-scm.com/docs/git-cherry-pick
[git rebase]: https://git-scm.com/docs/git-rebase
[git clone]: https://git-scm.com/docs/git-clone
[git remote]: https://git-scm.com/docs/git-remote
[git fetch]: https://git-scm.com/docs/git-fetch
[git pull]: https://git-scm.com/docs/git-pull
[git push]: https://git-scm.com/docs/git-push
