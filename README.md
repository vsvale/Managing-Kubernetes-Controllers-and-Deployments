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
kubectl apply -f deployment.yaml