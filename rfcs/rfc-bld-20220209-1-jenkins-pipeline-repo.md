### Summary:
The purpose of this feature is to provide the O3DE Jenkins Pipeline setup to developers. This includes maintainers/contributors that will submit improvements to the O3DE Jenkins pipeline and game developers that want to use the AR pipeline for their own project.

The goal is to move the pipeline configuration from an internal process to a public one managed by the Build SIG with all the required files stored in a repo under the O3DE org. The current setup has a combination of CloudFormation stacks and manually deployed resources managed by an internal Amazon team.

The release of this feature will not change the behavior of the AR pipeline, but it will allow the O3DE community to maintain and contribute changes to the Jenkins setup.

### What is the relevance of this feature?
This is important because it allows developers to contribute changes to the pipeline setup and enables the following:

1. Ownership of the pipeline can be fully transferred to the Build SIG.
2. Other developers can also fork this repo and easily setup the AR pipeline to run on their own game project.

### Feature design description:
All the scripts and configs required to setup our Jenkins infrastructure will be packaged in a repo. A pipeline will automatically deploy the required resources when updates are merged in.

A CDK package will also be included to easily deploy the service to AWS. This includes a CodePipeline instance that will be used to build, test, and deploy the service.

Use cases:
- **Update the Jenkins server setup**: a user will clone the repo and make changes (e.g. install a new plugin, update the jenkins version, etc.). Then they can test these changes locally by building the docker image and running the container on their local machine. After testing, the user can then submit a PR with their changes. After the changes are reviewed and merged, the pipeline will take the change, deploy it to staging, then finally to production.

- **Use the AR for game projects**: a user will fork the repo and make any customizations they need to the Jenkins server setup. Using this configuration they can easily setup the AR pipeline to run on their project. They can host Jenkins on their own server or a cloud provider using the templates provided.

### Technical design description:
Jenkins config components:
- Dockerfile: Generates the Jenkins server docker image.
- Jenkins Configuration as Code (JCasC) configs: YAML files stored in the repo used by the JCasC plugin that manages the configuration of the Jenkins server.
- plugins.txt: Defines the plugins/versions to install. A script included in the docker image installs the plugins.
- Job DSL configs: Defines the jobs that run the pipelines and also used as templates for new jobs. This is separate from the actual pipeline config defined in `scripts/build/Jenkins/Jenkinsfile` in the O3DE repo.

AWS Hosting components:
- ECS/Fargate: Hosts the Jenkins server container.
- CodePipeline: Coordinates the stages to run the server by detecting config changes, building the container image, and deploying the container to production.
- Cloud Development Kit (CDK) Stacks: Automates the deployment of the resources required to host the service in AWS.

![image](rfc-bld-20220209-1-jenkins-pipeline-repo/workflow_diagram_1.png)

Deployment workflow:
 1. User submits a pull request to change one of the config files or scripts (e.g. dockerfile). After the PR is approved the change is merged into the repo.
2. The update triggers CodeBuild to build a new image. After the image is created it is pushed to a private ECR repo.
  a. CodeBuild uses the dockerfile, plugins.txt, and buildspec.yaml file stored in the repo to build the jenkins image
3. The pipeline triggers the deployment of the updated CDK stack.
4. Fargate deploys a new task using the new image stored in ECR.
5. The task definition config defines the image tag, memory and CPU spec, and other container options.

The core part of this setup is the docker container used to run the Jenkins server. To build the image, we pull down the latest LTS version of the Jenkins docker image and run our Dockerfile with our customizations.

```
FROM jenkins/jenkins:2.319.2-lts-jdk11
```

#### Testing Deployments

Here are the options to test changes before they are deployed to production:
1. Locally: Users can build the docker image and run the container to host the Jenkins instance on their local machine. Users can also run `cdk synth` to validate changes to the CDK packages prior to deployment and use `cdk deploy` to test the changes in their own AWS account.
2. Staging: The pipeline will take the changes merged in and deploy them to the staging instance for automated and manual testing
3. PR checks: When submitting a PR to the repo, we can also add automated checks for the jenkins configs and CDK stacks prior to merging it to the repo.

#### Rollback

In the event that issues are identified in the prod stack, a rollback mechanism will be required to redeploy the last known good version. 

These issues should be identified with automated mechanisms to rollback quickly and effortlessly. We should also provide a manual rollback mechanism if issue are identified outside of automated testing. 

CDK Pipelines doesn't provide a built-in mechanism to rollback yet, however the CodeDeploy resource within the Jenkins ECS stack can be utilized to perform a rollback if necessary. 

#### Deployment Permissions

For simplicity, a manual approval step will be added to promote changes from staging to prod. We will need to investigate additional mechanisms to promote these changes. This includes maintenance windows, merge queues, etc. We will need to work with the build SIG to identify the users that will have access to promote changes.

Additionally, a subset of build SIG maintainers will need direct access to the AWS account hosting the pipeline. This will be required in the event an investigation requires access to the AWS account to resolve an issue. 

### What are the advantages of the feature?
- Makes it easier to maintain our jenkins infrastructure and keep it up-to-date with security patches for jenkins and its plugins.
- Makes it easier for developers to run the same AR pipeline on their own O3DE project. This combined with the `Jenkinsfile` in the o3de repo, developers can use this setup to run the same pipe on their project and customize as needed. 
- All config changes to the Jenkins server will be version controlled and will be reviewed through PRs.

### What are the disadvantages of the feature?
- Each config change to the Jenkins server setup will require a redeployment. Interruptions will be minimized by setting up deployment windows during off hours.
- This will be an opinionated setup of the jenkins pipeline and optimized for O3DE. It may not work for all developers, however they will be able fork the repo and customize as needed.

### Are there any alternatives to this feature?
An alternative is to migrate our pipeline workflows to GitHub actions. While we have plans to utilize this mechanism for smaller automated workflows, there are some limitations that prevent us from migrating the entire pipeline to GH actions. This is mainly due to storage and compute restrains for O3DE builds. It's possible to utilize self-hosted runners, so we would still maintain some of our current build infrastructure with this solution. That said, there are planned investigations into how we can integrate GH actions into our pipeline. 

### How will users learn this feature?
- Our current Jenkins Pipeline guide will be updated to include info on how changes can be tested and deployed through this setup. We will also update our docs to include a guide for users that want to use the AR pipeline on their own project.
- Our contribution docs will also be updated to reference this setup.

### Are there any open questions?
- Will this repo also be used to deploy config updates to production and how will it be protected?
  - Yes, a separate branch from `main` will be created for our production deployments and branch protection will be enabled to require approvals and successful checks. The pipeline used for deployment will also have additional protections when moving changes from staging to prod.
- Is this setup only compatible with AWS?
  - No, the generated docker container to run Jenkins can be hosted on any number of platforms. A CDK template will be included in the initial rollout, since our current infrastructure is hosted in AWS. It's possible for the community to provide templates for other cloud providers as needed. 

