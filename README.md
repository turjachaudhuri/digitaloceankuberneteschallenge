# digitaloceankuberneteschallenge
https://www.digitalocean.com/community/pages/kubernetes-challenge

Digital Ocean Kubernetes Challenge - 

I took part in this challenge , pretty late , on 30.12.2021 .
So , I had basically 1 day to finish the challenge .

As part of this challenge , you can choose from any of the 3 categories - 
Category 1 - New to Kubernetes
Category 2 - Used Kubernetes before
Category 3 - Kubernetes expert

Since I did not have much time at hand , I went with Category 1 . 

Category 1 had 4 challenges you can choose from - 
1. Deploy an internal container registry
2. Deploy a log monitoring system
3. Deploy a scalable SQL database cluster
4. Deploy scalable NoSQL database cluster

For the purpose of this challenge , I chose Challenge 2 - Deplo a log monitoring system .
Details of this challenge - "So your applications produce logs. Lots of logs. How are you supposed to analyze them? A common solution is to aggregate and analyze them using the ELK stack, alongside fluentd or fluentbit."

So , the basic idea is to create an observability system , via which you can push all the logs of the cloud native applications that you have deployed on k8 onto a enterprise grade tool like ELK ( Elastic + Logstash + Kibana ).

To do this challenge , the first thing we need to do is to create a DigitalOcean account if we do not have one .
I signed up for one , and as part of that I got 100USD credit .
But , at the same time , since I signed up for this challenge , in the Category 1 , I also got 60 USD credit for the same ( You can get 120 USD credit for the tougher levels )

With the credit in place , next I deployed a managed k8 cluster within DigitalOcean . 
Managed k8 is awesome because we donot have to do a lot of the infra/maintenance tasks , as those are taken care of by the MSP and we do not have to worry about those tasks .

Creating a DigitalOcean K8 cluster is pretty straightforward .
Step 1 - Select k8 version , I chose the latest
Step 2 - Select datacenter region , I chose Bangalore as I am from India
Step 3 - Choose VPC Network , I chose default , as I did not have any difficult networking requirements / on-prem connectivity needed for this use-case
Step 4 - Cluster capacity , I chose 2 nodes in a nodepool , ( 2vCPU + 4GB / node ) .

With that very quickly I had my managed k8 setup hosted in DigitalOcean , named -> digitalocean-k8-challenge

Once the cluster is created , I had to connect to the cluster using kubectl from my local machine/developer workstation .
This is also pretty simple . 
The idea is to download the kubeconfig file , but we do not need to do it manually . 
I used DigitalOcean Client API tool - doctl .

Steps to install doctl can be found here - https://docs.digitalocean.com/reference/doctl/how-to/install/ ( they are extremely straightforward )

Once doctl was installed , I downloaded the kubeconfig directly via doctl - doctl kubernetes cluster kubeconfig save 3d08b923-a43c-49f7-9e6c-15ca6a2bdaf8 ( in my case ) , and post that , I could leverage standard kubectl commands , as if I am directly connected to that k8 setup.

At this point , I had everything ready to create the actual deployments for the logging stack , all pre-requisites were in place .

Once this was done , I followed this blog (https://www.digitalocean.com/community/tutorials/how-to-set-up-an-elasticsearch-fluentd-and-kibana-efk-logging-stack-on-kubernetes) from DigitalOcean ( step-by-step ) to create the ELK stack , and push container logs via fluentd into the elastic index and visualize it via kibana .

To do the deployment , i create a namespace ( logical segregation ) called kube-logging , all deployment artifacts was pushed there .
Then I deployed the Elastic statefulset and headless service , then deployed Kibana service , and finally the fluentd daemon set .

For all the deployment , the manifest files are uploaded in the souce control , and can be used to re-create the entire thing from scratch again . Also , all detailed steps can be found at the blog link shared above .
