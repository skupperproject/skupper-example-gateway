# Skupper local server

[![main](https://github.com/ssorj/skupper-example-local-server/actions/workflows/main.yaml/badge.svg)](https://github.com/ssorj/skupper-example-local-server/actions/workflows/main.yaml)

#### Accept connections to a local process from a remote service

This example is part of a [suite of examples][examples] showing the
different ways you can use [Skupper][website] to connect services
across cloud providers, data centers, and edge sites.

[website]: https://skupper.io/
[examples]: https://skupper.io/examples/index.html

#### Contents

* [Overview](#overview)
* [Prerequisites](#prerequisites)
* [Step 1: Install the Skupper command-line tool](#step-1-install-the-skupper-command-line-tool)
* [Step 2: Access your Kubernetes cluster](#step-2-access-your-kubernetes-cluster)
* [Step 3: Set up your Kubernetes namespace](#step-3-set-up-your-kubernetes-namespace)
* [Step 4: Install Skupper in your Kubernetes namespace](#step-4-install-skupper-in-your-kubernetes-namespace)
* [Step 5: Install the Skupper gateway](#step-5-install-the-skupper-gateway)
* [Step 6: Deploy the frontend and backend services](#step-6-deploy-the-frontend-and-backend-services)
* [Step 7: Expose the backend](#step-7-expose-the-backend)
* [Step 8: Expose the frontend](#step-8-expose-the-frontend)
* [Step 9: Sleep](#step-9-sleep)
* [Cleaning up](#cleaning-up)
* [About this example](#about-this-example)

## Overview

This example is a multi-service HTTP application deployed across a
Kubernetes cluster and a bare-metal host or VM.

It contains two services:

* A backend service that exposes an `/api/hello` endpoint.  It
  returns greetings of the form `Hi, <your-name>.  I am <my-name>
  (<pod-name>)`.

* A frontend service that sends greetings to the backend and
  fetches new greetings in response.

The frontend runs on Kubernetes and the backend runs on your local
machine.  Skupper enables the frontend to connect to the backend
using a dedicated service network.

<!-- <img src="images/entities.svg" width="640"/> -->

## Prerequisites

* A working installation of Docker ([installation
  guide][install-docker]) or Podman ([installation
  guide][install-podman])

* The `kubectl` command-line tool, version 1.15 or later
  ([installation guide][install-kubectl])

* Access to a Kubernetes cluster, from [any provider you
  choose][kube-providers]

* XXX `pip install starlette uvicorn`

[install-docker]: https://docs.docker.com/engine/install/
[install-podman]: https://podman.io/getting-started/installation
[install-kubectl]: https://kubernetes.io/docs/tasks/tools/install-kubectl/
[kube-providers]: https://skupper.io/start/kubernetes.html#providers

## Step 1: Install the Skupper command-line tool

The `skupper` command-line tool is the primary entrypoint for
installing and configuring Skupper.  You need to install the
`skupper` command only once for each development environment.

On Linux or Mac, you can use the install script (inspect it
[here][install-script]) to download and extract the command:

~~~ shell
curl https://skupper.io/install.sh | sh
~~~

The script installs the command under your home directory.  It
prompts you to add the command to your path if necessary.

For Windows and other installation options, see [Installing
Skupper][install-docs].

[install-script]: https://github.com/skupperproject/skupper-website/blob/main/docs/install.sh
[install-docs]: https://skupper.io/install/index.html

## Step 2: Access your Kubernetes cluster

The procedure for accessing a Kubernetes cluster varies by
provider. [Find the instructions for your chosen
provider][kube-providers] and use them to authenticate and
configure access.

[kube-providers]: https://skupper.io/start/kubernetes.html#providers

## Step 3: Set up your Kubernetes namespace

Use `kubectl create namespace` to create the namespace you wish
to use (or use an existing namespace).  Use `kubectl config
set-context` to set the current namespace for your session.

_**Console for hello-world:**_

~~~ shell
kubectl create namespace hello-world
kubectl config set-context --current --namespace hello-world
~~~

_Sample output:_

~~~ console
$ kubectl create namespace hello-world
namespace/hello-world created

$ kubectl config set-context --current --namespace hello-world
Context "minikube" modified.
~~~

## Step 4: Install Skupper in your Kubernetes namespace

The `skupper init` command installs the Skupper router and service
controller in the current namespace.

**Note:** If you are using Minikube, [you need to start `minikube
tunnel`][minikube-tunnel] before you install Skupper.

[minikube-tunnel]: https://skupper.io/start/minikube.html#running-minikube-tunnel

_**Console for hello-world:**_

~~~ shell
skupper init
~~~

_Sample output:_

~~~ console
$ skupper init
Waiting for LoadBalancer IP or hostname...
Skupper is now installed in namespace 'hello-world'.  Use 'skupper status' to get more information.
~~~

## Step 5: Install the Skupper gateway

_**Console for hello-world:**_

~~~ shell
skupper gateway init --type docker
~~~

## Step 6: Deploy the frontend and backend services

_**Console for hello-world:**_

~~~ shell
kubectl create deployment frontend --image quay.io/skupper/hello-world-frontend
(cd backend && python python/main.py --host localhost --port 8081) &
~~~

_Sample output:_

~~~ console
$ kubectl create deployment frontend --image quay.io/skupper/hello-world-frontend
deployment.apps/frontend created
~~~

## Step 7: Expose the backend

_**Console for hello-world:**_

~~~ shell
skupper service create backend 8080
skupper gateway bind backend localhost 8081
~~~

## Step 8: Expose the frontend

_**Console for hello-world:**_

~~~ shell
skupper service create frontend 8080
skupper service bind frontend deployment/frontend
skupper gateway forward frontend 8080
~~~

## Step 9: Sleep

_**Console for hello-world:**_

~~~ shell
sleep 86400
~~~

## Cleaning up

To remove Skupper and the other resources from this exercise, use
the following commands.

_**Console for hello-world:**_

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

Skewer provides some utilities for generating the README and running
the example steps.  Use the `./plano` command in the project root to
see what is available.

To quickly stand up the example using Minikube, try the `./plano demo`
command.
