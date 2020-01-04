*   **_Core Concepts (13%)_**
*   **_Multi-Container Pods (10%)_**
*   **_Pod Design (20%)_**
*   **_State Persistence (8%)_**
*   **_Configuration (18%)_**
*   **_Observability (18%)_**
*   **_Services and Networking (13%)_**

Core Concepts (13%)
===================

Practice questions based on these concepts

*   Understand Kubernetes API Primitives
*   Create and Configure Basic Pods


**_1\. List all the namespaces in the cluster_**
```
kubectl get namespaces
kubectl get ns
```

**_2\. List all the pods in all namespaces_**

```
kubectl get po --all-namespaces
```

**_3\. List all the pods in the particular namespace_**

```
kubectl get po -n <namespace name>
```

**_4\. List all the services in the particular namespace_**

```
kubectl get svc -n <namespace name>
```

**_5\. List all the pods showing name and namespace with a json path expression_**

```
kubectl get pods -o=jsonpath="{.items[*]['metadata.name', 'metadata.namespace']}"
```

**_6\. Create an nginx pod in a default namespace and verify the pod running_**

```
// creating a pod  
kubectl run nginx --image=nginx --restart=Never

// List the pod  
kubectl get po
```

**_7\. Create the same nginx pod with a yaml file_**

```
// get the yaml file with --dry-run flag  
kubectl run nginx --image=nginx --restart=Never --dry-run -o yaml > nginx-pod.yaml

apiVersion: v1  
kind: Pod  
metadata:  
  creationTimestamp: null  
  labels:  
    run: nginx  
  name: nginx  
spec:  
  containers:  
  - image: nginx  
    name: nginx  
    resources: {}  
  dnsPolicy: ClusterFirst  
  restartPolicy: Never  
status: {}

// create a pod   
kubectl create -f nginx-pod.yaml
```

**_8\. Output the yaml file of the pod you just created_**

```
kubectl get po nginx -o yaml
```

**_9\. Output the yaml file of the pod you just created without the cluster-specific information_**

```
kubectl get po nginx -o yaml --export
```

**_10\. Get the complete details of the pod you just created_**

```
kubectl describe pod nginx
```

**_11\. Delete the pod you just created_**

```
kubectl delete po nginx
kubectl delete -f nginx-pod.yaml
```

**_12\. Delete the pod you just created without any delay (force delete)_**

```
kubectl delete po nginx --grace-period=0 --force
```

**_13\. Create the nginx pod with version 1.17.4 and expose it on port 80_**

```
kubectl run nginx --image=nginx:1.17.4 --restart=Never --port=80
```

**_14\. Change the Image version to_ 1.15-alpine for the pod you just created and verify the image version is updated**

```
kubectl set image pod/nginx nginx=nginx:1.15-alpine

kubectl describe po nginx

// another way it will open vi editor and change the version  
kubectl edit po nginx

kubectl describe po nginx
```

**_15\. Change the Image version back to_ 1.17.1 for the pod you just updated and observe the changes**

```
kubectl set image pod/nginx nginx=nginx:1.17.1  
kubectl describe po nginx  
kubectl get po nginx -w 
# watch it
```

**_16\. Check the Image version without the describe command_**

```
kubectl get po nginx -o jsonpath='{.spec.containers[].image}{"\n"}'
```

**_17\. Create the nginx pod and execute the simple shell on the pod_**

```
// creating a pod  
kubectl run nginx --image=nginx --restart=Never

// exec into the pod  
kubectl exec -it nginx /bin/sh
```

**_18\. Get the IP Address of the pod you just created_**

```
kubectl get po nginx -o wide
```

**_19\. Create a busybox pod and run command ls while creating it and check the logs_**

```
kubectl run busybox --image=busybox --restart=Never -- ls
kubectl logs busybox
```

**_20\. If pod crashed check the previous logs of the pod_**

```
kubectl logs busybox -p
```

**_21\. Create a busybox pod with command sleep 3600_**

```
kubectl run busybox --image=busybox --restart=Never -- /bin/sh -c "sleep 3600"
```

**_22\. Check the connection of the nginx pod from the busybox pod_**

```
kubectl get po nginx -o wide// check the connection  
kubectl exec -it busybox -- wget -o- <IP Address>
```

**_23\. Create a busybox pod and echo message ‘How are you’ and delete it manually_**

```
kubectl run busybox --image=nginx --restart=Never -it -- echo "How are you"
kubectl delete po busybox
```

**_24\. Create a busybox pod and echo message ‘How are you’ and have it deleted immediately_**

```
// notice the --rm flag  
kubectl run busybox --image=nginx --restart=Never -it --rm -- echo "How are you"
```

**_25\. Create an nginx pod and list the pod with different levels of verbosity_**

```
// create a pod  
kubectl run nginx --image=nginx --restart=Never --port=80

// List the pod with different verbosity  
kubectl get po nginx --v=7
kubectl get po nginx --v=8
kubectl get po nginx --v=9
```

**_26\. List the nginx pod with custom columns POD\_NAME and POD\_STATUS_**

```
kubectl get po -o=custom-columns="POD_NAME:.metadata.name, POD_STATUS:.status.containerStatuses[].state"
```

**_27\. List all the pods sorted by name_**

```
kubectl get pods --sort-by=.metadata.name
```

**_28\. List all the pods sorted by created timestamp_**

```
kubectl get pods --sort-by=.metadata.creationTimestamp
```

Multi-Container Pods (10%)
==========================

Practice questions based on these concepts

