Starting with Kubernetes is a struggle if you do not know what you are doing. However, it might also be the best way to learn, because watching a myriad of videos often does not cut it. After multiple failed attempts, I just finished setting up my own Kubernetes cluster at home. I will take you through the steps on how you can do it yourself.
<span style="white-space: pre-wrap;">Step 0 - do we really want Kubernetes?</span>


My guess is that you already convinced yourself you need Kubernetes for some reason. It still is important to reason if you really require Kubernetes. In my opinion, this is dependent on two things. Is there some functional need for Kubernetes, and secondly, do we have the expertise necessary to run a Kubernetes cluster? 
On the functional need, I typically read the same advice online: "You do not need Kubernetes if you are not at least a medium-sized business". The arguments for this point of view are quite simple. Kubernetes is complex to set up and maintain, and it is more expensive to run than a simple server. Just deal with some discomforts of running a bare-metal server, and save yourself a headache and $\frac{9}{10}$ of the money. And while this is true, there are many advantages of running Kubernetes. I think anyone who has had the privilege of running a 10-year-old bare-metal server can attest to the horrific amount of maintenance debt and quick hacks that are accrued over time. Tales of how an update of a single PHP module crashed multiple other services are also not uncommon. There are many ways to fix these kinds of issues, and Kubernetes can be one of them. So running a maintainable server can be a good enough reason to start running Kubernetes. 
A second reason can just be to learn. Even if Kubernetes is overkill for your setup, learning how to use it cannot really backfire in any way. Not so incidentally, this is exactly what I am doing. I am a student finishing up my masters and have run some servers in the past. They would never last because after months of not touching the server, a problem would arise, and I would have forgotten how I configured it in the first place. Now I am maintaining a server for my rowing club, and it has the exact same issues as described in the previous chapter. I want to upgrade their setup to Kubernetes, however, I am slightly hesitant to do it on critical infrastructure. Therefore, I am learning at home first. Not only will it help me transition the rowing club to Kubernetes, it will also help me with a lot of other things I have on my to-do list for a long time. For example, it will also help me setup coding environments, with development, staging, and production environments to properly run my code in. I can start running automated tests without GitHub. I can automatically build my own images. And host all my own data. In the future I want to disconnect as much of my data from MAMAA (Meta, Apple, Microsoft, Amazon, and Alphabet) as possible. 
On the second part, do you have the expertise to host your own Kubernetes cluster? Be honest with yourself. If you just started learning about servers, just hosting a bare-metal home server which has a reasonable backup might solve most of your problems. It will probably be hard enough to learn as is, and you once you start running into problems, you can start to get how more complicated setups might be or not be what you need. Trying to learn without the right background is  just going to make you struggle even more, setting up a cluster. I would require that you know quite a bit of background knowledge in multiple categories of information technology. A quick and dirty summary of necessary knowledge:
Setting up and managing a Linux-based server (installation, file rights, commands, user:groups, services, networking, SSH, mounting drives)
Basic cryptography (TLS, asymmetric encryption, symmetric encryption)
Networking (ports, IP addresses, port-forwarding, firewalls, DNS, domains)
Containerization (images, registries, container specific networking, container lifecycle)
Kubernetes basics (cluster configurations, kubectl, Kubernetes types such as ingresses, services, deployments, pods and more)
You need to know how all the parts in your cluster are working together, because Kubernetes setups can differ quite substantially, and you often need to recognize these differences and adapt your own setup accordingly. If you do not, you will constantly run into head scratching errors that make no sense, making the process frustrating in the best case. I just finished my 6th year of university, and have run probably over 10 different servers and hundreds of different applications, and I still get very confused why stuff is not working. I can only imagine the headache if I did not have this background. This is your warning, make sure you do not jump in the lake if you cannot swim.
tl;dr: Hosting Kubernetes can be complex and costly. Make sure you have the expertise and the right reasons to set up your own cluster. You probably do not really need Kubernetes, but there are still good reasons to do it. Trying to learn is always a good reason. Just make sure you have enough expertise before you dive into the deep end.<span style="white-space: pre-wrap;">Step 1: acquiring hardware</span>


The first thing you need to determine is how you want to set up a cluster. The easiest way is to not create a cluster at all. Kubernetes can run on a single computer using something like Minikube. If you want to create an actual cluster, you will need multiple PC's. On a small scale home lab, you can probably get away with a simple cluster with one control plane and a few workers. For my setup, I use a single Raspberry Pi as a control plane, and two Raspberry Pi's as worker nodes. You can also go full beans and set up a full high availability cluster. If you are reading this guide, I am assuming you are new to the world of Kubernetes, so we are starting small. A high availability cluster brings a lot more complexity than just adding some computers. The parts list:
3 Raspberry Pis (4 or newer)
3 micro SD cards (32 Gb or more)
3 proper USB-C power supplies
A network attached storage device that can handle NFS
A switch with enough ports to connect all the Raspberry Pis
Mount/case for Raspberry Pis (optional)
Tip: if you have a bit of money left over, try experimenting with power-over-Ethernet.A crude representation of the setup we are going to make.


