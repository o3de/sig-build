# Migrate Nightly Pipeline to O3DE

### Summary:
The purpose of this RFC is to document the build SIGs intent to migrate the nightly AR (Automated Review) pipeline over to O3DE's build infrastructure and to highlight the blockers for certain platforms.

This effort will require an investigation into which platforms we want to migrate and support in the nightly pipeline prior to starting this work. 


#### Goals

Here are the main goals of migrating the pipeline over to O3DE:

- **Public visibility:** We want the O3DE community to be able to view the status of the nightly jobs and contribute fixes to the build failures. Currently the nightly pipelines run on internal infrastructure that is only accessible by Amazon teams.
- **Transfer Ownership/Costs:** The ownership of these pipelines and their platforms should be owned by the O3DE SIGs. This also includes the costs of running the pipeline.
- **Stay within budget:** The Linux Foundation has specified a budget to spend on the AR infrastructure. We need to consider costs when migrating platforms to O3DE.
- **Useful State:** The nightly pipeline should provide useful information to developers regarding build failures related to code changes. The requires that the pipeline be in a stable state before it's migrated to O3DE. 

#### Pipelines

- Development Nightly Clean
- Development Nightly Incremental

Note: There are also nightly pipelines created for stabilization branches prior to a release.

### Blockers
This section contains details on the platforms that we cannot migrate in their current state.

#### Platforms
| Platform | Blockers |
| ------- | --- |
| Mac/iOS | Costs will exceed budget. Autoscaling currently not supported. Dedicated hosts are required to deploy Mac EC2 instances which are billed while the hosted instance is shutdown. Minimum 24hr allocation. |
| Windows GPU | Costs will exceed budget. Dedicated hosts not required, so it's possible to autoscale. A solution is required to auto-connect instances when they startup. |


### Migration Plan

The main strategy of the migration plan is to transfer the nightly jobs to O3DE in phases. This will allow us to meet the goals identified above while we address the blockers.

| Task | Description | GitHub Issue/PR |
| --- | --- | ---|
| Identify platforms to migrate in the first pass | First we need to identify the platforms that we want to support in the nightly pipelines, so that we can make the plan to migrate in phases. Check with the required SIGs for approval. | Completed |
| Split up nightly pipeline into separate tags | Update the build_config.json file for each platform to separate the nightly pipeline using the platforms identified in the previous task. This will allow us to run the excluded platforms in an internal pipeline while we address the blockers. | https://github.com/o3de/o3de/pull/9406 |
| Create Jenkins pipelines | Create nightly pipelines on O3DE Jenkins instances to run the separate nightly pipelines created in the previous task. | https://github.com/o3de/o3de/issues/9427 |
| Create plan to address Mac/iOS blockers | Investigate solutions to address the blockers to migrate Mac/iOS platforms to O3DE. This includes evaluating other mac hosting services. | https://github.com/o3de/o3de/issues/9428 |
| Create plan to address Windows GPU blockers | Investigate solutions to address the blockers to migrate the Windows GPU platform to O3DE.  | https://github.com/o3de/o3de/issues/9429 |
