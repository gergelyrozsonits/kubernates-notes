# High level overview
Runs over the containerized application and and make them avaiable when and were you wanted.
Kubernates == cluster of physical or virtual machines, makes possible to deploy containerized applications to the claster, but hide the resources behind the cluster.
Kubernates automates the distribution af these cont. apps based on the deployments.
Every claster has a master and one or more workers called nodes (runs the applications).
The master coordinates all activities in your cluster:
- scheduling applications
- maintaining applications' desired state
- scaling applications
- and rolling out new updates
Nodes are workers, each has a Kubelet which is an agent for communicate with the master + runs the containerized applications.

# Installation (with native docker)
You can download the binary from: https://github.com/kubernetes/minikube. Documentation is also available here.

According to https://github.com/kubernetes/minikube/blob/master/docs/drivers.md#hyperV-driver in case of Hyper-V:
> Please note: By default it will try to use VirtualBox so in case of native docker (with HyperV) please create a virtual switch according to the following description: https://docs.docker.com/machine/drivers/hyper-v/ and start minikube with the following command:
```./minikube start --vm-driver="hyperv" --hyperv-virtual-switch="<virtual switch name>"```

In this case as a first step please consider to setup a virtual network switch by following the following steps:
https://docs.docker.com/machine/drivers/hyper-v/#4-create-the-nodes-with-docker-machine-and-the-microsoft-hyper-v-driver

Please note: only the virtual network switch has to be initiated, the VM will be created by minikube

If for some reasone the installation process fails or something has been miscunfigured, then the vm can be removed (after it has been stopped) from Hyper-V Manager. If you need a fresh install then the base configs has to be removed from ```~/.docker/machine/machines``` and ```~/.minikube/machines directory```. Minikube also soports a ```minikube delete``` command.
*During the installation please dont forget to use a terminal in Administrator mode.*

To play with Kubernates, Kubernetes CLI (kubectl) needs to be downloaded as well (and it has to be available on the path as well). According to https://kubernetes.io/docs/tasks/tools/install-kubectl/ it can be downloaded with ```curl -LO https://storage.googleapis.com/kubernetes-release/release/v1.8.0/bin/windows/amd64/kubectl.exe```

> Also please note: i had issues with starting minikube any other shell than ``cmd``` or ```Power Shell```, also the terminal must run in Administrator mode.

If any problem occures during installation the verbose logs can be accessed by starting Minikube with the following command:

```
./minikube start --vm-driver="hyperv" --hyperv-virtual-switch="minikube" -v 9999
```

# First steps

If both minkube and kubectl are available and the minikube cluster is running you can verify your system with:

```
kubectl get nodes
```

As a result all the available nods has be listed (in case of minikube only one should be seen of course), the result should looks like this:

```
PS C:\minikube-windows> ./kubectl get nodes
NAME       STATUS    ROLES     AGE       VERSION
minikube   Ready     <none>    4d        v1.8.0
```

Now as the nodes are ready to use, lets deploy a container to it.

# Deployment
To run an app in Kubernates the following command can be used:

```
./kubectl.exe run <deployment name> --image=<docker image to be done> --port=8080
```

To test your local minikube claster one can deploy a test app with the following command:

```
./kubectl run kubernetes-bootcamp --image=docker.io/jocatalin/kubernetes-bootcamp:v1 --port=8080
```

Whith the above described way the given image will be deployed to the cluster. The deployment will be orchestrated by the master node of the claster, it will tries to find a suitable node where the instance can be deployed, schedule the deployment of the app.
Later on one can list the deployments with the following command:

```
./kubectl get deployments
```

Even if the deployment has been created after the execution of this command further deployment issues could occure. The troubleshooting can be challenging in this case. First of the pod list has to be accessed to check which pod is assigned to a certain deployment (the command can be fine tuned if needed: https://kubernetes.io/docs/tasks/access-application-cluster/list-all-running-container-images/):

```
./kubectl get pods
```

Now with having the pod name one can access the status of the deployed app with (the ```POD_NAME``` is an optional parameter here, if not porvided then all pod statuses will be listed):

```
./kubectl described pods <POD_NAME>
```

The result should look like this:

```
PS C:\minikube-windows> ./kubectl get pods
NAME                                   READY     STATUS    RESTARTS   AGE
kubernetes-bootcamp-6db74b9f76-rqzq2   1/1       Running   0          1h
```

Also the logs can be accessed with having the pod name with the following command:

```
./kubectl logs kubernetes-bootcamp-6db74b9f76-rqzq2
```

> Please note: The same infomration can be retrieved by using the admin page as well.

# Accessing the apps

Even if the deployment will be scheduled by Kubernates automatically, by default all pods are private in Kubernates therefore even if the pods can be accessed from inside the claster it will not be exposed by Kubernates from outside of the claster. To check wether its working or not a proxy has to be created with the following command:

```
./kubectl proxy
```

The proxy will be running and makes the inner, private network available. Via proxy Kubernates admin page becomes also available on ```http://localhost:8001/ui```. Here beside almost all available options one can access the deployments or pod informations in a fancy way.

To access the dummy test app (kubernetes-bootcamp) via proxy first the Pod name is needed, so please check the pod name with in the earlier described way:

```
./kubectl get pods
```

Now, with having the pod name the deployed container can be accessed via the proxy with:

```
http://localhost:8001/api/v1/proxy/namespaces/default/pods/<NAME>/
```

Commands:
Delete deployment ```./kubectl delete deployment <NAME>```