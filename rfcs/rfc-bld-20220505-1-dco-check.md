## Summary:

The purpose of this feature is to provide a scriptable way to perform a DCO (Developer Certificate of Origin) check on PRs, branch pushes, and local environments. The O3DE organization currently checks all commits for sign-off to ensure legal ownership of the contributions.

To accomplish this, this proposal is looking to use a fork of the dco-check script: [https://github.com/christophebedard/dco-check](https://github.com/christophebedard/dco-check) and use Github Actions to automate its use on PRs and branch pushes. This will differ from the current implementation, which uses a Github App (https://github.com/dcoapp/app)

The goal is to provide a visible and controllable environment for the DCO check to be performed and modified. Currently, the DCO Github App's ownership has been in flux, and has not taken in new PRs in months. In addition, there have been numerous outages of the App, which is using backend infrastructure not controlled by O3DE or Github. This has blocked PRs during critical release schedules.

By creating a fork of the existing script, the O3DE community can increase contributor velocity and bring in features/fixes that can be pushed upstream. The O3DE community would own the implementation of the automation and business logic.

The release of this feature will not change the behavior of the PR process, but it will allow the O3DE community to maintain and contribute changes to the DCO setup.

## What is the relevance of this feature?

This is important because it allows developers to contribute changes to the DCO check process and enables the following:

1.  Transfer the ownership of the workflow and codebase of the DCO check to the Build SIG and upstream maintainer.
2.  Migrates the automation and infrastructure to Github Actions, a tier-1 service.
2.  Allow other developers to fork this repo and easily setup the DCO check to run on their own project.
3.  Allow developers to run this the DCO check script locally as a pre-check.
4.  Allow the DCO check to be enabled on a per-push basis on every branch, to provide an early warning system before too many commits make the rebase process complicated.

## Feature design description:

All the scripts and configs required to setup DCO checks will be packaged in a repo. A Github Action will automatically deploy the required resources when updates are merged in. This is configured inside of the .github folder in every repo.

The repo has additional Github actions that will be used to build, test, and package the scripts.

In addition, the script can be used to gate PRs and have a means to "fix-up" the PR by use of a empty commit with a message that signs-off for the current history.

Use cases:

- **Enable the use of DCO check in a repo**: a repo administrator will add a `dco.yaml` file to `.github/workflow`. Github will automatically trigger Github Actions to consume this configuration in the repo. The output of a check will signal to the user if the commits in the branch is signed. If used as a branch protection mechanism, the Github Action will trigger with every PR and block merges if used as a check criteria.
- **Update the DCO check setup**: a user will clone the repo and make changes (e.g. install a new dependent library, update the python version, etc.). Then they can test these changes locally by running the python script on their local machine. After testing, the user can then submit a PR with their changes. After the changes are reviewed and merged, the updates will be available to all branches and PRs.  
- **Use the DCO check on other projects**: a user will fork the repo and make any customization they need to the Github Action and repo setup. Using this configuration they can easily setup the DCO check to run on their project.

## Technical design description:

### DCO check components:

-   DCO check repo: A repo containing all scripts, tests, and automations for the dco\_check.py script. Forked from [https://github.com/christophebedard/dco-check](https://github.com/christophebedard/dco-check)
-   `dco\_check.py`: A script with minimal dependancies contains all the business logic to validate and check commits
-   tests/: Scripts using pytest that will validate dco\_check.py. Automatically runs on PR and push via Github Actions.

### Github Action components:

-   `dco.yaml`: YAML file that configures the flow control, steps, environment (OS), and dependencies (Python, repo location). Committed to the .github/workflows folder of any repo.

### Deployment workflow:

1.  User submits a pull request to change one of the config files or scripts (e.g. dco\_check.py)
2.  The pull request triggers Github Action to test the commit
3.  If the pull request passes tests, the PR can be merged into the main branch

### DCO Check workflow

1.  User commits in a DCO controlled repo branch (whether it is primary or fork)
2.  A Github Action is kicked off to scan the commit on push
3.  If the check succeeds, no action
4.  If it fails, the Github Action will exit a failure code
    -   In the error message, a message is printed with instructions to fix the issue
    -   In addition, an email is sent to the committer, linking them directly to the failed run

### DCO Check fix workflow

1.  On a DCO check failure, the user can do the following:
    -   Do a Git rebase and sign each unsigned/invalid commit
    -   Issue an empty "fix-up" commit with a key phrase in the commit message, such as: `I sign-off on all past commits`
2.  If a fix-up commit is created, the dco\_check.py script will need an argument to accept fix-ups. This can be enabled by default
    -   With a fix-up commit, all past commits within the branch are not scanned, as the fix-up will stand in for those commits. Future commits after the fixup commit are still scanned and flagged, however.

### Testing Deployments

Here are the options to test changes before they are deployed to production:

1.  Locally: Users can clone the repo and run their changes for dco\_check.py on another repo
2.  Branches: The Github Action pipeline will trigger on any push to the repo
3.  PR checks: When submitting a PR to the repo, we will be running pytests on the code to validate before merging the code

### Rollback

In the event that issues are identified in PR or per branch checks, the Github Action configured can pull from the previous version. Otherwise, changes can be rolled back in the DCO check repo

These issues should be identified with user reporting, as long as merge tests pass correctly.

### Deployment Permissions

For simplicity, a PR is used to promote changes from development to main.

Additionally, only certain teams will be able to merge into the .github/workflows folder in the repo, such as SIG-Build and administrators.

## What are the advantages of the feature?

-   Makes it easier to maintain the DCO check scripts with involvement from contributors
-   Makes it easier for developers to run the same DCO check on their own O3DE project
-   All config changes to the DCO check will be version controlled and will be reviewed through PRs
-   Github infrastructure is used end-to-end, instead of outside infrastructure

## What are the disadvantages of the feature?

-   Each config change could impact DCO checks for PRs in progress. Notifications to all contributors will be needed to maintain visibility.
-   This will be an opinionated script and workflow for O3DE. It may not work for all developers, however they will be able fork the repo and customize as needed.

## Are there any alternatives to this feature?

An alternative is to continue to use the Github DCO App or utilize another script. Most solutions investigated are based in Javascript, which may impact maintainability.

## How will users learn this feature?

-   The DCO check script has a upstream maintainer, which has documentation on how to modify and deploy the scripts
-   Our contribution docs will also be updated to reference this setup.

## Are there any open questions?

-   Will this repo also be used to deploy config updates to production and how will it be protected?
    -   Yes, a separate branch from `main` will be created for our production deployments and branch protection will be enabled to require approvals and successful checks.
-   Will the fix-up sign-off allow someone to sign-off on behalf or take credit of others?
    -   There is potential to take credit for others that did not sign-off. We will need policy and training for the maintainers to manage these situations, such as instances where the un-signed committer is not available.
-   Can this configuration be disabled?
    -   Yes, the configuration can disabled in the dco.yaml file on a per branch basis. We will be putting this into a .gitignore file to prevent changes from flowing into the production branch.
    -   Notifications can also be disabled through the user's Github profile, though it is global.
-   What if Github Actions stops working?
    -   We can utilize manual checks or use self-hosted runners to perform the same check
    -   Worst case, the action can be disabled or rolled back to using the Github DCO app