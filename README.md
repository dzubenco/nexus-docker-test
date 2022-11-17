# Installation methods

Official documentation:
[https://help.sonatype.com/repomanager3/installation-and-upgrades](https://help.sonatype.com/repomanager3/installation-and-upgrades)

<details>
<summary>**Install Nexus using Yum**</summary>

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
<summary>Install Nexus manually</summary>

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

![Screenshot_20220505_103625.png](https://bitbucket.org/repo/Ag9LKzq/images/433202221-Screenshot_20220505_103625.png)

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

![Screenshot_20220506_103304.png](https://bitbucket.org/repo/Ag9LKzq/images/1251430095-Screenshot_20220506_103304.png)


**Post install:** Login as admin to Nexus

To ensure the system begins with a secure state, Nexus Repository Manager generates a unique random password during the system's initial startup which it writes to the data directory (in our case it's "sonatype-work/nexus3") in a file called admin.password.

So you can use the value from this file:
```
sudo vi /opt/sonatype-work/nexus3/admin.password
```

And then go to http://your_host:8081/ in your browser to log in as "admin" user using the password from the file above.
</details>


<details> 
<summary>Run Nexus as a Docker container</summary>

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
<summary>Install Nexus instance within a Kubernetes cluster</summary>

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
<summary>Anonymous access & Local Authorizing Realm</summary>

#
During initial configuration of Nexus repository you should remain the following checkbox and choose "Local Authorizing Realm" in the Realm dropdown:

![img](https://bitbucket.org/repo/Ag9LKzq/images/3880051481-Screenshot_20220922_100138.png)

In case if you've missed this, you can find this setting in the [ 1) Admin panel -> 2) Anonymous access ] panel as shown below:

![img](https://bitbucket.org/repo/Ag9LKzq/images/2991281466-Screenshot_20220922_100414.png)

Then go to [ 1) Admin panel -> 2) Realms ] and add Local Authorizing Realm to the active block.

</details>


<details> 
<summary>Create Cleanup policies for each types of repo</summary> 

#
1) Login to Nexus as Admin

2) Navigate to Admin panel at the very top of Nexus UI

![Screenshot_20220922_111946.png](https://bitbucket.org/repo/Ag9LKzq/images/1268056392-Screenshot_20220922_111946.png)

3) At the Repository section choose "Cleanup policies"

![Screenshot_20220922_112117.png](https://bitbucket.org/repo/Ag9LKzq/images/373771384-Screenshot_20220922_112117.png)

4) Click at the "Create Cleanup Policy" button

The next steps (as an example) will be described for a maven type of repository:

![Screenshot_20220922_112522.png](https://bitbucket.org/repo/Ag9LKzq/images/1892634485-Screenshot_20220922_112522.png)

**1)** Specify the name of cleanup policy --> **2)** Choose the type of repository (at the screenshot above it's maven2) --> **3)** Choose Cleanup criteria (at the screenshot above it's about to delete components that haven't been downloaded in 3 days)

These steps should be repeated for all the type of repositories for which you need to have a cleanup job configured.
In my case it's the following list: apt, conda, docker, helm, maven, npm, pypi

![Screenshot_20220922_113509.png](https://bitbucket.org/repo/Ag9LKzq/images/3607471725-Screenshot_20220922_113509.png)

</details>


# Setup Docker repositories

Nexus Repository Manager support Docker registries as the Docker repository format for hosted and proxy repositories. Official documentation from Sonatype on how to proxy Docker: [link](https://help.sonatype.com/repomanager3/nexus-repository-administration/formats/docker-registry/proxy-repository-for-docker)

Prerequisite: Go to "server administration and configuration" section -> Choose "Security" -> "Realms" option on the left sidebar -> Add Docker Bearer Token Realm to the active block

<details>
<summary>Setup Proxy Docker repository</summary>

#
Go to "server administration and configuration" section -> Choose "repositories" option on the left sidebar, then click "create repository" button at the very top of the screen -> Choose "docker (proxy)" type

[Open a screenshot in a fullscreen](https://bitbucket.org/repo/Ag9LKzq/images/792556447-Screenshot_20220516_125027.png)

1) Provide the name of proxy

2) Check the "HTTP" checkbox and provide a Port value you may use for this repository (at the screenshot it's 8181)

3) Check "allow anonymous docker pull"

4) Provide the URL of the remote storage (for example, https://registry-1.docker.io). Note: each proxy repository can use only one remote storage

5) For the Docker index, select Use Docker Hub

6) Check "Allow Nexus Repository Manager to download and cache foreign layers" checkbox. Remain the regexp by default

7) Please don't forget to apply to the repository the cleanup policy which has been created at the [Post-Install steps] -> [Create Cleanup policies for each types of repo] section of this guide
 
</details>

<details>
<summary>Setup Hosted Docker repository</summary>

</details>

<details>
<summary>Setup Group Docker repository</summary>

</details>

<details>
<summary>Client configuration & How to use</summary>

</details>