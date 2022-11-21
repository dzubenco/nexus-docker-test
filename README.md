# Installation methods

Official documentation:
[https://help.sonatype.com/repomanager3/installation-and-upgrades](https://help.sonatype.com/repomanager3/installation-and-upgrades)

<details>
<summary><h4>Install Nexus using Yum</h4></summary>

## Prerequisite steps:
* Rocky Linux 8

## Installation steps:
### 1. Install Sonatype Nexus 3 repo and Nexus 3 itself

following the steps from https://github.com/sonatype-nexus-community/nexus-repository-installer#yum-setup

### 2. Fine-tune the memory requirements

by editing the file `/opt/sonatype/nexus3/bin/nexus.vmoptions` (for weaker systems set

```
-Xms512m
-Xmx512m
-XX:MaxDirectMemorySize=512m
```

### 3. Enable and start Nexus 3 service

via `sudo systemctl enable nexus-repository-manager --now`

### 4. Login to Nexus as `admin`.

To ensure the system begins with a secure state, Nexus Repository Manager generates a unique random password during the system’s initial startup which it writes to the data directory (in our case it's "sonatype-work/nexus3") in a file called admin.password.

So you can use the value from this file:

`sudo cat /opt/sonatype/sonatype-work/nexus3/admin.password`

And then go to http://your_host:8081/ in your browser to log in as "admin" user using the password from the file above.
</details>

<details>
<summary><h4>Install Nexus manually</h4></summary>

## Prerequisite steps:

* Install wget utility in case if you don't have it:
```
sudo yum install wget -y

```

* Install OpenJDK 1.8 in case if you don't have it (to check the version run "java -version")
```
sudo yum install java-1.8.0-openjdk.x86_64 -y
```

## Installation steps: 

**1) Move to your /opt directory**
```
cd /opt
```

**2) Download the latest version of Nexus**

You can get the latest download links for nexus from [here](https://help.sonatype.com/repomanager3/product-information/download) (for example, *https://download.sonatype.com/nexus/3/nexus-3.38.1-01-unix.tar.gz*)
```
sudo wget -O nexus.tar.gz https://download.sonatype.com/nexus/3/latest-unix.tar.gz
```

**3) Extract the tar file**
```
sudo tar -xvzf nexus.tar.gz
```
You should see two directories: nexus files directory (it's name is "nexus-3.20.1-01" at the screenshot below) and nexus data directory (it's name is "sonatype-work" at the screenshot below).

![1.png](images/1.png)

Rename the nexus files directory
```
sudo mv nexus-3* nexus
```

**4) Create new user which will run the service**

As a good security practice, it is not advised to run nexus service with root privileges. So create a new user named "nexus" to run the nexus service
```
sudo adduser nexus
```

Change the ownership of nexus files directory and nexus data directory to nexus user
```
sudo chown -R nexus:nexus /opt/nexus
sudo chown -R nexus:nexus /opt/sonatype-work
```

**5) Edit "nexus.rc" file**

Open /opt/nexus/bin/nexus.rc file
```
sudo vi  /opt/nexus/bin/nexus.rc
```

Uncomment run_as_user parameter and set it as follows
```
run_as_user="nexus"
```

**6) Edit "nexus.vmoptions"**

Open the file in editor
```
sudo vi /opt/nexus/bin/nexus.vmoptions
```

In case if you need to change the default nexus data directory You need to adjust the "-Dkaraf.data" value .

Also I've notices that the service is not starting at all without any logging in case if it's not enough memory to start. So if default values of "-Xms" and "-Xmx" are too huge, you'd need to decrease them.

Below are the values I've used in my setup:
```
-Xms512m
-Xmx512m
-XX:MaxDirectMemorySize=512m
-XX:+UnlockDiagnosticVMOptions
-XX:+LogVMOutput
-XX:LogFile=../sonatype-work/nexus3/log/jvm.log
-XX:-OmitStackTraceInFastThrow
-Djava.net.preferIPv4Stack=true
-Dkaraf.home=.
-Dkaraf.base=.
-Dkaraf.etc=etc/karaf
-Djava.util.logging.config.file=etc/karaf/java.util.logging.properties
-Dkaraf.data=../sonatype-work/nexus3
-Dkaraf.log=../sonatype-work/nexus3/log
-Djava.io.tmpdir=../sonatype-work/nexus3/tmp
-Dkaraf.startLocalConsole=false
-Djdk.tls.ephemeralDHKeySize=2048
-Dinstall4j.pidDir=/opt/tmp
-Djava.endorsed.dirs=lib/endorsed
```

**7) Start the service**

You can configure the repository manager to run as a service with "init.d" or "systemd".

Both these methods you can find described at the following [page](https://help.sonatype.com/repomanager3/installation-and-upgrades/run-as-a-service).

In this guide we will use "update-rc.d" - a tool that targets the initscripts in "init.d" to run the nexus service.

Symlink "opt/nexus/bin/nexus" to "/etc/init.d/nexus":
```
sudo ln -s /opt/nexus/bin/nexus /etc/init.d/nexus
```

Then activate the service
```
cd /etc/init.d
sudo update-rc.d nexus defaults
sudo service nexus start
```

**Note:** default settings of Port and Host values which nexus uses once the service is started can be found in "/opt/nexus/etc/nexus-default.properties":

![2.png](images/2.png)


**Post install:** Login as admin to Nexus

To ensure the system begins with a secure state, Nexus Repository Manager generates a unique random password during the system's initial startup which it writes to the data directory (in our case it's "sonatype-work/nexus3") in a file called admin.password.

So you can use the value from this file:
```
sudo vi /opt/sonatype-work/nexus3/admin.password
```

And then go to http://your_host:8081/ in your browser to log in as "admin" user using the password from the file above.
</details>


<details> 
<summary><h4>Run Nexus as a Docker container</h4></summary>

## 

You can find instructions at:
[https://github.com/sonatype/docker-nexus3](https://github.com/sonatype/docker-nexus3)

Or create a docker-compose file similar to the following:
[link](https://github.com/dzubenco/nexus-docker-test/blob/main/docker-compose.yml)

Then run via the following commands:

```
docker-compose pull
docker-compose up -d
```

It can take some time (2-3 minutes) for the service to launch in a new container. 
You can the status using the following command  to determine once Nexus is ready:

```
docker-compose ps
```


</details>

<details>
<summary><h4>Install Nexus instance within a Kubernetes cluster</h4></summary>

## Links to the Charts:

Official Helm chart:
[https://artifacthub.io/packages/helm/sonatype/nexus-repository-manager](https://artifacthub.io/packages/helm/sonatype/nexus-repository-manager)

Community Helm chart:
[https://artifacthub.io/packages/helm/stevehipwell/nexus3](https://artifacthub.io/packages/helm/stevehipwell/nexus3)
</details>


# Backup and restore (blob stores and database)

https://help.sonatype.com/repomanager3/planning-your-implementation/backup-and-restore


# Post-install steps

<details>
<summary><h4>Anonymous access & Local Authorizing Realm</h4></summary>

#
During initial configuration of Nexus repository you should remain the following checkbox and choose "Local Authorizing Realm" in the Realm dropdown:

![3.png](images/3.png)

In case if you've missed this, you can find this setting in the [ 1) Admin panel -> 2) Anonymous access ] panel as shown below:

![4.png](images/4.png)

Then go to [ 1) Admin panel -> 2) Realms ] and add Local Authorizing Realm to the active block.

</details>


<details> 
<summary><h4>Create Cleanup policies for each types of repo</h4></summary> 

#
1) Login to Nexus as Admin

2) Navigate to Admin panel at the very top of Nexus UI

![5.png](images/5.png)

3) At the Repository section choose "Cleanup policies"

![6.png](images/6.png)

4) Click at the "Create Cleanup Policy" button

The next steps (as an example) will be described for a maven type of repository:

![7.png](images/7.png)

**1)** Specify the name of cleanup policy --> **2)** Choose the type of repository (at the screenshot above it's maven2) --> **3)** Choose Cleanup criteria (at the screenshot above it's about to delete components that haven't been downloaded in 3 days)

These steps should be repeated for all the type of repositories for which you need to have a cleanup job configured.
In my case it's the following list: apt, conda, docker, helm, maven, npm, pypi

![8.png](images/8.png)

</details>

<details>
<summary><h4>Setup cleanup tasks (!)</h4></summary>

#
1) Login to Nexus as Admin

