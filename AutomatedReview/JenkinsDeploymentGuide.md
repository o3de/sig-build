# Jenkins Deployment Guide
The purpose of this guide is to instruct Build SIG maintainers how to trigger a deployment of the O3DE Jenkins server. Deployments are automated by the pipeline detailed here: [O3DE Jenkins Pipeline Repo RFC](https://github.com/o3de/sig-build/blob/main/rfcs/rfc-bld-20220209-1-jenkins-pipeline-repo.md)

## Getting Started 

The Build SIG maintains the Jenkins pipeline and approves changes to the infrastructure required to run it. A select group of maintainers within the SIG have access to the AWS account hosting the underlying infrastructure. Contact the chair/co-chair of [#sig-build](https://discord.com/channels/805939474655346758/816043576034328636) for more info. 

### Resources

| Jenkins Instance | Description | Git Repo/Branch |
| --- | --- | --- |
| https://jenkins.build.o3de.org/ | O3DE Prod Instance | o3de-jenkins-pipeline / branch: o3de-prod |
| https://jenkins-sandbox.build.o3de.org/ | O3DE Sandbox Instance | o3de-jenkins-pipeline / branch: o3de-sandbox

GitHub Repo URL: https://github.com/o3de/o3de-jenkins-pipeline 


## Testing

Testing can be completed on a docker container running on your machine in addition to the sandbox instance.

1. Local testing: Update the files in your workspace, create an image, and run the container. Detailed steps are located in the project [README](https://github.com/o3de/o3de-jenkins-pipeline/blob/main/README.md). 
2. Sandbox: Checkout the the sandbox branch `o3de-sandbox`, push changes directly to it, and the pipeline will deploy the updated config. 


### Plugins

The version numbers for plugin updates can be found on the update center page: `$JENKINS_URL/pluginManager/`

If a version shows up as **Unavailable**, this means it requires a newer version of Jenkins. This may appear even if the instance is running on the latest LTS version.


### Jenkins Update

The latest available LTS version can be found here (https://www.jenkins.io/changelog-stable/). Also the associated docker tag can be found here (https://hub.docker.com/r/jenkins/jenkins/tag).

Note: Specify the version number when updating the tag. For example use `2.277.4-lts-jdk11` instead of `lts-jdk11`)


## Prod Deployment

Proceed to these steps after the changes have been tested in the sandbox. More info on the deployment and testing pipelines can be found here: [O3DE Jenkins Pipeline Deployment](https://github.com/o3de/sig-build/blob/main/rfcs/rfc-bld-20220504-2-jenkins-pipeline-deployment.md)

1. Check out the `o3de-prod` branch and get latest.
2. Create a new branch to stage the changes to deploy.
3. Merge in changes that were tested in the sandbox branch.
```
# Merge individual files with:
git checkout <branch-name> <file-name>
```
4. Commit and push files to the branch
5. Submit a pull request with `o3de-prod` as the target. Main will be selected by default, so make sure this is changed.
6. A maintainer on `sig-build` with the required access will review and merge in the pull request.
7. The change will be deployed through the pipeline.


## Monitoring Deployments

Build SIG maintainers that have access to the AWS account can view the progress of the deployment in CodePipeline.

There are 3 main sections to the pipeline:

1.  Build: The CDK packages are generated and changes are deployed if required, including changes to the pipeline itself.
2.  Assets: A new docker image is created and uploaded where it will be deployed by ECS.
3.  Deploy: The target stack is updated to use the new docker image. This results in the deployment of a new ECS task. Traffic will be routed to the new task once it's online.

### Logs

Click on the **Details** link on each section to view the related logs. 

If a new task is failing to start, you can view the logs in the ECS service page. Click on the task ID then click on the Logs tab. The older task will not be stopped until the new one passes the health checks. 


## Troubleshooting Deployments

### Failed to install plugins

Error:
```
Some plugins failed to download! Not downloaded: <plugin_name>
```

Causes:
- Possible network issues or the incorrect plugin name is provided in plugins.txt.  
    - Fix: Verify the plugin name and version in `$JENKINS_URL/pluginManager/` otherwise retry the build
- The latest plugin version is not supported in the Jenkins LTS version.
    - Fix: Update only to the latest supported version in the plugins.txt file
