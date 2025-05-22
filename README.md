# AWS Kubernetes Capstone Project


## Description
This Capstone project is build inside the EC2 instance (kube-capstone). It can be access via the aws console or using Github repo. Examples are used from class activities, part 2 has process of how the resources were created.  


# Part 1:  Deployment Server 

##  For AWS Console access
* Log in to eruser102 account (can be provided)
* Start kube-capstone EC2 instance and EC2 connect to the server
* Run this command aws sts get-caller-identity to see if you have access key attached, (it should show, if not key will be provided)


##  Check installation
* Kubernetes - type kubectl, then press RETURN to test.
* eksctl - type eksctl, then press ENTER.
* Docker - type docker, ENTER then "sudo systemctl enable docker" and "sudo systemctl start docker" to boot and start the service. (Docker hub is not used.)
* git - type git and then press ENTER.
* node - type npm and then press ENTER.
* helm - type helm and then press ENTER. (but only for practice)
* Mariadb - installed but only used for practice 


##  ECR repo
* 2 repos should be active with its images. Below "Create ECR repos" section has more information.
* There are 3 version of images in events-website. All the commands are on mentioned on part 2.
* If there are any issue it must be that the ECR image is using either latest or 2.0 version.


##  Create eks cluster & nodes
* ls and cd into eksctlcluster
* Run eks cluster using : eksctl create cluster -f cluster.yaml
* Check AWS console -> EKS cluster -> "kc-cluster" will be created
* Takes about 20-30mins to create
* Check EC2, 3 t3.medium nodes will be created and running
* All resources will be created.


