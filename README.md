# Installation methods

<details>
<summary>Install Nexus using Yum</summary>

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