A quick aside for a high availability cluster. You can absolutely create one if you feel comfortable adding a few more moving parts. A proper high available Kubernetes cluster would have something that can act as a load balancer, such as HAProxy or NGINX. This load balancer needs to be redundant. Achieving redundancy for the load balancer can be accomplished using protocols like the Virtual Router Redundancy Protocol (VRRP), with keepalived serving as a prime example of an implementation of VRRP. Lastly, you will need three control-plane nodes, tainted to not execute pods themselves, and three worker nodes. This totals to eight computers, so it is quite the investment.[^1]: A quick aside for a high availability cluster. You can absolutely create one if you feel comfortable adding a few more moving parts. A proper high available Kubernetes cluster would have something that can act as a load balancer, such as HAProxy or NGINX. This load balancer needs to be redundant. Achieving redundancy for the load balancer can be accomplished using protocols like the Virtual Router Redundancy Protocol (VRRP), with keepalived serving as a prime example of an implementation of VRRP. Lastly, you will need three control-plane nodes, tainted to not execute pods themselves, and three worker nodes. This totals to eight computers, so it is quite the investment.


<span style="white-space: pre-wrap;">Installing k(8-5)s</span>


K3s is a simplifies and optimizes the installation of Kubernetes. We use it because it is optimized for edge applications, so it will run on hardware with low specs. It also runs on the ARM64, which makes our lives a lot easier. The plan is easy, install Ubuntu server 64-bit (or any other distro you like) on all the Raspberry Pis. Make sure all Pis are up-to-date, then install k3s. I will be completely honest, I used a tutorial just like this one to set up my own cluster. I will recap the commands in short.
Flashing an OS
Flashing an OS is quite simple, grab a micro SD card and your favourite flashing software (for example Raspberry Imager or Etcher), and you are off to the races. Choose a 64-bit ARM64 operating system and flash it onto the SD card. Raspberry Imager has the added bonus of adding configuration settings to your image while flashing it to the SD card. I added the SSH key from my PC and an admin username/password. I also set the correct timezone.
Configuring the OS
Configuring the OS is downloading and installing all OS updates and adding some Raspberry Pi/k3s specific configurations. # Update and upgrade distro
$ sudo apt update && sudo apt upgrade -y

# Install docker
$ sudo apt install -y docker.io

# Enable the required container features
$ sudo sed -i '$ s/$/ cgroup_enable=cpuset cgroup_enable=memory cgroup_memory=1 swapaccount=1/' /boot/firmware/cmdline.txt

# We need to reboot for the kernel flags to take effect.
$ sudo reboot


Installing k3s
To install k3s, we first create a control plane node, which generates a node token. Then we join the worker nodes by specifying the IP and the node token of the control node. Important note, a k3s control-plane node is not a "pure" control-plane node. It will still run pods like worker nodes. If you do not want the control-plane to run pods, you need to taint these nodes.
Creating a control plane nodecurl -sfL https://get.k3s.io | sh -s - server --cluster-init
better option

$ curl -sfL https://get.k3s.io | sh -

# This node-token is needed when installing the worker nodes
$ sudo cat /var/lib/rancher/k3s/server/node-token


Creating worker nodes# Don't forget to replace $YOUR_SERVER_NODE_IP and $YOUR_CLUSTER_TOKEN
$ curl -sfL https://get.k3s.io | K3S_URL=https://$YOUR_SERVER_NODE_IP:6443 K3S_TOKEN=$YOUR_CLUSTER_TOKEN sh -

curl -sfL https://get.k3s.io | K3S_TOKEN=K102df304d807414f5efe9090f22ab7f37ffab4edfc423e53893cfe394035ec9299::server:c78fc9da6b83cede41d5da78a488093e sh -s - server --server https://192.168.1.201:6443
better option


That is all there is to creating a Kubernetes cluster! Now we can install kubectl on our pc and import the kube config file that is created by the control-plane node, and we can start running things on our cluster.
Accessing the cluster
Install kubectl on your development pc, we will use this to talk to the Kubernetes API on the control-plane. Of course, just follow the latest instructions to install on your OS of choice. 
Installing Kubernetes tools
Then we grab the kubeconfig from the control-plane node:$ cat /etc/rancher/k3s/k3s.yaml


Copy the output to a file named config and move it to the .kube folder, which is where kubectl takes it configuration from. The .kube folder should be in ~/.kube/config. The kubeconfig contains the keys to completely control everything in your cluster, so treat it like you would any ssh private key.
Change the server value from 127.0.0.1 to your ip!

ufw disable!!
<span style="white-space: pre-wrap;">Adding network attached storage</span>



<span style="white-space: pre-wrap;">Things I learned</span>


Bitnami sealed secrets do not refresh if you update the manifest and rerun kubeseal for some reason. Even after deleting the sealed secret with kubectl, the secret remains unchanged. 
Always always always update your DNS before you try to get a certificate from Let's Encrypt.
IngressRoute might not work for TLS certificates if you use cert-manager instead of the Traefik build in certificate manager.
You do not need a separate router to build a cluster.
Raspberry Pis use the ARM64 architecture. Therefore, all your images must also be ARM64. If they are not available then congratulations, you get to build them all on your own, and it takes a long time.
Mounting a volume "deletes" all existing files in that particular path.
PersistantVolumeClaims are immutable and thus need to be recreated with a new name if you make a mistake.
Do not use Kubernetes limits
Everybody turns of swap on Kubernetes, which can be an issue when using high memory applications
