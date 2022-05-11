# O3DE Jenkins Pipeline Deployment

### Summary:
This RFC provides additional details on the deployment stage for the Jenkins pipeline outlined here: https://github.com/o3de/sig-build/blob/main/rfcs/rfc-bld-20220209-1-jenkins-pipeline-repo.md


### Deployment stage details:
The purpose of this RFC is to have the deployment stage setup with two pipelines (we initially assumed a single pipeline). In this workflow, changes are tested in the staging branch/environment separately. To deploy the change to prod, a PR is created to merge the changes from the staging branch to the prod branch.

The main goal of the separate pipelines setup is to reduce the amount of PRs created for each deployment. By having a separate pipeline for staging, these changes can be tested before creating a single a PR for the deployment.

<img width="625" alt="deployment" src="https://user-images.githubusercontent.com/19914798/166326101-c36478bf-81e6-407d-b997-5afacfd1681a.png">

### Jenkins Deployment Issues
There are some caveats to setting up and deploying Jenkins that may complicate a single pipeline setup. Those issues are detailed below.

#### Jenkins Plugin Dependencies

Updating plugins requires an additional level of testing in the staging environment. This is due to the complexity of the dependencies in Jenkins plugins. Here are some examples:

- Plugin is updated to latest. However, it drops another plugin as a dependency. Removing this plugin breaks the functionality of some of our jobs. A new PR is created to update plugins.txt to re-add the plugin.
- Plugin is updated to latest. After upgrading Jenkins, a newer supported version of the plugin is now available. A new PR is created to update plugins.txt with the newer version.
- Plugin is updated to latest. After testing in sandbox, an issue is identified with the latest version. A new PR is created to rollback to a previous version.

### What are the advantages of the separate pipelines?
- All changes can be tested together in the staging environment before it is deployed to prod.
- This will reduce the number of PRs and changes that need to be submitted through the prod pipeline to update Jenkins and its plugins. 

### What are the disadvantages of the separate pipelines?
- There several changes that can accumulate in the staging branch. To avoid this we'll need to setup a process to minimize the changes made to staging at one time and break apart large changes.
- Untested changes are submitted to the prod branch. We'll need to setup branch protection on the prod branch and implement a policy that changes are only merged from the staging branch.

### FAQs?
- How can devs test their changes prior to submitting to staging?
  The docker container for Jenkins can be tested locally. Also the CDK package included in the repo can be used to setup a development environment. We'll need to investigate setting up an O3DE hosted environment.
- Will Jenkins config and infrastructure changes flow through the same pipeline?
  For now yes, since the Jenkins configs are now defined with the JCasC (Jenkins Config as Code) plugin, changes to these configs will trigger a new CodeBuild action to create a new docker image. It's possible that we can later decide to make a separate pipeline for config only changes.
- Is it a permanent decision if we change to a separate pipeline setup?
  No, we can easily switch back to a single pipeline setup if we decide to rollback this change. In fact, the current CDK setup will deploy a staging environment in the same pipeline if a staging certificate parameter is provided.
