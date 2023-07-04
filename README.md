# 5G-NR
5G SETUP - 5NR GNB (CABLE FREE) &amp; OPEN5GS (5G CORE NETWORK) ON KUBERNETES 

5G - CORE NETWORK ON K8S 
Setup Microk8s : 

Microk8s:- 

MicroK8s is the easiest and fastest way to get Kubernetes up and running.MicroK8s is the go-to platform for mission-critical workloads. Quickly spin nodes up in your CI/CD and reduce your production maintenance costs 
MicroK8s architecture and OS compatibility allows you to deploy on COTS hardware and develop on any workstation. 

Reference link:- 
https://ubuntu.com/tutorials/install-a-local-kubernetes-with-microk8s#1-overview

Install a local Kubernetes with MicroK8s : 
Deploying MicroK8s:
sudo snap install microk8s –classic 
  
NOTE: You may need to configure your firewall to allow pod-to-pod and pod-to-internet communication: 
sudo ufw allow in on cni0 && sudo ufw allow out on cni0 
sudo ufw default allow routed 

Enable addons: 
microk8s enable dns  
microk8s enable dashboard 
microk8s enable storage 
 
NOTE : These addons can be disabled at anytime by running the microk8s disable, 
With microk8s status you can see the list of available addons and the ones currently enabled. 

Accessing the Kubernetes dashboard: 
NOTE: Now that we have enabled the dns and dashboard addons we can access the available dashboard. To do so we first check the deployment progress of our addons with microk8s kubectl get all --all-namespaces 

Kubernetes dashboard : 

NOTE : EX :- As we see above the kubernetes-dashboard service in the kube-system namespace has a ClusterIP of 10.152.183.64and listens on TCP port 443. The ClusterIP is randomly assigned, so if you follow these steps on your host, make sure you check the IP adress you got. Point your browser to https://10.152.183.64:443 and you will see the kubernetes dashboard UI. 

token=$(microk8s kubectl -n kube-system get secret | grep default-token | cut -d " " -f1) 
microk8s kubectl -n kube-system describe secret $token 

Here ,the Microk8s is successfully deployed. Next deploy both open5gs and edgexfoundry in the cluster on microk8s using helm charts which is the package manage of k8s. 

Setup Open5gs using helm charts :
Open5gs : 

Open5GS is a C-language Open Source implementation for 5G Core and EPC, i.e. the core network of LTE/NR network (Release-17) open5gs.org. 

Open5gs Helm charts: 

Helm charts :- A Helm chart is a set of YAML manifests and templates that describes Kubernetes resources (Deployments, Secrets, CRDs, etc.) and defined configurations needed for the Kubernetes application, and is also easy to deploy in a Kubernetes cluster or in a single node with just one command. 

Reference link:- 
https://www.datree.io/helm-chart/open5gs-cgiraldo# 

Install Open5gs using Helm charts: 
Add Charts repsoitory to helm : 
helm repo add gradiant-openverso https://gradiant.github.io/openverso-charts/ 

Install Charts : 

Helm install my-open5gs gradiant-openverso/open5gs --version 0.4.1 


Access Open5gs UI: - https://<External IP>:3000(Service IP-Webui ) 
Here ,Open5gs helm charts has been installed on microk8s .Check by Helm list command . 

 

Setup Edgex using helm charts : 

 

Edgex: EdgeX Foundry is an open source, vendor neutral, flexible, interoperable, software platform at the edge of the network, that interacts with the physical world of device, sensors, actuators, and other IoT objects. In simple terms, EdgeX is edge middleware - serving between physical sensing and actuating "things" and our information technology (IT) systems. 

 

Reference link for edgex : 

https://docs.edgexfoundry.org/2.2/ 

 

EdgeX Foundry on Kubernetes using helm charts: 

Follow the below commands to install helm charts (Edgex) on k8s. 

git clone https://github.com/edgexfoundry/edgex-examples.git 

cd edgex-examples 

cd deployment/helm 

kubectl create namespace edgex 

helm install edgex-kamakura -n edgex . 

 

 

 

Reference link -https://github.com/edgexfoundry/edgex-examples/tree/v2.2.0/deployment/helm 

 

Access Edgex UI: https://<External IP>:4000 – (Service IP – Edgex-UI) 

 

 

Here , Edgex is helm chart will be intsalled successfully on k8s. Both the open5gs and Edgex has been deployed in the k8s cluster (Microk8s). 

 

5NR – GNB(CABLE FREE) 

 

Connect Core network to Gnb(cable free) : 

 

After installing open5gs on Microk8s , check whether the pod ,service are successfully deployed on the K8S cluster. 

 

On the K8S Cluster ,service (cluster IP) of AMF,UPF,SMF change by editing the Cluster IP to NodePort for Internal to External traffic. 

 

When you change the service type to NodePort, K8s allocates a port from a predefined range (usually between 30000-32767) on each node in the cluster. This allocated port is then mapped to the same port on the service within the cluster. 

Here's an overview of the process: 

1. Edit the service YAML: Locate the service definition in your Kubernetes manifest file (usually a YAML file) and change the `type` field from `ClusterIP` to `NodePort`. Save the changes. 
 

2. Apply the changes: Use the `kubectl apply` command to apply the modified YAML file to the cluster. This will update the service with the new configuration. 
 

3. Verify the changes: Run `kubectl get services` to view the list of services in the cluster. You should see your service listed with its new type as `NodePort`. Additionally, the output will show the allocated node port number for your service. 
 

By changing the service type from Cluster IP to NodePort, you effectively make the service accessible externally by using the node's IP address and the allocated node port. This enables traffic from outside the cluster to reach the service running on the pods. 

Copy the Service NodePort IP of the AMF and saved it for to use in Gnb Configuration File(enb.cfg). 

 

Add sim details in the Database of open5gs UI: 

 

Add the sim details in Open5gs Ui to stores in the Open5gs mongo-database . 

Required sim details like IMSI ,Ki and OPC ,which is added to the database. 

 

 

 

 

Configmap – enabling port:  

 

In the configmap ,Especially MME,PCRF,HSS,SMF Diameter NFV We should enable the specific port -3868 ,secport - 5868 By uncommenting from the configmap. 

 

 

 

5G NR GNB : 

 

Check whether the Gnb is connected in the same network with Core network and reachable to all the micro service running the K8S cluster by adding default route to the Gnb machine . 

 

1. Verify Network Connectivity: Ensure that the Gnb machine is connected to the same network as the Core network. You can do this by checking the network configuration on the Gnb machine and comparing it with the network configuration of the Core network. Ensure that both networks have compatible IP address ranges and subnet masks. 

 

2. Check Default Gateway: Verify that the Gnb machine has a default gateway configured. The default gateway is the IP address of the router or gateway device that connects the Gnb machine to the wider network, including the Core network as static ip. You can check the network configuration on the Gnb machine to confirm the presence of a default gateway.(Example :- sudo ip r add 0.0.0.0/0 via <Core static ip>) 

 

3. Validate Reachability: To ensure that the Gnb machine is reachable by all the microservices running on the K8S cluster, you can perform the following steps: 

 

a. From a microservice within the K8S cluster, attempt to ping the IP address of the Gnb machine and calico network of K8S cluster. If the ping is successful, it indicates that the Gnb machine is reachable from within the K8S cluster. 

 

b. If the Gnb machine has a firewall enabled, ensure that it allows incoming connections from the IP addresses or subnets used by the K8S cluster. Check the firewall configuration on the Gnb machine and make any necessary adjustments to allow incoming connections. 

 

c. Verify Routing: Check the routing tables on the Gnb machine to ensure that it has a route to reach the IP addresses used by the K8S cluster. This includes the IP addresses assigned to the microservices running on the cluster. If the Gnb machine does not have a route to the K8S cluster's IP addresses, you may need to add a specific route or adjust the default route configuration on the Gnb machine. 

 

Switch to the root user to edit the configuration files of root/lte.../config/enb.cfg by adding the AMF IP address and Change the gtpu address of Gnb IP address (static ip where we set) ,save and exit. 

 

Check whether the Gnb and Core are connected by using  Cable free UI ,where Gnap is connected to the service IP of AMF or Log file generate in gnb0.log file(path nano /tmp/gnb0.log). If Gnb is connected ,then check the (UE device which the sim is attached) Cable free outdoor CPE has connected to the Gnb in Cable free UI. 

 

Check whether the CPE(UE) has IP address .UPF (Core network – NFV) will assign an IP address for CPE(UE) by creating the tunnel in UPF. 

 

1 . The CPE/UE can have an IP address assigned to it by the UPF (User Plane Function) in the core network, which is a component of the 5G network architecture based on NFV (Network Function Virtualization). 

2 . When a CPE/UE connects to the network, it establishes a communication link with the UPF via a tunnel. This tunnel is created between the CPE/UE and the UPF to facilitate the transmission of data packets. 

3.Once the tunnel is established, the CPE/UE can communicate with the core network and other external networks using the assigned IP address. The UPF acts as a gateway for the CPE/UE, forwarding the data packets between the CPE/UE and the appropriate network destinations. 

 

Thus ,Using these Setup , We can achieve 5G low latency for Application level devices.  

 
 
 

 

 
