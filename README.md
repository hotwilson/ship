This repo provides you step-by-step instructions 
to get a new custom program running on a server in the cloud, 
then make a change. 

The components varied during this exercise include these decisions:

   * Repositories GitHub vs. GitLab vs. Bitbucket
   * Continuous integration utilities Jenkins vs. Circle-CI vs. Shippable, etc.
   * Scripting with Maven vs. Ansible declarative statements
   * Bare metal vs. virtual instances vs. Docker containers
   * Operating system CentOS vs. RedHat 
   * Build from scratch vs. Dockerhub vs. reuse builds in Artifactory
   * Instantiate using scripts vs. Kubernetes
   * Instantiate database as a separate server vs. mlab vs. compose clouds
   * Databases used by the application MongoDB vs. Postgres, etc.
   * Cloud environment Digital Ocean vs. Heroku vs. AWS vs. Azure vs. Google vs. Rackspace

We conduct this exercise so you can realistically compare 
the true effort needed for each choice of technologies.

Let's start with using GitHub, Jenkins, Dockerhub on Digital Ocean,
referencing a database in mlab.
These all offer free usage options.

The intended audience for this demo are experienced
<strong>DevOps</strong> cloud system administrators supporting
<strong>developers</strong>.

Our <strong>pipeline</strong> is to use a Jenkins instance 
to build the two apps,
then if successful, automatically push Docker images into Dockerhub.com
with a unique tag (a semantic build version number).

The major steps in this end-to-end pipeline:

   0. <a href="#InGitHub">Define apps in GitHub.com</a>
   0. <a href="#Dockerize">Dockerize apps</a>
   0. <a href="#JenkinsIn">Setup Jenkins instance</a>
   0. <a href="#JenkinsBuild">Build in Jenkins</a>
   0. <a href="#JenkinsAutoTests">Auto Test Trigger in Jenkins</a>

   0. <a href="#TagDocker">Add Docker semantic tags</a>
   0. <a href="#Push2Dockerhub">Push to Dockerhub.com</a>
   0. <a href="#Push2Dockerhub">Push to Dockerhub.com</a>
   0. <a href="#TagGitHub">Tag GitHub release</a>
   0. <a href="#CommitTrigger">Hook GitHub to trigger Build on commit</a>
   0. <a href="#SlackNotification">Slack notification</a>

   0. <a href="#DefineAWS">Define AWS Test and Prod instances</a>
   0. <a href="#DeployAWS">Auto Deploy to AWS</a>
   0. <a href="#VaryDockerOptions">Vary Docker Options</a>
   0. <a href="#SmokeTests">Smoke Test Trigger</a>


<a name="InGitHub"></a>

## Get into GitHub #
 
For sample apps we have 2 micro-services:

   0. A cron service called <strong>Box</strong>. 
   This updates into MongoDB a periodic heart beat 
   to show it's still up and running<br />
   (https://github.com/avinci/box)

   0. A visualizer UI called <strong>dv</strong>
   that connects to the MongoDB Box updated by Box and 
   draws whatever cron services are updating TTL<br />
   (https://github.com/avinci/dv)

We have two apps because we want to see when
one of its underlying services i.e. box or dv changes,
only the changed image should be deployed.
We don’t want to deploy the entire app when one component changes.


QUESTION: What is eye2eye now?

<a name="Dockerize"></a>

## Dockerize the app #


<a name="JenkinsIn"></a>

## Setup Jenkins Instance #
 

<a name="TagDocker"></a>

## Add Docker semantic tags

   * <a target="_blank" href="https://docs.docker.com/engine/reference/commandline/tag/">
   About Docker tags</a>


<a name="Push2Dockerhub"></a>

## Push into Dockerhub #
 


<a name="TagGitHub"></a>

## Tag GitHub release #

* Tester flags a build as ready to add to a release, release manager approves and adds it to a release


https://help.github.com/articles/creating-releases/

 
<a name="CommitTrigger"></a>

## Hook GitHub to trigger Build on commit #

- Now we create a APP release anytime any of the BOX or DV images change and 
this is also a semantic version number that is independent of the above

- We then deploy this Release into a AWS cluster called Test. 
Deployment is automatic every time a new release is created

- We need have a manual step where the release that is currently deploy to test, 
will be updated with “RC” tag.


<a name="JenkinsAutoTests"></a>

## Jenkins Auto Test Trigger #
 
* Trigger automated tests of build 



<a name="VaryDockerOptions"></a>

## Vary Docker Options #

- We also want to use different docker options for BOX in test vs prod

- we also want to use different env vars for BOX in test vs prod


<a name="SlackNotification"></a>

## Slack notification #
 
- we want to get notified into a slack room called test and prod 
depending on what env what service and what version is getting deployed

QUESTION: Which room?  We need a public one not associated with a vendor.


<a name="DefineAWS"></a>

## Define AWS Test and Prod instances #


 
<a name="DeployAWS"></a>

## Auto Deploy to AWS #

- We should then be able to trigger a manual deployment into production by 
supplying a valid RC release from the list of RC tagged releases


<a name="SmokeTests"></a>

## Smoke Test Trigger #
 
Trigger automated tests after successful deployment to a Test environment


<a name="FullReleaes"></a>

## Full Release #

* Manage a full release (e.g. combination of versions across multiple services)


<a name="Artifactory"></a>

## Artifactory #

QUESTION: We using this?