2) Navigate to Admin panel at the very top of Nexus UI

![55.png](images/5.png)

3) At the System section choose "Tasks"

![20.png](images/20.png)

4) Click on "Create task" button

![21.png](images/21.png)

5) Choose **"Cleanup Service (Admin - Cleanup repositories using their associated policies)"**

This task will clean up all the items which are valid to be cleaned up according to the **cleanup policies** set up for each repository separately.

- define the name of the task

- define frequency of running (e.g. daily)

6) Choose **"Admin - Compact Blob Store"**

It's a kind of hard delete. Any clean up (task, or individual deletions) done via NXRM3 is soft deleted in case it's removed the wrong thing, you can restore. Compact blob store task finishes the job (removing all soft deleted items).

- define the name of the task

- choose the blobstore this task will be applied to

- define frequency of running (e.g. daily)

7) Choose **"Delete blob store temporary files"**

- define the name of the task

- choose the blobstore this task will be applied to

- define frequency of running (e.g. daily)

8) Choose **"Docker - delete incomplete upload tasks"**

- define the name of the task

- define "age in hours" value - incomplete docker uploads that are older than the specified age in hours will be deleted

- define frequency of running (e.g. daily)

9) Choose **"Docker - delete unused manifests and images"**

- define the name of the task

- provide the nexus repository name to clean up

- define "deploy offset in hours" value - Manifests and images deployed within this period before the task starts will not be deleted

- define frequency of running (e.g. daily)

</details>

<details>
<summary><h4>Setup Users and User roles for contributing</h4></summary>

#
Example will contain info about how to create **user role** and **user** that are able to download/upload artifacts to **Docker** Nexus repositories.

Login as Admin user -> Go to Admin Panel -> Expand "Security" section -> Choose "Roles" -> Click "Create Role"

Provide the ID and Name of User role, move the following Privileges from Active to Given section as at teh screenshot:

```
nx-repository-admin-docker-docker-group-*
nx-repository-admin-docker-docker-hosted-*
nx-repository-view-docker-docker-group-*
nx-repository-view-docker-docker-hosted-*
```

![18.png](images/18.png)

Click "Save" button.

Go to Admin Panel -> Expand "Security" section -> Choose "Users" -> Click "Create Local User"

Fill the form: 

ID: any description e.g. "docker-contributor"

First Name, Last Name, Email: any dummy values

Password: it will be used for authentication

Status: choose "active"

Roles: move previously created role (in my case it's "docker-contributor") from "Available" section to "Granted"

Save the user

![19.png](images/19.png)


</details>


# Setup Docker repositories

Nexus Repository Manager support Docker registries as the Docker repository format for hosted and proxy repositories. Official documentation from Sonatype on how to proxy Docker: [link](https://help.sonatype.com/repomanager3/nexus-repository-administration/formats/docker-registry/proxy-repository-for-docker)

Prerequisite: Go to "server administration and configuration" section -> Choose "Security" -> "Realms" option on the left sidebar -> Add Docker Bearer Token Realm to the active block

<details>
<summary><h4>Setup Proxy Docker repository</h4></summary>

#
Go to "server administration and configuration" section -> Choose "repositories" option on the left sidebar, then click "create repository" button at the very top of the screen -> Choose "docker (proxy)" type

![9.png](images/9.png)

1) Provide the name of proxy

2) Check the "HTTP" checkbox and provide a Port value you may use for this repository (at the screenshot it's 8181)

3) Check "allow anonymous docker pull"

4) Provide the URL of the remote storage (for example, https://registry-1.docker.io). Note: each proxy repository can use only one remote storage

5) For the Docker index, select Use Docker Hub

6) Check "Allow Nexus Repository Manager to download and cache foreign layers" checkbox (info: [link](https://help.sonatype.com/repomanager3/nexus-repository-administration/formats/docker-registry/foreign-layers)). Remain the regexp by default

7) Please don't forget to apply to the repository the cleanup policy which has been created at the [Post-Install steps] -> [Create Cleanup policies for each types of repo] section of this guide

![10.png](images/10.png)

</details>

<details>
<summary><h4>Setup Hosted Docker repository</h4></summary>

#
If you want to have an ability to push your own Docker images to the Nexus, you would need to have Hosted Repository set up.

The creation of Hosted Docker repository in Nexus is pretty similar to the Proxy Docker repository set up described above.

The differences are that:

1) When choosing the repository type to be created, choose "docker (hosted)"

