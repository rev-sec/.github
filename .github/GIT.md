# Git Conventions

## Commit Hygiene

------

### Key Times to Commit

* Completion of a logical task: Commit as soon as you finish a single unit of work, such
  as a bug fix, a new function, or a discrete UI change.
* Before starting a new task: This keeps different features or fixes separated in your
  history.
* At stable checkpoints: Commit when your code is in a "working" state (it builds and
  passes initial tests) to create a safe point you can return to if future changes break
  things.
* Before a major refactor: Create a commit before you begin rewriting large sections of
  code so you have a guaranteed fallback if the refactor goes wrong.
* End of a work session: It is good practice to commit (and push to a private branch)
  before ending your day to ensure your progress is backed up. These commits should be
  squashed before merging with any team branches so that there are no "work-in-progress"
  commits.

### Best Practices

* Keep commits atomic: Each commit should focus on one specific change. Avoid
  "mega-commits" that bundle multiple unrelated features or fixes.
* Never commit broken builds to shared branches: While local "work-in-progress" commits
  are fine for your own history, only share code that compiles and passes tests when
  pushing to main or team branches (use squashing to combine WIP commits).
* Write clear commit messages: Follow the 50/72 rule—a short summary of up to 50
  characters, followed by a detailed body if necessary.
* Use the staging area: Use the git add command to selectively pick which changes belong
  in your next commit.
* Commit locally, push selectively: In distributed systems like Git, you can commit to
  your local machine as often as every few minutes, but only push to the remote server
  when you are ready to share or back up your work.

## Branching

------

The table below is a quick reference for the different types of branches. This is
general and some types may not be applicable to all repositories.

| Type          | Naming                         | Use                                          | Notes                                                         |
|:--------------|:-------------------------------|:---------------------------------------------|:--------------------------------------------------------------|
| Release       | `stable`                       | The latest code deployed to production       | Accepts merges from `main` and hotfixes                       |
| Working       | `main`                         | The main working branch for core development | Accepts merges from all branches except hotfixes and `stable` |
| Feature       | `feature/<tid>/<feature-name>` | New features and enhancements                | Always branch from/merge to the tip of `main`                 |
| Bugfix        | `bugfix/<tid>/<bug-name>`      | Fixing bugs in the code                      | Always branch from/merge to the tip of `main`                 |
| Hotfix        | `hotfix/<tid>/<bug-name>`      | Fixing urgent bugs in production             | Always branch from/merge to a release in `stable`             |
| Refactor      | `refactor/<tid>/<description>` | Improving code without affecting function    | Always branch from/merge to the tip of `main`                 |
| Test          | `test/<tid>/<test-name>`       | Writing/improving automated tests            | Always branch from/merge to the tip of `main`                 |
| Documentation | `doc/<tid>/<description>`      | Documentation updates                        | Always branch from/merge to the tip of `main`                 |

Where:

* `<tid>`: represents the issue number if present. If not present that that section
  should be removed.
* `<feature-name>`/`<bug-name>`/`<test-name>`: represents a short descriptive name for a
  feature, bug, or test with words separated by hyphens (`-`) instead of spaces.
* `<description>` represents a description of the refactor or documentation. Words are
  separated by hyphens (`-`) instead of spaces and should be as short as possible.

## Main Branches

The repository will always hold two evergreen branches:

* `main`
* `stable`

The main branch should be considered `origin/main` and will be the main branch where the
source code of `HEAD` always reflects a state with the latest delivered development
changes for the next release. As a developer, you will be branching and merging from
`main`.

**Merges to `main` and `stable` are initiated via pull requests only**!

Consider `origin/stable` to always represent the latest code deployed to production.
During day to day development, the `stable` branch will not be interacted with.

When the source code in the `main` branch is stable and has been deployed, all the
changes will be merged into `stable` and tagged with a release number.

## Feature Branches

Feature branches are used when developing a new feature or enhancement which has the
potential of a development lifespan longer than a single deployment. When starting
development, the deployment in which this feature will be released may not be known. No
matter when the feature branch will be finished, it will always be merged back into the
`main` branch via a pull request.

During the lifespan of the feature development, the lead should watch the `main` branch
to see if there have been commits since the feature was branched. Any and all changes to
`main` should be merged into the feature before merging back to `main`; this can be done
at various times during the project or at the end, but time to handle merge conflicts
should be accounted for. Local incremental commits must be squashed before synchronising
with the remote repository as no work-in-progress commits are allowed in the repository.

Feature branches must:

* Branch from: `main`
* Merge back into: `main`

## Bug Branches

Bug branches differ from feature branches only semantically. Bug branches will be
created when there is a bug in a release that should be fixed and merged into the next
deployment. For that reason, a bug branch typically will not last longer than one
deployment cycle. Additionally, bug branches are used to explicitly track the difference
between bug development and feature development. No matter when the bug branch will be
finished, it will always be merged back into `main`.

Although likelihood will be less, during the lifespan of the bug development, the lead
should watch the `main` branch to see if there have been commits since the bug was
branched. Any and all changes to `main` should be merged into the bug before merging
back to `main`; this can be done at various times during the project or at the end, but
time to handle merge conflicts should be accounted for. Local incremental commits must
be squashed before synchronising with the remote repository as no work-in-progress
commits are allowed in the repository.

Bug branches must:

* Branch from: `main`
* Merge back into: `main`

## Hotfix Branches

A hotfix branch comes from the need to act immediately upon an undesired state of a live
production version. Additionally, because of the urgency, a hotfix is not required to be
pushed during a scheduled deployment. Due to these requirements, a hotfix branch is
always branched from a tagged stable branch. This is done for two reasons:

* Development on the `main` branch can continue while the hotfix is being addressed.
* A tagged stable branch still represents what is in production. At the point in time
  where a hotfix is needed, there could have been multiple commits to `main` which would
  then no longer represent production.

Hotfix branches must:

* Branch from: tagged `stable`
* Merge back into: `main` and `stable`
