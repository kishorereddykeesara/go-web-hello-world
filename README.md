Task 0: Install a ubuntu 16.04 server 64-bit in a virtual machine
Steps followed:
* Download virtualbox and install in windows machine
* Download ubuntu 16.04 server 64-bit ISO image
* Launch a VM from virtualbox and boot it with ISO image
  RAM size: 4 GB
  Disk: 32 GB
* Proceed with installation instructions while booting ISO image

Task 1: ssh to guest VM (ubuntu) from host and update the system to latest
* Install ssh client on host side (example: Bitwise) and ssh server on server (guest VM) side
* Configure port forward from virtualbox:
  Name: SSH, HostIP: 127.0.0.1, Host Port: 2222, Guest IP: 10.0.2.15 & Guest Port: 22
* Login to VM through ssh client
* Update the system/VM to latest packages:
  sudo apt update
  sudo apt upgrade -y
  sudo apt install ssh
* Upgrade kernel to latest (Used 5.4 in this demo)
  ** Download kernel packages and install
     sudo wget https://kernel.ubuntu.com/~kernel-ppa/mainline/v5.4/linux-headers-5.4.0-050400_5.4.0-050400.201911242031_all.deb
     sudo wget https://kernel.ubuntu.com/~kernel-ppa/mainline/v5.4/linux-headers-5.4.0-050400-generic_5.4.0-050400.201911242031_amd64.deb
     sudo wget https://kernel.ubuntu.com/~kernel-ppa/mainline/v5.4/ linux-image-unsigned-5.4.0-050400-generic_5.4.0-050400.201911242031_amd64.deb
     sudo wget https://kernel.ubuntu.com/~kernel-ppa/mainline/v5.4/linux-modules-5.4.0-050400-generic_5.4.0-050400.201911242031_amd64.deb
  ** Install packages using dpkg -i *.deb
     Issue faced: libssl incompatability with linux-headers
	 dpkg: dependency problems prevent configuration of linux-headers-5.5.0-050500-generic:
     linux-headers-5.5.0-050500-generic depends on libssl1.1 (>= 1.1.0); however:
     Package libssl1.1 is not installed.

	 Fix: Download libssl1.1_1.1 and install it
	 userdemo@ubuntudemovm:~/5.5$ sudo dpkg -i libssl1.1_1.1.0g-2ubuntu4_amd64.deb
     See https://github.com/teejee2008/ukuu/releases/tag/v18.5.1

Reference link to follow the instructions about kernel upgrade:
https://www.howtoforge.com/tutorial/how-to-upgrade-linux-kernel-in-ubuntu-1604-server/

  ** sudo update-grub
  ** sudo reboot
  ** Check kernel version after upgrade
     userdemo@ubuntudemovm:~$ uname -msr
     Linux 5.4.0-050400-generic x86_64
NOTE: Noticed an issue with DNS some time during upgrade of kernel, fix is to update /etc/resolve.conf as suggested in
https://stackoverflow.com/questions/24821521/wget-unable-to-resolve-host-address-http
Error: example: wget: unable to resolve host address ‘kernel.ubuntu.com’


Task2: Install gitlab.ce version in the host:

References followed:
https://about.gitlab.com/install/#ubuntu?version=ce
https://www.howtoforge.com/tutorial/how-to-install-and-configure-gitlab-on-ubuntu-16-04/
Steps:
 * sudo apt install gitlab-ce
 * Configure Gitlab Main URL -> sudo vi /etc/gitlab/gitlab.rb -> change external_url to "http://127.0.0.1"
 * sudo gitlab-ctl reconfigure
 * Change username and password:
 https://docs.gitlab.com/ee/security/reset_root_password.html
 * Port forward: HTTP rule, port 8080 on host and 80 on guest
Access Gitlab via http://127.0.0.1:8080

Task 3: Create a demo project in gilab http://127.0.0.1:8080
Steps:
* Create project group: demo
* Create project: go-web-hello-world
* Create go app: hello.go (see content in the repo, with listening port 8081)
* Access project from http://127.0.0.1:8080/demo/go-web-hello-world/

Task 4: Build the go web app and expose via 8081 port
 * Port forward: HTTP rule, port 8081 on host and 8081 on guest
   Rule:GO-WEB-APP-HTTP, HostIP:127.0.0.1,Host Port:8081,Guest IP: 10.0.2.15, Guest Port:8081
 * Install go on VM:
   ** curl -O https://storage.googleapis.com/golang/go1.11.2.linux-amd64.tar.gz
   ** tar zxf go1.11.2.linux-amd64.tar.gz
   ** sudo mv go /usr/local
   ** set GOPATH and PATH See https://medium.com/@patdhlk/how-to-install-go-1-9-1-on-ubuntu-16-04-ee64c073cd79
   ** Create hello.go as mentioned in Task 3
   ** go run hello.go
   ** Access go web app via http://127.0.0.1:8081/
 
Task 5: Install Docker
Steps followed according to below reference
Reference:
https://www.digitalocean.com/community/tutorials/how-to-install-and-use-docker-on-ubuntu-16-04

Task 6: Build docker image and run a container for go web app
 ** mkdir go-web-hello-world (inside /home/<user> in VM)
 ** cd go-web-hello-world
 ** copy hello.go to new dir
 ** Create Dockerfile (example: https://tutorialedge.net/golang/go-docker-tutorial/)
 ** docker build -t go-web-hello-world .
 ** docker run -d -p 8083:8081 go-web-hello-world
 ** port forward 8083 (guest and host)
 http://127.0.0.1:8083/

Task 7: Tag docker image
Reference:
https://ropenscilabs.github.io/r-docker-tutorial/04-Dockerhub.html
Steps:
* docker tag 006254a8467d kishoredocker2017/go-web-hello-world:v0.1
* docker push kishoredocker2017/go-web-hello-world:v0.1
* Issue: denied: requested access to the resource is denied
* Fix: docker login   https://stackoverflow.com/questions/41984399/denied-requested-access-to-the-resource-is-denied-docker
* docker push kishoredocker2017/go-web-hello-world:v0.1
* Check that image is availble in docker hub
https://hub.docker.com/repository/docker/kishoredocker2017/go-web-hello-world
