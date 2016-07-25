This repo provides you step-by-step instructions 
to get a new custom program running on a server in the cloud.
We then make a change and update the server through the automated 
<a href="#Pipeline">""pipepline"</a>. 

The intent of this effort is to vary the 
components deployed so you can realistically compare 
the true effort needed for each choice of technologies.

   * Repositories GitHub vs. GitLab vs. Bitbucket
   * Continuous integration utilities Jenkins vs. Circle-CI vs. Shippable, etc.
   * Scripting with Maven vs. Ansible declarative statements
   * Bare metal vs. virtual instances vs. Docker containers
   * Operating system CentOS vs. Ubuntu vs. RedHat 
   * Build from scratch vs. Dockerhub vs. reuse builds in Artifactory
   * Instantiate using Kubernetes
   * Instantiate database as a separate server vs. mlab vs. compose clouds
   * Databases used by the application MongoDB vs. Postgres, etc.
   * Cloud environment Digital Ocean vs. Heroku vs. AWS vs. Azure vs. Google vs. Rackspace

Let's start with using GitHub, CentOS, Jenkins, on Digital Ocean,
referencing a database in mlab.
These all offer free usage options.

The intended audience for this demo are experienced
<strong>developers</strong> who want to focus on innovation and
<strong>DevOps</strong> cloud system administrators supporting developers.


<a name="Pipeline"></a>

## Pipeline #

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

   0. <a href="#DefineDO">Define Test and Prod instances in Digital Ocean</a>
   0. <a href="#DeployDO">Auto Deploy to Digital Ocean</a>
   0. <a href="#VaryDockerOptions">Vary Docker Options</a>
   0. <a href="#SmokeTests">Smoke Test Trigger</a>


<a name="InGitHub"></a>

## App source into GitHub #

<a title="shippable sample app 20160725-192x294-c71" href="https://cloud.githubusercontent.com/assets/300046/17109659/c4ef30e8-524d-11e6-8a97-eb816206a1ef.png">
<img align="right" width="192" height="294" src="https://cloud.githubusercontent.com/assets/300046/17109659/c4ef30e8-524d-11e6-8a97-eb816206a1ef.png">
                                             