2) Provide a name of repository, choose the blobstore (or remain it default) and apply a cleanup policy if needed (it should be set up as at the [Post-Install steps] -> [Create Cleanup policies for each types of repo] section of this guide)

3) Don't forger to provide a HTTP connector at specified port as at the screenshot below. The port should be different from other HTTP connector ports specified for other created repos.

![11.png](images/11.png)

Then you will be able to push your own images to such repository. 
Example of pushing to Docker hosted repo can be found at the **"Client configuration & How to use"** section below

</details>

<details>
<summary><h4>Setup Group Docker repository</h4></summary>

#
Several Docker repositories can be grouped in order to simplify access if you're going to use different remote repos at the same time.
For more details please refer to the [guide](https://help.sonatype.com/repomanager3/nexus-repository-administration/formats/docker-registry/grouping-docker-repositories) .

In our case, Nexus contains the Docker Group repository which includes all the Proxy Docker repos and Hosted Docker repo. 
So, accessing the only one HTTP connector of Group repository, we will be able to **download** any image from all these repos (please **note** that Nexus Repository OSS **does not support pushing** into a group repository, so only pulling from group repository is available. Explicit push to the hosted repository is described in the **"Client configuration & How to use"** section below):

![12.png](images/12.png)
</details>

<details>
<summary><h4>Client configuration & How to use</h4></summary>

#

1) Go to /etc/docker/daemon.json and change it's content as follows:
```
{ "features" : { "buildkit": true},
"insecure-registries": ["nexus_address:http_connector_group_repo", "http://nexus_address:http_connector_group_repo", "nexus_address:http_connector_hosted_repo", "http://nexus_address:http_connector_hosted_repo"],
"registry-mirrors": ["http://nexus_address:http_connector_group_repo", "http://nexus_address:http_connector_hosted_repo"],
"debug": true
 }

```

for example, in my case it would be:

```
{ "features" : { "buildkit": true}, 
"insecure-registries": ["localhost:8183", "http://localhost:8183", "localhost:8182", "http://localhost:8182"], 
"registry-mirrors": ["http://localhost:8183", "http://localhost:8182"], 
"debug": true 
 }
```

2) Create a file /etc/default/docker and put the following line:

```
DOCKER_OPTS="--config-file=/etc/docker/daemon.json"
```

![13.png](images/13.png)

3) Go to **~/.docker/config.json**. In case if it contains a record with docker.io, delete it (otherwise docker will work with docker hub instead of proxy)

4) Restart the docker service:

```
sudo systemctl restart docker
```

Run docker info command:

```
docker info
```

At the bottom you should see records similar to the following:

![14.png](images/14.png)

Now if you run in your console:

```
docker pull
# or
docker push
```

your docker will point to the Nexus instance.

**Example of pushing** to Docker hosted repo:
General approach is described [here](https://help.sonatype.com/repomanager3/nexus-repository-administration/formats/docker-registry/pushing-images)

I've chose one of the available images for pushing:

![15.png](images/15.png)

Then made a tag:

![16.png](images/16.png)

Then authenticated as "docker-contributer" user (password: 123123123) and pushed the image:

![17.png](images/17.png)

</details>

# Setup Helm repositories

Official documentation from Sonatype on how to proxy Helm: [link](https://help.sonatype.com/repomanager3/nexus-repository-administration/formats/helm-repositories)

<details>
<summary><h4>Setup Proxy Helm repository</h4></summary>

#
Note: each created repository can proxy only one remote repository.

The list of Helm repositories for proxying:

```
https://oxyno-zeta.github.io/helm-charts-v2/
https://argoproj.github.io/argo-helm/
https://charts.bitnami.com/bitnami
https://aws.github.io/eks-charts
https://charts.crossplane.io/stable
https://charts.bitnami.com/bitnami
https://dapr.github.io/helm-charts
```

In general proxy repository can be set up as follows:

![22.png](images/22.png)

1) Go to "server administration and configuration" section

2) Choose "repositories" option on the left sidebar, then click "create repository" button at the very top of the screen

3) Choose "helm (proxy)" type

![23.png](images/23.png)

