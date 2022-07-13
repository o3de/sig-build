## Meeting Details

- **Date/Time:** July 13, 2022 @ 05:00 pm UTC / 10:00 am PDT
- **Location:** [Discord SIG-Build Voice Room](https://discord.gg/wDNAQmatpq)
- **Moderator:** Mike Chang
- **Note Taker** Brian Herrera

The [SIG-Build Meetings](https://github.com/o3de/sig-build/tree/main/meetings) repo contains the history past calls, including a link to the agenda, recording, notes, and resources.

## SIG Updates

**What happened since the last meeting?**

No updates

## Meeting Agenda


### support in the build system for VS2022: https://github.com/o3de/o3de/issues/6707
- Build node created with VS2022 installed. Currently testing in a branch.
- There is one issue with the filter in the cmake detection. This will need to be updated. 


### Validation step in front of the parallel build pipeline
- The job was updated to run on a Linux node
- Discovered that the unmount step on the linux nodes were pointing to the wrong folder causing a timeout. A change will also be madeo the to the incremental build script.
- The update will add about 4-5min time to AR runtime.


### Build Failure Analysis improvements
- Shirang is currently rewriting the CDK packages on his desktop due to an issue encountered on his laptop. 
- Will also start work on the Lambda functions. 


###  New o3de-extras and canonical.o3de.org repo and workflow
- New repos created
  - https://github.com/o3de/o3de-extras
  - https://github.com/o3de/canonical.o3de.org
- The canonical repo will store the gem repo info and the licenses files.
- The installer will be updated to point to this repo.


### DCO check PoC
- The current DCO check app went down recently. This requires a force push to kick off a rescan.
- The proposed DCO check allows a fixup commit to sign previous unsigned commits. Mike will follow up with Royal to determine if this is acceptable.
- This will replace current DCO scanner.


## Other topics for discussion

- Static code analysis
  - Currently evaluating CodeQL
  - A timeout error is being encountered in the current runs. Also running into the 60GB storage limit on the hosted runners. 
  - An workaround for these limitations would include using self-hosted runners or using jenkins to run the job. There is also the option to clean up the storage during the pre-build step.
  - Only a minimal build is required to run the analysis and may not have to run all profiles.
- Installer ownership: https://github.com/o3de/community/pull/136
  - Mike commented to accept ownership for installer generation workflows. Other components (UI and workflow) TBD.
  - No response yet.

## Outcomes from Agenda



#### SIG reviewers and maintainers groups
- @brianherrera 
- @amzn-changml 

## Action Items

- Chairs: No action
- Maintainers/Reviewers: As noted in the agenda outcome

## Open Discussion Items

Triage backlog grooming: https://github.com/o3de/o3de/issues?q=is%3Aissue+is%3Aopen+label%3Asig%2Fbuild+label%3Aneeds-triage+no%3Aassignee

List any additional items below!
