## 1. Fast-Forward (ff)

  ### Conceptual

  A Fast-Forward occurs when the branch being merged into has no new commits since the feature branch diverged.

  Because there are no diverging changes, Git does not need to combine code or create a merge commit—it simply moves ("fast-forwards") the branch
  pointer ahead to match the target.

    Before:
    A --- B (main)
           \
            C --- D (feature)

    After Fast-Forward:
    A --- B --- C --- D (main, feature)
  ### Script-wise

    # Checkout main and merge feature using fast-forward (default behavior if possible)
    git checkout main
    git merge feature
    
    # Force fast-forward only (fails with an error if a true merge commit is required)
    git merge --ff-only feature

    # Prevent fast-forward (creates a merge commit even if ff is possible, useful to group PR commits)
    git merge --no-ff feature -m "Merge feature branch into main"
  ──────
  ## 2. 3-Way Merge (git merge)

  ### Conceptual

  Used when both branches have diverged: main has progressed, and feature has progressed independently.

  Git finds the common ancestor commit, inspects the diffs from both branches, merges them, and creates a new merge commit that has two parent
  commits.
    Before:
            C --- D (feature)
           /
    A --- B --- E (main)

    After Merge:
            C --- D (feature)
           /       \
    A --- B ------- E --- M (main)

  • Pros: Preserves the exact chronological history and context of when a feature branch existed. Non-destructive (does not rewrite commit IDs).
  • Cons: Branch history can quickly turn into a messy "railway map" of merge commits.

  ### Script-wise

    # 1. Switch to the target branch
    git checkout main

    # 2. Merge the feature branch
    git merge feature
    
    # If there are conflicts:
    # - Git pauses and marks conflict markers (<<<<<<<, =======, >>>>>>>)
    # - Edit the files to resolve conflicts, then stage them:
    git add <conflicted-file>

    # - Complete the merge:
    git commit -m "Merge feature branch and resolve conflicts"

    # - Or abort if something goes wrong:
    git merge --abort
  ──────
  ## 3. Rebase (git rebase)
  ### Conceptual

  Instead of creating a merge commit to join two histories, rebase rewrites history.
  It takes the commits unique to your branch, temporarily sets them aside, moves your branch's base to the latest commit of the target branch, and
  "replays" your commits one by one on top. Each replayed commit gets a new commit hash (C', D').
    Before:
            C --- D (feature)
           /
    A --- B --- E (main)

    After `git rebase main` on feature:
    A --- B --- E (main)
                 \
                  C' --- D' (feature)

  Now, merging feature back into main can be a clean Fast-Forward!

    Final Fast-Forward into main:
    A --- B --- E --- C' --- D' (main, feature)
  • Pros: Completely linear history without cluttering merge commits; makes git log and git bisect clean and easy to navigate.
  • Cons: Rewrites history (changes commit hashes).
  │ Caution
  │ The Golden Rule of Rebasing: Never rebase branches that are public/shared with other collaborators, as rewriting shared commits causes
  conflicts
  │ for everyone else pulling that branch.

  ### Script-wise
  #### Standard Rebase Workflow
    # 1. Make sure your local main is up to date
    git checkout main
    git pull origin main

    # 2. Switch to your feature branch and rebase onto main
    git checkout feature
    git rebase main

    # If conflicts occur during rebase:
    # - Fix the conflict in your editor, then stage it:
    git add <conflicted-file>
    # - Continue to the next commit (do NOT run git commit):
    git rebase --continue
    # - Or cancel the entire operation and return to your original state:
    git rebase --abort

    # 3. Bring main up to date via Fast-Forward
    git checkout main
    git merge --ff-only feature

    # 4. If feature was already pushed to remote, push with lease:
    git push --force-with-lease origin feature

  #### Interactive Rebase (git rebase -i)

  Allows you to clean up, squash, reorder, or edit commits before merging:

    # Rebase the last 3 commits interactively
    git rebase -i HEAD~3

  Git will open your editor with options:

    pick 1a2b3c4 Add login endpoint
    pick 5d6e7f8 Fix typo in login
    pick 9g0h1i2 Add test cases

    # Commands:
    # p, pick  = use commit
    # r, reword = edit commit message
    # e, edit  = stop to amend commit contents
    # s, squash = meld into previous commit and combine commit messages
    # f, fixup  = meld into previous commit, discarding this commit's message
    # d, drop   = remove commit

  Changing pick to fixup on the second commit will merge "Fix typo in login" into "Add login endpoint", keeping history clean.
  ──────
  ## Summary Cheat Sheet

   Feature             │ Fast-Forward (--ff)         │ 3-Way Merge (git merge)                   │ Rebase (git rebase)
  ─────────────────────┼─────────────────────────────┼───────────────────────────────────────────┼─────────────────────────────────────────────────
   New commit created? │ No                          │ Yes (Merge commit M)                      │ Yes (Re-creates commits C', D')
   Rewrites history?   │ No                          │ No                                        │ Yes (New hashes)
   Commit history      │ Linear                      │ Branching / graph                         │ Linear
   Best used for...    │ Simple merges with no       │ Integrating large, shared branches (e.g.  │ Updating a local feature branch with the latest
                       │ divergence                  │ main to staging)                          │ main