1) Provide the name of proxy

2) Provide the URL of the remote storage (for example,  https://kubernetes-charts.storage.googleapis.com/ )

3) (Optional, can be remained by default) Choose a blob store for the repository if you need to separate it from the default one.

4) Please don't forget to apply to the repository the cleanup policy which has been created at the **cleanup policies section** of this guide

![24.png](images/24.png)

As a result, repository like this should appear:

![25.png](images/25.png)

</details>

<details>
<summary><h4>Setup Hosted Helm repository</h4></summary>

#

If you want to have an ability to push your own Helm charts to the Nexus, you would need to have Hosted Repository set up.

The creation of Hosted Helm repository in Nexus is pretty similar to the **Proxy Helm repository** creation.

The differences are that:

1) When choosing the repository type to be created, choose "helm (hosted)"

2) Provide a name of repository, choose the blobstore (or remain it default) and apply a cleanup policy if needed (it should be set up as above in the **cleanup policies setup** section)

</details>

<details>
<summary><h4>Client configuration & How to use</h4></summary>

#

### **How to fetch Helm charts from helm-proxy repo**

Once you have Helm up and running you'll want to run a command similar to the following to add a Helm repo:
```
helm repo add <helm_repository_name> http://<host>:<port>/repository/<nexus_repository_name>/ --username <username> --password <password>
```

The below command will fetch the latest chart or with the version:
```
1. helm fetch <helm_repository_name>/<chart_name>
2. helm fetch <helm_repository_name>/<chart_name> --version <chart_version>
```

For example, Nexus Repository has a Helm proxy repository called **helm-proxy** and your Nexus Repository is running on localhost:8081 where username is admin and password is admin. 
You would like to add this repository to Helm client. Also, you would like to fetch the latest MySQL chart. To accomplish this, you would do the following:
```
1. helm repo add nexusrepo http://localhost:8081/repository/helm-proxy/ --username admin --password admin
2. helm fetch nexusrepo/mysql
```

If you want to fetch a chart with a specific version, just run it, like so:
```
helm fetch nexusrepo/mysql --version 1.4.0
```

---
### **How to push Helm charts to the helm-hosted repo**

![26.png](images/26.png)

1) I've created test chart

2) I've checked that chart directory has been created with default content

3) Made an archive out of the chart

Then, using "helm-contributor" user with "123123123" password we can push the chart to the helm-hosted repo.
The following command should be used:

```
curl -X 'POST' \
  'http://localhost:8082/service/rest/v1/components?repository=helm-hosted' \
  -u 'helm-contributor:123123123' \
  -H 'accept: application/json' \
  -H 'Content-Type: multipart/form-data' \
  -F 'helm.asset=@test_chart-0.1.0.tgz;type=application/x-compressed-tar'
```
</details>

# Setup Maven repositories

<details>
<summary><h4>Setup Proxy Maven repository</h4></summary>

#

</details>

<details>
<summary><h4>Setup Hosted Maven repository</h4></summary>

#

</details>

<details>
<summary><h4>Setup Group Maven repository</h4></summary>

#

</details>

<details>
<summary><h4>Client configuration & How to use</h4></summary>

</details>

# Setup Conda repositories

<details>
<summary><h4>Setup Proxy Conda repository</h4></summary>

#

</details>

<details>
<summary><h4>Client configuration & How to use</h4></summary>

</details>

# Setup Npm repositories

<details>
<summary><h4>Setup Proxy Npm repository</h4></summary>

#

</details>

<details>
<summary><h4>Setup Hosted Npm repository</h4></summary>

#

</details>

<details>
<summary><h4>Setup Group Npm repository</h4></summary>

#

</details>

<details>
<summary><h4>Nginx configuration to redirect npm audit</h4></summary>

</details>

<details>
<summary><h4>Client configuration & How to use</h4></summary>

</details>

# Setup Pypi repositories

<details>
<summary><h4>Setup Proxy Pypi repository</h4></summary>

#

</details>

<details>
<summary><h4>Setup Hosted Pypi repository</h4></summary>

#

</details>

<details>
<summary><h4>Setup Group Pypi repository</h4></summary>

#

</details>

<details>
<summary><h4>Client configuration & How to use</h4></summary>

</details>

# Debian

# How to configure S3 Blobstore in Nexus