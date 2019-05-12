+++
date = "2019-04-25"
description = "Trying to sum up a basic usage of Docker"
language = "en-us"
title = "A Docker Primer"
slug = "docker" 
+++

 <img src="https://www.docker.com/sites/default/files/social/docker_facebook_share.png" alt="Docker logo" class="center"></img> 

## What is Docker ?

Docker is a software overlay allowing its users to develop apps in environments
called *containers*. It leverages [kernel-side virtualisation technologies][5]
in order to make guests systems run as if they were their own.

Docker differs from lower-level virtualisation engines like [VMWare][2] 
or [VirtualBox][3] because Docker containers all share the same kernel,
while VBoxes for example use each a completely separate stack, from the
emulated BIOS upwards.

This allows Docker containers to : 

 - Run extremely fast, because they're native and not emulated
 - Take less HDD space ([vanilla debian weighs around 50MB][4])
 - Be easy to build : see Dockerfile examples below.

Of course, it also has a few drawbacks :

 - Containers are system specific (no Windows containers on Linux...)
 - Security can be a concern, *there is actually a lot going on there*.

[Docker][1] is also the name of the very successful company who is developping
the aforementionned solution, as well as a few other services and products
like `docker-compose` and the Docker Hub which will be presented in more
detail on the next sections.

[1]: https://www.docker.com
[2]: https://www.vmware.com
[3]: https://www.virtualbox.org

[4]: https://hub.docker.com/_/debian?tab=tags
[5]: https://en.wikipedia.org/wiki/Docker_(software)#Technology
[6]: https://to.be.disclosed/no-reference-yet

## Installing Docker on your system

Docker install process greatly varies regarding your host system.
On Windows and Mac, you can get an installer from the official download page,
which will install the whole up-to-date Docker toolset (called Docker Stack).

On Linux, regarding your target distribution, you can either set a repository,
download a packet, or fetch and compile the executables by hand.
</br>
On a Debian Stretch `amd64` system, you can do as follows.

```bash
sudo apt install apt-transport-https ca-certificates
wget https://download.docker.com/linux/debian/gpg -O docker.pub
# Verify integrity using SHA256sum and key fingerprint
sudo apt-key add docker.pub

URL="https://download.docker.com/linux/debian stretch stable"
sudo echo "deb [arch=amd64] "$URL >> /etc/apt/sources.list
sudo apt update
sudo apt install docker-ce
```
For other systems, please have a look at the Docker [online docs][doc].
Once you've got Docker on your system, you can check its version by typing
the usual `docker -v`.

[doc]: https://docs.docker.com

**Security warning** : While adding your standard user to the Docker group
can be tempting to avoid constant sudoing when using `docker`, it is bad
practice to do so because the Docker group basically gives you root rights.
For example, an attacker who compromised your user account could erase 
your standard system logs by typing this one-liner.

```bash
docker exec alpine --mount /var/log:/root/logs rm -rf ~/logs
```

## Running your first container

Now you have Docker on your system, let's take quick tour of what's possible.
First we'll search the Docker Hub to get our containers :

```bash
sudo docker search alpine
sudo docker search debian
sudo docker search ubuntu
```
These commands should each output a list of images vaguely looking like this.

```

root@laptop:~# docker search alpine --limit 8
NAME                                   DESCRIPTION                                     STARS               OFFICIAL            AUTOMATED
alpine                                 A minimal Docker image based on Alpine Linux…   5201                [OK]                
mhart/alpine-node                      Minimal Node.js built on Alpine Linux           428                                     
anapsix/alpine-java                    Oracle Java 8 (and 7) with GLIBC 2.28 over A…   406                                     [OK]
alpine/git                             A  simple git container running in alpine li…   79                                      [OK]
kiasaki/alpine-postgres                PostgreSQL docker image based on Alpine Linux   43                                      [OK]
easypi/alpine-arm                      AlpineLinux for RaspberryPi                     32                                      
jfloff/alpine-python                   A small, more complete, Python Docker image …   24                                      [OK]
hermsi/alpine-sshd                     Dockerize your OpenSSH-server upon a lightwe…   18                                      [OK]

```
Each list item corresponds to a Docker image. The list also shows the image tags,
popularity, and official status to help you make yourself an idea of how much
trust you can put in what you're about to download.

