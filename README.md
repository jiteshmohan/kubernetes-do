# Guide to creating a Kubernetes cluster on a coreos instance on Digital Ocean with three nodes.
## Creating coreos cluster on Digital Ocean
1. Create a node with the following configuration
 * 1GB/1CPU 30GB 2TB nodehttps://discovery.etcd.io/new?size=3
 * Select an appropriate region
 * CoreOS 76.4.0(Stable release)
 * Turn on private networking
 * Select User Data option
 * In the textbox for user data provide information as specidied in [this file](https://github.com/jiteshmohan/kubernetes-do/blob/master/cloud-config)
   - Use this URL [https://discovery.etcd.io/new?size=3](https://discovery.etcd.io/new?size=3) to generate a discovery token. The 3 at the end of the token specifies the number of nodes that are to be present while initiating the cluster. A minimum of these many number of nodes should be present in the cluster for it to be connected through fleet.
   - Replace the token generated above in line 7 of the cloud-config file.
  * CoreOS instances can only be connected to via ssh and hence a public-key needs to be set up in the digital ocean dashboard. Use this [link](https://www.digitalocean.com/community/tutorials/how-to-set-up-ssh-keys--2) to learn how to setup ssh keys with digital ocean.
  * In a similar manner create 2 more nodes.
2. Once the nodes are created you can securely connect to them using ssh. ssh -A core@public-ip-address
3. Once you are connected to the device you can clone this repository<br>`git clone https://github.com/jiteshmohan/kubernetes-do.git`
4. Create a new directory `/opt/bin/`<br>`sudo mkdir -p /opt/bin/`
5. `cd kubernetes-do`
6. Copy all files from the folder executables into the `/opt/bin/` directory.<br>`sudo cp executables/* /opt/bin/`
7. Next you need to copy the service files to `/etc/systemd/system/`. However the files are separate for master and minion.
 * On the master: `sudo cp service-files/master/*.service /etc/systemd/system/`
 * On minion: `sudo cp service-files/minion/*.service /etc/systemd/system/`
8. Digital ocean creates a file `/etc/environment` when spawning the instance. The contents of this file are as follows:<pre><code>$ cat /etc/environment
  COREOS_PRIVATE_IPV4=\<ip_addr\>
  COREOS_PUBLIC_IPV4=\<ip_addr\>
</code></pre>
   To this file add additional two lines in the case of the minion. The lines that need to be added are<pre><code>MASTER_PUBLIC_IPV4=\<ip_addr\>
  MASTER_PRIVATE_IPV4=<ip_addr>
</code></pre>
9. Ensure that all the executable copied into `/opt/bin` have execute permissions.
10. Enable the services that we created.<pre><code>cd /etc/systemd/system
  sudo systemctl enable *
</code></pre>
11. On master start the following services<pre><code>sudo systemctl restart flanneld
  sudo systemctl restart docker
  sudo systemctl restart kube-apiserver
  sudo systemctl restart kube-controller-manager
  sudo systemctl restart kube-scheduler
  sudo systemctl restart kube-proxy
  sudo systemctl restart kubelet</code></pre>
12. On the minions start the following services<pre><code>sudo systemctl restart flanneld
  sudo systemctl restart docker
  sudo systemctl restart kube-scheduler
  sudo systemctl restart kube-proxy
  sudo systemctl restart kubelet</code></pre>
13. After starting all the above services use<pre><code>systemctl status \<service-name\></code></pre> to check the status of each service.
14. On the master run the following command  to get the list of machines in the kubernetes cluster.<pre><code>kubectl get nodes</code></pre>
15. To access the Kubernetes dashboard perform the following steps<pre><code>cd
  git clone https://github.com/kubernetes/kubernetes.git
  cd kubernetes
  kubectl create -f cluster/addons/kube-ui/kube-ui-rc.yaml --namespace=kube-system
  kubectl create -f cluster/addons/kube-ui/kube-ui-svc.yaml --namespace=kube-system</code></pre>
16. To access the UI use the link https://\<master-public-ip\>:6443/ui/

# References
+ https://www.digitalocean.com/community/tutorials/how-to-install-and-configure-kubernetes-on-top-of-a-coreos-cluster
+ https://www.digitalocean.com/community/tutorials/how-to-set-up-a-coreos-cluster-on-digitalocean
+ http://kubernetes.io/v1.0/docs/user-guide/ui.html
