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

We have 2 micro-services, each defined in a separate repo:

0. A cron service called <strong>Box</strong>.
   This updates into MongoDB a periodic heart beat
   to show it's still up and running<br />
   (<a target="_blank" href="https://github.com/ttrahan-beta/box">https://github.com/ttrahan-beta/box</a>)

0. A visualizer UI called <strong>dv</strong>
   that connects to the MongoDB Box updated by Box and
   draws whatever cron services are updating TTL (Time To Live)<br />
   (<a target="_blank" href="https://github.com/ttrahan-beta/dv">https://github.com/ttrahan-beta/dv</a>)

Because we have two apps, we can see when
one service i.e. box or dv changes,
only the changed image should be deployed.
We don’t want to deploy the entire app when one component changes.

Integrated tests are at
https://github.com/ttrahan-beta/test

QUESTION: What about different branches of https://github.com/aye0aye/box

Files in these repos are described when appropriate below.

   * **package.json** defines Node packages to be installed.
   * **server.js** is the file Node invokes.

   * **Gruntfile.js**

   * **bower.json** and bower_components folder
   * **static** folder

   * **shippable.yml**
   * <a href="#Dockerize">**Dockerfile**</a> to "Dockerize" the app

   * **build.sh**
   * **boot.sh**
   * **push.sh**


<a name="Pipelines"></a>

## Automated Pipelines #


<hr />

<a name="MostCommonDevOps"></a>

## Most common DevOps technologies #

GitHub, CentOS, Jenkins, on Digital Ocean,
referencing a database in mlab.
These all offer free (or nearly free) usage options.

The major steps in this workflow:
 
![ci pipeline node v01-20160729-650x346](https://cloud.githubusercontent.com/assets/300046/17258341/a2878404-5583-11e6-8be6-144a2a3dab63.jpg)

<strong>Details of tasks to achieve the above is currently being actively updated.</strong>

   0. <a href="#Dockerize">Apps are Dockerized with a Dockerfile in GitHub</a>
   0. <a href="#InGitHub">The repo is tagged manually</a>
   0. <a href="#CommitTrigger">Hook GitHub to trigger Build on commit</a>
   0. <a href="#JenkinsIn">A CI server (running Jenkins) is instantiated (in Digital Ocean)</a>
   0. <a href="#JenkinsIn">The CI server obtains the repos by a checkout</a>

   0. <a href="#JenkinsIn">Only Java programs that compiles WAR files need Artifactory</a>
   0. <a href="#JenkinsBuild">Build in Jenkins with a tag</a>
   0. <a href="#JenkinsAutoTests">Auto Test Trigger in Jenkins</a>
   0. <a href="#Push2Dockerhub">Push to DockerHub (or other image repository)</a>
   0. <a href="#PullDockerhub">Pull from DockerHub</a>

   0. <a href="#TagGitHub">Register Tag release</a>
   0. <a href="#DeployDO">Provision VPC</a>
   0. <a href="#DefineDO">Docker run to create container</a>
   0. <a href="#SmokeTests">UAT Smoke Test on server</a>
   0. <a href="#VaryDockerOptions">Logging</a>

   0. <a href="#VaryDockerOptions">Monitoring</a>
   0. <a href="#VaryDockerOptions">Scaling tests</a>
   0. <a href="#SlackNotification">Slack notification server is ready for use</a>

   0. <a href="#VaryDockerOptions">Vary Docker Options</a>
