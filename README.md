# gh-qstack - GitHub CLI Quick Stack

Love the [gh stack](https://github.com/github/gh-stack) CLI but want to be quick? `gh-qstack` to the rescue for a single command `gh qstack` that:
* initializes a GitHub Stack
* automatically picks a branch name
* checks out that branch in the stack
* adds all changes
* commits them given your message, which also becomes the PR title
* pushes
* starts submit GH PR flow

Now when you want to go back to an existing layer to update something? `gh qstack fixup`
* adds all changes
* uses `git history fixup` to update the current commit and upstream branches
* pushes all updated branches

A few lines of bash is all you need [gh-qstack](/gh-qstack)!

## Commands

1. `gh qstack "message"` for adding a new layer with the init, branch name, checkout, add, commit, push, and submit.
2. `gh qstack f` or `gh qstack fixup` for modifying an existing layer and fixing things up the stack based on the recent [git history fixup](https://lalitm.com/post/git-history/). Note, this needs `git` version 2.55 or later.
3. `gh qstack --help` for getting help text on commands and their flag

There's also flags if you want to skip submitting and pushing
* `--no-submit` - will not run `gh stack submit` on `gh qstack`
* `--no-push` - will not run `gh stack push` on `gh qstack` and `gh qstack fixup`. Note: Since push is disabled, submit won't be run either

If you want to save even more keystrokes, try out these aliases
```sh
$ gh alias set qs 'qstack'
$ gh alias set qsf 'qstack fixup'
```
## Install

Quite similar to other `gh` CLI [extensions](https://cli.github.com/manual/gh_extension_install):
```sh
$ gh extension install harsh183/gh-qstack
```

Whenever there's an update, `gh` cli will tell you automatically after using `qstack`:
```sh
A new release of qstack is available: 8603e16354eacd7ca77aa658b113bcc80a8dbe6d → 08d6f3689d3793e0cb8727da96947947af651102
To upgrade, run: gh extension upgrade qstack
git@github.com:harsh183/gh-qstack.git
```

## Requirements

1. [gh cli](https://cli.github.com/) as recent as you can. The [gh-stack](https://github.com/github/gh-stack) official docs say it should be v2+
2. [git](https://git-scm.com/install/) as recent as can if you want the `gh qstack fixup` command to work since `git history fixup` was introduced recently in `v2.55` where it's still experimental and it will get fixes over time. A lot of system git versions are actually fairly old, so make sure to upgrade. If you don't want `fixup`, an older `git` version should probably work.

## Example

Based off dog fooding this very project:

1. When there's no stack because you are on your default trunk branch, here `main`. Run `gh qstack "message"`:
```sh
~/E/gh-qstack (main|●1✚1) $ vim gh-qstack
~/E/gh-qstack (main|●1✚1) $ gh qstack "V1 of the script"
✓ Created stack: main ← quick/20260827100519
  You're on quick/20260827100519 (top of stack).

What's next:
  • commit your work as usual, then add a layer:  gh stack add
  • see your stack any time:                      gh stack view
  • when ready to open PRs:                       gh stack submit
[quick/20260827100519 492c191] V1 of the script
 1 file changed, 44 insertions(+)
 create mode 100755 gh-qstack
Pushing 1 branch to origin...
✓ Pushed 1 branches
To create PRs for this stack, run `gh stack submit`
Checking stack state...
Pushing to origin...
✓ Created PR #1 for quick/20260827100519
✓ Pushed and synced 1 branches
~/E/gh-qstack (quick/20260827100519|✔) $
```

This made [PR 1](https://github.com/harsh183/gh-qstack/pull/1)

2. Now do some more changes, then it's simply `gh-qstack` again. Here we're also showing the `--no-submit` param in case you don't want to make a PR right away.
```sh
~/E/gh-qstack (quick/20260827100519|✔) $ vim gh-qstack
~/E/gh-qstack (quick/20260827100519|✚1) $ gh qstack "Change Default Branch Name" --no-submit
✓ Created branch 08-27-change_default_branch_name (layer 2) with commit 64bdefd8ee55b4962159d97f3820a07deae211df
Pushing 2 branches to origin...
✓ Pushed 2 branches
To create PRs for this stack, run `gh stack submit`
~/E/gh-qstack (08-27-change_default_branch_name|✔) $
```

3. Wait, I want to change something in a given layer. Lets use the `fixup` flow using `gh qstack f`
```sh
~/E/gh-qstack (08-27-change_default_branch_name|✔) $ vim gh-qstack
~/E/gh-qstack (08-27-change_default_branch_name|✚1) $ gh qstack f
Pushing 2 branches to origin...
✓ Pushed 2 branches
To create PRs for this stack, run `gh stack submit`

```

4. Going on, adding yet another layer where I'll submit my current and previous PR.
```sh
~/E/gh-qstack (08-27-change_default_branch_name|✚1) $ gh qs "Flesh out README"
✓ Created branch 08-27-flesh_out_readme (layer 3) with commit 39edf804a703f1d83793caa7496b48a32b9a33a9
Pushing 3 branches to origin...
✓ Pushed 3 branches
To create PRs for this stack, run `gh stack submit`
Checking stack state...
Pushing to origin...
PR #1 for quick/20260827100519 is up to date
✓ Created PR #2 for 08-27-change_default_branch_name
✓ Created PR #3 for 08-27-flesh_out_readme
✓ Stack created on GitHub with 3 PRs (stack #4)
✓ Pushed and synced 3 branches
~/E/gh-qstack (08-27-flesh_out_readme|✚1) $
```

This made [PR2](https://github.com/harsh183/gh-qstack/pull/2) and [PR3](https://github.com/harsh183/gh-qstack/pull/3)

5. Actually I want to go back and change the default branch again. Let's do that
```sh
~/E/gh-qstack (08-27-flesh_out_readme|✔) $ gs down
✓ Checked out 08-27-change_default_branch_name, 1 branch down
~/E/gh-qstack (08-27-change_default_branch_name|✔) $ vim gh-qstack
~/E/gh-qstack (08-27-change_default_branch_name|✚1) $ gh qsf
Pushing 3 branches to origin...
✓ Pushed 3 branches
Run `gh stack view` to see your stack of PRs
```
Now this updated [PR2](https://github.com/harsh183/gh-qstack/pull/2), and you can actually see the result of the force push in the PR page with the [diff](https://github.com/harsh183/gh-qstack/compare/4d7f0a5cfae7551c086b600019082bb8776d7f96..bd03c066b4360af410a66e0aa1dabd27836fd7f8)

6. Okay now I want to add everything I just did and wrap up this README
```sh
~/E/gh-qstack (08-27-flesh_out_readme|✚2) $ gh qs "Finish README" --no-push
✓ Created branch 08-27-finish_readme (layer 4) with commit 0575e77aaf6fbc72eeb896f7c7bb07adff375c67
~/E/gh-qstack (08-27-finish_readme|✔) $ gs submit
Checking stack state...
Pushing to origin...
PR #1 for quick/20260827100519 is up to date
PR #2 for 08-27-change_default_branch_name is up to date
PR #3 for 08-27-flesh_out_readme is up to date
✓ Created PR #5 for 08-27-finish_readme
✓ Stack updated on GitHub with 4 PRs (stack #4)
✓ Pushed and synced 4 branches
~/E/gh-qstack (08-27-finish_readme|✔) $
```

This made [PR4](https://github.com/harsh183/gh-qstack/pull/5)

here's what your stack looks like after more dogfooding in the UI
<img width="904" height="470" alt="image" src="https://github.com/user-attachments/assets/03a0eb62-7c4d-4570-96d2-01b425ce50a2" />

## Why This Exists

Now `gh stack` has a [Abbreviated workflow](https://github.com/github/gh-stack#abbreviated-workflow) that uses `gs add -Am "commit message"` that covers branch name, checkout, add, commit. I've had a good time using it, and I suggest starting with that. But I was missing
* having a similar flow on `gs init`
* pushing and making a PR right away

I had achieved similar UX with my earlier project [gh-chain](https://github.com/harsh183/gh-chain/) which had a similar flow before GitHub's official stacking feature was available.
