# Introduction to Docker

## What is Docker ?

Docker is a software overlay allowing its users to develop apps in environments
called *containers*. It leverages kernel-side virtualisation technologies in 
order to make guests systems run as if they were their own.

Docker differs from lower-level virtualisation engines like VMWare or VirtualBox
because Docker containers all share the same kernel, while VBoxes for example
use each a complete different stack, from the emulated BIOS upwards.

This allows Docker containers to : 

 - Run extremely fast, because they're native and not emulated
 - Take less HDD space (vanilla debian weighs around 100MB)
 - Be easy to build : see Dockerfile examples below.

Of course, it also has a few drawbacks :

 - Containers are system specific (no Windows containers on Linux...)
 - Security can be a concern, there is actually a lot going on there.

Docker is also the name of the very successful company who is developping
the aforementionned solution, as well as a few other services and products
like `docker-compose` and the Docker Hub which will be presented in more
detail on the next sections.

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

URL="https://download.docker.com/linux/debian/stretch stable"
sudo echo "deb [arch=amd64] "$URL >> /etc/apt/sources.list
sudo apt update
sudo apt install docker-ce
```
For other systems, please have a look at the Docker download page.
Once you've got Docker on your system, you can check its version by typing
the usual `docker -v`.

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
Put docker output here
```
Each list item corresponds to a Docker image. The list also show the image tags,
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
Docker will then get the correponding image layers and install the image.
You can find it under `/var/lib/docker` but it may require a little search.
More on Docker objects and internals on the corresponding section.

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

RUN	apk add tcpdump bro suricata && \
	useradd # Make this non-interactive
	
USER	analyser
WORKDIR	/home/analyser

# CMD or ENTRYPOINT
```

```bash
#!/bin/sh
# Autostart suricata
suricata -i eth0
```

For each Dockerfile directive, a new image layer is created. For this reason,
`RUN` commands tend to be very compact, with each instruction linked to the 
previous with a double ampersand `&&`.

I recommend keeping an eye on the Dockerfile reference, on the best practice
webpage and on the official Docker tutorial anyway while experimenting 
with Docker builds.

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
sudo docker push user/image	# Requires a Hub account
sudo docker save imagename 	# Missing args to gzip result
```
Okay, so now we have a cool custom image, but ideally we would like to customize
its environment so that it can run network analysis directly on a dedicated host 
interface. How do we do that ?

### Setting up the image environment

There are actually two ways of setting up a Docker environment : arguments and
`docker-compose`. We briefly saw the first one with `--network` and `--mount`
options, but there are a lot more. They are listed here.

The second way implies creating a configuration file called `docker-compose.yml`
and using the `docker-compose` tool. On Windows and Mac, this tool ships with
Docker by default, but on Linux you may need to install it separately (see here).

Here is an example. </br>
Here is docker-compose reference.

As you can see, docker-compose keywords are almost identical to the previous
arguments. It's just another way to provide the same information.

## Wrapping up : Docker development workflow

 - Pull a base image
 - Work on it with a Dockerfile
 - Build it (and clean previous remains)
 - Share it using the Hub or gzipped archives
 - Run it with arguments or using `docker-compose`

Or using a nice picture : there (asciinema)

## Docker security good practice

Let's call it "okay practice" instead and define it in simple terms :

 - Design and build a security model for your system as soon as possible,
	and audit regularly. This means code audit, pentesting, and architecture
	review. A. Shostack has published an excellent guide about
	threat modelling I advise anyone to read, and you might also find
	interesting info about the subject there.

 - Don't blindly trust your base image. Check the build and the runtime
	environment, and restrict every unecessary right and cap you can.
	You can use inspection tools such as Lynis, Anchore and OpenVAS,
	and enforcement tools such as AppArmor/SELinux/Tomoyo and AIDE.

 - Check for vulnerabilities in your software stack by reading CVE news,
	and patch/update anytime there's a critical security bug.

## Conclusion

Docker is one of several kernel-level virtualisation software
available today (other solutions include LXC and Hyper-V).
Its cross-platform availability and simplicity of use greatly contributed
to the huge popularity it currently has. With such a tool, it becomes
easy to build and seal your software stack by shipping each block as an
independent container on a monitored, system-internal network.

With such a power comes however the reponsibilty of clearly defining your
trust zones and securing your accesses, as Docker runs with near-to-root
rights and prerogatives.

<!-- IMAGE of SpiderMan or StanLee there -->