##  Run the commands 
* kubectl cluster-info (info about Kubernetes control plane and CoreDNS )
* kubectl get services (kubernetes ClusterIP)
* kubectl get nodes ( 3 running nodes)
* kubectl get pods (this won't show any results yet)


##  Useful commands for eks
- To run the cluster
* eksctl create cluster -f cluster.yaml in cd 
- To update cluster
* eksctl update cluster -f 
- To delete cluster
* eksctl delete cluster -f cluster.yaml


## Kubernetes Deployments

## For events-api
* cd out of ekscluster
* cd ~/eventsapp/kubernetes-config
* kubectl apply -f api-deployment.yaml
* kubectl get deployments (events-api will be ready)
* kubectl get pods and kubectl describe pod (for the details)


## For events-web
* be in kubernetes-config
* kubectl apply -f web-deployment.yaml
* kubectl get deployments (this will show both events)
* kubectl get pods (3 pods will run as this pod has 2 replicas set) 


## For ClusterIP 
* from kubernetes-config create: events-api-svc by kubectl apply -f api-service.yaml
* kubectl get service -> events-api-svc, cluster-IP will be created


## For Load Balancer
* from kubernetes-config create: events-web-svc by using kubectl apply -f web-service.yaml
* kubectl get service ( 3 service will run)
* copy and paste the long EXTERNAL-IP: a062e8c70fae4403597ec3ccf0541222-1415513312.us-east-1.elb.amazonaws.com
* this events app page


## Modify replicas
* change replicas from 2 to 3 in web-deployment.yaml file to test the application
* nano or vi web-deployment.yaml (to exit nano, cntrl o and cntrl x then enter) (vi -> :wq!)
* kubectl apply -f web-deployment.yaml
* kubectl get pods ->  1 new pod will start runnning.
* change the replica back to 2 and it will terminate new one.
* nano web-deployment.yaml -> replicas 3 to 2, kubectl apply -f web-deployment.yaml (to reconfigure)
* kubectl get pods (show the terminating pod)
* alternatively, you can delete this new pod using following: 
* * ex: kubectl delete pod events-web-5d58474486-7s8k9

## Adding Resource Requests, Limits and Autoscaling
* cd out of kubernetes-config
* cd ~/eventsapp/HPA-demo/
* run kubectl apply -f deployment.yaml to create autoscale-app from deployment.apps
* kubectl get deployments
* kubectl get pods ( 2 new pods will be running)
* kubectl top pods (shows CPU and memory)

## For horizontal pod autoscaler (HPA)
* kubectl apply -f autoscale.yaml 
* kubectl get hpa (shows target cpu 0% to 60%, min & max pods, replicas, etc. )
* kubectl top pods
* kubectl get service -> copy the external IP from 

## For testing
* cd ~/eventsapp/HPA-demo -> ls -> cat service.yaml to check the ports 80 and targetport 8080. 
* kubectl apply -f service.yaml
* kubectl get service
* copy External IP for autoscale-app-svc and paste it in the browser which loads with demo app version 1.0 (can take some time to load)
- Note keep this browser running


## Locate the for loop in the app 
* kubectl get pods
* grab a pod name from the list then edit POD-NAME-HERE and run kubectl exec -i -t POD-NAME-HERE -- /bin/bash
* ls
* cat app.js ( to check the loop)
|var x = 0.0001;
   |for (var i = 1; i <= 4000000; i++) {
    |x += Math.tan(i*Math.random());
   |}
* exit


## Autoscaling the App - Load test

Open 2 new sessions via EC2:
- working session: use for loop, add the saved External IP for autoscale-app-svc: while true; do curl http://ac43dcff309ae4646b06914679c99c30-593099355.us-east-1.elb.amazonaws.com/; done;  
- 2nd session: toinvestigate HPA and pods scaling continue executing the commands:
  * kubectl get hpa (target cpu will change after every running loop)
  * kubectl top pods
  * kubectl get pods
- 3rd session run the while loop again
- Use CTRL+C to stop loops in both windows. After a while the targets will come down to cpu: 0% - 60%. Use above commands to check the status:
* kubectl get hpa
* kubectl top pods
* kubectl get pods


## Test updating your application with a blue/green update
## Rolling Updates (test)
* cd ~/eventsapp/kubernetes-config/
* nano web-deployment.yaml file edit replication to 4 
* run kubectl apply -f web-deployment.yaml (few will get terminated and new will be created)
- reload the External IP browser to check if its still running. (events app browser)

## Upgrade to Version 2.0
* cd ~/eventsapp/kubernetes-config/ -> nano web-deployment.yaml
* modify the container image version from latest to 2.0 "URI: 441257995286.dkr.ecr.us-east-1.amazonaws.com/events-website:2.0"
* check kubectl get pods to see new running pods
* reload the External IP browser and the Version changes to 2.0 (my latest version is v1.0)
* in another window apply kubectl get pods -w (watch the pod)
* kubectl rollout undo deployments/events-web (on the main server windor to undo the deployment)
* change the version back to latest in web-deployment.yaml then apply kubectl apply -f web-deployment.yaml

## Blue/Green Deployments
* cd ~/eventsapp/kubernetes-config/
* cp web-deployment.yaml web-deployment-v2.yaml
* nano web-deployment-v2.yaml file edit from “name: events-web” to “name: events-web-v2.0”
* modify ver:1.0 to ver:2.0 (where applicable)
* modify container image url from version latest to 2.0 or just copy URI image from the ECR events-web, version 2
* kubectl apply -f web-deployment-v2.yaml configure new deployment.
* kubectl get deployments
* kubectl get pods

## Switch the Load Balancer Labels
* edit the web-service.yaml
* in the selector, change “ver: v1.0” to “ver: v2.0”
* kubectl apply -f web-service.yaml
* reload the app in External IP browser until you see Version 2.0

## Rollback
* edit the web-service.yaml
* in the selector, change “ver: v2.0” to “ver: v1.0”
* kubectl apply -f web-service.yam
* reload the app in External IP browser until you see no Version on the heading. 

## Canary Release
* edit web-deployment-v2.yaml file, replicas from 4 to 1
* apply kubectl apply -f web-deployment-v2.yaml file
* nano web-service.yaml remove “ver: v1.0” line from selector (this will cause the load balancer to select all pods not matter the version)
* kubectl apply -f web-service.yaml

## Testing the Canary Release
* 4 copies of version 1.0 will load and only 1 copy of version 2.0 will load. Will need to refresh page couple of times.
* Or run the script with External IP link: while true; do curl http://EXTERNAL-IP/ | grep -i "version 2.0" && sleep 1; done; (Whenever “version 2.0” is found, it will be displayed)
* ctrl C to stop the script.
* nano web-service.yaml and add the “ver: v1.0” line back and apply kubectl apply -f web-service.yaml
* set the replicas to 2 in web-deployment.yaml file run kubectl apply command
- For this project "events-web-v2.0” is stay acive

## Deploying StatefulSets (incomplete) 
* cd ~/eventsapp/statefulset-demo/
* kubectl apply -f statefulset-demo.yaml
* kubectl get pods
* kubectl get pvc
* can take several minutes to load the demo, you can also check in the AWS console under EKS cluster resources -> StatefulSets
* tried to create v2 but it failed as well (this didn't work for me so I didn't complete)

## Running Kubernetes Jobs (incomplete)
* cd eventsapp/database-initializer
* new repo was created for the job using docker build, cronjob.yaml file was created and deleted using kubectl apply -f cronjob.yaml and kubectl delete cronjob hello
* api-deployment.yaml has been edited, kubectl apply -f api-deployment.yaml


## Delete
* kubectl delete deployment autoscale-app
* kubectl delete hpa autoscale-app-hpa
* kubectl delete service autoscale-app-svc
* kubectl get service
* kubectl delete service events-api-svc
* kubectl delete service events-web-svc 
* kubectl delete service kubernetes 
* kubectl get pods
* kubectl delete pod (namespace)

* For deleting main cluster - cd ~/ekscluster
  * eksctl delete cluster -f cluster.yaml
  * will terminated all the remaining infra along with 3 nodes
  * lastly stop Ec2 instance 




# Part 2:  Details on the process steps ##

## Create and configure the deployment environment
 
Create new t2.micro instance with exisitng AMI, security group with all SSH, HTTP and HTTPS rule allowed. [Access and Secrets key is based on eruser102 user key.]
* Connect to the newly build server and run "aws configure" add the access and secrets key - use "aws sts get-caller-identity" to confirm access in the account.


## Practice test
Check the ports:
Under EC2, check security group -> enable TCP protocols
* Port - 8082 
* Port - 8080
* Port - 22 

To test the ports:
Create 'mkdir eventsapp' folder and downloaded the repo inside this new folder: 
* git init
* git pull https://github.com/msutton150/eventsappstart.git

cd into events-api then ran these commands:
* npm install
* node server.js

Also, do same for events-website folder in another EC2 connect session.

To test the app:
Use the public IP address of the EC2 instance http://instancepublicip:8080 and paste in the browser.

Edit the title name inside ~/eventsapp/events-website/views/layouts/default.hbs
re run following commands in both folders:
* npm install
* node server.js

Refresh and check the browser changes will be updated. CTRL+C in both sessions to exit.


## Containerize your application and store it in a repository


## Create ECR repos

Create 2 mutebale repositories in ECR - Elastic Container Registry with encryption type as AES-256
* events-api
* events-website
Grab the URI image for both repos.

## Push images 

Use the commands to login and build and push the image to the registry. Edit URI for events-api folder:
* docker login -u AWS -p $(aws ecr get-login-password --region us-east-1) 441257995286.dkr.ecr.us-east-1.amazonaws.com/events-website
* docker build -t events-website .
* ocker tag events-api:latest 441257995286.dkr.ecr.us-east-1.amazonaws.com/events-website
* docker push 441257995286.dkr.ecr.us-east-1.amazonaws.com/events-website:latest

## Create another version

Edit /eventsapp/events-website/views/layouts/default.hbs by adding **Version 2.0** with in <h1><your name> Events App **Version 2.0**</h1>
then ran above commands from $ docker build onwards and add :2.0 instead of :latest.

- Example: docker push 441257995286.dkr.ecr.us-east-1.amazonaws.com/events-website:2.0

Also, create another version 3.0 to check everything is working fine. 


## Deploy an EKS cluster

Create cluster.yaml file
* ls
* mkdir eksctlcluster
* touch cluster.yaml (or touch cluster-config.yaml to create new file)
* nano cluster.yaml
* Paste cluster config code from: https://roiworkshopmaterials.s3.us-west-2.amazonaws.com/cluster.yaml

Edit 
* cluster name, region, and availability zones.
* Length constraints: maximum length of 128. So, make sure to reduce the name of the cluster, example: my-cluster. 
* change the managedNodeGroups:
    instanceType: t3.medium
    minSize: 3
    desiredCapacity: 3
    maxSize: 6 

Save and create the cluster using following command 

* eksctl create cluster -f cluster.yaml 

- Check the progress in AWS console -> CloudFormation stacks, if anything fails you can easily delete the cluster using following command and check the stack to find more info.

* eksctl delete cluster -f cluster.yaml

- You can also check AWS EKS cluster: overview, resources, add-ons, etc for the progress.
- Can take 20-30 mins to deploy.

Once the cluster is created, check kubectl cluster access to see everything is running:

* kubectl cluster-info (information about the cluster)
* kubectl get services (service named Kubernetes)
* kubectl get nodes (3 running nodes in EC2 instance)



## Deploy your web-based application, including the backend database


## Create yaml files (pre-created)

* cd eventsapp 
* mkdir kubernetes-config 
* cd kubernetes-config
* touch api-deployment.yaml and touch web-deployment.yaml
* nano api-deployment.yaml and paste the file (edit URI with events-web URI image, update app name as web in 5 places and port as 8080)

apiVersion: apps/v1
kind: Deployment
metadata:
 labels:
   app: events-api
 name: events-api
spec:
 replicas: 1
 selector:
   matchLabels:
     app: events-api
     ver: v1.0
 template:
   metadata:
     labels:
       app: events-api
       ver: v1.0
   spec:
     containers:
     - image: 441257995286.dkr.ecr.us-east-1.amazonaws.com/events-api:latest (URI is located in ECR repositories)
       name: events-api
       ports:
       - containerPort: 8082

## Create pods

* kubectl apply -f api-deployment.yaml and kubectl apply -f web-deployment.yaml 

Verify deployments
* kubectl get deployments
* kubectl get pods

Investigate ReplicaSets
* kubectl delete pod <paste pod name here>
* kubectl get pods [new pod was started to replace the deleted one]
* replicas are changed to 3 in the web-deployment.yaml file which is configured by above kubectl apply command. 
* 2 new pods will run then change replicas back to 1.
* Use kubectl apply -f web-deployment.yaml and kubectl get pods to configure ans watch the pods getting terminated.

You can check the progress in EKS cluster.


## Create a ClusterIP and Load Balancer

* check the deployments are running
* kubectl get deploy


## Create api-service.yaml file

* cd ~/eventsapp/kubernetes-config
* touch api-service.yaml
* nano api-service.yaml 
* paste the yaml file
 
apiVersion: v1
kind: Service
metadata:
 labels:
   app: events-api-svc
 name: events-api-svc
spec:
 ports:
 - port: 8082
   protocol: TCP
   targetPort: 8082
 selector:
   app: events-api
   ver: v1.0
 type: ClusterIP


## Modify web-deployment.yaml with following environment variable (SERVER)

env:
- name: SERVER
  value: "http://events-api-svc:8082"

* kubectl apply -f web-deployment.yaml

# Create web-service.yaml file

* touch web-service.yaml
* nano web-service.yaml and paste 

apiVersion: v1
kind: Service
metadata:
 labels:
   app: events-web-svc
 name: events-web-svc
spec:
 ports:
 - port: 80
   protocol: TCP
   targetPort: 8080
 selector:
   app: events-web
   ver: v1.0
 type: LoadBalancer

* kubectl apply -f web-service.yaml --to deploy--
* kubectl get service
* copy and paste EXTERNAL-IP for the events-web-svc in new browser tab example: http://a8d6bc86d9c7444079cbf9154fa8041b-893998906.us-east-1.elb.amazonaws.com/ it should run with your given name and 2 events. 
* change replicas to 3 in web-deployment.yaml file to test the application, repeated kubectl apply command and get pods to check the running pods.
* change back to 1 replica.



