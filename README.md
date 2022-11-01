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