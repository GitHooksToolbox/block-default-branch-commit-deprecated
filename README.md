<p align="center">
    <a href="https://github.com/GitToolbox/">
        <img src="https://cdn.wolfsoftware.com/assets/images/github/organisations/gittoolbox/black-and-white-circle-256.png" alt="GitToolbox logo" />
    </a>
    <br />
    <a href="https://github.com/GitToolbox/block-default-branch-commit/actions/workflows/pipeline.yml">
        <img src="https://img.shields.io/github/workflow/status/GitToolbox/block-default-branch-commit/pipeline/master?style=for-the-badge" alt="Github Build Status">
    </a>
    <a href="https://github.com/GitToolbox/block-default-branch-commit/releases/latest">
        <img src="https://img.shields.io/github/v/release/GitToolbox/block-default-branch-commit?color=blue&label=Latest%20Release&style=for-the-badge" alt="Release">
    </a>
    <a href="https://github.com/GitToolbox/block-default-branch-commit/releases/latest">
        <img src="https://img.shields.io/github/commits-since/GitToolbox/block-default-branch-commit/latest.svg?color=blue&style=for-the-badge" alt="Commits since release">
    </a>
    <br />
    <a href=".github/CODE_OF_CONDUCT.md">
        <img src="https://img.shields.io/badge/Code%20of%20Conduct-blue?style=for-the-badge" />
    </a>
    <a href=".github/CONTRIBUTING.md">
        <img src="https://img.shields.io/badge/Contributing-blue?style=for-the-badge" />
    </a>
    <a href=".github/SECURITY.md">
        <img src="https://img.shields.io/badge/Report%20Security%20Concern-blue?style=for-the-badge" />
    </a>
    <a href="https://github.com/GitToolbox/block-default-branch-commit/issues">
        <img src="https://img.shields.io/badge/Get%20Support-blue?style=for-the-badge" />
    </a>
    <br />
    <a href="https://wolfsoftware.com/">
        <img src="https://img.shields.io/badge/Created%20by%20Wolf%20Software-blue?style=for-the-badge" />
    </a>
</p>

## Overview

No matter which style of git workflow you use, it is generally (but not by everyone) agreed that committing directly into the `default branch` is a bad idea.

> We use the term `default branch` to mean any branch you have configured as your default, this is often `master` or `main` but can be configured on a repo by repo basis. 

Some of the reasons for this include (and there are many many more):

* If you push a work-in-progress state to remote, the default branch is potentially broken.
* If another developer starts work for a new feature from the default branch, they start with a potentially broken state. This slows down development.
* Different features/bug-fixes are not isolated, so that the complexity of all ongoing development tasks is combined in one branch. This increases the amount of communication necessary between all developers.
* You cannot do pull requests which are very good mechanism for code reviews.
* You cannot squash commits/change git history in general, as other developers might already have pulled the `default branch` in the meantime.

## What can we do ?

There are 2 main solutions that can be used to attempt to solve this problem. There are others but these are the most common.

1. Protect the branch - This is a server side solution and requires no involvement from the end user.
2. Protect the commit - This is a client side solution and required a small amount of setup work from the end user.

### Protect the branch

This solution works by making it is impossible to push directly to the branch. In Github (and Gitlab and others) you can give a branch `protected` status. This has the effect of stopping anyone from pushing to it. Some of the basic pros and cons for this approach are:

**Pros:**

* Stop any changes being made directly to the branch
* No end user interaction required
* Works across repeated clones

**Cons:**

* All or nothing solution
* Only stops the push not the local commit (requires user to unpick the local issues)

### Protect the commit

This solution works by stopping the user from committing code locally to the branch rendering a push the branch pointless as there are no local changes to push. This is most often achieved with the use of pre-commit hooks.

#### What is a pre-commit hook?

A pre-commit hook is run first, before you even type in a commit message. It's used to inspect the snapshot that's about to be committed, to see if you've forgotten something, to make sure tests run, or to examine whatever you need to inspect in the code.

By using a pre-commit hook you can inspect the branch the user is attempting to commit to and take any required actions. Some of the basic pros and cons for this approach are:

**Pros:**

* Stop any changes being committed to the local branch
* Doesn't require any server side work

**Cons:**

* Has to be configured for each new clone

### Block or not?

When it comes to local branch protection there are 2 main ways to approach the problem:

1. Block - Simply block the commit (This solution).
2. Prompt - Warn the user they are trying to commit and prompt for confirmation (see our [prompt-default-branch-commit](https://github.com/GitToolbox/prompt-default-branch-commit) repo for this solution).

### Blocking the commit

This solution implements a complete block when the user attempts to commit changes to the local `default branch`.

#### Example

```shell
# git commit -m "Some nice commit message"
You cannot commit to the default branch - Aborting!
```

### Installing the hook

Copy the [script](src/block-default-branch-commit) to the .git/hooks/pre-commit at the root of the desired repository (and ensure that it is executable [*chmod +x]*). At this point the hook will run (fire) every time you attempt to commit a change to the repository.

> The name pre-commit is a special name used internally by git so must be the name used when you copy the script into the repository.

#### Root is where?

If you are not sure where the root of the repository is, then execute the following from within any directory in your repository.

```shell
r=$(git rev-parse --git-dir) && r=$(cd "$r" && pwd)/ && cd "${r%%/.git/*}" && pwd
```

The above will give you the full path to the root of your repository no matter which directory you are in (even inside the .git directory or any of its subdirectories).

If you are sure you are in just a normal directory you could issue the following command instead. It is easier to remember but isn't going to work 100% of the time.

```shell
git rev-parse --show-toplevel
```

> If you see something similar to **fatal: this operation must be run in a work tree** then try the first command instead.

### Changing the defaults

By default this script uses `master` as the default branch name, but this can be changed in one of 2 ways.

1. Change the name in the global variable [script](https://github.com/GitToolbox/block-default-branch-commit/blob/master/src/block-default-branch-commit#L14).
2. Turn on automatic branch identification. [script](https://github.com/GitToolbox/block-default-branch-commit/blob/master/src/block-default-branch-commit#L9).

> Automatic branch identification does result in a short pause while it identifies the default branch from the remote end.

## Running multiple scripts

A pre-commit (and all other git hooks) hook can only be a single script, so if you want to run multiple different scripts as part of a 'pipeline' then you need to make use of our [Git Hook Multiplexer](https://github.com/GitToolbox/git-hook-multiplexer).

