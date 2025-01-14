# Basic Deployment

In this exercise, you will deploy an nginx web server to a Kubernetes Cluster, and access the default nginx _welcome page_ in your browser.

It is not expected that you understand everything going on, yet.

## Learning Goals

- Use the Kubernetes command line interface (CLI) `kubectl`.
- Run and access an application in Kubernetes.

## Introduction

In Kubernetes, we run containers, but we don't manage containers directly. Containers are instead placed inside `pods`.
A `pod` _contains_ containers.

Pods in turn are managed by controllers, the most common one is called a `deployment`.

In this exercise, you will create a `deployment`, which will create a `pod`, in which your container will be running.

### Interacting with Kubernetes using kubectl

We will be interacting with Kubernetes using the command line.

The Kubernetes CLI is called `kubectl`, and allows us to manage our applications and Kubernetes itself.

To use it, type `kubectl <subcommand> <options>` in a terminal.

## Exercise

### Overview

- Run an application in a `pod` using the `kubectl create` command
- Make the application accessible from the Internet

### Step by step instructions

<details>
<summary>
Step by step:
</summary>

## Run application using kubectl create command

We will use the [nginx](https://nginx.org/en/) webserver as an example of an application you might want to run in Kubernetes.

Here is the command to do it:

```
kubectl create deployment nginx --image=nginx:latest
```

Expected output:

```
deployment.apps/nginx created
```

We can ask Kubernetes about what resources it has, such as our `pod`.

We do this using the `kubectl get <kind>` command, in this case the `<kind>` will be `pod(s)`.

Verify that your pod was created and is running using kubectl:

```
kubectl get pods
```

Expected output:

```
NAME                     READY   STATUS    RESTARTS   AGE
nginx-6d666844f6-tjvk5   1/1     Running   0          15s
```

Awesome! Nginx is running.

## Make the application accessible from the internet

We are getting a little ahead of our exercises here, but to illustrate that we actually have
a functioning webserver running in our pod, let's try exposing it to the internet and access it from a browser!

First use the following command to create a `service` for your `deployment`:

> :bulb: A `service` is a networking abstraction that enables a lot of the neat networking features of Kubernetes.
> We will cover `services` in detail in a later exercise, so just go with it for now :-)

```
kubectl expose deployment nginx --port 80 --type NodePort
```

Expected output:

```
service/nginx exposed
```

Get the `service` called `nginx` and note down the NodePort, by finding the `PORT(S)` column and noting the number on the right side of the colon `:`

```
kubectl get service nginx
```

Expected output:

```
NAME        TYPE       CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
nginx       NodePort   10.96.223.218   <none>        80:32458/TCP   12s
```

In this example, Kubernetes has chosen port `32458`, you will most likely get a different number.

Finally, look up the IP address of a node in the cluster with:

```
kubectl get nodes -o wide
```

> :bulb: The `-o wide` flag makes the output more verbose, i.e. to include the IPs

Expected output:

```
NAME    STATUS   . . . INTERNAL-IP  EXTERNAL-IP     . . .
node1   Ready    . . . 10.123.0.8   35.240.20.246   . . .
node2   Ready    . . . 10.123.0.7   35.205.245.42   . . .
```

In the example your external IPs are either `35.240.20.246` or `35.205.245.42`.

Since your `service` is of type `NodePort` it will be exposed on _all_ of the nodes. The service will be exposed on the port with the number you noted down above.

Choose one of the `EXTERNAL-IP`'s, and point your web browser to the address: `<EXTERNAL-IP>:<PORT>`.

In this example, the address could be `35.240.20.246:32458`, or `35.205.245.42:32458`.

You should see the default nginx webpage in your browser.

</details>

### More details

This section has some optional extra details.

<details>
<summary>More details about pods</summary>

A `pod` (_not container_) is the smallest building-block/worker-unit in Kubernetes,
it has a specification of one or more containers and exists for the duration of the containers;
if all the containers stop or terminate, the Pod is stopped.

</details>

<details>
<summary>More details about deployments</summary>

Usually a `pod` will be part of a `deployment`; a more controlled or _robust_ way of running `pod`s.
A deployment can be configured to automatically delete stopped or exited Pods and start new ones,
as well as run a number of identical Pods e.g. to provide high-availability.

</details>

### Clean up

Delete the resources you have created using `kubectl delete <kind> <name>`

```
kubectl delete deployment nginx
kubectl delete service nginx
```
