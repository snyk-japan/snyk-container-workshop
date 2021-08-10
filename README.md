# Introduction to Snyk Container Workshop

Snyk Container helps you find and fix vulnerabilities in container images. With snyk container Scale your security capabilities by enabling developers to quickly eliminate a multitude of vulnerabilities by upgrading to a more secure base image or rebuilding when the base image is outdated

You may not always have access to the original source code that runs in your containers, but vulnerabilities in your code dependencies are still important. Snyk can detect and monitor open source dependencies for popular languages as part of the container scan

In this hands-on workshop we will achieve the follow:

* Step 1 Fork the Goof Application
* Step 2 Configure GitHub Integration 
* Step 3 Configure Docker Hub Integration 
* Step 4 Test using the “Add to Project” Docker Hub Integration
* Step 5 Find vulnerabilities in Goof’s Dockerfile
* Step 6 Fix the Dockerfile FROM tag using a Pull Request
* Step 7 Container Test using the Snyk CLI
* Step 8 Container Reporting Dashboard

## Prerequisites

* public GitHub account - http://github.com
* git CLI - https://git-scm.com/downloads
* snyk CLI - https://support.snyk.io/hc/en-us/articles/360003812538-Install-the-Snyk-CLI
* Registered account on Snyk App - http://app.snyk.io
* Docker Desktop running locally - https://www.docker.com/products/docker-desktop

# Workshop Steps

_Note: It is assumed your using a mac for these steps but it should also work on windows or linux with some modifications to the scripts potentially_

## Step 1 Fork the Goof Application

Navigate to the following GitHub repo - https://github.com/snyk/goof

* Click on the "**Fork**" button
* Ensure you are forking this repo to your public GitHub account
* Click done

