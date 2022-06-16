## Meeting Details

- **Date/Time:** June 15, 2022 @ 05:00 pm UTC / 10:00 am PDT
- **Location:** [Discord SIG-Build Voice Room](https://discord.gg/wDNAQmatpq)
- **Moderator:** Deepak Sharma
- **Note Taker** Mike Chang

The [SIG-Build Meetings](https://github.com/o3de/sig-build/tree/main/meetings) repo contains the history past calls, including a link to the agenda, recording, notes, and resources.

## SIG Updates

**What happened since the last meeting?**

- Migrated nightly pipeline to O3DE.

## Meeting Agenda

- Discuss agenda from proposed topics
  - Migrated nightly pipeline: https://github.com/o3de/sig-build/issues/34
  - Intermittent Tests RFC needs a review from Build-SIG - https://github.com/o3de/sig-testing/issues/43
  - Proposed RFC - GitHub PR Workflow - https://github.com/o3de/sig-build/issues/44
  - Triage the new issues in the queue - https://github.com/o3de/o3de/issues?q=is%3Aissue+is%3Aopen+label%3Asig%2Fbuild+label%3Aneeds-triage+no%3Aassignee. 

## Agenda notes

### Migrated nightly pipeline: https://github.com/o3de/sig-build/issues/34
dshmz: Status on the nightly builds
shirang: A set of the nightly jobs have been migrated. Next steps are to add ios and mac jobs, but at a reduced build cadance
brian: There's a GHI on the mac and ios blockers

### Intermittent Tests RFC needs a review from Build-SIG - https://github.com/o3de/sig-testing/issues/43
mike: The codeowners work is already being investigated by shirang
shirang: Correct, I'm working looking at assigning failures by codeowners, and another modification to look at the asset to see if there's an owner
mike: Is there a github issue for this work?
shirang: No, but I will make one now
shirang: There's also an ask from sig-testing to run a periodic build. The failure rate for rapid multipule runs will influence the overall average.
dshmz: Work with sean to discuss the rate of failure is a concern

### Proposed RFC - GitHub PR Workflow - https://github.com/o3de/sig-build/issues/44

dweiss: Question about rate limiting, what methods do we have to handle failures? How do we handle situations where commands are failing to be processed?
mike: The fallback is to go to the Jenkins job and run it manually
shirang: For rate limiting, we could just check if there's a build already in progress and not process any further commands until it is done. We can also throw a comment that the command was rejected
mike: That's a good point about rejection commands. We should have some sort of response to the commands, but maybe not successes
brian: Yes, I was looking to reduce the amount of noise, so I was looking into just the failure response, but leave successes evident in the check
mike: The one issue with rejecting commands on a current build is situations where a current build is having intermittent failures. Someone could be checking in code while a build is happening and wants to queue another one

## Outcomes from Agenda

Brian - Followup on the GHI on mac and ios builds for nightly with shirang
Shirang - Comment on the RFC for intermittent tests on any existing work from sig-build and reach out to sean on periodic build failure concerns
Brian - Update Github PR RFC following feedback

#### SIG reviewers and maintainers groups
- @brianherrera 
- @amzn-changml 

## Action Items

- Chairs: No action
- Maintainers/Reviewers: As noted in the agenda outcome

## Open Discussion Items

List any additional items below!
