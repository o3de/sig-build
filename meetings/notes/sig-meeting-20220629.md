## Meeting Details

- **Date/Time:** June 29, 2022 @ 05:00 pm UTC / 10:00 am PDT
- **Location:** [Discord SIG-Build Voice Room](https://discord.gg/wDNAQmatpq)
- **Moderator:** Mike Chang
- **Note Taker** Brian Herrera

The [SIG-Build Meetings](https://github.com/o3de/sig-build/tree/main/meetings) repo contains the history past calls, including a link to the agenda, recording, notes, and resources.

## SIG Updates

**What happened since the last meeting?**

- GHI created for iOS/Mac blockers for the daily periodic pipeline metrics: https://github.com/o3de/o3de/issues/9428
- Updates from sig-testing regarding the periodic pipeline work:
  - shirang: No additional pipeline setup required. We will run the metrics scripts on the existing pipe.
  - There is some overlap between sig-testing's RFC, however they will remain separate
- GitHub workflow RFC: RFC will be merged in today if there are no other comments

## Meeting Agenda

Discuss agenda from proposed topics:

- Build failure RCA improvement RFC - https://github.com/o3de/sig-build/pull/48
- Intermittent Tests RFC needs a review from Build-SIG - https://github.com/o3de/sig-testing/issues/43
- Proposed RFC - GitHub PR Workflow https://github.com/o3de/sig-build/issues/44 


## Agenda notes

### Build failure RCA improvement RFC - https://github.com/o3de/sig-build/pull/48
- Discussed any additional feedback
- There were concerns that Shirang addressed regarding solutions for rate limiting in the event there is a spike in SNS messages sent to the topic. An SQS queue was added for rate limiting.

### Proposed RFC - GitHub PR Workflow https://github.com/o3de/sig-build/issues/44 
- No other comments. PR to be merged in today. 


### Fail fast Option - https://github.com/o3de/o3de/pull/10415
- Mike: PR opened for fail fast option for AR. This will not work on the first run and will leave the fail fast option off as the default for now. 
- Question about how this will affect metrics and what we need to provide to the metrics team
  - Brian: Metrics team just needs access to the fail fast parameter for each run so they can seprarate that in the metrics
  - Mike: All parameters should be provided in the SNS message. Will need to check. 

### Add installers as a responsibility of SIG-build - https://github.com/o3de/community/pull/136
- No response yet on which SIG will own the installer experience. 


### Dependabot updates made recently for python dependency CVEs
-  We addressed 3 CVE items in the past week.
-  There's a bug with the get_python script that causes issues when dependencies are updated. This needs to be address to resolve the open issues with python dependencies.


## Other topics for discussion

- Any thoughts on adding support in the build system for VS2022? (Royal)
  - This update is coming soon. There's a question on whether we need to support both VS2022 and VS2019
  - Royal: Can this be selectable and only support VS2022? 
  - Mike: Yes, however we need to determine if we need to support both in our build system

- Using Incredibuild in our build system (Royal)
  - Mike: This requires infra for the coordinator and licenses. There are also different support levels across different platforms. 
  - Royal: This can be something that IB can work on and help address. We would also treat this integration as any other third-party intergration. 
  - Colin: Looks good as long as we can disable it.
  - Royal: Yes, we wouldn't soley rely on a third-party for our builds.  



## Outcomes from Agenda

- Mike - Review AR SNS messages and verify the required parameters are provided for the metrics team. 
- Brian - Merge in GitHub workflow RFC


#### SIG reviewers and maintainers groups
- @brianherrera 
- @amzn-changml 

## Action Items

- Chairs: No action
- Maintainers/Reviewers: As noted in the agenda outcome

## Open Discussion Items

List any additional items below!
