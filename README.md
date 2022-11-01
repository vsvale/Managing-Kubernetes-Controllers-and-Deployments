# Managing-Kubernetes-Controllers-and-Deployments


## Notes
Inside the kube-system namespace, there's a collection of controllers supporting parts of the cluster's control plane
How'd they get started since there's no cluster when they need to come online? Static Pod Manifests
`kubectl get --namespace kube-system all`

Let's look more closely at one of those deployments, requiring 2 pods up and runnning at all times.
`kubectl get --namespace kube-system deployments coredns`

Daemonset Pods run on every node in the cluster by default, as new nodes are added these will be deployed to those nodes.
There's a Pod for our Pod network, router and one for the kube-proxy.
`kubectl get --namespace kube-system daemonset`

One daemonset per node
`kubectl get nodes`

Creating a Deployment Imperatively, with kubectl create, you have lot's of options available to you such as image, container ports, and replicas
`kubectl create deployment hello-world --image=gcr.io/google-samples/hello-app:1.0 --replicas=5`

Check out the status of our imperative deployment
`kubectl get deployment`

Now let's delete that and move towards declarative configuration.
`kubectl delete deployment hello-world`

Let's start off declaratively creating a deployment with a service.
`kubectl apply -f https://raw.githubusercontent.com/vsvale/Managing-Kubernetes-Controllers-and-Deployments/main/deployment.yaml`

Check out the status of our deployment, which creates the ReplicaSet, which creates our Pods
`kubectl get deployments hello-world`

The first replica set created in our deployment, which has the responsibility of keeping of maintaining the desired state of our application but starting and keeping 5 pods online. In the name of the replica set is the pod-template-hash
`kubectl get replicasets`

The actual pods as part of this replicaset, we know these pods belong to the replicaset because of the pod-template-hash in the name
`kubectl get pods`

But also by looking at the 'Controlled By' property
`kubectl describe pods | head -n 15`

It's the job of the deployment-controller to maintain state. Let's look at it a litte closer. The selector defines which pods are a member of this deployment.
Replicas define the current state of the deployment, we'll dive into what each one of these means later in the course. In Events, you can see the creation and scaling of the replica set to 5
`kubectl describe deployment`

Remove our resources
`kubectl delete deployment hello-world`
`kubectl delete service hello-world`

Deploy a Deployment which creates a ReplicaSet
`kubectl apply -f https://raw.githubusercontent.com/vsvale/Managing-Kubernetes-Controllers-and-Deployments/main/deployment.yaml`
`kubectl get replicaset`

Let's look at the selector for this one...and the labels in the pod template
`kubectl describe replicaset hello-world`

Let's delete this deployment which will delete the replicaset
`kubectl delete deployment hello-world`
`kubectl get replicaset`

Deploy a ReplicaSet with matchExpressions
`kubectl apply -f https://raw.githubusercontent.com/vsvale/Managing-Kubernetes-Controllers-and-Deployments/main/deployment-me.yaml`

Check on the status of our ReplicaSet
`kubectl get replicaset`

Let's look at the Selector for this one...and the labels in the pod template
`kubectl describe replicaset hello-world`

Deleting a Pod in a ReplicaSet, application will self-heal itself
`kubectl get pods`
`kubectl delete pods hello-world`[tab][tab]
`kubectl get pods`

Isolating a Pod from a ReplicaSet
`kubectl get pods --show-labels`


Edit the label on one of the Pods in the ReplicaSet, the replicaset controller will create a new pod
`kubectl label pod hello-world-[tab][tab] app=DEBUG --overwrite`
`kubectl get pods --show-labels`

Taking over an existing Pod in a ReplicaSet, relabel that pod to bring it back into the scope of the replicaset...what's kubernetes going to do?
`kubectl label pod hello-world-[tab][tab] app=hello-world-pod-me --overwrite`

One Pod will be terminated, since it will maintain the desired number of replicas at 5
`kubectl get pods --show-labels`
`kubectl describe ReplicaSets`




#Demo 5 - Node failures in ReplicaSets
#Shutdown a node
ssh c1-node3
sudo shutdown -h now


#c1-node3 Status Will go NotReady...takes about 1 minute.
kubectl get nodes --watch


#But there's a Pod still on c1-node3...wut? 
#Kubernetes is protecting against transient issues. Assumes the Pod is still running...
kubectl get pods -o wide


#Start up c1-node3, break out of watch when Node reports Ready, takes about 15 seconds
kubectl get nodes --watch


#That Pod that was on c1-node3 goes to Status Error then it will be restarted on that Node.
kubectl get pods -o wide 


#It will start the container back up on the Node c1-node3...see Restarts is now 1, takes about 10 seconds
#The pod didn't get rescheduled, it's still there, the container restart policy restarts the container which 
#starts at 10 seconds and defaults to Always. We covered this in detail in my course "Managing the Kuberentes API Server and Pods"
kubectl get pods -o wide --watch

#Shutdown a node again...
ssh c1-node3
sudo shutdown -h now


#Let's set a watch and wait...about 5 minutes and see what kubernetes will do.
#Because of the --pod-eviction-timeout duration setting on the kube-controller-manager, this pod will get killed after 5 minutes.
kubectl get pods --watch


#Orphaned Pod goes Terminating and a new Pod will be deployed in the cluster.
#If the Node returns the Pod will be deleted, if the Node does not, we'll have to delete it
kubectl get pods -o wide


#And go start c1-node3 back up again and see if those pods get deleted :)


#let's clean up...
kubectl delete deployment hello-world
kubectl delete service hello-world