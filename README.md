# Anaconda Repo Runbook

The following runbook walks through the steps needed to install Anaconda Repo. The runbook is designed for two audiences: those who have direct access to the internet for installation and those where such access is not available or restricted for security reasons. For these restricted a.k.a. "Air Gap" environments, Continuum ships the entire Anaconda Repo repository to the organization on portable storage medium or as a downloadable TAR archive.  Where necessary, additional instructions for Air Gap environments are noted. If you have any questions about the instructions, please contact your sales representative or Priority Support team, if applicable, for additional assistance.

![](https://www.lucidchart.com/publicSegments/view/591eb3b5-b326-49fb-addd-d733e6ceae18/image.png)
## Requirements
### Hardware Requirements

* Physical server or VM
* CPU: 2 x 64-bit 2 2.8GHz 8.00GT/s CPUs or better
* Memory: 32GB RAM (per 50 users)
* Storage: Recommended minimum of 300GB; Additional space is recommended if the repository is will be used to store packages built by the customer.
* Software Requirements
* RHEL/CentOS 6.7 (Other operating systems are supported, however this document assumes RHEL or CentOS 6.7)
* MongoDB version 2.6
* Anaconda Repo license file - given as part of the welcome packet - contact your sales representative or support representative if you cannot find your license.
* Security Requirements
* Privileged (root) access or sudo capabilities
* Ability to make (optional) iptables modifications

**NOTE**: SELinux does not have to be disabled for Anaconda Repo operation 
### Network Requirements
#### TCP Ports
* Inbound TCP 8080 (Anaconda Repo)
* Inbound TCP 22 (SSH)
* Outbound TCP 443 (Anaconda Cloud)
* Outbound TCP 25 (SMTP)
* Outbound TCP 389/636 (LDAP(s))

### Other Requirements
Assuming the above requirements are met, there are no additional dependencies necessary for Anaconda Repo.

### Air Gap Media
This document assumes that the Air Gap media is located at /installer on the server where the installation is taking place. 
Air Gap media contents:

```
/installer
└── anaconda-suite
   ├── archive
   ├── miniconda
   └── pkgs
mongodb-org-tools-2.6.8-1.x86_64.rpm
mongodb-org-shell-2.6.8-1.x86_64.rpm
mongodb-org-server-2.6.8-1.x86_64.rpm
mongodb-org-mongos-2.6.8-1.x86_64.rpm
mongodb-org-2.6.8-1.x86_64.rpm
```
**NOTE:** Many of the steps below contain special instructions for Air Gap installs, denoted by an "**Air Gap**" section immediately following the step in question.

## Anaconda Repo Installation
The following sections detail the steps required to install Anaconda Repo.

### 1. Install MongoDB
##### 1.1. Download MongoDB packages:

* **Air Gap Installation:** Skip this step.

* **Regular Installation:**

        RPM_CDN="https://820451f3d8380952ce65-4cc6343b423784e82fd202bb87cf87cf.ssl.cf1.rackcdn.com"
        curl -O $RPM_CDN/mongodb-org-tools-2.6.8-1.x86_64.rpm
        curl -O $RPM_CDN/mongodb-org-shell-2.6.8-1.x86_64.rpm
        curl -O $RPM_CDN/mongodb-org-server-2.6.8-1.x86_64.rpm
        curl -O $RPM_CDN/mongodb-org-mongos-2.6.8-1.x86_64.rpm
        curl -O $RPM_CDN/mongodb-org-2.6.8-1.x86_64.rpm


##### **1.2.** Install MongoDB packages:

* **Air Gap Installation:**
    
        sudo yum install -y /installer/mongodb-org*

* **Regular Installation:**

        sudo yum install -y mongodb-org*



##### **1.3.** Start mongodb:


    sudo service mongod start


##### **1.4.** Verify mongod is running:

    sudo service mongod status
    mongod (pid 1234) is running...


**NOTE:** Additional mongodb installation information can be found here.

### 2. Create Anaconda Repo administrator account

In a terminal window, create a new user account for Anaconda Repo named “binstar”

    sudo useradd -m binstar

**NOTE:** The binstar user is the default for installing Anaconda Repo. Any username can be used, however the use of the root user is discouraged.

### 3. Create Anaconda Repo directories

    sudo mkdir -m 0770 /etc/binstar
    sudo mkdir -m 0770 /var/log/anaconda-server
    sudo mkdir -m 0770 -p /opt/anaconda-server/package-storage
    sudo mkdir -m 0770 /etc/binstar/mirrors

### 4. Give the binstar user ownership of directories:

    sudo chown -R binstar. /etc/binstar
    sudo chown -R binstar. /var/log/anaconda-server
    sudo chown -R binstar. /opt/anaconda-server/package-storage
    sudo chown -R binstar. /etc/binstar/mirrors
    
### 5. Switch to the Anaconda Repo administrator account

    sudo su - binstar

### 6. Install Miniconda bootstrap version
##### **6.1.** Fetch the download script using curl: 

* **Air Gap Installation:** Skip this step.

* **Regular Installation:**

        curl 'http://repo.continuum.io/miniconda/Miniconda-latest-Linux-x86_64.sh' > Miniconda.sh

##### **6.2.** Run the Miniconda.sh installer script:

* **Air Gap Installation:** Skip this step.
        
        sh /installer/anaconda-suite/miniconda/Miniconda-latest-Linux-x86_64.sh
        
* **Regular Installation:**

        sh Miniconda.sh

##### **6.3.** Review and accept the license terms:

```
Welcome to Miniconda (by Continuum Analytics, Inc.)
In order to continue the installation process, please review the license agreement.  
Please, press ENTER to continue. Do you approve the license terms? [yes|no] yes
```
##### **6.4.** Accept the default location or specify an alternative:

```    
Miniconda will now be installed into this location:
/home/binstar/miniconda2  
-Press ENTER to confirm the location 
-Press CTRL-C to abort the installation
-Or specify a different location below
 [/home/binstar/miniconda2] >>>" [Press ENTER]
 PREFIX=/home/binstar/miniconda2
```

##### **6.5.** Update the binstar user's path:

```
Do you wish the installer to prepend the Miniconda install location to PATH in your /home/binstar/.bashrc ? [yes|no] yes
```

##### **6.6.** For the new path changes to take effect, “source” your .bashrc:
```
source ~/.bashrc
```

### 7. Install Anaconda Repo Enterprise Packages

##### **7.1.** Install the Anaconda Repo channel:

* **Air Gap Installation:** Install the Anaconda Repo channel from local files.

        conda config --add channels  file:///installer/anaconda-suite/pkgs/
        conda config --remove channels defaults --force

* **Regular Installation:**  Install the Anaconda Repo channel from Anaconda Cloud.

        export TOKEN=<your Anaconda Cloud token>
        conda config --add channels https://conda.anaconda.org/t/$TOKEN/binstar/
        conda config --add channels https://conda.anaconda.org/t/$TOKEN/anaconda-server/

Where **“TOKEN”** is the Anaconda Repo token you should have received from Continuum Support. This adds the correct channels to conda by updating the `/home/binstar/.condarc` file.


##### **7.2.** Install the Anaconda Repo packages via conda:

    conda install anaconda-client binstar-server binstar-static cas-mirror

### 8.Configure Anaconda Repo
##### **8.1.** Initialize the web server for Anaconda Repo:

    anaconda-server-config --init

##### **8.2.** Set the Anaconda Repo package storage location:

    anaconda-server-config --set fs_storage_root /opt/anaconda-server/package-storage

##### **8.3.** Create an initial “superuser” account for Anaconda Repo:

    anaconda-server-create-user --username "superuser" --password "yourpassword" --email "your@email.com" --superuser

**NOTE:** to ensure the bash shell does not process any of the characters in this password, limit the password to lower case letters, upper case letters and numbers, with no punctuation. After setup the password can be changed with the web interface.

##### **8.4.** Initialize the Anaconda Repo database:

    anaconda-server-db-setup --execute

### 9. Set up automatic restart on reboot, fail or error

##### **9.1.** Configure Supervisord:

    anaconda-server-install-supervisord-config.sh

This step:

* creates the following entry in the binstar user’s crontab:
	

    `@reboot /home/binstar/miniconda/bin/supervisord`

	
* generates the `/home/binstar/miniconda/etc/supervisord.conf` file

##### **9.2.** Verify the server is running:

    supervisorctl status

    binstar-server RUNNING   pid 10831, uptime 0:00:05
    binstar-worker RUNNING   pid 2784, uptime 0:00:04

### 10. Install Anaconda Repo License
Visit **http://your.anaconda.server:8080**. Follow the onscreen instructions and upload your license file. Log in with the superuser user and password configured above. After submitting, you should see the login page.

**NOTE:** Contact your sales representative or support representative if you cannot find or have questions about your license. 


### 11. Mirror Anaconda Repo
Now that Anaconda Repo is installed, we want to mirror packages into our local repository. If mirroring from Anaconda Cloud, the process will take hours or longer, depending on the available internet bandwidth. Use the `anaconda-server-sync-conda` command to mirror all Anaconda packages locally under the "anaconda" user account.


* **Air Gap Installation:** Since we're mirroring from a local filesystem, some additional configuration is necessary.


    **1.** Create a mirror config file:

        sudo vi /etc/binstar/mirrors/conda.yaml

     Add the following:
 
        channels:
          - file:///installer/anaconda-suite/pkgs

    **2.** Mirror the Anaconda packages:

        anaconda-server-sync-conda --mirror-config /etc/binstar/mirrors/conda.yaml

* **Regular Installation:** Mirror from Anaconda Cloud.

        anaconda-server-sync-conda

**NOTE:** Depending on the type of installation, this process may take 30-90 minutes. 


To verify the local Anaconda Repo repo has been populated, visit **http://your.anaconda.server:8080/anaconda** in a browser.


### Optional: Mirror the Anaconda Cluster Management channel
If the local Anaconda Repo will be used by Anaconda Cluster nodes (head or compute), the recommended method is to mirror using an “anaconda-cluster” user. To mirror the Anaconda Cluster Management repo, create the mirror config YAML file below:

* **Regular Installation:**

    **1.** Create a mirror config file:

        vi /etc/binstar/mirrors/anaconda-cluster.yaml

    **2.** Add the following:

        channels:
          - https://conda.anaconda.org/t/<TOKEN>/anaconda-cluster

    **3.** Mirror the Anaconda Cluster Management packages:

        anaconda-server-sync-conda --mirror-config /etc/binstar/mirrors/anaconda-cluster.yaml --account=anaconda-cluster



Where **“TOKEN”** is the Anaconda Cluster Mangagement token you should have received from Continuum Support. 

* **Air Gap Installation:**

    **1.** Create a mirror config file:
    
        vi /etc/binstar/mirrors/anaconda-cluster.yaml

    **2.** Add the following:

        channels:
          - file:///installer/anaconda-suite/pkgs

    **3.** Mirror the Anaconda Cluster Management packages:

        anaconda-server-sync-conda --mirror-config /etc/binstar/mirrors/anaconda-cluster.yaml --account=anaconda-cluster


**NOTE:**Ignore any license warnings. Additional mirror filtering/whitelisting/blacklisting options can be found here.














### Optional: Adjust iptables to accept requests on port 80

The easiest way to enable clients to access an Anaconda Repo on standard ports is to configure the server to redirect traffic received on standard HTTP port 80 to the standard Anaconda Repo HTTP port 8080.


**NOTE:** These commands assume the default state of iptables on CentOS 6.7 which is “on” and allowing inbound SSH access on port 22. Take caution; mistakes with iptables rules can render a remote machine inaccessible.


**Allow inbound access to tcp port 80:**

```
sudo iptables -I INPUT -i eth0 -p tcp --dport 80 -m comment --comment "# Anaconda Repo #" -j ACCEPT
```

**Allow inbound access to tcp port 8080:**

```
sudo iptables -I INPUT -i eth0 -p tcp --dport 8080 -m comment --comment "# Anaconda Repo #" -j ACCEPT
```

**Redirect inbound requests to port 80 to port 8080:**

```
sudo iptables -A PREROUTING -t nat -i eth0 -p tcp --dport 80 -m comment --comment "# Anaconda Repo #" -j REDIRECT --to-port 8080
```

**Display the current iptables rules:**

```
sudo iptables -L -n
Chain INPUT (policy ACCEPT)
target     prot opt source               destination         
ACCEPT     tcp  --  0.0.0.0/0            0.0.0.0/0           tcp dpt:8080 /* # Anaconda Repo # */ 
ACCEPT     tcp  --  0.0.0.0/0            0.0.0.0/0           tcp dpt:80 /* # Anaconda Repo # */ 
ACCEPT     all  --  0.0.0.0/0            0.0.0.0/0           state RELATED,ESTABLISHED 
ACCEPT     icmp --  0.0.0.0/0            0.0.0.0/0           
ACCEPT     all  --  0.0.0.0/0            0.0.0.0/0           
ACCEPT     tcp  --  0.0.0.0/0            0.0.0.0/0           state NEW tcp dpt:22 
REJECT     all  --  0.0.0.0/0            0.0.0.0/0           reject-with icmp-host-prohibited 

Chain FORWARD (policy ACCEPT)
target     prot opt source               destination         
REJECT     all  --  0.0.0.0/0            0.0.0.0/0           reject-with icmp-host-prohibited 

Chain OUTPUT (policy ACCEPT)
target     prot opt source               destination  
```

**NOTE:** the PREROUTING (nat) iptables chain is not displayed by default; to show it, use:

```
sudo iptables -L -n -t nat
Chain PREROUTING (policy ACCEPT)
target     prot opt source               destination         
REDIRECT   tcp  --  0.0.0.0/0            0.0.0.0/0           tcp dpt:80 /* # Anaconda Repo # */ redir ports 8080 

Chain POSTROUTING (policy ACCEPT)
target     prot opt source               destination         

Chain OUTPUT (policy ACCEPT)
target     prot opt source               destination       
```

Write the running iptables configuration to /etc/sysconfig/iptables:
 
```
sudo service iptables save
```