For sample apps we have 2 micro-services:

   0. A cron service called <strong>Box</strong>. 
   This updates into MongoDB a periodic heart beat 
   to show it's still up and running<br />
   (<a target="_blank" href="https://github.com/avinci/box">https://github.com/avinci/box</a>)

   0. A visualizer UI called <strong>dv</strong>
   that connects to the MongoDB Box updated by Box and 
   draws whatever cron services are updating TTL (Time To Live)<br />
   (<a target="_blank" href="https://github.com/avinci/dv">https://github.com/avinci/dv</a>)

QUESTION: What about different branches of https://github.com/aye0aye/box

Files in these repos are described when appropriate below.

   * **package.json** defines Node packages to be installed.
   * **server.js** is the file Node invokes.

   * **Gruntfile.js** 

   * **bower.json** and bower_components folder
   * **static** folder

   * **shippable.yml** 
   * **Dockerfile** 

   * **build.sh** 
   * **boot.sh** 
   * **push.sh** 

Because we have two apps, we can see when
one service i.e. box or dv changes,
only the changed image should be deployed.
We don’t want to deploy the entire app when one component changes.


<a name="Dockerize"></a>

## Dockerize the app #

Dockerizing an application is the process of converting an application to run within a Docker container. 

   <pre>
FROM node:0.10.44-slim
ADD . /home/demo/box/
RUN cd /home/demo/box && npm install
ENTRYPOINT ["/home/demo/box/boot.sh"]
   </pre>

While dockerizing most applications is straight-forward, there are a few problems that need to be worked around each time. Two common tasks during dockerization are:

0. To avoid embedding API keys and database login credentials in images,
   move them to a file external from files in the image, such as
   APP_CONFIG=/etc/dev.config

0. Use volumes to bind mount the configuration data at run time

0. Use wrapper scripts that modify configuration data with tools like sed that environment variable

0. Use coding that send application logs to a central logging system via
   STDOUT/STDERR when it defaults to files in the container’s file system.

Some use a small Golang application 
http://github.com/jwilder/dockerize
to simplify the dockerization process.

See http://jasonwilder.com/blog/2014/10/13/a-simple-way-to-dockerize-applications/


<a name="JenkinsIn"></a>

## Setup Jenkins Instance #
 
   * https://hub.docker.com/_/jenkins/
   * https://github.com/jenkinsci/docker
   * https://github.com/docker-library/official-images (stackbrew)
   * https://www.cloudbees.com/blog/jenkins-docker-hub-plugin-image-based-workflows

    docker pull jenkins

<a name="DockerDaemon"></a>

## Run Docker Daemon #

The automated pipeline 

0. On a Mac, use Homebrew cask to install the UI app within a Virtualbox:

   <tt><strong>
   brew cask install dockertoolbox
   </strong></tt>

   The response:

   <pre>
Warning: The default Caskroom location has moved to /usr/local/Caskroom.
&nbsp;
Please migrate your Casks to the new location and delete /opt/homebrew-cask/Caskroom,
or if you would like to keep your Caskroom at /opt/homebrew-cask/Caskroom, add the
following to your HOMEBREW_CASK_OPTS:
&nbsp;
  --caskroom=/opt/homebrew-cask/Caskroom
&nbsp;
For more details on each of those options, see https://github.com/caskroom/homebrew-cask/issues/21913.
==> Satisfying dependencies
==> Installing Cask dependencies: virtualbox
virtualbox ...
==> Downloading http://download.virtualbox.org/virtualbox/5.1.2/VirtualBox-5.1.2
==> Verifying checksum for Cask virtualbox
==> Running installer for virtualbox; your password may be necessary.
==> Package installers may write to any location; options such as --appdir are i
Password:
   </pre>

0. Enter your system password when prompted.

   The additional response:

   <pre>
==> installer: Package name is Oracle VM VirtualBox
==> installer: Upgrading at base path /
==> installer: The upgrade was successful.
🍺  virtualbox was successfully installed!
done
complete
==> Downloading https://github.com/docker/toolbox/releases/download/v1.11.2/Dock
######################################################################## 100.0%
==> Verifying checksum for Cask dockertoolbox
==> Running installer for dockertoolbox; your password may be necessary.
==> Package installers may write to any location; options such as --appdir are i
Password:
==> installer: Package name is Docker Toolbox
==> installer: Installing at base path /
==> installer: The install was successful.
🍺  dockertoolbox was successfully installed!
   </pre>

0. Use docker-machine

   Refer to https://docs.docker.com/machine/get-started/

0. On Linux server, start the Docker daemon background process:

   <tt><strong>
   dockerd
   </strong></tt>

   PROTIP: This would be placed in Jenkins start-up scripts.

   It listens on the Unix socket unix:///var/run/docker.sock

   In most cases, the system administrator configures a process manager such as SysVinit, Upstart, or systemd to manage the docker daemon’s start and stop.

   Volumes are units of highly available block storage that you can attach to your Droplet.
   Use volumes to store assets, backups, databases, and more. 
   Volumes can be moved between Droplets and you can increase their size as needed.

0. Run Docker using a command such as:

   <tt><strong>
   docker run  -t -i ubuntu:14.04 /bin/echo 'Hello world'
   </strong></tt>

   If you don't have the Docker Daemon running, you'll see a message such as:

   <pre>
docker: Cannot connect to the Docker daemon. Is the docker daemon running on this host?.
See 'docker run --help'.
   </pre>

   See https://docs.docker.com/machine/get-started/

   See https://docs.docker.com/v1.8/userguide/dockerizing/

0. Create docker machine:

   docker-machine create --driver virtualbox default

0. List docker machines created:

   <tt><strong>
   docker-machine ls
   </strong></tt>

   The response:

   <pre>
   NAME      ACTIVE   DRIVER       STATE     URL                         SWARM   DOCKER   ERRORS
   default   *        virtualbox   Running   tcp://192.168.99.187:2376           v1.9.1
   </pre>

0. Define

   docker-machine env default

0. Connect to shell:

   eval "$(docker-machine env default)"



<a name="TagDocker"></a>

## Add Docker semantic tags

   * <a target="_blank" href="https://docs.docker.com/engine/reference/commandline/tag/">
   About Docker tags</a>


<a name="Push2Dockerhub"></a>

## Push into Dockerhub #
 
   * Read <a target="_blank" href="https://docs.docker.com/engine/tutorials/dockerrepos/">
   https://docs.docker.com/engine/tutorials/dockerrepos</a>

   * https://jenkins.io/solutions/docker/

   * https://wiki.jenkins-ci.org/display/JENKINS/Docker+Plugin


0. Login:

   <tt><strong>
   docker login
   </strong></tt>


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


<a name="DefineDO"></a>

## Define Test and Prod instances in Digital Ocean #

0. Sign up for an account at DigitalOcean.com

0. Provide a credit card or Paypal account enough for a $5 droplet.

0. Open a Terminal shell window to a folder where you generate SSH key pair:

   <tt><strong>
   cd ~/.ssh
   </strong></tt>

   See https://www.digitalocean.com/community/tutorials/how-to-use-ssh-keys-with-digitalocean-droplets

   <tt><strong>
   ssh-keygen -t rsa
   </strong></tt>

0. Save the file as "do_s" or another file of your choosing.
   Unlike other programs, we won't be using the default file names on your machine.

0. Copy the public key file into your clipboard:

   <tt><strong>
   pbcopy < ~/.ssh/do_s.pub
   </strong></tt>

   Alternately, use a text editor to open the file to copy, such as:

   <tt><strong>
   atom ~/.ssh/id_rsa.pub
   </strong></tt>

0. Switch back to the Create Droplet webpage to click "New SSH Key".

0. Click in the box and press command+V to paste.

0. Specify a name.

0. Select the location, etc.

0. Select Droplet image.

   QUESTION: Select my own.

0. Copy the hostname to your clipboard and paste to your notes.

   ubuntu-512mb-nyc1-01

0. Click the green Create button.

0. Copy the IP Address to your clipboard and paste to your notes.
   For example:

   192.241.155.147

0. Click More to Access Console.

   NOTE: Each Droplet spun up is a new VPS for your personal use.

   Alternately,

   <tt><strong>
   ssh root@192.241.155.147
   </strong></tt>

   The first time you connect, you'll see a message such as:

   <pre>
The authenticity of host '192.241.155.147 (192.241.155.147)' can't be established.
ECDSA key fingerprint is SHA256:sdfsdfsdfu5+Xsaf4COHb0UaRTxeoycUh4tLj1kwNQ.
Are you sure you want to continue connecting (yes/no)? 
   </pre>

   Type <strong>yes</strong> because this is expected behavior.

0. Enter your password.

0. Click API at the top menu.

   Register the app.

0. Add image

   See https://cloud.digitalocean.com/images/snapshots

Aditionally: 

   * https://www.docker.com/products/docker-toolbox
   * https://www.digitalocean.com/support
   * https://www.digitalocean.com/community

<a name="DeployDO"></a>

## Auto Deploy to Digital Ocean #

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
