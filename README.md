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
![ManagedK8SetupinDigitalOcean](https://user-images.githubusercontent.com/6042946/147797094-52096097-90df-44be-927b-7791fc8c7a0a.PNG)

Once the cluster is created , I had to connect to the cluster using kubectl from my local machine/developer workstation .
This is also pretty simple . 
The idea is to download the kubeconfig file , but we do not need to do it manually . 
I used DigitalOcean Client API tool - doctl , which is a client tool , and authenticates with the DigitalOcean servers via API token , as shown below -
![doctlAPItoken](https://user-images.githubusercontent.com/6042946/147797118-543f4eb8-c953-46c6-ad9d-b46f55b1ece4.PNG)

Steps to install doctl can be found here - https://docs.digitalocean.com/reference/doctl/how-to/install/ ( they are extremely straightforward )

Once doctl was installed , I downloaded the kubeconfig directly via doctl - doctl kubernetes cluster kubeconfig save 3d08b923-a43c-49f7-9e6c-15ca6a2bdaf8 ( in my case ) , and post that , I could leverage standard kubectl commands , as if I am directly connected to that k8 setup.
As you can see below , I am connected to the Digital Ocean managed k8 cluster setup in the cloud -
![KubectlGetNodes](https://user-images.githubusercontent.com/6042946/147797135-1498e43a-1e47-49d7-a2d7-99e81d344602.PNG)

Actually , using doctl , it is easy to download the kubeconfig details , and append it to the kubeconfig file in the local user profile , as shown below -
![localkubeconfigwithdigitaloceanremoteclusterdata](https://user-images.githubusercontent.com/6042946/147797165-770553b2-9da4-4125-b593-2a670cbb2925.PNG)

At this point , I had everything ready to create the actual deployments for the logging stack , all pre-requisites were in place .

Once this was done , I followed this blog (https://www.digitalocean.com/community/tutorials/how-to-set-up-an-elasticsearch-fluentd-and-kibana-efk-logging-stack-on-kubernetes) from DigitalOcean ( step-by-step ) to create the ELK stack , and push container logs via fluentd into the elastic index and visualize it via kibana .

To do the deployment , i create a namespace ( logical segregation ) called kube-logging , all deployment artifacts was pushed there .
Then I deployed the Elastic statefulset and headless service , then deployed Kibana service , and finally the fluentd daemon set .

A capture of all the kubernetes objects that are deployed into the kube-logging namespace can be found below -
![AllResourcesDeployedinK8LoggingNamespace](https://user-images.githubusercontent.com/6042946/147797029-45f75238-e383-41d1-a13d-4ae11b3deae6.PNG)

All the services are exposed via PortForwarding , and then accessed from localhost , for ease of use during POC .
For example , the Kibana service is exposed via PortForwarding , as shown below :
![KibanaPortForwardingCommand](https://user-images.githubusercontent.com/6042946/147796944-ff31144c-80a7-4bd4-ab20-3aafdd92d3be.PNG)

As soon as the port-forwarding is done , the kibana service can be accessed via localhost , just like any other Web UI , as shown below -
![KibanaRunningOnLocalhostInK8ViaPortForwarding](https://user-images.githubusercontent.com/6042946/147797005-66bf652b-4e1f-4174-8670-dc3baa48ceaf.PNG)

The same is done for the elasticsearch service as well -
Port forwarding done via command - 
![ESPortForwardingCommand](https://user-images.githubusercontent.com/6042946/147797052-124bf90a-ba62-4d7d-aae6-211c9aa49301.PNG)

Service is listening on assigned port in localhost -
![ESRunningonK8exposedviaPortForwarding](https://user-images.githubusercontent.com/6042946/147797075-d9f68064-985f-4603-bfea-7bfca207529a.PNG)

For all the deployment , the manifest files are uploaded in the souce control , and can be used to re-create the entire thing from scratch again . Also , all detailed steps can be found at the blog link shared above .

We also deploy fluentd as a daemonset that will route all log data from the containers to elasticsearch -
![fluentd_configuration](https://user-images.githubusercontent.com/6042946/147813648-bbebae71-6969-4053-be53-58ead4d8655f.PNG)

For the other deployment manifests please refer to the source control files .

A sample Kibana dashboard is shown below -
![SampleKibanaDashboard](https://user-images.githubusercontent.com/6042946/147813536-b355688e-cf11-4fef-876d-8fe50c8e8cab.PNG)
