
- [**Important**:](#important)
- [Before you pull the image](#before-you-pull-the-image)
- [Requirements](#requirements)
  - [Linux](#linux)
  - [Windows](#windows)
  - [macOS](#macos)
- [Installation](#installation)
- [Run](#run)
    - [GNU/Linux](#gnulinux)
    - [Other](#other)
    - [Run troubleshooting](#run-troubleshooting)
    - [Licenses](#licenses)
- [Connection](#connection)
    - [SAPGUI](#sapgui)
    - [User and Passwords](#user-and-passwords)
    - [Browser](#browser)
    - [SAP Cloud Connector](#sap-cloud-connector)
- [Additional Information](#additional-information)
    - [Stop](#stop)
    - [Start again](#start-again)
    - [abapGit](#abapgit)
- [Known issues](#known-issues)
    - [Error message when starting SAP Cloud Connector (SCC)](#error-message-when-starting-sap-cloud-connector-scc)
    - [Creating a new container](#creating-a-new-container)
- [Primary contacts](#primary-contacts)


<h1><a id="important">Important</a></h1> 
To pull the image, click on the tab **Tags** and choose the correct Docker command from there.
**DO NOT** attempt to pull the image from this page (Overview).


<h1><a id="before-you-pull-the-image">Before you pull the image</a></h1>

- You understand the principles of docker container technology
- You know the entities docker image / docker container and their relationship
- You know the basic commands to work with images and containers

<h1><a id="requirements">Requirements</a></h1>

Windows and Mac users must make sure they have assigned enough resources to their Desktop Docker because their Docker runs in a VM which contains GNU/Linux and that underlying VM does not share hardware resources with the host machine without explicit assignment. 

**_Please note: We highly recommend 32GB RAM to run the ABAP Platform Trial image. The following requirements only cover the resources needed for the Docker environment itself._**

<h2><a id="linux">Linux</a></h2>

- 4 CPUs
- 16GB RAM
- 150GB Disk

<h2><a id="windows">Windows</a></h2>

- 4 CPUs for Docker Desktop
- 16GB for Docker Desktop
- 170GB disk for Docker Desktop

<h2><a id="macos">macOS</a></h2>

- 4 CPUs for Docker Desktop
- 16GB for Docker Desktop
- 170GB disk for Docker Desktop

<h1><a id="installation">Installation</a></h1>

Note: To be able to start the system you must agree to SAP DEVELOPER License which will be presented to you when you start the docker container.

Before you start, please make sure you have assigned enough disk space to your Docker set up. The image has around 23GB of size when compressed and >53GB after decompressing. It would be frustrating to lose time downloading the image to only find out docker does not have enough disk space to unpack the image.

To be able to successfully run the system, it's necessary to assign at least 16GB RAM to Docker Desktop.

The image is not available to anonymous users and therefore you must have an account in [Docker Hub](https://hub.docker.com/). To be able to pull the image you must run the command `docker login` or login via Docker Desktop at first.

Finally you can run the command 
```bash
docker pull sapse/abap-platform-trial:<TAGNAME>
```

<h1><a id="run">Run</a></h1>

The system expects host name be *vhcala4hci*, all other host names will prevent the system from starting.
Please, use the following command and watch the output carefully:

<h2><a id="gnulinux">GNU/Linux</a></h2>

```bash
docker run --stop-timeout 3600 -it --name a4h -h vhcala4hci sapse/abap-platform-trial:<TAGNAME>
```

<h2><a id="other">Other</a></h2>

```bash
docker run --stop-timeout 3600 -i --name a4h -h vhcala4hci -p 3200:3200 -p 3300:3300 -p 8443:8443 -p 30213:30213 -p 50000:50000 -p 50001:50001 sapse/abap-platform-trial:<TAGNAME> -skip-limits-check
```

We start the container in interactive mode (*-i*) for being able to stop the system gracefully using the key stroke Ctrl-C. However, we also use the parameter `--stop-timeout` which causes that Docker will give the SAP HANA database (HDB) enough time to write its In-Memory database onto disk upon shutdown request.

We name the container *a4h* for easier reference in future commands.

If you plan to stop and start the container to keep your changes in the system, it is recommended to also use the parameter *-agree-to-sap-license*. The parameter will make sure **you will not need to manually accept the license agreement**.

After all the services are successfully started, it is a good idea to wait until the CPU load goes down and the amount of used Memory stops from growing before you attempt to logon to the system.

<h2><a id="run-troubleshooting">Run troubleshooting</a></h2>

The init process of the container run checks for the correct hostname and for the Linux kernel limits.

In the case you want to skip the Linux kernel limits check, add the parameter `-skip-limits-check` to *docker run* command line you can find above.

If you used the docker run command several lines above on this page and the container exited with the following error message:

```
Cannot continue because of insufficient system limits configuration!

If you want to continue without recommended limits,
run again with the parameter -skip-limits-check
```

Appending the parameter `-skip-limits-check` to the run command and executing the run command again will most probably lead to a container name collision error with the following symptoms:

```
Error response from daemon: Conflict. The container name "/a4h" is already in use by container XYZ. You have to remove (or rename) that container to be able to reuse that name..
```

That's because the first command started a container named *a4h* which immediately exited but docker didn't remove it and now you are trying to start another container with the same name. Correct, that's what we advised you to do. However, to be able to start the container again you must remove the previous instance using the command `docker rm -f a4h` and only then you can issue the enhanced docker run command again.

The following limits are checked:

- kernel.shmmax (allowed for docker run --sysctl) > 21474836480
- kernel.shmmni (allowed for docker run --sysctl) > 32768
- kernel.shmall (allowed for docker run --sysctl) > 5242880
- kernel.msgmni (allowed for docker run --sysctl) > 1024
- kernel.sem (allowed for docker run --sysctl) > 1250 256000 100 8192 
- Number of Opened File descriptors (RLIMIT_NOFILE, ulimit -n, docker run --ulimit nofile=1048576:1048576) > 1048576
- vm.max_map_count (must be set on the host via sysctl on GNU/Linux) > 2147483647
- fs.file-max (must be set on the host via sysctl on GNU/Linux) > 20000000
- fs.aio-max-nr (must be set on the host via sysctl on GNU/Linux) > 18446744073709551615

The limits enabled by Docker can be passed on the *docker run* command line as: 

*--sysctl kernel.shmmax=21474836480 --sysctl kernel.shmmni=32768 --sysctl kernel.shmall=5242880 --sysctl kernel.msgmni=1024 --sysctl kernel.sem="1250 256000 100 8192" --ulimit nofile=1048576:1048576*

The sysctl parameters which are not enabled for modification by Docker must be changed on the Docker host (on the machine where you enter the docker commands). Unfortunately only Linux users can do so and therefore **Mac and Windows users must always use the parameter _-skip-limits-check_**. We apologize for not mentioning this pseudo-requirement sooner, but we wanted to make sure you are fully aware of the fact that there are some limits which were not met and it may have negative effects.

In the case you want to skip the hostname check, add the parameter `-skip-hostname-check` to *docker run* command line.

<h2><a id="licenses">Licenses</a></h2>

**_ABAP Platform (AS ABAP)_**

You can check the expiry date of your ABAP license in the transaction **SLICENSE** in SAPGUI. You may wish to set a reminder to update your license, since it is easier to do so before expiry.

**Updating the license via SAPGUI (SLICENSE)** 
The ABAP license supplied with the Docker image lasts only three months. Therefore, you should download and import the demo license as follows:

1. Logon to your ABAP system with the user SAP*, client 000, same password as for DEVELOPER (DEVELOPER , client 001, is locked).
2. Start transaction SLICENSE; copy the hardware key.
3. Get the license from minisap , choosing the system A4H.
4. Back in your ABAP System, log off, then log on with the user DEVELOPER, client 001.
5. Start SLICENSE again; remove the old invalid licenses. (sap* is not allowed to delete licenses).

**Updating the license via Docker**
The image contains a script which is able to update the AS ABAP license from the file you bind mount or copy to the container. Just save the text file onto your local file system and push it to the container at the path */opt/sap/ASABAP_license*. The hardware key necessary for creation of the license file is printed out during start up phase of the container. 

**Updating via Docker: New container**:  Update the *docker run* command with `-v <local path the key file>:/opt/sap/ASABAP_license`. Please, make sure the *-v* parameter is on your command line before the Docker image name (*sapse/abap-platform-trial:<TAGNAME>*) because the parameter belongs to *docker run* and everything behind the image name is passed to  programs inside the container.
 
**Updating via Docker: Existing container**:  Copy the key file to the container with the command `docker cp <local path the key file> a4h:/opt/sap/ASABAP_license`. If the container was stopped, the file will be applied when you start the container again. If the container is running, you can either stop and start the container or you can trigger the license update functionality via `docker exec -it a4h /usr/local/bin/asabap_license_update`.

If you run into trouble with the AS ABAP license update script, you can prevent the container from executing this functionality by passing the parameter *-no-asabap-license-update* or by creating the file */opt/sap/.no_ASABAP_license_update* in the container.

**_HDB_**

The image is shipped with a valid HDB license; it's not necessary to re-apply this until just before it expires. You can check the expiry date of your HDB license in **DBA Cockpit > System Information > License**. You may wish to set a reminder to update your license, since it is easier to do so before expiry.


The image contains a script which is able to update the HDB license from the file you bind mount or copy to the container. So, if you run into the need to update HDB license, just save the text file onto your local file system and push it to the container at the path */opt/sap/HDB_license*. The hardware key necessary for creation of the license file is printed out during start up phase of the container. 

**New container**:  Update the *docker run* command with `-v <local path the key file>:/opt/sap/HDB_license`.  Please, make sure the *-v* parameter is on your command line before Docker image name (*sapse/abap-platform-trial:<TAGNAME>*) because the parameter belongs to to *docker run* and everything behind the image name is passed to programs inside the container.

**Existing container**:  Copy the key file to the container with the command `docker cp <local path the key file> a4h:/opt/sap/HDB_license`. If the container was stopped, the file will be applied when you start the container again. If the container is running, you can either stop and start the container or you can trigger the license update functionality via `docker exec -it a4h /usr/local/bin/hdb_license_update`.

If you run into troubles with the license update script, you can prevent the container from executing this functionality by passing the parameter *-no-asabap-license-update* or by creating the file */opt/sap/.no_HDB_license_update* in the container.


**Open Source Legal Notices**
[Open Source Legal Notice](https://support.sap.com/content/dam/launchpad/en_us/osln/osln/73555000100900006355_20231015054035.pdf)


<h1><a id="connection">Connection</a></h1>


The following list defines ports used by the container:
- 3200: SAPGUI Instance 00
- 3300: RFC Instance 00
- 8443: SAP Cloud Connector
- 30213: SAP HANA MDC Database
- 50000: AS ABAP HTTP
- 50001: AS ABAP HTTPS

If you need to access the container outside the docker host (from a different machine or a VM) or you are not lucky enough to run GNU/Linux, please expose the ports using the parameter *-p* (the lower case p, case matters) (e.g. for SAPGUI add the following to docker run command: `-p 3200:3200`).

For your convenience, here is the string exposing all relevant ports which you can copy and paste to your docker run command:

`-p 3200:3200 -p 3300:3300 -p 8443:8443 -p 30213:30213 -p 50000:50000 -p 50001:50001`

If you run into the need to expose too many ports, you can consider using `--net=host` instead of exposing ports one by one but the option will cause that all container's port will be available outside the docker host and you will not be able to start another such container. Unfortunately, it appears that this option is available to GNU/Linux users only.

Please, do not use the parameter *-P* (the capitalized P, case matters) because that exposes container ports on random host ports and many SAP clients requires exact ports which cannot be changed (e.g. if the container's port 3200 is exposed as the port 54356, as far as we know you will not be able to configure SAPGUI for Windows to connect to that port).

In the case you are on Windows and you want to connect to the containers IP directly without the need to expose the ports with the parameter *-p*, you may need to update their IP routes to get their TCP/IP packets correctly routed from their host machine to the docker container (which is running in a virtualized GNU/Linux). Self-study materials:
- https://docs.docker.com/docker-for-windows/networking/
- https://github.com/docker/for-win/issues/221

Mac users must always publish the required ports because of the know Docker for Mac limitations:
- https://docs.docker.com/docker-for-mac/networking/#known-limitations-use-cases-and-workarounds

In the case you want run more than 1 container and you do not use GNU/Linux you can play with publish port numbers. For example you can expose the container's port 3200 as the port 3201 (*-p 3201:3200*) and then you can connect to SAPGUI with the instance number 01 instead of the default 00.


<h2><a id="sapgui">SAPGUI</a></h2>


1. Add a custom-specified system with the Application Server `<the container's IP>`or *localhost* if you exposed the port 3200 (i.e. `-p 3200:3200`) or *vhcala4hci* if you updated your *hosts* file. 
2. Finally use Instance `00` and SID `A4H`.


<h2><a id="user-and-passwords">User and Passwords</a></h2>

The user name is *DEVELOPER*. 
The password is:
- for ABAP Platform Trial 1909, initial shipment *`Htods70334`* (Note that we no longer provide this version)
- for ABAP Platform Trial 1909, SP01 *`ABAPtr1909`*

This is also predefined (same password) for client 000, client 001:  SAP* , DDIC.


<h2><a id="browser">Browser</a></h2>


Accessing the port HTTP or HTTPS services via an internet browser does not have any special requirements as long as you use the port 50000 for HTTP or the port 50001 for HTTPS and the correct host.

The host value depends on the way how you started the container.  If you exposed all the required ports, then you can use *localhost*. If your Operating System allows you to configure IP routing the way that you can reach out the container's IP directly, you can use `<the container's IP>`.

When you get redirected to your browser from SAPGUI, the URL will have host set to `vhcala4hci` which will not be reachable unless you modify your **hosts** file ([Hosts file](https://en.wikipedia.org/wiki/Hosts_(file))).  It is necessary to add a new entry which will make sure your Operating System will be able to translate the hostname *vhcala4hci* to an IP address. Contents of the new entry depends on how you started the container. 

**Manually exposed ports or --net=host** - append the line `127.0.0.1  vhcala4hci`

**No explicit port exposure** - append the line `<the container's IP>  vhcala4hci`


<h2><a id="sap-cloud-connector">SAP Cloud Connector</a></h2>


To be able to use SAP Cloud Connector, you must start an additional service via the following commands:

```bash
docker exec -it a4h bash
/usr/local/sbin/rcscc_daemon start
```

SAP Cloud Connector status can be checked by:

```bash
docker exec -it a4h bash
/usr/local/sbin/rcscc_daemon status
```

The last command will start a daemon process which must be stopped before you can leave the container. Please, use the following command:

```bash
/usr/local/sbin/rcscc_daemon stop
exit
```

You can connect to the instance of SAP Cloud Connector at:

`https://<the container's IP>:8443`

with the user *Administrator* and the password *manage*.


<h1><a id="additional-information">Additional Information</a></h1>

<h2><a id="stop">Stop</a></h2>


We must make sure SAP HANA has enough time to write all its data into files on your disk.
To stop the container gracefully, hit Ctrl-C in the command window
where you started the container or run the following command:

```bash
docker stop -t 7200 a4h
```


<h2><a id="start-again">Start again</a></h2>


You can start a stopped container via the command *docker start*. 

```bash
docker start -ai a4h
```

- `-i` = We must start it in the interactive mode to be able to respond to the possible start problems 
- `-a` = we must "attach" to the container to be able  to see text output



<h2><a id="abapgit">abapGit</a></h2>

 abapGit is available to download here, along with complete instructions:


[abapGit Installation](https://docs.abapgit.org/user-guide/getting-started/install.html)

Note that the report you download is named **`ZABAPGIT_STANDALONE`**.

<h1><a id="known-issues">Known issues</a></h1>

<h2><a id="error-message-when-starting-sap-cloud-connector-scc">Error message when starting SAP Cloud Connector (SCC)</a></h2>

```bash
ERROR: shell command for retrieving PID of process bound to SCC port ...java.io.IOEcception..
```
The error message does not affect the functionality of SAP Cloud Connector (SCC) and will be removed in a future version of SCC.



<h2><a id="creating-a-new-container">Creating a new container </a></h2>

Do not omit the following parameter: 

```bash
-agree-to-sap-license 
```

The script asks for the agreement if it's missing but you probably run into a problem when you stop and start the container again. 

<h1><a id="primary-contacts">Primary contacts</a></h1>

- [Julie Plummer](mailto:julie.plummer@sap.com)
- [Ralf Henning](mailto:ralf.henning@sap.com)
- [Jakub Filak](mailto:jakub.filak@sap.com)
