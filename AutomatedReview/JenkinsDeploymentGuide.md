# Jenkins Deployment Guide

The purpose of this guide is to provide steps to automate and troubleshoot Jenkins pipeline deployments. The steps here will focus on O3DE's use cases including CDK Pipeline deployments and local testing. 

## Getting Started 

To get started, clone the o3de-jenkins-pipeline repo and review the setup instructions provided in the README. 

- Repo: https://github.com/o3de/o3de-jenkins-pipeline/ 




## Testing

Testing can be completed on a docker container running on your machine in addition to the sandbox instance.

### Local testing

The main workflow is to update the files in your workspace, create an image, and run the container.

#### Authentication

- GitHub Login: Follow the instructions under the GitHub OAuth App in the [README](https://github.com/o3de/o3de-jenkins-pipeline/blob/main/README.md)
- AWS Credentials: If you will use components that requires AWS Auth (e.g. EC2 Plugin), you'll need to generate AWS credentials for the container.


#### Setup Steps

Setup workspace:
1. Clone repo.
2. Create a local testing branch to make it easier to track changes.
	- Do not commit secrets to the repo. 
3. Install and start docker
4. Create a volume.

Setup configs:
1. Update jenkins.yaml to setup GitHub login
    - Add required values to `clientID` and `clientSecret`. Remember: Do not commit secrets to the repo. 
    - It's also possible to pass these values through environment variables. 
2. Update other .yaml files as necessary for testing. 

Run container:
1. Create image: `docker build -t jenkins .`
2. Run container:
```
# Simple run command using named volume:
docker run -d -p 8080:8080 --mount type=volume,target=/var/jenkins_home,source=<volume-name> jenkins

# Example with AWS credentials
docker run -e "AWS_REGION=us-west-2" -e <AWS_ACCESS_KEY_ID> -e <AWS_SECRET_ACCESS_KEY> -d -p 8080:8080 --mount type=volume,target=/var/jenkins_home,source=<volume-name> jenkins
```

### Plugins

The version numbers for plugin updates can be found on the update center page: `$JENKINS_URL/pluginManager/`

If a version shows up as **Unavailable**, this means it requires a newer version of Jenkins. This may appear even if the instance is running on the latest LTS version.


### Jenkins Update

The latest available LTS version can be found here (https://www.jenkins.io/changelog-stable/). Also the associated docker tag can be found here (https://hub.docker.com/r/jenkins/jenkins/tag).

Note: Specify the version number when updating the tag. For example use `2.277.4-lts-jdk11` instead of `lts-jdk11`)

## Jenkins Startup Details

This section contains details on how the Jenkins service starts up using the scripts defined in the docker container. This startup process is different from how it runs on a typical server install. 

### Jenkins Base Image

This base image is maintained by the Jenkins community and published to Docker hub. This is the image we use when creating our own custom docker container. 
- GitHub repo: https://github.com/jenkinsci/docker
- Docker Hub: https://hub.docker.com/r/jenkins/jenkins

#### JENKINS_HOME

One important note about JENKINS_HOME is that it is captured as a Volume in the base image. This prevents us from editing the contents of the directory when creating downstream images. In order to add contents to JENKINS_HOME, files must be copied into the $REF directory (set to /usr/share/jenkins/ref). The Jenkins start up scripts are setup to copy the contents of this directory into JENKINS_HOME when the container starts.  

### Entrypoint

To run Jenkins, the container is configured to using the following entrypoint. This is the command that the container will execute on start up. 

```
ENTRYPOINT ["/usr/bin/tini", "--", "/usr/local/bin/jenkins.sh"]
```

Tini is a init utility for containers: https://github.com/krallin/tini 

The jenkins.sh script performs these main actions:
- Copies the contents of $REF to $JENKINS_HOME by running the jenkins-support.sh script.
    - NOTE: If symlinks are added to $REF, the contents of the target file/directory are copied not the symlink itself. 
- Loads Java and Jenkins options ($JAVA_OPTS and $JENKINS_OPTS)
- Runs the Jenkins WAR file to start the service using the options that are provided.]

### O3DE Jenkins Image

Using the base image above, we create a dockerfile to generate our own Jenkins container image. This allows us to install our required plugins and load other custom options. 
- Dockerfile: https://github.com/aws-lumberyard/o3de-jenkins-pipeline/blob/main/Dockerfile 

#### Custom Entrypoint

We utilize a custom entrypoint to override the original one mainly to clear out the contents of the plugins directory prior to startup. The default location of this directory is in JENKINS_HOME which is stored on a shared filesystem and there currently isn't an option to change this location. This results in stale plugin data that needs to be deleted or manually uninstalled from the Jenkins UI. 

The custom entrypoint is configured to perform our required tasks then call the original entrypoint listed above. 

### Jenkins Startup

After the start up scripts are executed and Jenkins starts, it also performs the following actions:

- Loads the plugins installed into $JENKINS_HOME/plugins
- Uses the JCasC plugin to load the config files and job DSL scripts. The CASC_JENKINS_CONFIG environment variable is used to tell the plugin where to find the files. 
    - The env var is set to ${JENKINS_LOCAL}/configs in our Dockerfile. 
    - This is moved off the default location of $JENKINS_HOME to avoid stale configs when the config files are updated. 



## Jenkins Configuration as Code

Configs and templates are stored in the /configs directory of the repo. Any .yaml or .yml files stored in this directory will be picked up by the JCasC plugin. 

### Decrypting credentials

The configs of an existing Jenkins server can be exported to a YAML file that can be used to generate new config files. 

If there are credentials in the configs, they are encrypted by the Jenkins server. To decrypt, go to the script console on the Jenkins server used to export the file and run the following command:
```
println(hudson.util.Secret.decrypt("{ABC}"))
```
Replace 'ABC' with the string listed in the exported config. 


## Monitoring Deployments

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

### Jenkins Home Directory

The Jenkins home directory is stored in EFS. There are times when you may need to access the home directory to update configs in the event the server fails to start up. More info on the home directory can be found here: https://www.jenkins.io/doc/book/managing/system-configuration/ 

To access the home directory for each instance, spin up an EC2 instance in that AWS account and perform the following steps:

1. Add the same security group as the ECS Task used to run Jenkins. This can be found in ECS Cluster > Services > Networking. This will allow the host to reach the EFS instance.
2. Mount the EFS access point: https://docs.aws.amazon.com/efs/latest/ug/mounting-access-points.html 
3. Navigate to the home directory using the mount point.

Remember to terminate the instance when completed.

### Jenkins fails to start with GitHub Auth plugin

When starting up Jenkins, the login page fails to load and the following error is displayed:

```
com.thoughtworks.xstream.mapper.CannotResolveClassException: org.jenkinsci.plugins.GithubSecurityRealm
[...]
Caused: java.io.IOException: Unable to read ${JENKINS_HOME}/config.xml
```

Cause:
Plugins were updated as part of the latest deployment and there is an incompatibility with the current version of the Github Auth plugin and Matrix Auth plugin. This can happen when only one of the plugins are updated.

Fix:
1. Reset the security realm to the following: `<securityRealm class="hudson.security.HudsonPrivateSecurityRealm">`
    - This will allow you to login with the initial admin user or another user account created in Jenkins. Go to JENKINS_HOME/secrets/initialAdminPassword for the admin password.
2. Update both GitHub Auth plugin and Matrix Auth to latest
3. Go to Settings > Configure Global Security then re-enable GitHub authentication