## Meeting Details

- **Date/Time:** Mar 23, 2022 @ 05:00 pm UTC / 10:00 am PDT
- **Location:** [Discord SIG-Build Voice Room](https://discord.gg/wDNAQmatpq)
- **Moderator:** Mike Chang
- **Note Taker** Mike Chang

The [SIG-Build Meetings](https://github.com/o3de/sig-build/tree/main/meetings) repo contains the history past calls, including a link to the agenda, recording, notes, and resources.

## SIG Updates

**What happened since the last meeting?**

- OpenSSL dependancies for Linux changed to system OpenSSL, as per: https://github.com/o3de/o3de/issues/4898
- VS studio version updated to 16.11+ for CLI builds in project manager, as per: https://github.com/o3de/o3de/issues/7665

## Meeting Agenda

**Discuss agenda from proposed topics**

- OpenSSL lib dependency change
- VS2022 proposed changes
- DCO plugin investigation: https://github.com/o3de/sig-build/issues/21
- Review and Triage the Backlog for Build SIG - https://github.com/o3de/o3de/issues?q=is%3Aissue+is%3Aopen+label%3Asig%2Fbuild+label%3Aneeds-triage+no%3Aassignee

## Outcomes from Discussion topics

**Discuss outcomes from agenda**

- OpenSSL 
  - (DM) Only for default system libs, not for 3p like zlib
  - Any system libs that we maintain need to have its downstream dependancies included
  - (Mike) Agreed

- VS2022
  - Allow users to build for VS2022 for project manager through cmake configure
  - We still need to update the VS2022 dependancies on the Jenkins nodes to validate
  - Users can still build with VS2022 using the cmake build scripts via command lines
  - (DM) VS2022 warning are being flagged as errors, being reported via Discord
  - (Mike) We should start investigations on automated builds for latest versions of VS2022 (or any latest stable versions)
  - (DM) Agreed
  - (Brian) Preview versions are inherently unstable, it would be hard to take action as things are being fixed

- DCO
  - There has been frequent merge issues with DCO, even with new LF owner
  - (DM) Not seen merge errors recently, but wonder if there are ways for Github to pick up only the user's commit
  - (Mike) There may be libraries that we can call from Github Action


## Action Items

- Mike
  - Create a GHI for automated canary builds for latest stable versions of Visual Studio
  - Continue research for DCO alternatives

**Create actionable items from proposed topics**

- As per action items

## Open Discussion Items

- None discussed
