# Contents

[Reference Architecture of Container Service/Devops 2](#_Toc53389433)

[Setup the Virtual Private Network 3](#_Toc53389434)

[Setup a CI server with ECS (Elastic Compute Service) 3](#_Toc53389435)

[Setup Docker CI build In ECS Instance 5](#_Toc53389436)

[Set up the Kubernetes CLUSTER 6](#_Toc53389437)

[Create container registry 8](#_Toc53389438)

[Deploy applicaton to Kubnertes 11](#_Toc53389439)

[Make Change on application 14](#_Toc53389440)

[Make RELASE OF on application 17](#_Toc53389441)

[Blue-Green Deploymen for new version of app 18](#_Toc53389442)

[Auto Scaling Rule on Infrastructure 20](#_Toc53389443)

[Administrator Access and Manage Environment 20](#_Toc53389444)

[Compare Polardb vs Aws Aurora 22](#_Toc53389445)

# Reference Architecture of Container Service/Devops

![](RackMultipart20201019-4-6f5pdh_html_9241afdb9d9183c7.png)

Every organization nowadays embark digital transformation that helps delivery fast time to market products and services, DevOps becoming important factor to shift-left the life-cycle of development and operation with given supports of platform/tools and process (LEAN, Scrum, Kanban).

The Reference Architecture describes the approach of implementing product life-cycle of CI/CD in the modern DevOps practices for cloud native micro service architecture, there are area contains

☐ Source Code Control **– Tool in use for this DEMO - GitHub**

- Create/Maintain artifacts
- Version controlled/ validate/code review and auto test
- Initial concept to production release

☐ Continuous Build (CI) – **Tool in use for this DEMO – Container Registry**

- Plan and pre-production Build
- Automation and Continuous Integration
- Transparent monitoring of development process

☐ Continuous Build (CD) – **Tool in use for this DEMO – Container Kubernetes Service**

- Release, Deployment and Coordination
- Continuous Configuration Automation (Canary, Blue-green deployment)
- Infrastructure, Monitor, Log and Alert Management

![](RackMultipart20201019-4-6f5pdh_html_7cbb38ece3b19098.gif)

# Homework Tips Checklist for Parents

# Setup the Virtual Private Network

☐ Create Virtual Private Network

- Create VPC – **vpc-k8s** , switch – **vswitch-k8s**
- VPC CIDR – 192.168.0.0/16, switch IPv4 CIDR - 192.168.0.0/24
- Zone – Malaysia Zone A

![](RackMultipart20201019-4-6f5pdh_html_515422f521a65b45.png)

![](RackMultipart20201019-4-6f5pdh_html_1c98e31449d3d75f.png)

# Setup a CI server with ECS (Elastic Compute Service)

☐ Under vswitch, and create instance of ECS, this is ensure it will created under vswitch that setup in Malaysia Zone A , and this is for build server purpose

![](RackMultipart20201019-4-6f5pdh_html_a5b5dc5d895ff576.png)

☐ Create ECS Instance

- 2vCPU 4Gib memory Shared Compute
- Using Ubuntu 20.04 64 bit – Disk Standard SSD
- Networking – it will pre-selected vpc-k8s and vswitch-k8s that created early to ensure it sit within the Region replicate.
- Create Security Group – SG-Web-SLB
  - add allowed 8080/8080 since web app using this expose
- Set Logon password for root : Admin123!@# (you can secured it later with SSH Key Pair)
- Set Instance name = k8s-cicd

![](RackMultipart20201019-4-6f5pdh_html_ee47f2107a9b55c1.png)

☐ Review the Instance that created in the list and you can see Internet IP/Private IP, and Status running.

- It is advised not to create directly of internet IP host in ECS instead either create separate VPC zone, SLB or Jump-host design, however this is to demonstrate the purpose of CI for lab purpose.

![](RackMultipart20201019-4-6f5pdh_html_1616b2dcaa4b0c96.png)

☐ Using Putty to connect to Internet IP that has been created in Instance (save in on my stored session)

![](RackMultipart20201019-4-6f5pdh_html_b5bba4c5e399007b.png)

☐ Connecting to Linux ECS instance using SSH protocol (you may change to private/public key access for secured access, for this lab purpose demonstrate I&#39;m using simple password access)

![](RackMultipart20201019-4-6f5pdh_html_99cea319b93a64a5.png)

- Container Based - Install Docker Follow [https://www.digitalocean.com/community/tutorials/how-to-install-and-use-docker-on-ubuntu-20-04](https://www.digitalocean.com/community/tutorials/how-to-install-and-use-docker-on-ubuntu-20-04)

# Setup Docker CI build In ECS Instance

☐ Once you inside the remote SSH console of ECS Instance

- Install Docker Follow [https://www.digitalocean.com/community/tutorials/how-to-install-and-use-docker-on-ubuntu-20-04](https://www.digitalocean.com/community/tutorials/how-to-install-and-use-docker-on-ubuntu-20-04)
  - sudo apt update – update list of packages.
  - sudo apt install apt-transport-https ca-certificates curl software-properties-common – install docker&#39;s pre-requisite packages for Ubuntu – 20.04
  - curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add - - add GPG Key for official Docker repository for CI server system
  - sudo add-apt-repository &quot;deb [arch=amd64] https://download.docker.com/linux/ubuntu focal stable&quot; - add Docker repository to APT sources
  - sudo apt update – APT sources from newly added repo
  - apt-cache policy docker-ce – check and see output of version for Docker from official
  - sudo apt install docker-ce – Install Docker
  - sudo systemctl status docker – Check system deamon on docker status
- Make a webapp directory , and execute git clone
  - git clone https://github.com/turbotmy/docker-hello-world-spring-boot

![](RackMultipart20201019-4-6f5pdh_html_f4392e751ccbca6e.png)

  - Goto webapp folder and
    - Run Docker build -t=&quot;hellow-world-java&quot; .
    - ![](RackMultipart20201019-4-6f5pdh_html_85d5f460dcd10ab1.png)
    - Run Docker images , to check if images are built successfully.

docker images

REPOSITORY TAG IMAGE ID CREATED SIZE

hello-world-java latest 118e46b41dc9 About a minute ago 122MB

\&lt;none\&gt; \&lt;none\&gt; 76c2ba239ee3 About a minute ago 134MB

openjdk 8-jdk-alpine a3562aa0b991 17 months ago 105MB

maven 3.5.2-jdk-8-alpine 293423a981a7 2 years ago 116MB

    - Run Docker run -p 8080:8080 hello-world-java to test the webapp

![](RackMultipart20201019-4-6f5pdh_html_d742f08f80025eed.png)

    - Check on the browser on the public/internet IP with given 8080 port, you will see the page

![](RackMultipart20201019-4-6f5pdh_html_961265ea165dbc50.png)

# Set up the Kubernetes CLUSTER

☐ Create Kubernetes Cluster

- Choose Managed Cluster for demonstrate purpose
  - Cluster name = k8s

![](RackMultipart20201019-4-6f5pdh_html_55051800d6936bee.png)

- Worker configuration – select 2 vCPU core 4G memory , 3 unit worker nodes

![](RackMultipart20201019-4-6f5pdh_html_f6f26593cee62862.png)

- Worker Configurations Operation system – Aliyun Linux 2.1903 (Container-optmized OS)

Logon Type : Password

Logon Password : &quot;Admin123!@#&quot;

![](RackMultipart20201019-4-6f5pdh_html_2ecb04bd443260e2.png)

- Cluster Component Configurations Review

![](RackMultipart20201019-4-6f5pdh_html_eb6a8c29198f737.png)

- Review if created successfully in Container Kubernetes Cluster

![](RackMultipart20201019-4-6f5pdh_html_37546c621cb4b302.png)

# Create container registry

☐ Create namespace of bank for Container Registry.

- Namespace = myrbank (normally naming goes by company, business unit or teamname – to denote collection of repositories for the services/product)

![](RackMultipart20201019-4-6f5pdh_html_70b096b45a44527.png)

☐ Binding Github (source code repository provider) on the container registry

- Binding using my GitHub Repo account = Turbotmy

![](RackMultipart20201019-4-6f5pdh_html_e889a8fd94e21e86.png)

☐ Setup repository and push local codes to master repository of GitHub

- Using Window Terminal with WSL in Ubuntu (Local) , or you can use the ECS CI server setup previously (remote SSH into ECS CI Server)
- For demo purpose, I&#39;m using my local of Ubuntu but will be same as ECS CI Server.
- Run git remote show origin
- Head branch : master

![](RackMultipart20201019-4-6f5pdh_html_de4408c3e1d7d81e.png)

- Git remote path = [https://github.com/turbotmy/docker-hello-world-spring-boot](https://github.com/turbotmy/docker-hello-world-spring-boot)
- Thanks, on example of webapp source codes (docker-hello-world-spring-boot)

![](RackMultipart20201019-4-6f5pdh_html_3cde0fee6cad78e2.png)

☐ Create Repository and link source code on the binding account

- Create new repository in container registry, and binding to Github path
- Review the repository is created, and may view detail/manage
- Repository type = private and code repository = [https://github.com/turbotmy/docker-hello-world-spring-boot](https://github.com/turbotmy/docker-hello-world-spring-boot)

![](RackMultipart20201019-4-6f5pdh_html_6022b7c38099362b.png)

- Review the Build Rules
  - Built-in-rule on tags:release-v$version , that is default practice of release git version naming

![](RackMultipart20201019-4-6f5pdh_html_3b4f1ea68204d234.png)

- Follow Guide of Push image to registry from Alibaba repository that setup
  - sudo docker login
  - sudo docker tag (using short key) with version 1.0 (aligned with default built rules or customized as your company standard definition)
  - sudo docker push

![](RackMultipart20201019-4-6f5pdh_html_d638e9c575913a63.png)

- Once completed, you can inspect in the tags menu of repository

![](RackMultipart20201019-4-6f5pdh_html_29620a339a96c4ef.png)

# Deploy applicaton to Kubnertes

☐ Goto the webapp , top right click on Deploy application , select VPC and Kubernetes

![](RackMultipart20201019-4-6f5pdh_html_c4088675e8ef6d70.png)

☐ Click Deploy, will re-direct to Container Kubernetes Cluster that created, click on application deployments

- Enter a name &quot;hello-world-java-default&quot;

![](RackMultipart20201019-4-6f5pdh_html_e5ec0c83506d6aff.png)

- Click Next , Select image from Container Registry
  - Select the image path from Alicloud container registry that was created
  - Specific the image version using &quot;latest&quot;, and &quot;Always&quot; pull for later preparation when app change.

![](RackMultipart20201019-4-6f5pdh_html_6f800345e5be7cde.png)

  - Click Complete, and can see deployment task is submitted and succeeded

![](RackMultipart20201019-4-6f5pdh_html_58eda7a3af11a7ab.png)

  - Deployment overview can see Pods are created and Running

![](RackMultipart20201019-4-6f5pdh_html_dc64fd9b36906951.png)

  - Create Trigger for pulling the image
    - Copy the Trigger link address confirm, which will be in Container Registry

![](RackMultipart20201019-4-6f5pdh_html_49052817cbcd0f8a.png)

    - Copy the Trigger link address confirm, which will be in Container Registry

![](RackMultipart20201019-4-6f5pdh_html_b1033db2018056c8.png)

  - Goto Container Registry, create a webapp trigger using webhook management to push and redeploy in k8s when new images are built

![](RackMultipart20201019-4-6f5pdh_html_f48a039b933b8c04.png)

- Goto Services, and create service to expose public access through Sever Load Balancer

![](RackMultipart20201019-4-6f5pdh_html_668c67d0266f47f8.png)

- Once create, you can see the app is deployed with external endpoint

![](RackMultipart20201019-4-6f5pdh_html_b8f80fe4a6c3b623.png)

# Make Change on application

☐ Since I have source code in local machine, create a new branch (release-v1.1)

![](RackMultipart20201019-4-6f5pdh_html_ec70695685521ec9.png)

☐ Make a quick change of HelloController to include v1.1

![](RackMultipart20201019-4-6f5pdh_html_a5471091f0879e2e.png)

☐ do a quick check and stage the changes using git add .

![](RackMultipart20201019-4-6f5pdh_html_918c64ba36d3dceb.png)

☐ using git commit the changes

![](RackMultipart20201019-4-6f5pdh_html_3219d7db4d5d8ca4.png)

☐ using git push to origin with credential from the local release-v1.1 branch

![](RackMultipart20201019-4-6f5pdh_html_cf637c9c0823128c.png)

☐ Create a new release-v1.1 in GitHub that picked from release-v1.1 branch

![](RackMultipart20201019-4-6f5pdh_html_18f25fa87689c2cd.png)

☐ Goto Container registry of webapp, and you can see build status is in building, that&#39;s because the rule is met when release is created according to the build rule.

![](RackMultipart20201019-4-6f5pdh_html_e314db7d10c00876.png)

☐ Once the build is completed and Tags 1.1 version is created

![](RackMultipart20201019-4-6f5pdh_html_2b74ba4cc9dd6a1f.png)

☐ you also can see the access log of trigger

![](RackMultipart20201019-4-6f5pdh_html_bd0e4832ec062d04.png)

# Make RELASE OF on application

☐ You can decide when to release the newer version after the CI is passed however on deployment you can either create automatic deployment (according to rule, or using manual workload release

- The Cluster k8s allow you to decide of workload version which demonstrate how you want to plan your deployment strategy
- The 1.1 is where is images is build and passed in CI , and you can create/modify the deployment to pick up which versioning you are deploying through k8s pods and services.

![](RackMultipart20201019-4-6f5pdh_html_267fd9ffcfa60342.png)

- When I choose 1.1 , the entire stack was automatically pick and deploy, if I visit the external IP of k8s service, it reflect the changes of what I did ![](RackMultipart20201019-4-6f5pdh_html_bef1d0763478a6b2.png)

# Blue-Green Deploymen for new version of app

☐ Using Kubernetes Ingress function in Alibaba cloud allow you define weighting of release and apply re-route of traffic to the version you would like to test/roll-out.

- Create workload in container service cluster
- Default is v1.1 , and you have hello-world-java-v1

![](RackMultipart20201019-4-6f5pdh_html_85e2473435f1e85b.png)

- Create a service with hello-world-v1

![](RackMultipart20201019-4-6f5pdh_html_13ee38e35a544265.png)

- Create a service with hello-world-latest

![](RackMultipart20201019-4-6f5pdh_html_85782b8f4e4c3bcc.png)

- Create a Ingress service
  - Enable the Weight of 50% each

![](RackMultipart20201019-4-6f5pdh_html_fdc98eb8a85b5c09.png)

- Inspect the endpoint with 10 times to see the weight result
  - Execute command

curl [http://hello-world-java.cb1b65879e4e2400ab64d446163a10048.ap-southeast-3.alicontainer.com/?[1-10](http://hello-world-java.cb1b65879e4e2400ab64d446163a10048.ap-southeast-3.alicontainer.com/?%5B1-10)] \&gt; output.txt

  - Open the output.txt
    - You can see the result contain of &quot;Hello World&quot; and &quot;Hello World 1.0&quot; evenly distribute the responses with the weight rule define.

![](RackMultipart20201019-4-6f5pdh_html_dcc9a50730f63d36.png)

☐ The Strategy allow you to distribute the traffic , and monitor the result.

# Auto Scaling Rule on Infrastructure

☐ Since the Kubernetes is created with cluster resources

- Scaling Group is created with Worker nodes

![](RackMultipart20201019-4-6f5pdh_html_fde8be7801011de5.png)

- Where you can using the Alicloud Scaling group to create the scaling rules
  - The below is managed Kubernetes cluster where it&#39;s managed the scaling (simple scaling rule).

![](RackMultipart20201019-4-6f5pdh_html_760c9e3a53b82dd9.png)

  - You can also add and create additional scaling rule (e.g predictive scaling rule when average CPU over 80%)

![](RackMultipart20201019-4-6f5pdh_html_acb97bef60a63bd7.png)

# Administrator Access and Manage Environment

☐ There are several ways to access and manage the Kubernetes cluster

- Using RAM Role access to assign role specific on accessing group resources, and via Alicloud UI Console

![](RackMultipart20201019-4-6f5pdh_html_6bfcb81093984d89.png)

- Using Kubectl – Follow the instructions under the k8s cluster connection information

![](RackMultipart20201019-4-6f5pdh_html_6ab34caf4c1e2114.png)

- Using Kubectl – Follow instruction above , and once completed, you can execute the kubectl command to manage the cluster. e.g kubectl config view , kubectl get all, kubectl get pods -n default

![](RackMultipart20201019-4-6f5pdh_html_857a03b34f3728ba.png)

- One of the favorite commands (for admin) is to collect logs of the pod

![](RackMultipart20201019-4-6f5pdh_html_dd32298b03460a4d.png)

# Compare Polardb vs Aws Aurora

| **Area** | **Polar DB** | **AWS Aurora** |
| --- | --- | --- |
| **Service Vendor** | Alibaba Cloud Polar DB – first released on Sept 2017 | AWS Aurora – First released on October 2014 |
| **Availability and Fault Tolerance** | 16 copies across availability zones , through PrallelRaft technologies on leader node. | 6 ways replication across 3 availability zones |
| **Design Architecture** | Use zero-copy and RDMA technologies and focus on I/O requests | Using large amount of CPU to convert into innodb page |
| **Compatibility** | Design on innodb page and efficient on MySQL and PostgreSQL  | Deliver support to MySQL and PostgreSQL on RedoLog level and lead uncertainty to latest version compatibility |
| **Scalability** | Cloud-Native database, up to 100TB in storage and 88vCPU and 710GB memory for computation | Miminum 10GB up to 64TB storage (Sept 2020 AWS announce can up to 128TB Storage), however no official vCPU support and memory computation |
| **Backup** | Reduced to mere seconds and unrelated to size of underlying data , the backup stored within cluster. | Backup and restore for the length of backup retention and are stored in Amazon S3 |

![](RackMultipart20201019-4-6f5pdh_html_668cbf0e931b3fef.gif)
