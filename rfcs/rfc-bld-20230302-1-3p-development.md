Introduction
-----------------
The current system using Jenkins and AWS S3 helped bootstrap our move for our 3rd Party libraries to open source, but much of the current workflow depends on:

1.  Committing the 3p patch and build code in [https://github.com/o3de/3p-package-source/tree/main/package-system](https://github.com/o3de/3p-package-source/tree/main/package-system)
2.  Building the 3p package locally (which requires developers to maintain a build environment), using a "pull and build" script, then compressing the result as a tar.xz file
3.  Using a separate internal Jenkins to publish the build package up to AWS S3 bucket that is only accessible by AWS developers (Note: this is not validated against the code checked in to Github)
4.  Add a reference to the package in O3DE (defined here: [https://github.com/o3de/o3de/tree/development/cmake/3rdParty/Platform](https://github.com/o3de/o3de/tree/development/cmake/3rdParty/Platform)/<platform>/BuiltinPackages-<platform>.json) with their code change PR and AR it
5.  Using an internal Jenkins to promote the packages to O3DE's S3 bucket and Cloudfront CDN once tests pass

Internally, we did this to thoroughly verify legacy packages for NDA'ed code. However, these components (specifically 2, 3, 5) are not visible to the public and cannot be directly contributed or packaged without having access to the internal systems.

We aim to make this public and the intake process transparent, as the community currently has no direct means to make contributions. In order to do this, we will need a end to end workflow that accomplishes the following:

1.  Does not involve any internal-only tooling infrastructure
2.  Can run CI/CD under the same scope as the fork or branch, to allow for development independent of the production infrastructure
3.  The workflow and storage device is visible to the public, to make changes straightforward and transparent

$~$

Proposed Solution
-----------------
We propose using a combination of Github Actions and Github Packages to achieve our goals.

First, an overview of each of these services:

### Github Actions

**What is it**: A build pipeline service offered by Github, similar to Jenkins. They offer managed Windows, Ubuntu Linux, and Mac instances that come [pre-loaded with build dependencies](https://github.com/actions/virtual-environments) (such as msbuild/vc142, cmake, android SDK) for a variety of use cases, and can be loaded with other dependencies. It can also run builds on hardware hosted by the user through an agent.

**How do you use it**: Build scripts are stored in **`.github/workflows`**  and can be forked along with the repo to provide the same automation to the fork. It can pull scripts within its own repo and execute them as "stages". These can be triggered on [various actions](https://docs.github.com/en/actions/using-workflows/events-that-trigger-workflows), such as merge or PR creation.

**Cost**: Github actions is free for public repos and any forks made from it. Usage limits are not explicitly defined, but it is currently in use by large projects without known issues. For private repos, cost is broken down by a ["bucket" of minutes](https://docs.github.com/en/billing/managing-billing-for-github-actions/about-billing-for-github-actions).

**What does this replace:** Jenkins and local build infrastructure

$~$

### Github Packages

**What is it**: A package distribution service offered by Github. They offer storage for a variety of package systems, like nuget, npm, and docker (OCI) containers. This is backed by a managed CDN. Note: Github does not offer a service like S3, it must be a managed package format.

**How do you use it**: For this use case, we're looking to use a specification for OCI that allow arbitrary [artifacts to be stored as containers, which come with hash generation, binary layering, and signing](https://github.com/oras-project/artifacts-spec/blob/main/scenarios.md). These artifacts can be created as a tar.xz file with metadata (using [www RFC 1341](https://www.w3.org/Protocols/rfc1341/4_Content-Type.html) content type headers), converted to container form and uploaded to Github Packages using pre-existing open source libraries/clients (specifically, ["cosign"](https://github.com/sigstore/cosign))

**Cost**: Github Packages is free for public repos and any forks made from it. Usage limits are not explicitly defined, but it is currently in use by large projects without known issues. For private repos, cost is broken down by [storage and aggregate transfer rates](https://docs.github.com/en/billing/managing-billing-for-github-packages/about-billing-for-github-packages)

**What does this replace:** AWS S3 buckets and Cloudfront CDN  

With this in mind, some examples of the two main workflow components, packaging and building:

$~$

## OCI container packaging example

We will be using a client application called ["cosign"](https://github.com/sigstore/cosign) a Linux Foundation project that signs and packages arbitrary artifacts in OCI artifact format. This will be used to login to Github Packages, and publish a container to the root of the Github user or organization profile (it's not explicitly tied to a repo until assigned)

We can do the following to login to Github Packages. This creates a authentication token, stored in the home drive

`echo $GITHUB_ACCESS_TOKEN | cosign login https:http://ghcr.io/ -u $GITHUB_USERNAME --password-stdin`

Then we upload a pre-built sample package [openexr-3.1.3-rev4-windows](https://github.com/users/amzn-changml/packages/container/package/packages%2Fopenexr-3.1.3-rev4-windows "packages/openexr-3.1.3-rev4-windows"), including the associated SHA256 hash and json file:

`cosign upload http://ghcr.io/o3de/openexr-3.1.3-rev4-windows:1.0 ./openexr-3.1.3-rev4-windows.tar.xz`

If we take a look at the [uploaded package](https://github.com/users/amzn-changml/packages/container/3p-package-source%2Fopenexr-3.1.3-rev4-windows/24268585?tag=v1) in Github Packages, it comes with all the associated metadata. The `tar.xz` file is in the layers under the `"application/octet-stream"` media type.

```
{
  "digest": "sha256:03c26a2e95393ec57484c1b2beae6eb0ae569ed3b4c977ad2f59475315f91f56",
  "mediaType": "application/vnd.oci.image.manifest.v1+json",
  "size": 402,
  "config": {
    "digest": "sha256:44136fa355b3678a1146ad16f7e8649e94fb4fc21fe77e8310c060f61caaff8a",
    "mediaType": "application/vnd.unknown.config.v1+json",
    "size": 2
  },
  "layers": [
    {
      "digest": "sha256:44136fa355b3678a1146ad16f7e8649e94fb4fc21fe77e8310c060f61caaff8a",
      "mediaType": "application/octet-stream",
      "size": 2
    },
    {
      "digest": "sha256:c850268e849171751cdaefdab1952333ac38afbb771b999e99d67f9761706d98",
      "mediaType": "application/octet-stream",
      "size": 4838876
    }
  ]
}
```

If we want to pull the package, we can take the root digest hash of the generated container and curl the package (the bearer token is a generic credential used for all clients)

``` curl -L -H "authorization Bearer q==" http://ghcr.io/o3de/03c26a2e95393ec57484c1b2beae6eb0ae569ed3b4c977ad2f59475315f91f56 > openexr-3.1.3-rev4-windows.tar.xz ```

$~$

## Github Actions building example
Using the cosign application demonstrated, we will use an Ubuntu environment with some dependencies, run "[pull\_and\_build\_from\_git.py](https://github.com/amzn-changml/3p-package-source/blob/main/Scripts/extras/pull_and_build_from_git.py)" for a package, then upload the resulting .tar.xz file to Github packages

```
name: Package-CI-Build

on: 
 push: 
       branches: 
           - main 
           - stabilization-* 
        permissions: read-all

jobs: 
    build: 
       name: build 
       runs-on: ubuntu-latest 
       permissions: 
          id-token: write
          contents: read

 steps: 
           - uses: actions/checkout@2541b1294d2704b0964813337f33b291d3f8596b # v2.4.0 
           - uses: sigstore/cosign-installer@7e0881f8fe90b25e305bbf0309761e9314607e25 # v2.3.0 
           - uses: actions/setup-go@b22fbbc2921299758641fab08929b4ac52b32923 # v2.2.0 with: go-version: '1.17' check-latest: true
           - run: | 
                  python3 ${GITHUB_WORKSPACE}/Scripts/extras/pull_and_build_from_git.py $GITHUB_WORKSPACE/package-system/$PACKAGE_NAME
                  echo $GITHUB_ACCESS_TOKEN | cosign login https://ghcr.io -u $GITHUB_USERNAME --password-stdin
 cosign upload ghcr.io/o3de/$PACKAGE_NAME:$VERSION ./$PACKAGE_NAME.tar.xz
```

$~$

## Potential development workflow

### 3P Package Development Phase

1.  Contributors fork/branch the source and GitHub action scripts from the 3p-package-source repo
2.  During development they can build and package in their "dev" bucket on Github packages using a Github Action build step contained in the fork  
3.  The build step uses a Github Action to execute a build script (currently the `[build_packages.py](https://github.com/o3de/3p-package-scripts/blob/main/o3de_package_scripts/build_package.py)` script but can be agnostic), which will consume a primary metadata file for each platform  
    1.  This primary metadata file will be part of contribution prior to the build (currently the [`package_build_host_list_<platform>.json`](https://github.com/o3de/3p-package-source/blob/main/package_build_list_host_windows.json) file), which will define custom steps specific to each package
    2.  The package source path could include **`PackageInfo.json`**, a metadata file containing package version and licenses, but one can be generated as part of the build
4.  If the build step succeeds, a separate Github Actions step will create the OCI image from the output, then upload it to Github Packages. A download location and hash of the package will be generated in the build output. 
5.  To allow a developer to consume their own packages, they would add the Github Packages CDN URL in [cmake/3rdPartyPackages.cmake](https://github.com/o3de/o3de/blob/development/cmake/3rdPartyPackages.cmake#L32) or add to the **`LY_PACKAGE_SERVER_URLS`** environment variable  

$~$

### 3P Package Contribution Phase

1.  When the contribution is ready, a 3P package PR will be created by the contributor, targeting the 3p-package-source development branch  
    1.  Prechecks via Github Actions will get triggered on PR creation to validate the **`PackageInfo.json`** file for completeness and the license indicated in the file is "blessed" by the project.
    2.  Recursive dependencies and licenses **should** be called out, but a secondary license check using [Scancode Toolkit](https://github.com/nexB/scancode-toolkit) may be needed to validate
2.  A maintainer will manually execute the build step in Github Actions once all prechecks are green. An automated comment referencing a codeowners approval group can be added to the PR, which would notify maintainers.  
3.  If the build fails, the 3P Package PR is kicked back for re-work
4.  Once the build step is complete, the package is automatically uploaded to Github packages for the O3DE org. The 3P package PR should be merged into 3p-package-source.
5.  A download location and hash of the package will be generated in the build output. The package is now staged for consumption in a Pull Request for O3DE AR, but not ready for use in production
    1.  Note: The hash generated for the O3DE staged package will not match the developer fork package

$~$

### O3DE Contribution/AR Phase

1.  The package hash generated from the 3P package contribution phase will be used in a PR targeting O3DE's `**cmake/3rdParty/Platform/<platform>/BuiltInPackages_<platform>.cmake**` file for the staged package  
    1.  This can be manual initially, but automated PR can be triggered immediately after the 3P Package Contribution phase once there is enough confidence in the workflow
2.  Run the O3DE PR against AR for the new package hash and optionally, any engine changes
3.  If the AR fails, then package changes may need to be made, kicking the workflow back to the 3P Package Development phase for a new PR iteration of the package.
4.  If AR succeeds, then the PR is merged into O3DE development, creating a new reference to the 3P package revision in `**BuiltInPackages_<platform>.cmake**` in the development branch and "promoting" the 3p package to production

$~$

### Cleanup Phase

1.  Development packages that were not promoted past **x** date will be subject to cleanup through a scheduled Github Action, with notification
2.  Package versions in production can be kept up to **n** versions or indefinitely, depending on the need to keep past release versions

$~$

## User/Contributor workflow

### On the user and AR side

1.  The 3p cmake script will download the GitHub Package via Curl and unpack the Docker "image", creating the expected 3p tar.gz file. This will need to match up with the 3p packages hash file
2.  If the Github Package is unavailable or is on the old S3 bucket location, the 3P package downloader can fallback to the old location

$~$

Dependencies
------------

*   Github repos - Houses the patch files and scripts for building 3P packages  
*   Github actions - The infrastructure and scripts for building packages. Scripts housed in .github/workflow  
*   Github packages - The storage service for packages. Artifacts created and uploaded to this service in OCI container format
*   AWS Elastic Container Service (ECS) - A container service using auto scaled hosts for self-hosted runners  
*   AWS Elastic Container Registery (ECR) - A service that hosts pre-built container images  
*   S3 - A storage device that holds packages for internal repos
*   AWS Cloudfront - A content delivery network (CDN) which caches packages for internal repos at global sites

$~$

Additional Considerations
-------------------------
The advantages to this approach:

*   All scripts can be contained in the .github/workflow folder, which will follow along with any fork or branch.
*   Github Actions is invoked within the scope of the repo, fork, or branch, which decentrializes its output
*   Github Actions environments are containerized, so that a developer can reproduce builds locally
*   Github packages in OCI format come with "free" vulnerability scanning, supply chain protection (containers can be signed), and traceability (the artifact hash can be correlated to other public packages)
*   For public repos, this infrastructure is "free" of cost, with some nebulous limit but seems higher than the sum of our current packages. The only time costs come into play is for private repos, which already come with a bucket of Actions minutes and Packages storage space.

$~$

There's some cons however:
*   Because the development package output is decentralized, we won't have a central "development" S3 bucket. 
    - *Mitigation:* This means a contributor will have to ensure that they add the Github packages URL for their fork to the the list of CDN urls to be pulled via cmake for iterative development.
*   Github Actions uses a containerized infrastructure, with limited specs - something on the order of [2 CPUs with 7 GB RAM](https://docs.github.com/en/actions/using-github-hosted-runners/about-github-hosted-runners#supported-runners-and-hardware-resources) with 19~30GB of SSD storage. Large packages like QT and Python may take hours to build. 
    - *Mitigation:* There may be ways to utilize Linux Foundation's hardware as "agents" (called self-hosted runners) for Github Actions if we need higher specs
*   Support for versions of OS is on a ["window" of releases](https://github.com/actions/virtual-environments#available-environments). Some OS versions will fall out of support.
    - *Mitigation:* Older supported platforms will be augmented by self-hosted runners
*   We're at the mercy of Github for any support and if Github itself goes down, then users pulling 3p from an installed version of O3DE may get blocked. 
    - *Mitigation:* We will still use our current Cloudfront/S3 CDN on [o3debinaries.org](http://o3debinaries.org) as a backup, but other CDNs can be used as a potential mirror. This is controlled by the CDN list in [https://github.com/o3de/o3de/blob/development/cmake/3rdPartyPackages.cmake#L32](https://github.com/o3de/o3de/blob/development/cmake/3rdPartyPackages.cmake#L32)

Security and license Considerations
-------------------------
*   How to we prevent supply chain attacks?
    - Github Action trigger abuse
        - *Potential threat:* Threat actors DDoSing Github Actions or causing it to trigger automatically and merge without review
        - *Mitigation:* Only maintainers will be able to trigger the GHA manually. We require a PR of the contribution before it is merged, and must be signed off by 2 people
    - Remote execution on Github Action
        - *Potential threat:* Threat actors injecting remote execution code or binaries into the package
        - *Mitigation:* Static analysis and virus scan is performed on the code and any binary artifacts. The package itself should be vetted by a maintainer (the code should come from the original repo), and any CVEs are resolved before moving to the AR stage
    - Man in the Middle (MitM) attacks on the package transmission
        - *Potential threat:* The CDN address is changed or redirected to a new domain owned by a threat actor
        - *Mitigation:* The CDN address is using HTTPS and will warn if the SSL certificate doesn't match. The 3P downloader will not pull packages on SSL mismatch. The SSL certificate itself is owned by the Linux Foundation (through a domain provider) and has named maintainers
    - Existing package modified by a threat actor
        - *Potential threat:* An existing package is modified by a threat actor and reuploaded to Github Packages
        - *Mitigation:* Github Packages does not allow overwrite of an existing package. In addition, the container is signed through a trust chain to SigStore. Finally, the `tar.xz` file's hash is part of the metadata list file (example: https://github.com/o3de/o3de/blob/development/cmake/3rdParty/Platform/Windows/BuiltInPackages_windows.cmake), which will not match if modified. The 3P downloader will not extract and utilize a package if there is a hash mismatch.

*   How do we prevent package contributions with licenses incompatible with the O3DE project?
    - License incompatiblity
        - *Potential threat:* A contributor submits a package with a commercial or incompatible license
        - *Mitigation:* All packages are recommended to be MIT/Apache 2.0 licensed, though expections can be granted with TSC approval. Maintainers will be expected to validate this through manual and automated means
    - Incorrect licenses
        - *Potential threat:* A contributor submits a package without a license or misattributes the license
        - *Mitigation:* All packages will be scanned for SPDX headers. A metadata file (`build_config.json`) should be included in each package to be built, which will conatin the SPDX license shortname. An additonal automated scan will be performed during the build process to look for license files and will alert the maintainers if a mismatch is found.
        
