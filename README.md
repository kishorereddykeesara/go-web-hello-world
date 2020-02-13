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
  * Download kernel packages and install
     sudo wget https://kernel.ubuntu.com/~kernel-ppa/mainline/v5.4/linux-headers-5.4.0-050400_5.4.0-050400.201911242031_all.deb
     sudo wget https://kernel.ubuntu.com/~kernel-ppa/mainline/v5.4/linux-headers-5.4.0-050400-generic_5.4.0-050400.201911242031_amd64.deb
     sudo wget https://kernel.ubuntu.com/~kernel-ppa/mainline/v5.4/ linux-image-unsigned-5.4.0-050400-generic_5.4.0-050400.201911242031_amd64.deb
     sudo wget https://kernel.ubuntu.com/~kernel-ppa/mainline/v5.4/linux-modules-5.4.0-050400-generic_5.4.0-050400.201911242031_amd64.deb
  * Install packages using dpkg -i *.deb
     Issue faced: libssl incompatability with linux-headers
	 dpkg: dependency problems prevent configuration of linux-headers-5.5.0-050500-generic:
     linux-headers-5.5.0-050500-generic depends on libssl1.1 (>= 1.1.0); however:
     Package libssl1.1 is not installed.

	 Fix: Download libssl1.1_1.1 and install it
	 userdemo@ubuntudemovm:~/5.5$ sudo dpkg -i libssl1.1_1.1.0g-2ubuntu4_amd64.deb
     See https://github.com/teejee2008/ukuu/releases/tag/v18.5.1

Reference link to follow the instructions about kernel upgrade:
https://www.howtoforge.com/tutorial/how-to-upgrade-linux-kernel-in-ubuntu-1604-server/

  * sudo update-grub
  * sudo reboot
  * Check kernel version after upgrade
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
   * curl -O https://storage.googleapis.com/golang/go1.11.2.linux-amd64.tar.gz
   * tar zxf go1.11.2.linux-amd64.tar.gz
   * sudo mv go /usr/local
   * set GOPATH and PATH See https://medium.com/@patdhlk/how-to-install-go-1-9-1-on-ubuntu-16-04-ee64c073cd79
   * Create hello.go as mentioned in Task 3
   * go run hello.go
   * Access go web app via http://127.0.0.1:8081/
 
Task 5: Install Docker
Steps followed according to below reference
Reference:
https://www.digitalocean.com/community/tutorials/how-to-install-and-use-docker-on-ubuntu-16-04

Task 6: Build docker image and run a container for go web app
 * mkdir go-web-hello-world (inside /home/<user> in VM)
 * cd go-web-hello-world
 * copy hello.go to new dir
 * Create Dockerfile (example: https://tutorialedge.net/golang/go-docker-tutorial/)
 * docker build -t go-web-hello-world .
 * docker run -d -p 8083:8081 go-web-hello-world
 * port forward 8083 (guest and host)
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


Task 9: Install Kubernetes
* sudo apt-get update && sudo apt-get install -y apt-transport-https && curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
* echo "deb http://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee -a /etc/apt/sources.list.d/kubernetes.list && sudo apt-get update
* sudo apt install -y kubeadm  kubelet kubernetes-cni
* sudo swapoff -a
* sudo sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab
* sudo kubeadm init --pod-network-cidr=10.244.0.0/16 --apiserver-advertise-address=10.0.2.15 --ignore-preflight-errors=NumCPU

Your Kubernetes control-plane has initialized successfully!
To start using your cluster, you need to run the following as a regular user:
  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

References:
https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/
https://www.devdiaries.net/blog/Single-Node-Kubernetes-Cluster-Part-2/

Task 10: Deploy Go web app in K8S
* Prepare deployment file: go-web-hello-world-k8s-deployment.yml
* Deploy the app
  kubectl apply -f go-web-hello-world-k8s-deployment.yml
* Prepare service file: go-web-hello-world-k8s-service.yml
* Deploy the service (NodePort with port 31080)
  kubectl apply -f go-web-hello-world-k8s-service.yml
  Expect output like
userdemo@ubuntudemovm:~/go-web-hello-world$ kubectl get service
NAME                         TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)           AGE
go-web-hello-world-service   NodePort    10.101.22.250   <none>        31080:31080/TCP   19s
kubernetes                   ClusterIP   10.96.0.1       <none>        443/TCP           46h
* Port forward config in virtualbox: 31080/31080
* Access application from host on : http://127.0.0.1:31080/
References:
https://docs.docker.com/get-started/part3/
https://www.bogotobogo.com/GoLang/GoLang_Web_Building_Docker_Image_and_Deploy_to_Kubernetes.php
https://www.digitalocean.com/community/tutorials/how-to-deploy-resilient-go-app-digitalocean-kubernetes

Task 11 & 12: Install K8s dashboard & access with token

Install dashboard:
kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.0.0-rc5/aio/deploy/recommended.yaml

Access dashbord locally:
Run "kubectl proxy"
curl http://localhost:8001/api/v1/namespaces/kubernetes-dashboard/services/https:kubernetes-dashboard:/proxy/

Create secrets:
kubectl create secret generic kubernetes-dashboard-certs --from-file=$HOME/certs -n kubernetes-dashboard

Create a new user using Service Account mechanism of Kubernetes, grant this user admin permissions and login to Dashboard using bearer token tied to this user:
Create dashboard-adminuser-service-account.yaml with required content (see github)
Create dashboard-adminuser-clusterolebinding.yaml with required content (see github)
kubectl apply -f dashboard-adminuser-service-account.yaml
kubectl apply -f dashboard-adminuser-clusterolebinding.yaml

Copy token from output of below command:
kubectl -n kubernetes-dashboard describe secret $(kubectl -n kubernetes-dashboard get secret | grep admin-user | awk '{print $1}')

Edit dashboard config to expose via NodePort:
kubectl -n kubernetes-dashboard edit service kubernetes-dashboard
 * Change port to 31080
 * Add nodePort: 31080
 * Change type from ClusterIP to NodePort

Check the dashboard service:
userdemo@ubuntudemovm:~/go-web-hello-world$ kubectl -n kubernetes-dashboard get service kubernetes-dashboard
NAME                   TYPE       CLUSTER-IP      EXTERNAL-IP   PORT(S)           AGE
kubernetes-dashboard   NodePort   10.105.66.169   <none>        31081:31081/TCP   46h 

Port forward in virtualbox:

Access dashboard via:
https://127.0.0.1:31081/#/login
