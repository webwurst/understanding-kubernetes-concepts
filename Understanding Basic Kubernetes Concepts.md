Author: Puja Abbassi

Date: 2018-02-23

# Understanding Basic Kubernetes Concepts

This guide will help you understand the basic Kubernetes concepts. It consists of 5 chapters. In the first one I [explain the concepts of Pods, Labels, and Replica Sets](#pods-labels-replicas). In the second chapter [we talk about Deployments](#deployments). The third chapter [explains the Services concept](#services) and in the forth we look at [Secrets and ConfigMaps](#secrets-configmaps). In the final chapter we talk about [Daemon Sets and Jobs](#daemonsets-jobs).

There's lots of introductions and tutorials out there that help you get started with Kubernetes. However, most of them focus on showing how the commands work and how to get stuff running. Don't get me wrong, trying out the commands is important, but to really get into working productively with Kubernetes you need to go deeper and understand the concepts and their functionalities so you can actually use them in the way they were intended. This is especially hard coming from vanilla Docker, as the concepts of Docker don't directly translate to Kubernetes - at least if you want to use Kubernetes the right way.

## An Introduction To Pods, Labels, and Replicas {#pods-labels-replicas}

Let's start with the most basic concepts: pods, labels & selectors, and replica sets. I won't include any usage instructions, but focus solely on explaining the concepts, their functionality, and what you should use them for. These are only the most basic concepts and you need some more to get a full overview of Kubernetes. But rest assured I will explore further concepts in following chapters.

### Pods

"Pods are the smallest deployable units of computing that can be created and managed in Kubernetes." say the [official Kubernetes docs for pods](https://kubernetes.io/docs/concepts/workloads/pods/pod/). This sometimes leads to the confusion that pods are single containers, as that's what people are used to from Docker. While pods can contain one single container, they are not limited to one and can contain as many containers as needed.

<a data-flickr-embed="true"  href="https://www.flickr.com/photos/area53/15873184540/in/photolist-qbEdKw-72obGn-5spZFB-qAsi6x-8YJB9E-7pfbe6-fwnWQ5-xZ65XP-Bwmao-3ig83V-yNAGyD-Bwj3f-4dDmYB-7pj48E-gjXZYk-Bwm3n-5QNRqM-61H6EJ-hYgL8-4Hpip3-7kDxNp-5fVbZ4-6iCdvt-7KV1Tx-7pfbPt-7pj4oh-BwjrP-5Zrvfe-dDV5yM-bA7EP-BwiQg-4DLrcz-34u5KX-7pfbYz-qNMpSZ-6LdFU6-g9HwW3-4D33zo-7mUYHJ-pb3VkZ-iTTm-czDrdy-4egJbr-Brtm6-Bwrm2-4Tp16a-5tLUNi-6arDDU-6244vi-8KUh4w" title="Interesting Pod"><img src="https://c5.staticflickr.com/9/8604/15873184540_f107b5f651_b.jpg" width="1024" height="768" alt="Interesting Pod"></a><script async src="//embedr.flickr.com/assets/client-code.js" charset="utf-8"></script>

What makes these containers a pod, is that all containers in a pod run (nearly) as if they would have been running on a single host in pre-container world. Thus, they share a set of Linux namespaces and do not run isolated from each other. This results in them sharing an IP address and port space, and being able to find each other over `localhost` or communicate over the IPC namespace. Furthermore, all containers in a pod have access to shared volumes, that is they can mount and work on the same volumes if needed.

In order to gain all this functionality a pod is a single deployable unit. Each single instance or as we call it replica of a pod (with all its containers) is always scheduled together.

Now, for the typical Docker user this concept is quite new. For some it might sound like going back from the isolated "one process per container" to "deploying your whole LAMP stack together". However, this is not the intended use case for pods. The main motivation for the pod concept is supporting co-located, co-managed helper containers next to the application container. These include things like: logging or monitoring agents, backup tooling, data change watchers, event publishers, proxies, etc.. If you are not sure what to use pods for in the beginning, you can for now use them with single containers like you might be used to from Docker.

The pod is the most basic concept in Kubernetes. By itself, it is ephemeral and won't be rescheduled to a new node once it dies. If we want to keep one or more instances of a pod alive we need another concept: replica sets. But before that we need to understand what labels and selectors are.

### Labels & Selectors

Labels are key/value pairs that can be attached to objects, such as pods, but also any other object in Kubernetes, even nodes. They should be used to specify identifying attributes of objects that are meaningful and relevant to users. You can attach labels to objects at creation time and modify them or add new ones later. 

Labels can be used to organize and select subsets of objects. They are often used for example to identify releases (beta, stable), environments (dev, prod), or tiers (frontend, backend), but are flexible to support many other cases, too. They help get some order into the multi-dimensionality of modern deployment pipelines.

Labels are a key concept of Kubernetes as they are used together with selectors to manage objects or groups thereof. This is done without the actual need for any specific information about the object(s), not even the number of objects that exist.

Especially the fact that the number of objects is unkown should be kept in mind when working with label selectors. In general, you should expect many objects to carry the same label(s).

Currently, there are two types of selectors in Kubernetes: equality-based and set-based. The former use key value pairs to filter based on basic equality (or inequality). The latter are a bit more powerful and allow filtering keys according to a set of values.

Using label selectors a client or user can identify and subsequently manage a group of objects. This is the core grouping primitive of Kubernetes and used in many places. One example of its use is working with replica sets.

### Replica Sets (and Replication Controllers)

Replica sets, for those who have read a bit about Kubernetes before, are the next-generation of replication controllers. Currently, the main difference being that replica sets support the more advanced set-based selectors and thus are more flexible than replication controllers. However, the gist of the following explanation fits to both.

As mentioned above a pod by itself is ephemeral and won't be rescheduled if the node it is running on goes down. This is where the replica set comes in and ensures that a specific number of pod instances (or _replicas_) are running at any given time. Thus, if you want your pod to stay alive you make sure you have an according replica set specifying at least one replica for that pod. The replica set then takes care of (re)scheduling your instances for you.

As indicated above the replica set ensures a specific number of replicas are running. By modifying the number of replicas in the set's definition you can scale your pods up and down.

A replica set can not only manage a single pod but also a group of different pods selected based on a common label. This enables a replica set to for example scale all pods that together compose the frontend of an application together without having to have identical replica sets for each pod in the frontend.

You can include the definition of the pod directly in the definition of the replica set, so you can manage them together. However, there is a higher level concept called a _deployment_, which manages replica sets. Therefore, you usually won't need to create or manipulate replica set objects directly. It's still important knowing about this concept as without it you won't understand the specific workings of how Kubernetes will help you run and manage your applications. I will explain deployments and the many features they bring with them in a follow-up blog post, for now suffice to say that they will manage your replica sets for you.

### Further Reading

Now that you learned a bit about the basic concepts you can read more about them and their usage in the official Kubernetes docs:

- [Pods](http://kubernetes.io/docs/user-guide/pods/)
- [Labels & Selectors](http://kubernetes.io/docs/user-guide/labels/)
- [Replica Sets](http://kubernetes.io/docs/user-guide/replicasets/)

Try to play around with them and then maybe move on to a [more advanced example](https://github.com/kubernetes/kubernetes/tree/release-1.2/examples/guestbook) and try it out on a [local environement](https://blog.giantswarm.io/getting-started-with-a-local-kubernetes-environment/). Keep watching out for follow-ups to this blog post explaining deployments and more Kubernetes concepts. For feedback and requests just [get in touch](https://twitter.com/puja108).

## Using Deployments To Manage Your Services Declaratively {#deployments}

As you might have heard (or read) me saying before, in the Kubernetes community it is sometimes hard to find the best way to do things. On the service/app deployment side of things, this results in for example people deploying their containers manually with pods and then adding replication controllers on top to keep them alive and scale them and then at some point using `kubectl rolling-update` to do updates. This is at least in parts an imperative form of managing software. It usually involves manual work/commands and can be pretty opaque if you work in a team. 

<a data-flickr-embed="true"  href="https://www.flickr.com/photos/quinnanya/4873252184/in/photolist-8qCG6o-7vETFq-6bTRd7-drbPpz-3eFDqL-2Q7niP-6dgPhU-2ZdLZe-38UFxR-Br3qJV-zow44-2NqJ3K-aR8kPc-aR8jHR-aR8jp2-wy1x8-aR8f5X-6Gogt1-dZsPzj-aR8iLn-aR8eEz-9CehY2-aR8j5D-aR8hi2-aR8hLg-8b1E7c-8Tid8M-aR8k4i-aR8fXF-apkKvD-bVtSzM-aR8iba-aR8kq8-aR8fwx-2nVcip-fopepg-7h7DgM-71n1bz-25PEuq-dZn9dg-dZn9zx-dZn9q2-dZsQA7-dZsQ1d-dZsRaL-9RWZUv-zKvYt-26BQNz-CiGBTr-wxLTz" title="Use a declarative format"><img src="https://c1.staticflickr.com/5/4099/4873252184_5d74c87e90_b.jpg" width="1024" height="683" alt="Use a declarative format"></a><script async src="//embedr.flickr.com/assets/client-code.js" charset="utf-8"></script>

Therefor, the declarative _Deployment_ object was added in Kubernetes 1.2. However, looking around there's very few resources mentioning it and why you should use it. Sure, it is included in the [official docs](https://kubernetes.io/docs/user-guide/deployments/) and there was a nice [blog post](http://blog.kubernetes.io/2016/04/using-deployment-objects-with.html) about it on April 1st (maybe people thought it was a joke), but I don't see many people using it and even less explicitly mentioning it in their blog posts and examples. So I thought of writing a little piece on why I think you should use deployments.

_Sidenote: As of this writing usually system services or addons (e.g. DNS, Dashboard, Metrics) on most Kubernetes installs out there are still deployed with replication controllers, but some [official addons](https://github.com/kubernetes/kubernetes/tree/master/cluster/addons) are already being moved to deployments. Monitoring has done the move already and we merged a PR just a few days ago that moves Dashboard to deployments. On Giant Swarm all of these system services are already running as deployments._ 

### The Declarative vs. The Imperative Way

The main argument for Deployments is a general one between the declarative and the imperative way of deploying and managing software. It applies not only on the application-level, but also on the infrastructure-level (but that's for another post to contemplate). _Note: If you are already convinced that declarative management is the way to go you can skip this chapter and jump directly into the Kubernetes Deployments concept further below._

#### The Imperative Way

The imperative way is usually the one you use to try out things and get to a working system. You manually tweak it and at some point it is to your liking. However, if you keep on using this imperative style for deploying and managing your software you will encounter several problems (even if you automate the steps).

- If you want to know what has been last deployed, you can only resort to checking the currently deployed version, which is not necessarily what was planned to be deployed.
- If you change something manually, someone else might overwrite your changes or they might get reverted. For example because you only started new pods manually and didn't change the replication controller.
- Keeping track of how a service has developed over time is hard and needs to be documented somewhere separately. This is especially important when working in teams.
- You need several commands to get one service to the desired state. Even if these steps get automated, they might lead to different outcomes when run in different environments.

#### The Declarative Way

The declarative way on the other hand, is what you should come up with once you go into actually deploying and managing your software in production or integrating with continuous delivery pipelines. You might have tried out stuff the imperative way before, but once you know how it should look like, you sit down and "make it official" by writing it into a declarative definition. This avoids the above-mentioned problems and even brings some added benefits.

- It makes you think and plan how you want things to look once they are running (plan the state).
- It describes the way things should look like when running and not the steps that are needed to get there (define the desired state not the process).
- Changes can be easily documented by keeping track of (or versioning) the declarative definition, which makes work and communication in teams easier (but is also a good/clean approach for single devs).

### Deployments in Kubernetes

As mentioned above the deployments resource is pretty new and still not widely-used. However, for now it is the best primitive we have for deploying and managing our software in Kubernetes, so it is important to understand what it does and what you can use it for.

Before deployments, there were replication controllers, which managed pods and ensured a certain number of them were running. Now with deployments we move to replica sets, which are basically the next-generation of replication controllers. Only this time we don't manage them, but they get managed by the deployments we define. Thus, the chain is like following: Deployment -> Replica Set -> Pod(s). And we only have to take care of the first.

Additional to what replication controllers (or [replica sets](https://blog.giantswarm.io/understanding-basic-kubernetes-concepts-i-introduction-to-pods-labels-replicas/)) offer, deployments give you declarative control over the update strategy used for the deployment. This replaces the old `kubectl rolling-update` way of updating, but offers the same flexibility in terms of defining `maxSurge` and `maxUnavailable`, i.e. how many additional and how many unavailable pods are allowed. Defining this in a deployment enables you to "spec once use many times", which helps even more when working in teams or managing a multitude of deployments.

Deployments manage your updates for you. They even go as far as to check whether or not a new revision that is being rolled out is working and stop the rollout in case it is not.

You can additionally define a wait time (`minReadySeconds`) that a pod needs to be ready without any of its containers crashing before a pod is considered available, which again helps against "bad updates" and gives certain containers a bit more time to get ready for traffic.

You can further use the update/revision functionality of deployments to concurrently deploy multiple revisions of a deployment. This enables blue/green deployments or canary release strategies.

Furthermore, deployments keep a history of their revisions, which is used in rollback situations, as well as an event log, which you can use to audit releases and changes to your deployment.

### Getting Started With Deployments

Reading about all the cool stuff you can do with deployments, you might want to check it out yourself.

As deployments (and their replica sets) are a kind of successor to replication controllers, it is quite easy to change replication controllers that you already have to deployments. You only have to change:

```yaml
apiVersion: v1
kind: ReplicationController
```

to

```yaml
apiVersion: extensions/v1beta1
kind: Deployment
```

and add a line containing `matchLabels:` after the `selector:` line (before the actual selectors, do not forget to fix indentation), because the deployments resource supports [set-based label requirements](https://kubernetes.io/docs/user-guide/labels/#resources-that-support-set-based-requirements).

This should give you a workable deployment. As deployments feature more functionality than their predecessor, there are additional fields that you have not defined, yet. Luckily, these fields get filled automatically on creation with defaults. 

However, as we learned above this might result in different outcomes when deployed to different clusters. Sometimes this might be intended, but if not you could look at what you got when applying that deployment to your cluster with `kubectl edit deployment <deployment name>` and then copy (part of) that back into your manifest.

The right way, however, would be actually learning the correct usage of deployments by reading up on it in the [official documentation on deployments](https://kubernetes.io/docs/user-guide/deployments/) and then writing a manifest that works well for you. (You can still start with "translating" your existing replication controller manifests). The [Kubernetes blog post from April](http://blog.kubernetes.io/2016/04/using-deployment-objects-with.html) is also a good read and shows the way updates and rollbacks work with a nice example.

Have fun exploring deployments and look out for the next basic concepts post soon!

## Services Give You Abstraction {#services}

By now you should have a basic understanding of some of the fundamental primitives of Kubernetes. However, there are some concepts missing, still. One of the very central ones solves, especially when working with microservices, sets an abstraction layer on top of pods (and other services) so you can communicate with them without having to track every single pod as it starts, dies, and gets rescheduled. It is a basic building block for service discovery in Kubernetes.

<a data-flickr-embed="true"  href="https://www.flickr.com/photos/jeremybrooks/4164721895/in/photolist-7m2hHV-aajSs-jsXpJ-j4Ku9-j4Pnc-MqCWQ-j4PXG-j4xWN-jsXzN-j4GDj-j4yFX-j4Gjg-j4MDA-j4Rb8-j4EeS-jsYmZ-j4EKr-j4L77-hHZ5eu-j4Ev8-j4F1f-j4NLH-j4Pyw-j4Dth-j4GZg-j4zDg-j4CW2-j4DZk-j4BWF-4QBATu-j4KTF-j4BiB-j4G4Y-j4KGw-j4FeX-j4Nsk-j4Cf5-j4PLE-jsYzU-jsZwC-j4RGh-j4Fuj-RawF-Rax1-rphWGp-rFDBev-rpaurS-px4xot-nxqzNb-6NMqqk" title="Service"><img src="https://c8.staticflickr.com/3/2723/4164721895_2676f3891f_b.jpg" width="1024" height="683" alt="Service"></a><script async src="//embedr.flickr.com/assets/client-code.js" charset="utf-8"></script>

### Enter Services

As mentioned in the [first blog post](https://blog.giantswarm.io/understanding-basic-kubernetes-concepts-i-introduction-to-pods-labels-replicas/) in this series, pods are ephemeral and bound to get killed and started by replica sets (or replication controllers) dynamically. Thus, communicating with a pod or groups thereof calls for a concept that abstracts away the ephemeral pod.

This is where services come in. They are a basic concept that is especially useful when working with microservice architectures as they decouple individual services from each other. For example service A accessing service B doesn't know what kind of and how many pods are actually doing the work in B. The actual pods and even their implementation could change completely.

### How Services Work

Services work by defining a logical set of pods and a policy by which to access them. The selection of pods is based on label selectors (which we talked about in the first blog post). In case you select multiple pods, the service automatically takes care of load balancing and assigns them a single (virtual) service IP (which you can also set manually).

You can use the selector to choose a group of pods and define a `targetPort` on which to access them. Further helping with abstraction, this `targetPort` can also refer to the name of a port, which gives you more freedom to implement the actual pods behind the service. Even the port of each pod could be different as long as they carry the same name.

Additionally, services can abstract away other kinds of backends that are not Kubernetes pods. You can for example abstract away an external database cluster behind a service. This way you can for example use a simple local database for your development environment and a professionally managed database cluster in production without having to change the way that your other services talk to that database service.

You can use the same feature if some of your workloads are running outside of your Kubernetes cluster, i.e. either on another Kubernetes cluster (or [namespace](http://kubernetes.io/docs/user-guide/namespaces/)) or even completely outside of Kubernetes. Latter is especially interesting if you are just starting to migrate workloads.

### Talking To Services

You can discover and talk to services in your cluster either via environment variables or via [DNS](https://github.com/kubernetes/kubernetes/tree/release-1.2/cluster/addons/dns). Latter is a cluster addon, which comes OOTB in most Kubernetes installs (including Giant Swarm).

In case you want to use another type of service discovery and don't want the load balancing and single service IP provided by the service, there's an option to create a so-called "headless" services. You can use this if you are already using a service discovery or want to reduce coupling to the Kubernetes system.

### Get Started

Now that you know a bit more about the concept of services, you should read up on their usage in the [official documentation](http://kubernetes.io/docs/user-guide/services/). Then go ahead and build some [deployments](https://blog.giantswarm.io/understanding-basic-kubernetes-concepts-using-deployments-manage-services-declaratively/) that talk to each other over services.

Tip: If you have a service and a deployment that go together, you should usually start the service first and then the deployment. You can even deploy them with a single command by including both manifests in a single YAML file. You only have to separate them by a line containing `---`. Kubernetes will then create them one after the other.

## Secrets and ConfigMaps {#secrets-configmaps}

The third factor of the [12 factor app methodology](http://12factor.net/) is called [Config](http://12factor.net/config) and describes why you should store configuration in the environment. This is based on the fact that an application's configuration can change between environments (e.g. development, staging, production, etc.) and that you want your applications to be portable. Thus, you should store the config outside of the application itself.

Now with Docker and containers this means we should try to keep configuration out of the container image. This is even more needed when working with sensitive information, such as passwords, keys, auth tokens, because we might not want them to be available in a registry, even if that registry might be private.

In Docker we would use [`--env` or `--env-file`](https://docs.docker.com/engine/reference/commandline/run/#/set-environment-variables-e-env-env-file) for this no matter if we are working with sensitive information or just plain configuration.

<a data-flickr-embed="true"  href="https://www.flickr.com/photos/phunk/238010631/in/photolist-n2Sgz-efvdh-8po2PX-Hv92E-aFa9NU-o62d2N-6DGxCg-dcmDEG-6dMKmA-po5LLm-3dSNnN-4SxYyB-5QNDcr-eY1YE-5iXnZx-67qhyv-5Rdz8E-pJZ6Xs-PoQSz-xn8Ff-8UcSNJ-pBqZgX-4saSBE-4P6X1y-4wBSm-4v27DF-7ybZz-5R9361-9u5zKb-chr4iL-6arBZ4-qKxdQK-s5wmLX-3ivDWs-wikMVv-5Qkbca-tALd6-bfXD7Z-nFB7Wm-bWADhh-61CBtm-4UB6tG-h8hoTu-pZoMTS-gSXFSR-5QNDck-eWteMp-dnkiNh-79h48k-bd6CRz" title="Secret"><img src="https://c8.staticflickr.com/1/51/238010631_22f31b8f1e_o.jpg" width="1145" height="576" alt="Secret"></a><script async src="//embedr.flickr.com/assets/client-code.js" charset="utf-8"></script>

In Kubernetes we have two separate primitives for these use cases. The first is _Secrets_, which as the name suggest is for storing sensitive information. The second one is _ConfigMaps_, which you can use for storing general configuration. The two are quite similar in usage and support a variety of use cases.

### Secrets

Secrets can (and should) be used for storing small amounts (less than 1MB each) of sensitive information like passwords, keys, tokens, etc. Kubernetes creates and uses some secrets automatically (e.g. for accessing the API from a pod), but you can also create your own easily.

Using secrets is quite straightforward. You reference them in a pod and can then use them either as files from volumes or as environment variables in your pod. Keep in mind that each container in your pod that needs to access the secret needs to request it explicitly. There's no implicit sharing of secrets inside the pod.

There's also a special type of secret called `imagePullSecrets`. Using these you can pass a Docker (or other) container image registry login to the Kubelet, so it can pull a private image for your pod.

When updating secrets that are used by already running pods you need to be careful, as running pods won't automatically pull the updated secret. You need to explicitly update your pods (for example using the rolling update functionality of deployments explained in the [second blog post](https://blog.giantswarm.io/understanding-basic-kubernetes-concepts-using-deployments-manage-services-declaratively/) in this series).

Further keep in mind that you create a secret in a specific [namespace](http://kubernetes.io/docs/user-guide/namespaces/) and only pods in the same namespace can access the secret.

Secrets are kept in a tmpfs and only on nodes that run pods that use those secrets. The tmpfs keeps secrets from coming to rest on the node. However, they are transmitted to and from the API server in plain text, thus, be sure to have SSL/TLS protected connections between user and API server, but also between API server and Kubelets (Giant Swarm clusters do come with both enabled by default). 

### ConfigMaps

ConfigMaps are similar to Secrets, only that they are designed to more conveniently support working with strings that do not contain sensitive information. They can be used to store individual properties in form of key-value pairs. However, the values can also be entire config files or JSON blobs to store more information.

This configuration data can then be used as:

- Environment variables
- Command-line arguments for a container
- Config files in a volume

Good use cases for ConfigMaps are for example storing config files for tools like redis or prometheus. This way you can change the configuration of these without having to rebuild the container.

A difference to the secrets concept is that ConfigMaps actually get updated without the need to restart the pods that use them. However, depending on how you use the configs provided you might need to reload the configs, e.g. with an API call to prometheus to reload. 

### Towards more portable containers

Once, you're using Secrets and ConfigMaps, it's easy to differentiate between environments like dev, test, and prod. You can just use different secrets and configs to configure your containers for the respective environment.

These two concepts also make your containers more versatile in that they keep out some of the specifics and let different users deploy them in different ways. Thus, you can foster better re-use between teams or even outside of your organization.

Secrets (and in some use cases also ConfigMaps) are especially helpful when sharing with other teams and organizations or even more when sharing publicly. You can freely share your images (and manifests), maybe even keep them in a public repository, without having to worry about any company-specific or sensitive data being published.

## Daemon Sets and Jobs {#daemonsets-jobs}

In previous posts we looked at the basics of how to run our applications as pods in Kubernetes. There's two more ways to run pods that are a bit more specialized. One is the _Daemon Set_ and the other is called _Jobs_.

<a data-flickr-embed="true"  href="https://www.flickr.com/photos/phunk/4677641470/in/photolist-88m8QS-4ybrt-nYu1-pJkzSk-Nx3pw-achGi9-pSNFuN-7Qorv7-achGGw-8YEhXH-M5jtK-3EPYmM-2w4G2-2D8hX-4xSbL-6mwAAd-4ydBo-8NFkjt-oLjeHP-2NBd5w-2NBgBf-2NwQmi-7b3Dzs-4xTHz-4ydB4-prEWZx-cy5E5Q-nzidTs-4xSas-6mwBTU-4ydBR-7BFCz-4ydBY-4ydAJ-4ydzG-nfVEwr-pvNoXt-4ydzX-4ydA3-4ydBH-4ybra-4xS9P-4xS7D-4ybrC-aCLAND-4ybri-4ybqS-4xSc3-7EQPnq-A45V8s" title="Demon"><img src="https://c7.staticflickr.com/5/4018/4677641470_368e3094c7_o.jpg" width="1413" height="864" alt="Demon"></a><script async src="//embedr.flickr.com/assets/client-code.js" charset="utf-8"></script>

### Daemon Sets

A daemon set ensures that an instance of a specific pod is running on all (or a selection of) nodes in a cluster. It creates pods on each added node and garbage collects pods when nodes are removed from the cluster.

As the name suggests you can use daemon sets for running daemons (and other tools) that need to run on all nodes of a cluster. These can be things like cluster storage daemons (e.g. Quobyte, glusterd, ceph, etc.), log collectors (e.g. fluentd or logstash), or monitoring daemons (e.g. Prometheus Node Exporter, collectd, New Relic agent, etc.)

The simplest use case is deploying a daemon to all nodes. However, you might want to split that up to multiple daemon sets for example if you have a cluster with nodes of varying hardware, which might need adaptation in the memory and/or cpu requests you might include for the daemon.

In other cases you might want different logging, monitoring, or storage solutions on different nodes of your cluster. For these cases where you want to deploy the daemons only to a specific set of nodes instead of all, you can use a [node selector](https://github.com/kubernetes/kubernetes.github.io/tree/release-1.3/docs/user-guide/node-selection) to specify a subset of nodes for the daemon set. Note that for this to work you need to have labeled your nodes accordingly.

There are four ways to communicate with your daemons:

- Push: The pods are configured to push data to a service, so they do not have clients that need to find them.
- NodeIP and known port: The pods use a `hostPort` and clients can access them via this port on each NodeIP (in the range of nodes they are deployed to).
- DNS: The pods can be reached through a [headless service](http://kubernetes.io/docs/user-guide/services/#headless-services) by either using the `endpoints` resource or getting multiple A records from DNS.
- Service: The pods are reachable through a standard service. Clients can access a daemon on a random node using that service. Note that this option doesn't offer a way to reach a specific node.

Currently, you cannot update a daemon set. The only way to semi-automatically update the pods is to delete the daemon set with the `--cascade=false` option, so that the pods will be left on the nodes. Then you create a new daemon set with the same pod selector, but the updated template. The new daemon set will recognize the old pods, but not update them automatically. You will need to force the creation of pods with the new template, by manually deleting the old pods from the nodes.

### Jobs

Unlike the typical pod that you use for long-running processes, jobs let you manage pods that are supposed to terminate and not be restarted. A job creates one or more pods and ensures a specified number of them terminate with success.

You can use jobs for the typical batch-job (e.g. a backup of a database), but also for workers that need to work off a certain queue (e.g. image or video converters).

There are three kinds of jobs:

- Non-parallel jobs
- Parallel jobs with a fixed completion count
- Parallel jobs with a work queue

For non-parallel jobs usually only one pod gets started and the job is considered complete once the pod terminates successfully. If the pod fails another one gets started in its place.

For parallel jobs with a fixed completion count the job is complete when there is one successful pod for each value between 1 and the number of completions specified.

For parallel jobs with a work queue, you need to take care that no pod terminates with success unless the work queue is empty. That is even if the worker did its job it should only terminate successfully if it knows that all its peers are also done. Once a pod exits with success, then all other pods should also be exited or in the process of exiting.

For parallel jobs you can define the requested parallelism. By default it is set to 1 (only a single pod at any time). If parallelism is set to 0, the job is basically paused until it is increased.

Keep in mind that parallel jobs are not designed to support use cases that need closely-communicating parallel processes like for example in scientific computations, but rather for working off a specific amount of work that can be parallelized.

### The End?

This is the last post in this series of Kubernetes basics. However, this doesn't mean that having read all five blog posts makes you a Kubernetes master.

First, the primitives introduced, while maybe not limited to the most basic, do not cover the whole range of primitives available in Kubernetes. 

Second, there's new primitives coming with new releases, such as Pet Sets that just recently got introduced as an alpha resource in Kubernetes 1.3.

So most probably I will be writing about more primitives once I feel there's need for more or simpler explanations than what is available out there already.

Furthermore, just reading these blog posts and maybe looking through the Kubernetes documentation gives you a good foundation. However, you need to actually go and try it out and find ways to use the primitives to run and manage actual applications to get proficient in their usage. And there's no excuse that you don't have a cluster at hand, just try it out on a [local environment](https://blog.giantswarm.io/getting-started-with-a-local-kubernetes-environment/).