![alt tag](https://i.ibb.co/Gdf7N2W/snyk-starter-open-source-2.png)

## Step 2 Configure GitHub Integration

First we need to connect Snyk to GitHub so we can import our Repository. Do so by:

* Login to http://app.snyk.io Sign up if you haven't already.
* Navigating to Integrations -> Source Control -> GitHub
* Fill in your Account Credentials to Connect your GitHub Account.

![alt tag](https://i.ibb.co/bPqqybM/snyk-starter-open-source-1.png)

## Step 3 Configure Docker Hub Integration

Enable integration between Docker Hub and Snyk, to start managing your container vulnerabilities. To do that we must first connect to Docker Hub 

* Navigating to Integrations -> Container Registries -> Docker Hub 
* Enter your Docker Hub username and Access Token and then click Save

Snyk tests the connection values and the page reloads, now displaying Docker Hub integration information and the Add your Docker Hub images to Snyk button. A confirmation message that the details were saved also appears in green at the top of the screen. In addition, if the connection to Docker Hub failed, a notification appears

Note: As the access token, you can either use your DockerHub password or an [access token](https://docs.docker.com/docker-hub/access-tokens/) created in DockerHub. In case 2FA is activated on your account, access token only is applicable

![alt tag](https://i.ibb.co/hYyb7RD/snyk-container-1.png)

![alt tag](https://i.ibb.co/pWkKGmh/snyk-container-2.png)

## Step 4 Test using the “Add to Project” Docker Hub Integration

You may already have images in your Dockerhub Registries but lets go and add a new one to your Docker Hub account. 

* Login to Docker Hub as shown below. These will be the same credentials you used in Step 3 above.

```bash
$ docker login -u DOCKER_HUB_USERNAME -p YOIUR_ACCESS_TOKEN_OR_PASSWORD
Login Succeeded
```

* Pull down the following image as shown below. 

```bash
$ docker pull pasapples/docker-goof
Using default tag: latest
latest: Pulling from pasapples/docker-goof
Digest: sha256:be2d6b0f5041315f632f44e3528ea513c452cc95ed4a40bc3d70f943ab293e5f
Status: Downloaded newer image for pasapples/docker-goof:latest
docker.io/pasapples/docker-goof:latest
```

* Run the following commands to TAG and PUSH the image to your Docker Hub account. 
  
Note: Replace DOCKER_HUB_USERNAME with your Docker Bub username. 

```bash
$ docker tag pasapples/docker-goof:latest DOCKER_HUB_USERNAME/docker-goof:latest

$ $ docker push DOCKER_HUB_USERNAME/docker-goof:latest
The push refers to repository [docker.io/pasapples/docker-goof]
1bc5d83ccce7: Layer already exists
35bda1fbb3d0: Layer already exists
5f70bf18a086: Layer already exists
fbd39f37d7c2: Layer already exists
938fc2ad056c: Layer already exists
c24944d2eccc: Layer already exists
02a318dedbea: Layer already exists
7972420bc26e: Layer already exists
17c76043bf23: Layer already exists
496d6557f1e3: Layer already exists
867786449541: Layer already exists
92d17ee6d9da: Layer already exists
e54368741774: Layer already exists
5a6c4d956b5d: Layer already exists
86ab2c6c5d58: Layer already exists
latest: digest: sha256:be2d6b0f5041315f632f44e3528ea513c452cc95ed4a40bc3d70f943ab293e5f size: 3466
```

* Return to the Snyk Dashboard and click on "**Add your Docker Hub Images to Snyk**"

* Search for "**docker-goof**" and then select it and click "**Add Selected Repositories**"

![alt tag](https://i.ibb.co/mq421V8/snyk-container-3.png)

* This may take a few minutes, so you can view the Import using the link provided as follows

![alt tag](https://i.ibb.co/PQy4pzq/snyk-container-4.png)

* Once complete your container scan should appear as follows

![alt tag](https://i.ibb.co/NTX9KV2/snyk-container-5.png)

When Snyk Container scans an image, using any of the available integrations, we first find the software installed in the image, including:

1. dpkg, rpm and apk operating systems packages.
1. Popular unmanaged software, ie. installed outside a package manager.
1. Application packages based on the presence of a manifest file.

_Note: the container does not need to be run as Snyk reads the info from the file system; therefore, no container or foreign code needs to be run in order to successfully scan._

After we have the list of installed software, we look that up against our vulnerability database, which combines public sources with proprietary research

* Let's go ahead and click on the "**latest**" link to view the container issues as part of the scan

![alt tag](https://i.ibb.co/HGS4MSP/snyk-container-7.png)

One thing you will notice is recommendations for upgrading the base image. This is handy as we can remove a substantial amount of issues just by using an alternative base image from minor upgrades to major upgrades if available will be shown including what issues will remain if the basde image is changed and the container re-built.  

The supported base images can be found at this [link](https://snyk.io/docker-images/)

For each Vulnerability, Snyk displays the following ordered by our [Proprietary Priority Score](https://snyk.io/blog/snyk-priority-score/) :

1. The module (O/S, base image or user instruction layer) that introduced it and, in the case of transitive dependencies, its direct dependency
1. Details on the path and proposed Remediation, as well as the specific vulnerable functions
1. Overview
1. Exploit maturity
1. Links to CWE, CVE and CVSS Score
1. Social Trends
1. Plus more ...

## Step 5 Find vulnerabilities in Goof’s Dockerfile

Snyk detects vulnerable base images by scanning your Dockerfile when importing a Git repository. This allows you to examine security issues before building the image, so helps solve potential problems before they land in your registry or in production.

Now that Snyk is connected to your GitHub Account, import the Repo into Snyk as a Project as this contains a Dockerfile.

* Navigate to Projects
* Click "**Add Project**" then select "**GitHub**"
* Click on the Repo "goof" that you forked earlier at Step 1.

![alt tag](https://i.ibb.co/q9Rsxsh/snyk-starter-open-source-3.png)

_Note: The import can take up to one minute, so you can view the import log while it's running as shown below_

![alt tag](https://i.ibb.co/RQsX6jZ/snyk-starter-open-source-14.png)

* Once imported you should see a reference for the Dockerfile as shown below. 

![alt tag](https://i.ibb.co/1rNMFhC/snyk-container-9.png)

* Go ahead and click on the Dockerfile this is similar to what a scan of a container from a registry looks like BUT this tim we are scanning a Dockerfile itself versus the full container image.

In a Dockerfile project, you can find the relevant metadata of the Dockerfile and the base image used. If the base image is an [Official Docker image](https://docs.docker.com/docker-hub/official_images/), the results include recommendations for upgrades to resolve some of the discovered vulnerabilities

## Step 6 Fix the Dockerfile FROM tag using a Pull Request

TODO://

## Step 7 Container Test using the Snyk CLI

The Snyk CLI can run a container test on containers sitting in a registry and even your local docker deamon if you like. All the Snyk CLI needs is acces sto the registry itself which is for public Docker Hub images only requires a "docker login" to achieve that. The following examples show how to use the Snyk CLI to issue a container test.

* You have already built "docker-goof" so go ahead and test that as shown below

```bash
$ snyk container test pasapples/docker-goof:latest

...

Tested 412 dependencies for known issues, found 889 issues.

Base Image  Vulnerabilities  Severity
node:14.1   889              45 critical, 186 high, 196 medium, 462 low

Recommendations for base image upgrade:

Minor upgrades
Base Image  Vulnerabilities  Severity
node:14.17  530              9 critical, 46 high, 40 medium, 435 low

Major upgrades
Base Image   Vulnerabilities  Severity
node:16.5.0  344              3 critical, 31 high, 49 medium, 261 low

Alternative image types
Base Image                Vulnerabilities  Severity
node:16.6.0-slim          60               2 critical, 8 high, 5 medium, 45 low
node:16.6.1-buster-slim   60               2 critical, 8 high, 5 medium, 45 low
node:16.6.0-buster        343              3 critical, 30 high, 49 medium, 261 low
node:16.6.0-stretch-slim  78               6 critical, 11 high, 8 medium, 53 low
```

* The following container test is for a Spring Boot application 

```bash
$ snyk container test pasapples/spring-crud-thymeleaf-demo:latest

...
Organization:      pas.apicella-41p
Package manager:   deb
Project name:      docker-image|pasapples/spring-crud-thymeleaf-demo
Docker image:      pasapples/spring-crud-thymeleaf-demo:latest
Platform:          linux/amd64
Licenses:          enabled

Tested 652 dependencies for known issues, found 435 issues.
```

* Their is also a Distroless version if you would like to try with that

```bash
$ snyk container test pasapples/spring-crud-thymeleaf-demo:distroless
...
```

* Finally, we can monitor container images using the "snyk container monitor" command as shown below

```bash
$ snyk container monitor pasapples/spring-crud-thymeleaf-demo:latest --project-name=spring-crud-thymeleaf-demo-container

Monitoring pasapples/spring-crud-thymeleaf-demo:latest (spring-crud-thymeleaf-demo-container)...

Explore this snapshot at https://app.snyk.io/org/workshops-admin-org/project/1cce2457-a6ac-4f09-8465-9ca596a966fa/history/a253804a-bdb2-4ed1-a7d6-54a321e7f725

Notifications about newly disclosed issues related to these dependencies will be emailed to you.
```

![alt tag](https://i.ibb.co/vY63PXR/snyk-container-12.png)

* Return to the Snyk App and this time you will see the container image 

## Step 8 Container Reporting Dashboard

_Note: It can take up to an hour for report pages to show full details so if you see very little detail that would be why_

* Back to the Snyk App navigate to the projects page and select "**View report**" for the "**docker-goof**" project as shown below

* The following report page for the "**docker-goof**" container should be displayed

![alt tag](https://i.ibb.co/yPFVyt2/snyk-container-11.png)

Thanks for attending and completing this workshop

![alt tag](https://i.ibb.co/7tnp1B6/snyk-logo.png)

<hr />
Pas Apicella [pas at snyk.io] is an Solution Engineer at Snyk APJ
