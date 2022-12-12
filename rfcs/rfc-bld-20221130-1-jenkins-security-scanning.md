# O3DE Jenkins Pipeline - Security Scanning

## Summary:
This RFC the details the solutions to provide security scanning for the O3DE Jenkins Pipeline repo.

The main objective is to provide a mechanism to improve the security and best practices of the contributions to the cloud infrastructure and docker configurations in this repo. 

The components we want to scan in the repo are the CDK packages and Docker images. All contributions will be required to pass the configured checks. 

The scope of the security scanning is limited to the Jenkins server infrastructure and the components used to deploy it. This does not include the build node configs. Those configs are stored in the o3de repo: [Build Node Configs](https://github.com/o3de/o3de/tree/development/scripts/build/build_node/Platform)

## Feature design description:
The two main components that make up the O3DE Jenkins Pipeline repo are the CDK packages and the Docker image setup.

| Component| Usage | Environment |
| --- | --- | ---|
| CDK Packages | Defines and deploys the AWS Infrastructure to host the Jenkins pipeline server. | Python |
| Docker | Creates the image for the container that runs the Jenkins pipeline server. | Linux shell commands |


The CDK scanning solutions typically provide coverage using one of these two options:

1. Scan the CDK constructs
2. Scan the generated CloudFormation templates

The tools that scan the CDK constructs will evaluate the code within each of the CDK packages for any potential security vulnerabilities that will created in the final CloudFormation stack. The benefit of scanning CDK constructs is that the static analysis and synthesis steps can be performed at the same time during the cdk synth step. Developers can generate the CF templates and run the static analysis with a single command. However, this does require adding an additional construct within the CDK app to support this.

The other options scan the generated CloudFormation templates after the packages are created using `cdk synth`. Scanning the generated templates does not require any changes to the codebase and these scanners are typically stand-alone applications.


## CDK Package Solution: cdk-nag

- GitHub Repo: https://github.com/cdklabs/cdk-nag
- Guides: https://aws.amazon.com/blogs/devops/manage-application-security-and-compliance-with-the-aws-cloud-development-kit-and-cdk-nag/

Cdk-nag is an open source tool developed by AWS that provides the ability to scan CDK packages for security and best practice violations. To enable the checks, a cdk-nag construct needs to be added either to the top-level app or the individual stacks.

Example adding cdk-nag to the top-level app. This will enable the checks when running cdk synth or cdk deploy.

```
[...]
import cdk_nag

app = cdk.App()

Aspects.of(app).add(cdk_nag.AwsSolutionsChecks())
```

Example output:
```
cdk synth
[Error at /S3BucketWithCfnNagConstructStack/S3Bucket/Resource] AwsSolutions-S1: The S3 Bucket has server access logs disabled.
[Error at /S3BucketWithCfnNagConstructStack/S3Bucket/Resource] AwsSolutions-S2: The S3 Bucket does not have public access restricted and blocked.
[Error at /S3BucketWithCfnNagConstructStack/S3Bucket/Resource] AwsSolutions-S3: The S3 Bucket does not have default encryption enabled.
```


## Docker Image Solution: Docker Scan

- Product Info: https://docs.docker.com/engine/scan/

This tool is provided by Docker and provides the ability to scan the generated docker image for security vulnerabilities. This can run locally and in our pipeline, but does require that we first build the image prior to each scan.

Example setup and scan commands:
```
# Setup 
apt-get install docker-scan-plugin

# Build image
docker build -t jenkins .

# Run scan 
docker scan jenkins
```


## General Scanning

In addition to the options detailed above, this section will provide scanning options for other components in the O3DE Jenkins Pipeline repo.

**Dependabot**

- GitHub repo: https://github.com/dependabot/dependabot-core

Dependabot can be used to keep the dependencies defined in the repo up-to-date. Dependencies are defined in the following locations:

- Dockerfile: Our docker file uses the docker image published by Jenkins as its parent (e.g. FROM jenkins/jenkins:2.332.3-lts-jdk11) 
- cdk/requirements.txt: This file is used define the dependencies required to deploy the CDK stacks

Dependabot supports updating both docker and python dependencies. Enabling this tool in our repo can be found in its GitHub settings: `Repo > Settings > Code security and analysis`

To complete the setup, a config file also needs to be added to the repo at the following location: `.github/dependabot.yml`

Documentation on the .yml file can be found here: [Configuring dependabot version updates](https://docs.github.com/en/code-security/dependabot/dependabot-version-updates/configuring-dependabot-version-updates) 


## How will this be implemented

Github Actions/Pull Request checks
- The scanning solutions listed above will be setup as GitHub actions on the O3DE Jenkins Pipeline repo. This will allow the checks to run on pull requests and to commits submitted to branches. 


## Are there any alternatives to this feature?
Yes. Static analysis for CDK packages is still limited, so most of the other solutions scanned the generated CloudFormation templates instead of scanning the cdk code directly.

For Docker images, options to scan dockerfiles/shell commands directly used paid or restrictive licenses.

## Are there any open questions?
- Will we be able to run these checks locally?
	- Yes, the solutions covered in thie RFC can be executed locally during development. 
- Will these solutions scan code going into O3DE?
	- No, this security scanning setup will only cover code going into the Jenkins Pipeline for O3DE. Investigation for the O3DE repo is tracked here: https://github.com/o3de/o3de/issues/10032
- Can the scanner tools be swapped out later?
	- Yes, once we setup the GitHub Actions to run on commits and pull request, we can swap out the actual scanning tools later provided they support the environment we're running in. 
 
