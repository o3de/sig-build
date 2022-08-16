## Meeting Details

- **Date/Time:** Aug 10, 2022 @ 05:00 pm UTC / 10:00 am PDT
- **Location:** [Discord SIG-Build Voice Room](https://discord.gg/wDNAQmatpq)
- **Moderator:** Deepak Sharma
- **Note Taker** Mike Chang

The [SIG-Build Meetings](https://github.com/o3de/sig-build/tree/main/meetings) repo contains the history past calls, including a link to the agenda, recording, notes, and resources.

## SIG Updates

**What happened since the last meeting?**

See Agenda notes

## Meeting Agenda

Discuss agenda from proposed topics:

- Adding support in the build system for VS2022: https://github.com/o3de/o3de/issues/6707
- Increasing CMake version: https://github.com/o3de/o3de/issues/10939
- Increased Build Times RCA
- Build Failure Analysis improvements - Build logs are available from failed state, improvements are done, new RFC is required for the reporting.
- DCO check PoC - In Progress.

## Agenda notes

Mike: VS2022 AMI have been published and tested in nightly. They have failed nightly, but we will create bug tickets if they are truly an issue with the VS2022 compiler
Mike: In addition, the CMake update will be required to generate VS2022 solutions, which will be pushed out shortly.
Mike: We will also publish a proposal for platform support and length of time for support, to be reviewed by TSC
dshmz: When will that be ready?
Mike: By next meeting (8-18), but I won't be available to present it that day
dshmz: Let's postpone that for the week after, as I also won't be available next week
dshmz: Any updates on the Linux build time RCA?
Mike: I've collected some data in a private doc, but will have some internal review done before publishing this in the SIG-Build repo. Some fixes for NTP time syncs are pushed out in an experimetal branch last night.
dshmz: Any updates on the build failure analysis improvements?
Shirang: The changes for BFA have been pushed out and are currently displaying results in the build results. Will have a RFC for additional changes
dshmz: Any updates on the DCO check PoC?
Mike: No current progress, VS2022 and build time investigations are taking priority

### Add installers as a responsibility of SIG-build - https://github.com/o3de/community/pull/136
- No response yet on which SIG will own the installer experience. 

## Other topics for discussion

No topics put up for discussion

## Outcomes from Agenda

Mike - Create proposal for TSC on platform support

#### SIG reviewers and maintainers groups
- @brianherrera 
- @amzn-changml 

## Action Items

- Chairs: No action
- Maintainers/Reviewers: As noted in the agenda outcome

## Open Discussion Items

Triage backlog grooming: https://github.com/o3de/o3de/issues?q=is%3Aissue+is%3Aopen+label%3Asig%2Fbuild+label%3Aneeds-triage+no%3Aassignee