You can also search this on the [Hub Webpage](hub.docker.com) or if you use
[DuckduckGo](ddg.gg), using the associated `!docker` bang. The web results will
give you more detailed information (image building process, GitHub repo,
and so on) than just a plain CLI search without arguments.

Once you've decided which image to get, you can download it like this :

```bash
# Template : sudo docker pull repo/image:tag
# Repo and tag can be omitted if unambiguous
sudo docker pull alpine
```
Docker will then get the corresponding image layers and install the image.
You can find it under `/var/lib/docker` but it may require a little search.
More on Docker objects and internals on a next article, maybe.

Now you have your image, how about interacting with it ? </br>
Quick terminology tip : a running Docker image is a container.

```bash
sudo docker run -it alpine	# Dive inside the container
sudo docker exec alpine "id"	# Execute commands from outside
sudo docker logs alpine		# Get container logs (stdout)
```

You will quickly notice a few other things.

 1. You usually enter inside the container as root, but you may not be able to do
	everything the host root user can do. This actually depends on the
	container's capabilities, which can be set on startup.
	More on this topic in the Security section.

 2. Inside the running container, you have your own network interfaces, and your
	own directory tree. They are de-coupled with the host network and file
	system by default, but this can be changed if needed using `--network`
	and `--mount` options on startup. This however can lead to security
	leaks if not properly handled, be sure to check your threat model. 

 3. Once exited, the container is destroyed and any unsaved changes will be lost.

Let's now try to develop our own customized environment.

## Building with Docker

### Docker images

The canonical way of building images is by writing provisionning scripts called
*Dockerfiles*. In these files, you specify a base image to start `FROM`, then
you can `COPY` stuff into it and `RUN` scripts inside it until you get to the
expected result.

Then, you can specifiy the first command of your container
at startup : this is done by either `CMD` or `ENTRYPOINT`.
The first one passes a shell command, the second one a shell script
(*more or less...*)

You can also do other interesting things such as changing your base `USER`,
setting environment variables in the target system, or using custom arguments.

Here is an example of a simple Dockerfile for network traffic analysis.

```
FROM alpine:latest

RUN	apk add tcpdump ngrep tshark && \
	adduser --disabled-password --gecos "" analyser

USER analyzer
WORKDIR /home/analyzer

CMD ["/bin/sh"]
```

For each Dockerfile directive, a new image layer is created. For this reason,
`RUN` commands tend to be very compact, with each instruction linked to the 
previous with a double ampersand `&&`.

I recommend keeping an eye on the [Dockerfile reference][dref],
on the [best practice webpage][bprc] and on the official 
[Docker tutorial][tuto] anyway while experimenting with Docker builds.

[dref]: https://docs.docker.com/engine/rference/builder
[bprc]: https://docs.docker.com/develop/develop-images/dockerfile_best-practices
[tuto]: https://docs.docker.com/get-started

Once you have your Dockerfile ready, you can just run something like :

```bash
sudo docker build -t imagename path/to/Dockerfile
sudo docker images # Check it has been successfully added
sudo docker rmi 1234567890abcdef # In case of overwriting
sudo docker run -it imagename
```
 - When building several times over the same image, remains of previous
builds appear in the image list without any name. You can use `rmi`
to delete those if they're not useful anymore.

 - Oftentimes, Docker images are not meant to be CLI-interactive by default.
This can be the case when developping a service like a webserver for example.
In that case, the `-it` option should be discarded.

 - Sometimes you'll also want to share your image. In that case, you may either
push it on the Docker Hub or save it to a gzipped archive file.

```bash
sudo docker push user/image	# Requires a Hub account (?)
sudo docker save imagename 	# Missing args to gzip result
```
Okay, so now we have a cool custom image, but as you probably noticed,
you can't actually use the installed binaries yet, because you enter
the container as `analyzer` who has no admin rights.
Even if you delete line 4 in the Dockerfile, you still won't be able
to monitor what's going on over your host interface.

How can we setup the container environment in order to make it work ?

### Setting up the image environment

There are actually two ways of setting up a Docker environment : arguments and
`docker-compose`. We briefly saw the first one with `--network` and `--mount`
options, but there are a lot more. They are listed [here][dc-options].

