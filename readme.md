
## Intro 

<!-- 
Problem statement
-->

Developers want to try Kubernetes for many reasons. The problem is they are usually taken aback because of its complexity. So, what if we told you that with Weave GitOps you could spin up an Openshift cluster that is already GitOpsified? What if we told you you could do it in only two commands? And, finally, what if we told you that having the GitOps runtime installed in the cluster will allow you to manage everything – we mean EVERYTHING – from Git. Workloads, config and even infra is just a Pull Request from getting its new features. In this guide we will describe how the Weave GitOps installation works so that you can deliver your applications to an Openshift cluster fast. 

<!-- 
Solution
-->

GitOps allows developers to take control of Kubernetes from within Git. Everything is stored in Git which is the single source of truth for the declared state. Weave GitOps will reconcile what's in Git with the target environment running Kubernetes. All the time. Nothing will drift, everything has an audit trail, no changes go through without a Pull Request. That's the beauty of GitOps: Speed with complete control.

Convergent reconciliation – the magic behind GitOps – is promise theory made real for developers. Having your declared state instantiated immediately and converted in the actual state running in production is a bit of a shocker to most. Initially it feels like black magic but you get used to it, becoming trully cloud native is a matter of time. 

## Requirements

* [Weave GitOps 0.2.2](https://github.com/weaveworks/weave-gitops#weave-gitops)
* [Openshift 4.7](https://docs.openshift.com/container-platform/4.7/release_notes/ocp-4-7-release-notes.html)

## Steps

Let's first log in to the CLI using the `oc login` command and enter the required information when prompted. All relevant information about first steps in Openshift can be found in their [docs](https://docs.openshift.com/container-platform/4.1/cli_reference/getting-started-cli.html)

```
oc login -u username -p <password> https://api.crc.testing:6443
```

Now we will let Openshift know what namespace to use:

```
namespace="wego-system"
```

Now let's add the GitOps runtime provided bu Flux. This is made up of several controllers that will keep convergent reconciliation constant. All relevant information about Flux's controllers can be found in their [docs](https://fluxcd.io/docs/concepts/)

```
❯ oc adm policy add-scc-to-user privileged system:serviceaccount:$namespace:source-controller
❯ oc adm policy add-scc-to-user privileged system:serviceaccount:$namespace:kustomize-controller
❯ oc adm policy add-scc-to-user privileged system:serviceaccount:$namespace:image-automation-controller
❯ oc adm policy add-scc-to-user privileged system:serviceaccount:$namespace:image-reflector-controller
```

<!-- 
Not sure how to explain this
-->

Renaming the context name:

```
❯ kubectx openshift=. 
```

Now let's get going with Weave GitOps' installation:

```
wego gitops install
```

Openshift should've by now enabled all the pods required to provide the controllers with compute to run like so:

<!-- 
Add screenshot
-->

Now let's tell Weave GitOps what app workload to run for us in the target environment. App workloads are declarative and thus will have likely a YAML definition. 

```
wego app add .
```

You should be seeing something like this as the output of the CLI

```
Adding application:

Name: openshift-weave-gitops-demo
URL: ssh://git@github.com/openshift-fluxv2-poc/openshift-weave-gitops-demo.git
Path: ./
Branch: main
Type: kustomize

◎ Checking cluster status
:heavy_check_mark: Wego installed
✚ Generating deploy key for repo ssh://git@github.com/openshift-fluxv2-poc/openshift-weave-gitops-demo.git
uploading deploy key
✚ Generating Source manifest
✚ Generating GitOps automation manifests
✚ Generating Application spec manifest
Pull Request created: https://github.com/openshift-fluxv2-poc/openshift-weave-gitops-demo/pull/1

► Applying manifests to the cluster
```

What Weave GitOps will do now is open a Pull Request so that everyone in your team is aware of changes to the app workload. Imagine coworkers that would like to review your code, imagine managers that need to approve of it, imagine just running any CI automation to check for syntax errors and any security scanning, all that will be part of the Pull Request mechanism that Weave GitOps just opened for you. 

<!-- 
Add screenshot
-->

Now that as a dev we know we can manage an Openshift cluster from GitHub, why not create a dev environment in which to reconciliate our first changes to the app we are working on?

```
oc create namespace dev
```

Let's check everything is in place:

```
wego app status openshift-weave-gitops-demo
```

You should get this output:

```
Last successful reconciliation: 2021-08-23 19:41:02 +0700 +07

NAME                                            READY   MESSAGE                                                         REVISION                                        SUSPENDED
gitrepository/openshift-weave-gitops-demo       True    Fetched revision: main/feb77e1f794c710bb2433daa38a8c3315eaf5218 main/feb77e1f794c710bb2433daa38a8c3315eaf5218   False

NAME                                            READY   MESSAGE                                                         REVISION                                        SUSPENDED
kustomization/openshift-weave-gitops-demo       True    Applied revision: main/feb77e1f794c710bb2433daa38a8c3315eaf5218 main/feb77e1f794c710bb2433daa38a8c3315eaf5218   False
```


<!-- 
Add screenshot
-->