*   Understand multi-container pod design patterns (eg: ambassador, adaptor, sidecar)

**_29\. Create a Pod with three busy box containers with commands “ls; sleep 3600;”, “echo Hello World; sleep 3600;” and “echo this is the third container; sleep 3600” respectively and check the status_**

```
// first create single container pod with dry run flag
kubectl run busybox --image=busybox --restart=Never --dry-run -o yaml -- bin/sh -c "sleep 3600; ls" > multi-container.yaml

// edit the pod to following yaml and create it  
kubectl create -f multi-container.yaml

kubectl get po busybox

```

[multi-container.yaml](https://gist.githubusercontent.com/mehmetkut/d407f2980372fc0b6602dad7621f4f10/raw/863e606d8fcf6239729421d4fc1035b0efa9209e/multi-container.yaml)



**_30\. Check the logs of each container that you just created_**

```
kubectl logs busybox -c busybox1  
kubectl logs busybox -c busybox2  
kubectl logs busybox -c busybox3
```

**_31\. Check the previous logs of the second container busybox2 if any_**

```
kubectl logs busybox -c busybox2 --previous
```

**_32\. Run command ls in the third container busybox3 of the above pod_**

```
kubectl exec busybox -c busybox3 -- ls
```

**_33\. Show metrics of the above pod containers and puts them into the file.log and verify_**

```
kubectl top pod busybox --containers

// putting them into file  
kubectl top pod busybox --containers > file.log  

cat file.log
```

**_34\. Create a Pod with main container busybox and which executes this “while true; do echo ‘Hi I am from Main container’ >> /var/log/index.html; sleep 5; done” and with sidecar container with nginx image which exposes on port 80. Use emptyDir Volume and mount this volume on path /var/log for busybox and on path /usr/share/nginx/html for nginx container. Verify both containers are running._**

```
// create an initial yaml file with this  
kubectl run multi-cont-pod --image=busbox --restart=Never --dry-run -o yaml > multi-container.yaml

// edit the yaml as below and create it  
kubectl create -f multi-container.yaml

kubectl get po multi-cont-pod
```

[multi-container.yaml](https://gist.githubusercontent.com/mehmetkut/aa2046ccd6766b4ecbbaacf13ab828b4/raw/9679cdefce2f973d7981cdac86e6f313c0731d5e/multi-container.yaml)



**_35\. Exec into both containers and verify that main.txt exist and query the main.txt from sidecar container with _**curl localhost**_

```
// exec into main container  
kubectl exec -it  multi-cont-pod -c main-container -- sh  
cat /var/log/index.html

// exec into sidecar container  
kubectl exec -it  multi-cont-pod -c sidecar-container -- sh  
cat /usr/share/nginx/html/index.html

// install curl and get default page  
kubectl exec -it  multi-cont-pod -c sidecar-container -- sh  
# apt-get update && apt-get install -y curl  
# curl localhost
```

* * *

Pod Design (20%)
================

Practice questions based on these concepts

*   Understand how to use Labels, Selectors and Annotations
*   Understand Deployments and how to perform rolling updates
*   Understand Deployments and how to perform rollbacks
*   Understand Jobs and CronJobs

**_36\. Get the pods with label information_**

```
kubectl get pods --show-labels
```

**_37\. Create 5 nginx pods in which two of them is labeled env=prod and three of them is labeled env=dev_**

```
kubectl run nginx-dev1 --image=nginx --restart=Never --labels=env=dev  
kubectl run nginx-dev2 --image=nginx --restart=Never --labels=env=dev  
kubectl run nginx-dev3 --image=nginx --restart=Never --labels=env=dev

kubectl run nginx-prod1 --image=nginx --restart=Never --labels=env=prod
kubectl run nginx-prod2 --image=nginx --restart=Never --labels=env=prod
```

**_38\. Verify all the pods are created with correct labels_**

```
kubectl get pods --show-labels
```

**_39\. Get the pods with label env=dev_**

```
kubectl get pods -l env=dev
```

**_40\. Get the pods with label env=dev and also output the labels_**

```
kubectl get pods -l env=dev --show-labels
```

**_41\. Get the pods with label env=prod_**

```
kubectl get pods -l env=prod
```

**_42\. Get the pods with label env=prod and also output the labels_**

```
kubectl get pods -l env=prod --show-labels
```

**_43\. Get the pods with label env_**

```
kubectl get pods -L env
```

**_44\. Get the pods with labels env=dev and env=prod_**

```
kubectl get pods -l 'env in (dev,prod)'
```

**_45\. Get the pods with labels env=dev and env=prod and output the labels as well_**

```
kubectl get pods -l 'env in (dev,prod)' --show-labels
```

**_46\. Change the label for one of the pod to env=uat and list all the pods to verify_**

```
kubectl label pod/nginx-dev3 env=uat --overwrite
kubectl get pods --show-labels
```

**_47\. Remove the labels for the pods that we created now and verify all the labels are removed_**

```
kubectl label pod nginx-dev{1..3} env-
kubectl label pod nginx-prod{1..2} env-

kubectl get po --show-labels
```

**_48\. Let’s add the label app=nginx for all the pods and verify_**

```
kubectl label pod nginx-dev{1..3} app=nginx
kubectl label pod nginx-prod{1..2} app=nginx

kubectl get po --show-labels
```

**_49\. Get all the nodes with labels (if using minikube you would get only master node)_**

```
kubectl get nodes --show-labels
```

**_50\. Label the node nodeName=nginxnode_**

```
kubectl label node <nodename> nodeName=nginxnode
```

**_51\. Create a Pod that will be deployed on this node with the label nodeName=nginxnode_**

```
kubectl run nginx --image=nginx --restart=Never --dry-run -o yaml > pod.yaml

// add the nodeSelector like below and create the pod  
kubectl create -f pod.yaml
```

[pod.yaml](https://gist.githubusercontent.com/mehmetkut/157d6c56f6b70b4a13bbd853b45dceba/raw/0c2223a3c1a84f5c7a59bf39abad174f583e991d/pod.yaml)



**_52\. Verify the pod that it is scheduled with the node selector_**

```
kubectl describe po nginx | grep Node-Selectors
```

**_53\. Verify the pod nginx that we just created has this label_**

```
kubectl describe po nginx | grep Labels
```

**_54\. Annotate the pods with name=webapp_**

```
kubectl annotate pod nginx-dev{1..3} name=webapp  
kubectl annotate pod nginx-prod{1..2} name=webapp
```

**_55\. Verify the pods that have been annotated correctly_**

```
kubectl describe po nginx-dev{1..3} | grep -i annotations  
kubectl describe po nginx-prod{1..2} | grep -i annotations
```

**_56\. Remove the annotations on the pods and verify_**

```
kubectl annotate pod nginx-dev{1..3} name-  
kubectl annotate pod nginx-prod{1..2} name-

kubectl describe po nginx-dev{1..3} | grep -i annotations  
kubectl describe po nginx-prod{1..2} | grep -i annotations
```

**_57\. Remove all the pods that we created so far_**

```
kubectl delete po --all
```

**_58\. Create a deployment called webapp with image nginx with 5 replicas_**

```
kubectl create deploy webapp --image=nginx --dry-run -o yaml > webapp.yaml

// change the replicas to 5 in the yaml and create it  
kubectl create -f webapp.yaml
```
[webapp.yaml](https://gist.githubusercontent.com/mehmetkut/774fa46827d21b057a3445c1bdeed4b1/raw/4624eafdc3123f3772ad7f19f140907036133c87/webapp.yaml)


**_59\. Get the deployment you just created with labels_**

```
kubectl get deploy webapp --show-labels
```

**_60\. Output the yaml file of the deployment you just created_**

```
kubectl get deploy webapp -o yaml
```

**_61\. Get the pods of this deployment_**

```
// get the label of the deployment  
kubectl get deploy --show-labels

// get the pods with that label  
kubectl get pods -l app=webapp
```

**_62\. Scale the deployment from 5 replicas to 20 replicas and verify_**

```
kubectl scale deploy webapp --replicas=20

kubectl get po -l app=webapp
```

**_63\. Get the deployment rollout status_**

```
kubectl rollout status deploy webapp
```

**_64\. Get the replicaset that created with this deployment_**

```
kubectl get rs -l app=webapp
```

**_65\. Get the yaml of the replicaset and pods of this deployment_**

```
kubectl get rs -l app=webapp -o yaml  
kubectl get po -l app=webapp -o yaml
```

**_66\. Delete the deployment you just created and watch all the pods are also being deleted_**

```
kubectl delete deploy webapp
kubectl get po -l app=webapp -w
```

**_67\. Create a deployment of webapp with image nginx:1.17.1 with container port 80 and verify the image version_**

```
kubectl create deploy webapp --image=nginx:1.17.1 --dry-run -o yaml > webapp.yaml

// add the port section and create the deployment  
kubectl create -f webapp.yaml

// verify  
kubectl describe deploy webapp | grep Image
```
[webapp.yaml](https://gist.githubusercontent.com/mehmetkut/6f8a22f7b49a53ccbee54d74022c27b8/raw/169c549c98ce2f19cd2e532bb657fdaa7656ada7/webapp.yaml)


**_68\. Update the deployment with the image version 1.17.4 and verify_**

```
kubectl set image deploy/webapp nginx=nginx:1.17.4
kubectl describe deploy webapp | grep Image
```

**_69\. Check the rollout history and make sure everything is ok after the update_**

```
kubectl rollout history deploy webapp
kubectl get deploy webapp --show-labels  

kubectl get rs -l app=webapp  
kubectl get po -l app=webapp
```

**_70\. Undo the deployment to the previous version 1.17.1 and verify Image has the previous version_**

```
kubectl rollout undo deploy webapp
kubectl describe deploy webapp | grep Image
```

**_71\. Update the deployment with the image version 1.16.1 and verify the image and also check the rollout history_**

```
kubectl set image deploy/webapp nginx=nginx:1.16.1
kubectl describe deploy webapp | grep Image

kubectl rollout history deploy webapp
```

**_72\. Update the deployment to the Image 1.17.1 and verify everything is ok_**

```
kubectl rollout undo deploy webapp --to-revision=3
kubectl describe deploy webapp | grep Image

kubectl rollout status deploy webapp
```

**_73\. Update the deployment with the wrong image version 1.100 and verify something is wrong with the deployment_**

```
kubectl set image deploy/webapp nginx=nginx:1.100

kubectl rollout status deploy webapp (still pending state)

kubectl get pods (ImagePullErr)
```

**_74\. Undo the deployment with the previous version and verify everything is Ok_**

```
kubectl rollout undo deploy webapp  
kubectl rollout status deploy webapp

kubectl get pods
```

**_75\. Check the history of the specific revision of that deployment_**

```
kubectl rollout history deploy webapp --revision=7
```

**_76\. Pause the rollout of the deployment_**

```
kubectl rollout pause deploy webapp
```

**_77\. Update the deployment with the image version latest and check the history and verify nothing is going on_**

```
kubectl set image deploy/webapp nginx=nginx:latest

kubectl rollout history deploy webapp (No new revision)
```

**_78\. Resume the rollout of the deployment_**

```
kubectl rollout resume deploy webapp
```

**_79\. Check the rollout history and verify it has the new version_**

```
kubectl rollout history deploy webapp

kubectl rollout history deploy webapp --revision=9
```

**_80\. Apply the autoscaling to this deployment with minimum 10 and maximum 20 replicas and target CPU of 85% and verify hpa is created and replicas are increased to 10 from 1_**

```
kubectl autoscale deploy webapp --min=10 --max=20 --cpu-percent=85

kubectl get hpa
kubectl get pod -l app=webapp
```

**_81\. Clean the cluster by deleting deployment and hpa you just created_**

```
kubectl delete deploy webapp

kubectl delete hpa webapp
```

**_82\. Create a Job with an image node which prints node version and also verifies there is a pod created for this job_**

```
kubectl create job nodeversion --image=node -- node -v

kubectl get job -w  
kubectl get pod
```

**_83\. Get the logs of the job just created_**

```
kubectl logs <pod name> // created from the job
```

**_84.Output the yaml file for the Job with the image busybox which echos “Hello I am from job”_**

```
kubectl create job hello-job --image=busybox --dry-run -o yaml -- echo "Hello I am from job"
```

**_85\. Copy the above YAML file to hello-job.yaml file and create the job_**

```
kubectl create job hello-job --image=busybox --dry-run -o yaml -- echo "Hello I am from job" > hello-job.yaml

kubectl create -f hello-job.yaml
```

**_86\. Verify the job and the associated pod is created and check the logs as well_**

```
kubectl get job  
kubectl get po

kubectl logs hello-job-*
```

**_87\. Delete the job we just created_**

```
kubectl delete job hello-job
```

**_88\. Create the same job and make it run 10 times one after one_**

```
kubectl create job hello-job --image=busybox --dry-run -o yaml -- echo "Hello I am from job" > hello-job.yaml

// edit the yaml file to add completions: 10  
kubectl create -f hello-job.yaml
```

[hello-job.yaml](https://gist.githubusercontent.com/mehmetkut/441e3cd7cc46c019f1bc97306db17cc2/raw/3ef199357734849dcc410fb3fa5b97c5bb72ccb4/hello-job.yaml)


**_89\. Watch the job that runs 10 times one by one and verify 10 pods are created and delete those after it’s completed_**

```
kubectl get job -w  
kubectl get po

kubectl delete job hello-job
```

**_90\. Create the same job and make it run 10 times parallel_**

```
kubectl create job hello-job --image=busybox --dry-run -o yaml -- echo "Hello I am from job" > hello-job.yaml

// edit the yaml file to add parallelism: 10  
kubectl create -f hello-job.yaml
```

[hello-job.yaml](https://gist.githubusercontent.com/mehmetkut/e5e630d7f1f43eec227ec3fa066abfc2/raw/18658a3f1380104312ba47ba10ce7f7a83e553ce/hello-job.yaml)


**_91\. Watch the job that runs 10 times parallelly and verify 10 pods are created and delete those after it’s completed_**

```
kubectl get job -w  
kubectl get po

kubectl delete job hello-job
```

**_92\. Create a Cronjob with busybox image that prints date and hello from kubernetes cluster message for every minute_**

```
kubectl create cronjob date-job --image=busybox --schedule="*/1 * * * *" -- bin/sh -c "date; echo Hello from kubernetes cluster"
```

**_93\. Output the YAML file of the above cronjob_**

```
kubectl get cj date-job -o yaml
```

**_94\. Verify that CronJob creating a separate job and pods for every minute to run and verify the logs of the pod_**

```
kubectl get job  
kubectl get po

kubectl logs date-job-<jobid>-<pod>
```

**_95\. Delete the CronJob and verify all the associated jobs and pods are also deleted._**

```
kubectl delete cj date-job

// verify pods and jobs  
kubectl get po  
kubectl get job
```


* * *

State Persistence (8%)
======================

Practice questions based on these concepts

*   Understand PersistentVolumeClaims for Storage

**_96\. List Persistent Volumes in the cluster_**

```
kubectl get pv
```

**_97\. Create a hostPath PersistentVolume named task-pv-volume with storage 10Gi, access modes ReadWriteOnce, storageClassName manual, and volume at /mnt/data and verify_**

```
kubectl create -f task-pv-volume.yaml

kubectl get pv
```

[task-pv-volume.yaml](https://gist.githubusercontent.com/mehmetkut/d5946a14a488346e447177f709614b03/raw/d3d532ea77414f1e1623c8baae6a63a6fc3419d6/task-pv-volume.yaml)



**_98\. Create a PersistentVolumeClaim of at least 3Gi storage and access mode ReadWriteOnce and verify status is Bound_**

```
kubectl create -f task-pv-claim.yaml

kubectl get pvc
```
[task-pv-claim.yaml](https://gist.githubusercontent.com/mehmetkut/1e640f8253c534a44f269aa2687a5d67/raw/a923132845fabf8089e0d1fdef2918b45bd7a2b2/task-pv-claim.yaml)


**_99\. Delete persistent volume and PersistentVolumeClaim we just created_**

```
kubectl delete pvc task-pv-claim  
kubectl delete pv task-pv-volume
```

**_100\. Create a Pod with an image Redis and configure a volume that lasts for the lifetime of the Pod_**

```
// emptyDir is the volume that lasts for the life of the pod
kubectl create -f redis-storage.yaml

[redis-storage.yaml](https://gist.githubusercontent.com/mehmetkut/18767582495737164b2cc4da8ce1d6ed/raw/f58e461d655d2f3f3cf18d66312d853583c27f78/redis-storage.yaml)
```

**_101\. Exec into the above pod and create a file named file.txt with the text ‘This is called the file’ in the path /data/redis and open another tab and exec again with the same pod and verifies file exist in the same path._**

```
// first terminal  
kubectl exec -it redis-storage /bin/sh  
cd /data/redis  
echo 'This is called the file' > file.txt

//open another tab  
kubectl exec -it redis-storage /bin/sh  
cat /data/redis/file.txt
```

**_102\. Delete the above pod and create again from the same yaml file and verifies there is no file.txt in the path /data/redis_**

```
kubectl delete pod rediskubectl create -f redis-storage.yaml  
kubectl exec -it redis-storage /bin/sh  
cat /data/redis/file.txt // file doesn't exist

[redis-storage.yaml](https://gist.githubusercontent.com/mehmetkut/18767582495737164b2cc4da8ce1d6ed/raw/f58e461d655d2f3f3cf18d66312d853583c27f78/redis-storage.yaml)
```

**_103\. Create PersistentVolume named task-pv-volume with storage 10Gi, access modes ReadWriteOnce, storageClassName manual, and volume at /mnt/data and Create a PersistentVolumeClaim of at least 3Gi storage and access mode ReadWriteOnce and verify status is Bound_**

```
kubectl create -f task-pv-volume.yaml  
kubectl create -f task-pv-claim.yamlkubectl get pv  
kubectl get pvc
```

**_104\. Create an nginx pod with containerPort 80 and with a _PersistentVolumeClaim_ **_task-pv-claim_** **_and has a mouth path_** **"/usr/share/nginx/html"**

```
kubectl create -f task-pv-pod.yaml
```

[task-pv-pod.yaml](https://gist.githubusercontent.com/mehmetkut/6315f3dff31df30a16c6316fd458fe6b/raw/5bf25a3bb2d2616fa3e391885634f3182ff853c8/task-pv-pod.yaml
)

* * *

Configuration (18%)
===================

Practice questions based on these concepts

*   Understand ConfigMaps
*   Understand SecurityContexts
*   Define an application’s resource requirements
*   Create & Consume Secrets
*   Understand ServiceAccounts

**_105\. List all the configmaps in the cluster_**

```
kubectl get cm  
     or  
kubectl get configmap
```

**_106\. Create a configmap called myconfigmap with literal value appname=myapp_**

```
kubectl create cm myconfigmap --from-literal=appname=myapp
```

**_107\. Verify the configmap we just created has this data_**

```
// you will see under datakubectl get cm -o yaml  
         or  
kubectl describe cm
```

**_108\. delete the configmap myconfigmap we just created_**

```
kubectl delete cm myconfigmap
```

**_109\. Create a file called config.txt with two values key1=value1 and key2=value2 and verify the file_**

```
cat >> config.txt << EOF  
key1=value1  
key2=value2  
EOFcat config.txt
```

**_110\. Create a configmap named keyvalcfgmap and read data from the file config.txt and verify that configmap is created correctly_**

```
kubectl create cm keyvalcfgmap --from-file=config.txt

kubectl get cm keyvalcfgmap -o yaml
```

**_111\. Create an nginx pod and load environment values from the above configmap keyvalcfgmap and exec into the pod and verify the environment variables and delete the pod_**

```
// first run this command to save the pod yml 

kubectl run nginx --image=nginx --restart=Never --dry-run -o yaml > nginx-pod.yaml

// edit the yaml to below file and create  
kubectl create -f nginx-pod.yaml

// verify  
kubectl exec -it nginx -- env

kubectl delete po nginx
```

[nginx-pod.yaml](https://gist.githubusercontent.com/mehmetkut/520ba7ff44515d016887f103d139f7ec/raw/a41f891649aad1cc14c7e27c5adfc9a0d6bd4149/nginx-pod.yaml
)

**_112\. Create an env file file.env with var1=val1 and create a configmap envcfgmap_** **_from this env file and verify the configmap_**

```
echo var1=val1 > file.env  
cat file.env

kubectl create cm envcfgmap --from-env-file=file.env  
kubectl get cm envcfgmap -o yaml --export
```

**_113\. Create an nginx pod and load environment values from the above configmap envcfgmap and exec into the pod and verify the environment variables and delete the pod_**

```
// first run this command to save the pod yml  
kubectl run nginx --image=nginx --restart=Never --dry-run -o yaml > nginx-pod.yml

// edit the yml to below file and create  
kubectl create -f nginx-pod.yml

// verify  
kubectl exec -it nginx -- envkubectl delete po nginx
```

[nginx-pod.yaml](https://gist.githubusercontent.com/mehmetkut/0c455a26502bd81fa80e1d14b6ed2874/raw/3ea2b830e615f80e80c08a7a2cf0a302a32ad74e/nginx-pod.yaml
)

**_114\. Create a configmap called cfgvolume with values var1=val1, var2=val2 and create an nginx pod with volume nginx-volume which reads data from this configmap cfgvolume and put it on the path /etc/cfg_**

```
// first create a configmap cfgvolume  

kubectl create cm cfgvolume --from-literal=var1=val1 --from-literal=var2=val2

// verify the configmap  
kubectl describe cm cfgvolume

// create the config map   
kubectl create -f nginx-volume.yml

// exec into the pod  
kubectl exec -it nginx -- /bin/sh

// check the path  
cd /etc/cfg  
ls
```

[**nginx-volume.yml**](https://gist.githubusercontent.com/mehmetkut/f0f2989993e7697b60a8568792d4f74a/raw/3384c3e0feb2793653285ca42e88695086974db0/nginx-volume.yml)

**_115\. Create a pod called secbusybox with the image busybox which executes command sleep 3600 and makes sure any Containers in the Pod, all processes run with user ID 1000 and with group id 2000 and verify._**

```
// create yml file with dry-run  
kubectl run secbusybox --image=busybox --restart=Never --dry-run -o yaml -- /bin/sh -c "sleep 3600;" > busybox.yml// edit the pod like below and create  

kubectl create -f busybox.yml

// verify  
kubectl exec -it secbusybox -- sh  
id // it will show the id and group
```

[**busybox.yml**](https://gist.githubusercontent.com/mehmetkut/74f337e76eaebf059e8208204f476f7d/raw/fec5be6e1526521abe6798f57d0d251a26a1645b/busybox.yml)



**_116\. Create the same pod as above this time set the securityContext for the container as well and verify that the securityContext of container overrides the Pod level securityContext._**

```
// create yml file with dry-run  
kubectl run secbusybox --image=busybox --restart=Never --dry-run -o yaml -- /bin/sh -c "sleep 3600;" > busybox.yml

// edit the pod like below and create  
kubectl create -f busybox.yml

// verify  
kubectl exec -it secbusybox -- sh  
id // you can see container securityContext overides the Pod level
```

[**busybox.yml**](https://gist.githubusercontent.com/mehmetkut/bcf0335f2dfdfc44dbe5c4050645af1a/raw/b970222f6bd37e08c8d382132f70873793bfb1c9/busybox.yml)


**_117\. Create pod with an nginx image and configure the pod with capabilities **_NET_ADMIN_** and **_SYS_TIME_** verify the capabilities

```
// create the yaml file  
kubectl run nginx --image=nginx --restart=Never --dry-run -o yaml > nginx.yml

// edit as below and create pod  
kubectl create -f nginx.yml

// exec and verify  
kubectl exec -it nginx -- sh  
cd /proc/1  
cat status

// you should see these values  
CapPrm: 00000000aa0435fb  
CapEff: 00000000aa0435fb
```

[**nginx.yml**](https://gist.githubusercontent.com/mehmetkut/5e913c6b59bd70e11dd21ed5b08cf90f/raw/6dc7b7b7f47f6706bc4b5d94255ed34ab38540d3/nginx.yml)


**_118\. Create a Pod nginx and specify a memory request and a memory limit of 100Mi and 200Mi respectively._**

```
// create a yml file  
kubectl run nginx --image=nginx --restart=Never --dry-run -o yaml > nginx.yml

// add the resources section and create  
kubectl create -f nginx.yml

// verify  
kubectl top pod
```

[**nginx.yml**](https://gist.githubusercontent.com/mehmetkut/6a7fde1d3fd5b121d5e4d9932d5ace8e/raw/417640529d81d15c7d90385e94b0c718002a5404/nginx.yml)



**_119\. Create a Pod nginx and specify a CPU request and a CPU limit of 0.5 and 1 respectively._**

```
// create a yml file  
kubectl run nginx --image=nginx --restart=Never --dry-run -o yaml > nginx.yml

// add the resources section and create  
kubectl create -f nginx.yml

// verify  
kubectl top pod
```

[**nginx.yml**](https://gist.githubusercontent.com/mehmetkut/dcd72ded73ac984ab269de3a3e4a2ab7/raw/0febf63f5c15637ba2037912dababc07b37a08dd/nginx.yml)


**_120\. Create a Pod nginx and specify both CPU, memory requests and limits together and verify._**

```
// create a yml file  
kubectl run nginx --image=nginx --restart=Never --dry-run -o yaml > nginx.yml

// add the resources section and create  
kubectl create -f nginx.yml

// verify  
kubectl top pod
```

[**nginx.yml**](https://gist.githubusercontent.com/mehmetkut/7736b7830e8877d40bb26c8caed5082e/raw/e5c8086703ea707b3ba37e9c61a59768b013bff9/nginx.yml)


**_121\. Create a Pod nginx and specify a memory request and a memory limit of 100Gi and 200Gi respectively which is too big for the nodes and verify pod fails to start because of insufficient memory_**

```
// create a yml file  
kubectl run nginx --image=nginx --restart=Never --dry-run -o yaml > nginx.yml

// add the resources section and create  
kubectl create -f nginx.yml

// verify  
kubectl describe po nginx // you can see pending state
```

[**nginx.yml**](https://gist.githubusercontent.com/mehmetkut/5cd79444fe4d982e0bd1cf673ad0a619/raw/caec140b06dc0128fbe94ac7335d0197622c53bc/nginx.yml)


**_122\. Create a secret mysecret with values user=myuser and password=mypassword_**

```
kubectl create secret generic my-secret --from-literal=username=user --from-literal=password=mypassword
```

**_123\. List the secrets in all namespaces_**

```
kubectl get secret --all-namespaces
```

**_124\. Output the yaml of the secret created above_**

```
kubectl get secret my-secret -o yaml
```

**_125\. Create an nginx pod which reads username as the environment variable_**

```
// create a yml file  
kubectl run nginx --image=nginx --restart=Never --dry-run -o yaml > nginx.yml

// add env section below and create  
kubectl create -f nginx.yml

//verify  
kubectl exec -it nginx -- env
```
[**nginx.yml**](https://gist.githubusercontent.com/mehmetkut/ee51e63cfe84d8e53c6b945df3feb668/raw/7f533fc04e5527a2279b0a9e92385c558e799aff/nginx.yml)

**_126\. Create an nginx pod which loads the secret as environment variables_**

```
// create a yml file  
kubectl run nginx --image=nginx --restart=Never --dry-run -o yaml > nginx.yml

// add env section below and create  
kubectl create -f nginx.yml

//verify  
kubectl exec -it nginx -- env
```

[**nginx.yml**](https://gist.githubusercontent.com/mehmetkut/c925fce3af2ccf2d7327134ffedac322/raw/6a98ae970e8337381e2fd272838f8d5f1030452a/nginx.yml)



**_127\. List all the service accounts in the default namespace_**

```
kubectl get sa
```

**_128\. List all the service accounts in all namespaces_**

```
kubectl get sa --all-namespaces
```

**_129\. Create a service account called admin_**

```
kubectl create sa admin
```

**_130\. Output the YAML file for the service account we just created_**

```
kubectl get sa admin -o yaml
```

**_131\. Create a busybox pod which executes this command sleep 3600 with the service account admin and verify_**

```
kubectl run busybox --image=busybox --restart=Never --dry-run -o yaml -- /bin/sh -c "sleep 3600" > busybox.ymlkubectl create -f busybox.yml

// verify  
kubectl describe po busybox
```

[**busybox.yml**](https://gist.githubusercontent.com/mehmetkut/3c445ea90affc99c6dc8b721a7fb5409/raw/47b5421ce11d2fa03e721733296a01d207f3d10b/busybox.yml)


* * *

Observability (18%)
===================

Practice questions based on these concepts

*   Understand LivenessProbes and ReadinessProbes
*   Understand Container Logging
*   Understand how to monitor applications in kubernetes
*   Understand Debugging in Kubernetes

**_132\. Create an nginx pod with containerPort 80 and it should only receive traffic only it checks the endpoint / on port 80 and verify and delete the pod._**

```
kubectl run nginx --image=nginx --restart=Never --port=80 --dry-run -o yaml > nginx-pod.yaml

// add the readinessProbe section and create  
kubectl create -f nginx-pod.yaml

// verify  
kubectl describe pod nginx | grep -i readinesskubectl delete po nginx
```

[**nginx-pod.yaml**](https://gist.githubusercontent.com/mehmetkut/f17baca532059a0f24e23e5ae732d8f2/raw/0f7b5219a65b81b86fc31082a1f45dfea99e34df/nginx-pod.yaml)

**_133\. Create an nginx pod with containerPort 80 and it should check the pod running at endpoint / healthz on port 80 and verify and delete the pod._**

```
kubectl run nginx --image=nginx --restart=Never --port=80 --dry-run -o yaml > nginx-pod.yaml

// add the livenessProbe section and create  
kubectl create -f nginx-pod.yaml

// verify  
kubectl describe pod nginx | grep -i readinesskubectl delete po nginx
```

[**nginx-pod.yaml**](https://gist.githubusercontent.com/mehmetkut/c91864ae29d8c290537206dd63c0ef72/raw/cd622ebaeae59fb969d6c7964f7ea3e9ec87d640/nginx-pod.yaml)

**_134\. Create an nginx pod with containerPort 80 and it should check the pod running at endpoint /healthz on port 80 and it should only receive traffic only it checks the endpoint / on port 80. verify the pod._**

```
kubectl run nginx --image=nginx --restart=Never --port=80 --dry-run -o yaml > nginx-pod.yaml

// add the livenessProbe and readiness section and create  
kubectl create -f nginx-pod.yaml

// verify  
kubectl describe pod nginx | grep -i readiness  
kubectl describe pod nginx | grep -i liveness
```

[**nginx-pod.yaml**](https://gist.githubusercontent.com/mehmetkut/2d3d3508d2235899e6b4b485902a357d/raw/fa213e4cdd1baa866392d9823df43b937adee2ca/nginx-pod.yaml)

**_135\. Check what all are the options that we can configure with readiness and liveness probes_**

```
kubectl explain Pod.spec.containers.livenessProbe  
kubectl explain Pod.spec.containers.readinessProbe
```

**_136\. Create the pod nginx with the above liveness and readiness probes so that it should wait for 20 seconds before it checks liveness and readiness probes and it should check every 25 seconds._**

```
kubectl create -f nginx-pod.yaml
```

[**nginx-pod.yaml**](https://gist.githubusercontent.com/mehmetkut/26e769c805b0b8b74928da4f857c8d2e/raw/7787b23004b7f9fb1d83f88fdb6757ee8fdad8a0/nginx-pod.yaml)

**_137\. Create a busybox pod with this command “echo I am from busybox pod; sleep 3600;” and verify the logs._**

```
kubectl run busybox --image=busybox --restart=Never -- /bin/sh -c "echo I am from busybox pod; sleep 3600;"kubectl logs busybox
```

**_138\. copy the logs of the above pod to the busybox-logs.txt and verify_**

```
kubectl logs busybox > busybox-logs.txtcat busybox-logs.txt
```

**_139\. List all the events sorted by timestamp and put them into file.log and verify_**

```
kubectl get events --sort-by=.metadata.creationTimestamp

// putting them into file.log  
kubectl get events --sort-by=.metadata.creationTimestamp > file.logcat file.log
```

**_140\. Create a pod with an image alpine which executes this command ”while true; do echo ‘Hi I am from alpine’; sleep 5; done” and verify and follow the logs of the pod._**

```
// create the pod  
kubectl run hello --image=alpine --restart=Never  -- /bin/sh -c "while true; do echo 'Hi I am from Alpine'; sleep 5;done"

// verify and follow the logs  
kubectl logs --follow hello
```

**_141\. Create the pod with this_** **_kubectl create -f_** **_https://gist.githubusercontent.com/mehmetkut/560f7fcb9bbaa7c2b908b67e99e36e1d/raw/e9cfbf7a4e800eb1013815ee8a94b4f7e9ab30cb/not-running.yml. The pod is not in the running state. Debug it._**

```
// create the pod  

kubectl create -f [https://gist.githubusercontent.com/mehmetkut/560f7fcb9bbaa7c2b908b67e99e36e1d/raw/e9cfbf7a4e800eb1013815ee8a94b4f7e9ab30cb/not-running.yml](https://gist.githubusercontent.com/mehmetkut/560f7fcb9bbaa7c2b908b67e99e36e1d/raw/e9cfbf7a4e800eb1013815ee8a94b4f7e9ab30cb/not-running.yml)

// get the pod  
kubectl get pod not-running  
kubectl describe po not-running

// it clearly says ImagePullBackOff something wrong with image  
kubectl edit pod not-running

// it will open vim editor  
                     or  
kubectl set image pod/not-running not-running=nginx
```

**_142\. This following yaml creates 4 namespaces and 4 pods. One of the pod in one of the namespaces are not in the running state. Debug and fix it. https://gist.githubusercontent.com/mehmetkut/1013bf022036ed3f72297d48702e5004/raw/2ed97aaf6360f92f39e8e450a3739c6946d23a41/problem-pod.yaml._**

```
kubectl create -f [https://gist.githubusercontent.com/mehmetkut/1013bf022036ed3f72297d48702e5004/raw/2ed97aaf6360f92f39e8e450a3739c6946d23a41/problem-pod.yaml](https://gist.githubusercontent.com/mehmetkut/1013bf022036ed3f72297d48702e5004/raw/2ed97aaf6360f92f39e8e450a3739c6946d23a41/problem-pod.yaml)// get all the pods in all namespaces  
kubectl get po --all-namespaces

// find out which pod is not running  
kubectl get po -n namespace2

// update the image  
kubectl set image pod/pod2 pod2=nginx -n namespace2

// verify again  
kubectl get po -n namespace2
```

**_143\. Get the memory and CPU usage of all the pods and find out top 3 pods which have the highest usage and put them into the cpu-usage.txt file_**

```
// get the top 3 hungry pods  
kubectl top pod --all-namespaces | sort --reverse --key 3 --numeric | head -3

// putting into file  
kubectl top pod --all-namespaces | sort --reverse --key 3 --numeric | head -3 > cpu-usage.txt

// verify  
cat cpu-usage.txt
```

* * *

Services and Networking (13%)
=============================

Practice questions based on these concepts

*   Understand Services
*   Demonstrate a basic understanding of NetworkPolicies

**_144\. Create an nginx pod with a yaml file with label my-nginx and expose the port 80_**

```
kubectl run nginx --image=nginx --restart=Never --port=80 --dry-run -o yaml > nginx.yaml

// edit the label app: my-nginx and create the pod  
kubectl create -f nginx.yaml
```

[**nginx.yaml**](https://gist.githubusercontent.com/mehmetkut/f09fde088d68ba7f901909275b68ebc8/raw/ba14699f44f46b7340a5eecaacbf60115f20b752/nginx.yaml)

**_145\. Create the service for this nginx pod with the pod selector app: my-nginx_**

```
// create the below service  
kubectl create -f nginx-svc.yaml
```

[**nginx-svc.yaml**](https://gist.githubusercontent.com/mehmetkut/3f5a3a1785eaea10ed104bab0892cd09/raw/fa6466f8ba3bdbb2bd879d7c8f955416599ab1bd/nginx-svc.yaml)

**_146\. Find out the label of the pod and verify the service has the same label_**

```
// get the pod with labels  
kubectl get po nginx --show-labels

// get the service and chekc the selector column  
kubectl get svc my-service -o wide
```

**_147\. Delete the service and create the service with kubectl expose command and verify the label_**

```
// delete the service  
kubectl delete svc my-service

// create the service again  
kubectl expose po nginx --port=80 --target-port=9376

// verify the label  
kubectl get svc -l app=my-nginx
```

**_148\. Delete the service and create the service again with type NodePort_**

```
// delete the service  
kubectl delete svc nginx

// create service with expose command  
kubectl expose po nginx --port=80 --type=NodePort
```

**_149\. Create the temporary busybox pod and hit the service. Verify the service that it should return the nginx page index.html._**

```
// get the clusterIP from this command  
kubectl get svc nginx -o wide

// create temporary busybox to check the nodeport  
kubectl run busybox --image=busybox --restart=Never -it --rm -- wget -o- <Cluster IP>:80
```

**_150\. Create a NetworkPolicy which denies all ingress traffic_**

```yaml
apiVersion: networking.k8s.io/v1  
kind: NetworkPolicy  
metadata:  
  name: default-deny  
spec:  
  podSelector: {}  
  policyTypes:  
  - Ingress
```
