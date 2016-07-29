This repo provides you step-by-step instructions to get
<a href="#SampleApp">a sample app</a>
running on a server in the cloud.
We then make a change and update the server through an automated continuous integration
<a href="#Pipelines">"pipepline"</a>.

The intent of this effort is to vary the
DevOps technologies used so you can realistically compare
the true effort needed for each <a href="#TechChoices">
choice of technologies</a>.

   * Repositories GitHub vs. GitLab vs. Bitbucket
   * Continuous integration utilities Jenkins vs. Circle-CI vs. Shippable
   * Scripting with Maven scripts vs. declarative statements

   * Bare metal vs. virtual instances vs. Docker containers
   * Operating system CentOS vs. Ubuntu vs. RedHat
   * Build in VMs vs. Dockerhub
   * Build from scratch vs. reuse builds in Artifactory
   * Instantiate using Kubernetes

   * Databases used by the application MongoDB vs. Postgres, etc.
   * Instantiate database as a separate server vs. mlab vs. compose clouds
   * Cloud environment Digital Ocean vs. Heroku vs. AWS vs. Azure vs. Google vs. Rackspace

   * Testing of functional performance and capacity
   * Logging for traceability
   * Monitoring
   * Scalability for enterprise use

<hr />

<a name="TechChoices"></a>

## Choice of technologies #

From the above menu there can be many combinations.

So let's first build the <a href="#MostCommonDevOps">
most common DevOps technologies in use today</a>,<br />
then substitute one piece at a time to observe the difference.

The first substitute Jenkins with
<strong>Shippable</strong> for continuous integration
to see the impact of:

   * Scripting with Maven scripts in Jenkins vs. declarative statements
   * A single pipeline view in Jenkins vs. several on a single pane of glass (SPOG)
   * Sharing variables/state between jobs
   * Chaining jobs
   * Mix build triggers with parametized builds

BTW, the intended audience for this article are experienced
<strong>developers</strong> who want to focus on innovation and
<strong>DevOps</strong> cloud system administrators supporting developers.


<hr />

<a name="SampleApp"></a>

## Sample Apps #

<img align="right" width="192" height="294" src="https://cloud.githubusercontent.com/assets/300046/17109659/c4ef30e8-524d-11e6-8a97-eb816206a1ef.png">

As an example, let's consider 2 micro-services, each defined in a separate repo:

0. A cron service called <strong>Box</strong>.
   This updates into MongoDB a periodic heart beat
   to show it's still up and running<br />
   (<a target="_blank" href="https://github.com/ttrahan-beta/box">https://github.com/ttrahan-beta/box</a>)

0. A visualizer UI called <strong>dv</strong>
   that connects to the MongoDB Box updated by Box and
   draws whatever cron services are updating TTL (Time To Live)<br />
   (<a target="_blank" href="https://github.com/ttrahan-beta/dv">https://github.com/ttrahan-beta/dv</a>)

A Java (Spring Boot) sample app can also be considered,
but let's first look at the simpler Node.js example.

Because we have two apps, we can see when
one service i.e. box or dv changes,
only the changed image should be deployed.
We don’t want to deploy the entire app when one component changes.

Integrated tests are at
https://github.com/ttrahan-beta/test

QUESTION: What about different branches of https://github.com/aye0aye/box

Files in these repos are described when appropriate below.

   * **package.json** defines Node packages to be installed
   * **server.js** is the file Node invokes

   * **Gruntfile.js** to assemble the various sub-components

   * **bower.json** and bower_components folder
   * **static** folder

   * **shippable.yml**
   * <a href="#Dockerize">**Dockerfile**</a> to "Dockerize" the app

   * **build.sh**
   * **boot.sh**
   * **push.sh**

<hr />

<a name="MostCommonDevOps"></a>

## Most common DevOps technologies #

GitHub, CentOS, Jenkins, on Digital Ocean,
referencing a database in mlab.
These all offer free (or nearly free) usage options.

<a name="Pipelines"></a>

## Automating the Pipeline #

![ci pipeline node v01-20160729-650x352-i11](https://cloud.githubusercontent.com/assets/300046/17262747/87bc735c-559a-11e6-87d3-ae26d57c2884.jpg)

<strong>Details of tasks to achieve the above is currently being actively updated.</strong>

   0. <a href="#Dockerize">Apps are Dockerized with a Dockerfile</a>
   0. <a href="#InGitHub">A specific commit in GitHub is tagged manually</a>
   0. <a href="#JenkinsSetup">Instantiate the CI server (running Jenkins with packages, in Digital Ocean)</a>
   0. <a href="#CommitTrigger">Hook GitHub to trigger build on commit to the Develop branch</a>
   0. <a href="#JenkinsIn">The CI server obtains the repos by a checkout of Develop branch</a>

   0. <a href="#JenkinsIn">Define Docker credentials in CI (Jenkins)</a>
   0. <a href="#JenkinsIn">Only Java programs that compiles WAR files need Artifactory</a>
   0. <a href="#JenkinsBuild">Commit into Develop branch on GitHub</a>
   0. <a href="#JenkinsBuild">Invoke / connect Docker Jenkins package</a>
   0. <a href="#JenkinsBuild">Build Docker image with with a tag</a>

   0. <a href="#Push2Dockerhub">Push Docker image to DockerHub (or other image repository)</a>
   0. <a href="#PullDockerhub">Pull Docker image from DockerHub</a>
   0. <a href="#TagGitHub">Register APIs and external databases to be used</a>
   0. <a href="#DeployDO">Provision VPC</a>
   0. <a href="#DefineDO">Run Docker to create container from Docker image</a>

   0. <a href="#SmokeTests">Perform Smoke Test on server using Mocha or other test utility</a>
   0. <a href="#VaryDockerOptions">Verify Logging for traceability</a>
   0. <a href="#VaryDockerOptions">Verify Monitoring for workflow analytics</a>
   0. <a href="#VaryDockerOptions">Verify Scaling for enterprise use</a>
   0. <a href="#SlackNotification">Send notification (via Slack) that server is ready for use</a>

   0. <a href="#VaryDockerOptions">Vary Docker Options</a>
   0. <a href="#VaryDockerOptions">Update app source and cycle through again</a>

<hr />



Link to a hosted repository service



<a name="Push2Dockerhub"></a>

Test using <a target="_blank" href="http://requestb.in/">RequestBin (http://requestb.in)</a>, 
which gives you a URL that collects requests made to it 
and let you inspect them in a human-friendly way.
Use RequestBin to see what your HTTP client is sending or to inspect and debug webhook requests.

See https://docs.docker.com/docker-hub/webhooks/

If you have an automated build repository in Docker Hub, you can use Webhooks to cause an action in another application in response to an event in the repository. Docker Hub webhooks fire when an image is built in, or a new tag added to, your automated build repository.

With your webhook, you specify a target URL and a JSON payload to deliver. The example webhook below generates an HTTP POST that delivers a JSON payload:
