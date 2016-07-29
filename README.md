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

The sequence of tasks to establish such an automated pipeline is illustrated in the diagram below.


<a name="Pipelines"></a>

## Automating the Pipeline #

![ci pipeline node v01-20160729-650x352-i11](https://cloud.githubusercontent.com/assets/300046/17262747/87bc735c-559a-11e6-87d3-ae26d57c2884.jpg)

Manual tasks are listed above the blue box representing the Jenkins CI server.

<strong>Details of tasks to achieve the above is currently being actively updated.</strong>

   0. <a href="#Dockerize">Apps are Dockerized with a Dockerfile</a>
   0. <a href="#TagInGitHub">Tag a specific Git commit manually</a>
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

<a name="Dockerize"></a>

## Dockerize the app #

Dockerizing an application is the process of converting an application to run within a Docker container.

   <pre>
FROM node:0.10.44-slim
ADD . /home/demo/box/
RUN cd /home/demo/box && npm install
ENTRYPOINT ["/home/demo/box/boot.sh"]
   </pre>

See <a target="_blank" href="https://wilsonmar.github.io/docker-setup/">this blog</a>.

<hr />

<a name="TagInGitHub"></a>

## Tag a specific Git commit manually #

On local Git:

0. In Terminal, be in your repo's folder.
0. For more information, see https://git-scm.com/book/en/v2/Git-Basics-Tagging
0. List tags (in the singular):

   <tt><strong>
   git tag
   </strong></tt>

0. Add an annotated tag on the current commit:

   <tt><strong>
   git tag -a v01.04 -m "my version 01.04"
   </strong></tt>

   PROTIP: Come up with a standard on whether to zero-pad leading zeroes for sorting.
   Git and GitHub are smart enough to sort what comes before and after dots separately.
   But other programs may not be that smart.
   
   Alternately, add an annotated tag on a previous commit hash:

   <tt><strong>
   git tag -a v1.2 9fceb02
   </strong></tt>

0.  Transfer all of tags to the remote server that are not already there:

   <tt><strong>
   git push origin --tags
   </strong></tt>

   CAUTION: A separate request is necessary to push tags than code.

<hr />

<a name="JenkinsSetup"></a>

## Instantiate the CI server #

We are using a static (always on instance) within Digital Ocean.

0. Under "Manage Jenkins" -> "Manage Plugins", select and install plugins 
   for accessing GitHub and DockerHub:

   * Github Plugin from https://wiki.jenkins-ci.org/display/JENKINS/Github+Plugin
   * Git Plugin from https://wiki.jenkins-ci.org/display/JENKINS/Git+Plugin
   <br /><br />

0. Restart Jenkins so the installation takes.

0. In Manage Jenkins -> Configure System, 
   make sure the path to git is correctly set.

0. Choose "Manually manage hook URLs" under the "Github Web Hook" section. 

0. At the Jenkins home page, click "create new jobs".
0. Enter an item name such as "dv" for the Jenkins project.
0. Select "Freestyle project", then OK.
0. Under "Source Code Management", select "Git".
0. Type the Repository URL, such as https://github.com/hotwilson/dv.
0. Click Add, then Jenkins.
0. For Kind, select Certificate.

   PROTIP: Define a service account for Jenkins to use so several jobs can authenticate to Github.

0. Create a new set of private/public keys, and then either create a Github user for Bender or use the Github deploy keys feature. Follow those links for the excellent guides from Github.


0. Under "Build Triggers", tick "Build when a change is pushed to Github".

0. Save and build your job to get a successful build which clones the repository to the webhost.
 
0. Confirm by getting in the server and inspecting it.


The above are based on information from a variety of sources:

* http://fourkitchens.com/blog/article/trigger-jenkins-builds-pushing-github



<hr />

<a name="Push2Dockerhub"></a>

Test using <a target="_blank" href="http://requestb.in/">RequestBin (http://requestb.in)</a>, 
which gives you a URL that collects requests made to it 
and let you inspect them in a human-friendly way.
Use RequestBin to see what your HTTP client is sending or to inspect and debug webhook requests.

See https://docs.docker.com/docker-hub/webhooks/

If you have an automated build repository in Docker Hub, you can use Webhooks to cause an action in another application in response to an event in the repository. Docker Hub webhooks fire when an image is built in, or a new tag added to, your automated build repository.

With your webhook, you specify a target URL and a JSON payload to deliver. The example webhook below generates an HTTP POST that delivers a JSON payload:
