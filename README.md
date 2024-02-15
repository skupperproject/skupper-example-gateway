# Skupper Hello World using the gateway

[![main](https://github.com/ssorj/skupper-example-gateway/actions/workflows/main.yaml/badge.svg)](https://github.com/ssorj/skupper-example-gateway/actions/workflows/main.yaml)

#### Connect services running as system processes

This example is part of a [suite of examples][examples] showing the
different ways you can use [Skupper][website] to connect services
across cloud providers, data centers, and edge sites.

[website]: https://skupper.io/
[examples]: https://skupper.io/examples/index.html

#### Contents

* [Overview](#overview)
* [Prerequisites](#prerequisites)
* [Step 1: Install the Skupper command-line tool](#step-1-install-the-skupper-command-line-tool)
* [Step 2: Set up your Kubernetes namespace](#step-2-set-up-your-kubernetes-namespace)
* [Step 3: Create your Kubernetes site](#step-3-create-your-kubernetes-site)
* [Step 4: Set up the Skupper gateway](#step-4-set-up-the-skupper-gateway)
* [Step 5: Deploy the frontend and backend](#step-5-deploy-the-frontend-and-backend)
* [Step 6: Expose the backend](#step-6-expose-the-backend)
* [Step 7: Access the frontend](#step-7-access-the-frontend)
* [Cleaning up](#cleaning-up)
* [Summary](#summary)
* [Next steps](#next-steps)
* [About this example](#about-this-example)

## Overview

This example is a basic multi-service HTTP application deployed
across a Kubernetes cluster and a bare-metal host or VM.

It contains two services:

* A backend service that exposes an `/api/hello` endpoint.  It
  returns greetings of the form `Hi, <your-name>.  I am <my-name>
  (<hostname>)`.

* A frontend service that sends greetings to the backend and
  fetches new greetings in response.

The frontend runs on Kubernetes and the backend runs on your local
machine.  Skupper enables the frontend to connect to the backend
using a dedicated service network.

## Prerequisites

* A working installation of Docker ([installation
  guide][install-docker]) or Podman ([installation
  guide][install-podman])

* The `kubectl` command-line tool, version 1.15 or later
  ([installation guide][install-kubectl])

* Access to a Kubernetes cluster, from [any provider you
  choose][kube-providers]

* The `starlette` and `uvicorn` Python modules.  This is required to
  run the backend service locally.  To install the modules, run `pip
  install starlette uvicorn`.

[install-docker]: https://docs.docker.com/engine/install/
[install-podman]: https://podman.io/getting-started/installation
[install-kubectl]: https://kubernetes.io/docs/tasks/tools/install-kubectl/
[kube-providers]: https://skupper.io/start/kubernetes.html

## Step 1: Install the Skupper command-line tool

This example uses the Skupper command-line tool to deploy Skupper.
You need to install the `skupper` command only once for each
development environment.

On Linux or Mac, you can use the install script (inspect it
[here][install-script]) to download and extract the command:

~~~ shell
curl https://skupper.io/install.sh | sh
~~~

The script installs the command under your home directory.  It
prompts you to add the command to your path if necessary.

For Windows and other installation options, see [Installing
Skupper][install-docs].

[install-script]: https://github.com/skupperproject/skupper-website/blob/main/input/install.sh
[install-docs]: https://skupper.io/install/

## Step 2: Set up your Kubernetes namespace

Open a new terminal window and log in to your cluster.  Then
create the namespace you wish to use and set the namespace on your
current context.

**Note:** The login procedure varies by provider.  See the
documentation for your chosen providers:

* [Minikube](https://skupper.io/start/minikube.html#cluster-access)
* [Amazon Elastic Kubernetes Service (EKS)](https://skupper.io/start/eks.html#cluster-access)
* [Azure Kubernetes Service (AKS)](https://skupper.io/start/aks.html#cluster-access)
* [Google Kubernetes Engine (GKE)](https://skupper.io/start/gke.html#cluster-access)
* [IBM Kubernetes Service](https://skupper.io/start/ibmks.html#cluster-access)
* [OpenShift](https://skupper.io/start/openshift.html#cluster-access)

_**Kubernetes:**_

~~~ shell
# Enter your provider-specific login command
kubectl create namespace hello-world
kubectl config set-context --current --namespace hello-world
~~~

## Step 3: Create your Kubernetes site

A Skupper _site_ is a location where components of your
application are running.  Sites are linked together to form a
Skupper network for your application.

In Kubernetes, use `skupper init` to create a site.  This
deploys the Skupper router and controller.  Then use `skupper
status` to see the outcome.

**Note:** If you are using Minikube, you need to [start minikube
tunnel][minikube-tunnel] before you run `skupper init`.

[minikube-tunnel]: https://skupper.io/start/minikube.html#running-minikube-tunnel

_**Kubernetes:**_

~~~ shell
skupper init
~~~

_Sample output:_

~~~ console
$ skupper init
Waiting for LoadBalancer IP or hostname...
Waiting for status...
Skupper is now installed in namespace 'hello-world'.  Use 'skupper status' to get more information.
~~~

As you move through the steps below, you can use `skupper status` at
any time to check your progress.

## Step 4: Set up the Skupper gateway

The `skupper gateway init` command starts a Skupper router on
your local system and links it to the Skupper router in the
current Kubernetes namespace.

_**Kubernetes:**_

~~~ shell
skupper gateway init --type docker
~~~

_Sample output:_

~~~ console
$ skupper gateway init --type docker
Skupper gateway: 'fancypants-jross'. Use 'skupper gateway status' to get more information.
~~~

The `--type docker` option runs the router as a Docker
container.  You can also run it as a Podman container (`--type
podman`) or as a systemd service (`--type service`).

## Step 5: Deploy the frontend and backend

For this example, we are running the frontend on Kubernetes and
the backend as a local system process.

Use `kubectl create deployment` to deploy the frontend service
in your Kubernetes namespace.

Change to the `backend` directory and use `python
python/main.py` to start the backend process.  You can run this in a
different terminal if you prefer.

_**Kubernetes:**_

~~~ shell
kubectl create deployment frontend --image quay.io/skupper/hello-world-frontend
(cd backend && python python/main.py --host localhost --port 8081) &
~~~

_Sample output:_

~~~ console
$ kubectl create deployment frontend --image quay.io/skupper/hello-world-frontend
deployment.apps/frontend created

$ (cd backend && python python/main.py --host localhost --port 8081) &
INFO:     Started server process [208334]
INFO:     Waiting for application startup.
INFO:     Application startup complete.
INFO:     Uvicorn running on http://localhost:8081 (Press CTRL+C to quit)
~~~

## Step 6: Expose the backend

Use `skupper service create` to define a Skupper service called
`backend`.  Then use `skupper gateway bind` to attach your
running backend process as a target for the service.

_**Kubernetes:**_

~~~ shell
skupper service create backend 8080
skupper gateway bind backend localhost 8081
~~~

_Sample output:_

~~~ console
$ skupper gateway bind backend localhost 8081
2022/09/08 07:07:00 CREATE io.skupper.router.tcpConnector fancypants-jross-egress-backend:8080 map[address:backend:8080 host:localhost name:fancypants-jross-egress-backend:8080 port:8081 siteId:d187db66-cbda-43fe-ac3b-4be22bbad1c9]
~~~

## Step 7: Access the frontend

In order to use and test the application, we need external access
to the frontend.

Use `kubectl expose` with `--type LoadBalancer` to open network
access to the frontend service.

Once the frontend is exposed, use `kubectl get service/frontend`
to look up the external IP of the frontend service.  If the
external IP is `<pending>`, try again after a moment.

Once you have the external IP, use `curl` or a similar tool to
request the `/api/health` endpoint at that address.

**Note:** The `<external-ip>` field in the following commands is a
placeholder.  The actual value is an IP address.

_**Kubernetes:**_

~~~ shell
kubectl expose deployment/frontend --port 8080 --type LoadBalancer
kubectl get service/frontend
curl http://<external-ip>:8080/api/health
~~~

_Sample output:_

~~~ console
$ kubectl expose deployment/frontend --port 8080 --type LoadBalancer
service/frontend exposed

$ kubectl get service/frontend
NAME       TYPE           CLUSTER-IP      EXTERNAL-IP     PORT(S)          AGE
frontend   LoadBalancer   10.103.232.28   <external-ip>   8080:30407/TCP   15s

$ curl http://<external-ip>:8080/api/health
OK
~~~

If everything is in order, you can now access the web interface by
navigating to `http://<external-ip>:8080/` in your browser.

## Cleaning up

To remove Skupper and the other resources from this exercise, use
the following commands.

_**Kubernetes:**_

~~~ shell
kill $(ps -ef | grep 'python python/main\.py' | awk '{print $2}') 2> /dev/null
skupper gateway delete
skupper delete
kubectl delete service/frontend
kubectl delete deployment/frontend
~~~

## Next steps

Check out the other [examples][examples] on the Skupper website.

## About this example

This example was produced using [Skewer][skewer], a library for
documenting and testing Skupper examples.

[skewer]: https://github.com/skupperproject/skewer

Skewer provides utility functions for generating the README and
running the example steps.  Use the `./plano` command in the project
root to see what is available.

To quickly stand up the example using Minikube, try the `./plano demo`
command.
