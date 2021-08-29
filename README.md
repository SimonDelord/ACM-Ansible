# Introduction
This is my github repo for all ACM-Ansible related activities.

I am going to try to integrate some of the ACM functions with Ansible and see how they can be used together.

There is currently a single use case (August-2021).

## First Use Case - An infra/devops person creates an application and wants to deploy it in a High-availability (HA) mode.
   
   ![alt text](https://github.com/SimonDelord/ACM-Ansible/blob/main/images/ACM-Ansible-HA.png)



This sample application is a stateless application.
The K8 resources for it are defined in the deploy folder. It is a simple hello-world in this case (deployment, service, route). 

In this use case, ACM is managing 2 OCP clusters and deploys the same resources on both clusters as part of the application creation.

The first cluster is of the following type: *.apps.cluster-1.melbourneopenshift.com \ 
The second cluster is of the following type: *.apps.cluster-2.melbourneopenshift.com 

   
   
Now to deploy apps in an HA fashion, we want the same DNS entry for both on the two different clusters.\
For this, I use a not very refined approach (it could be done more elegantly).
   
   - ***First step*** Modify the Ingress Controller on both managed clusters
   
I modify the Ingress Controller Operator on each cluster to create for all apps an ha-apps.melbourneopenshift.com Custom Domain

Log into each of the two clusters and then type

oc edit ingresses.config/cluster -o yaml

replace the current 
spec:
  domain: apps.cluster-X.melbourneopenshift.com
   
with

spec:
  appsDomain: ha-apps.melbourneopenshift.com
  domain: apps.cluster-X.melbourneopenshift.com

Now, any app that will get deployed outside openshift namespaces will have by default a route named svc-name-namespace.ha-apps.melbourneopenshift.com \

     - ***Second step*** 
Ok now for the automation.
I created two Ansible-playbooks:
      - one to create the entries for *.ha-apps.melbourneopenshift.com mapped to the 2 IP @s of the OCP clusters (<cluster-1> and <cluster-2>)
      - one to do any post hook required (typically a curl command to the relevant URL).

Those playbooks need to be organised into folders.\
These folders (as of ACM2.2 and ACM2.3) need to be under another folder otherwise ACM doesn't seem to pick up that there are a Prehook and Posthook
folder in the top repo. 

In this demo, they are under the deploy folder.

From my experience you need a bit of time for the DNS entries to propagate, but after a few minutes the setup should work.
An example of the demo can be found there https://youtu.be/xFZD0Oh92wg   

