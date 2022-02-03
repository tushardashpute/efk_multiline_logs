# efk_multiline_logs
EFK Multiline Logs


The Log Collection In K8S
There are different ways to collect logs from a Kubernetes cluster. You can:

1.use sidecar containers
2.have the application push the logs into the logging service directly
3.deploy a log collector agent to each cluster node to collect the logs

I am using 3rd option, having a dedicated agent running on the cluster nodes.

So the basic idea in this case is to utilize the Docker engine under Kubernetes. When you are logging from a container to standard out/error, Docker is simply going to store those logs on the filesystem in specific folders.

Since a pod consists of Docker containers, those containers are going to be scheduled on a concrete K8S node, hence its logs are going to be stored on the node’s filesystem.

When you have an agent application running on every cluster-node, you can easily set up some watching for those log files and when something new is coming up, like a new log line, you can simply collect it.

Fluentd is one agent that can work this way. The only thing left is to figure out a way to deploy the agent to every K8S node. Luckily, Kubernetes provides a feature like this, it’s called DaemonSet. When you set up a DaemonSet – which is very similar to a normal Deployment -, Kubernetes makes sure that an instance is going to be deployed to every (selectively some) cluster node, so we’re going to use Fluentd with a DaemonSet.

**EFK Stack**
EFK – ElasticSearch, Fluentd, Kibana – stack. Kibana is going to be the visualization tool for the logs, ElasticSearch will be the backbone of Kibana to store the logs. And Fluentd is something we discussed already.

Everything in the stack is self-contained so a very simple apply is enough

**Running the example**
Build the docker image and the application

    $ docker build -t fluentd-multiline-java:latest .
    
Side note, if you are building for a minikube environment, you can issue the eval $(minikube docker-env) command to build the image for the K8S registry.

The next thing is to deploy the application to Kubernetes

    $ kubectl apply -f k8s/fluentd-multiline-java-deployment.yaml
    
Then deploy the EFK stack

    $ kubectl apply -f k8s/efk-stack.yaml
In case of minikube, set up tunneling since Kibana is exposed via a LoadBalancer.

Then access Kibana at the external IP of the LoadBalancer and the 5601 port.
