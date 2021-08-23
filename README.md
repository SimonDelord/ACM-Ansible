# Introduction
This is my github repo for all ACM-Ansible related activities.\
I am going to try to integrate some of the ACM functions with Ansible and see how they can be used together.

ACM is being used as a way of deploying:
   - triggers and pipelines to any cluster where developers will be using them
apps in both dev and prod environments

GitHub is used as the code repo environment:
   - some repositories have been used for ACM to deploy 

The use case is the following (3 steps demo):
   - **First Step** An infra/devops person creates various Tekton Tasks, Pipelines, Trigger components (TriggerTemplates, TriggerBindings) and uploads them into a GitHub repo. All these Tekton "capabilities" are then imported to the relevant OCP clusters via ACM.
   
   ![alt text](https://github.com/SimonDelord/ACM-Ansible/images/ACM-Ansible-HA.png)



This is the set of resources for the deployment of a "stateless" application (hello-world in this case) in a HA mode.
Basically, I use 2 OCP clusters and deploy the same resources on both clusters.
the first cluster is of the following type: *.apps.<cluster-1>.melbourneopenshift.com
the second cluster is of the following type: *.apps.<cluster-2>.melbourneopenshift.com

Now to deploy apps in an HA fashion, we want the same DNS entry for both on the two different clusters.
For this, I use the sledgehammer (it could 100% be done more elegantly).

I modify the Ingress Controller Operator on each cluster to create for all apps an ha-apps.melbourneopenshift.com Custom Domain

Log into each of the two clusters
then type

oc edit ingresses.config/cluster -o yaml

replace the current 
spec:
  domain: apps.<cluster-X>.melbourneopenshift.com
   
with

spec:
  appsDomain: ha-apps.melbourneopenshift.com
  domain: apps.<cluster-X>.melbourneopenshift.com

Now, any app that will get deployed outside openshift namespaces will have by default a route named svc-name-namespace.ha-apps.melbourneopenshift.com

Ok now for the automation.
I created two Ansible-playbooks, one to create the entries for *.ha-apps.melbourneopenshift.com mapped to the 2 IP @s of the OCP clusters (<cluster-1> and <cluster-2>)
one to do any post hook required (typically a curl command to the relevant URL).

Those playbooks need to be organised into folders (these folders need to be under another folder otherwise ACM doesn't seem to pick up that there are a Prehook and Posthook
folder). In this demo, they are under the deploy folder.

From my experience you need a bit of time for the DNS entries to propagate, but after a few minutes the setup should work.