The second way implies creating a configuration file called `docker-compose.yml`
and using the `docker-compose` tool. On Windows and Mac, this tool ships with
Docker by default, but on Linux you may need to install it separately. <br/>
In this case you might want to install it as a container from [there][dc-source].

Here is a docker-compose [example][dc-example]. </br>
Here is the docker-compose [reference][dc-refer].

[dc-options]: https://docs.docker.com/engine/reference/commandline/run/
[dc-example]: https://gist.github.com/blackstorm/d446814539daace544d0c9c61e842d18
[dc-source]: https://github.com/docker/compose/releases/download/1.24.0/run.sh
[dc-refer]: https://docs.docker.com/compose/compose-file

As you can see, docker-compose keywords are almost identical to the previous
arguments. It's just another way to provide the same information.
However, with compose files you can actually automate the building process
and orchestrate the interaction of several Docker containers, which is a big plus.

Below are the files we may use for our project, although we should check
beforehand for security policy compliance. Should this container really be able
to monitor our host interfaces ?

`Dockerfile`
```
FROM alpine:latest

RUN     apk add tcpdump ngrep tshark libcap && \
        adduser --disabled-password --gecos "" analyzer && \
        setcap CAP_NET_BIND_SERVICE+ep /usr/bin/ngrep && \
        setcap CAP_NET_ADMIN+ep /usr/bin/ngrep && \
        setcap CAP_NET_RAW+ep /usr/bin/ngrep

USER    analyzer
WORKDIR /home/analyzer

CMD     ["/bin/sh"]
```

`docker-compose.yml`

```yaml
version: '2.3'
services:
  main:
    build: .
    image: test
    network_mode: host
    cap_add:
      - NET_RAW
      - NET_ADMIN
      - NET_BIND_SERVICE
    volumes:
      - exchange:/data

volumes:
        exchange:

```
`commandline`

```bash
sudo docker-compose build
sudo docker-compose up		  # For non-interactive containers
sudo docker-compose run main	  # For interactive containers like ours
```

## Wrapping up : Docker development workflow

 - Pull a base image
 - Work on it with a Dockerfile
 - Build it (and clean previous remains)
 - Share it using the Hub or gzipped archives
 - Run it with arguments or using `docker-compose`

And as a live example :

<script id="asciicast-rHxl5SmiQH82aoH0lTb6nMiSJ" src="https://asciinema.org/a/rHxl5SmiQH82aoH0lTb6nMiSJ.js" async></script>

## Docker security good practice

Let's call it "okay practice" instead and define it in simple terms :

 - Design and build a security model for your system as soon as possible,
	and audit regularly. This means code audit, pentesting, and architecture
	review. A. Shostack has published an [excellent guide][tmodel] about
	threat modelling I advise anyone to read, and you might also find
	interesting info about the subject there.

 - Don't blindly trust your base image. Check the build and the runtime
	environment, and restrict every unecessary right and cap you can.
	You can use inspection tools such as [Lynis][S1], [Anchore][S2] and
	[OpenVAS][S3], and enforcement tools such as [AppArmor][S4],
	[SELinux][S5], [Tomoyo][S6] and [AIDE][S7].

 - Check for vulnerabilities in your software stack by reading [CVE][S8] news
	and patch/update anytime there's a critical security bug. Be sure to
	also check compliance using for example [CIS Benchmarks][S9].

[tmodel]: https://threatmodelingbook.com/

[S1]: https://linux-audit.com/lynis/
[S2]: https://github.com/anchore/anchore-engine
[S3]: http://openvas.org/
[S4]: https://wiki.ubuntu.com/AppArmor
[S5]: https://www.nsa.gov/what-we-do/research/selinux/
[S6]: http://tomoyo.osdn.jp/
[S7]: https://aide.github.io/
[S8]: https://cve.mitre.org/
[S9]: https://github.com/cismirror/benchmarks

## Conclusion

Docker is one of several kernel-level virtualisation software
available today (other solutions include BSD jails and Solaris zones).
Its cross-platform availability and simplicity of use greatly contributed
to the huge popularity it currently has. With such a tool, it becomes
easy to build and seal your software stack by shipping each block as an
independent container on a monitored, system-internal network.

With such a power comes however the responsibilty of clearly defining your
trust zones and securing your accesses, as Docker runs with near-to-root
rights and prerogatives.

