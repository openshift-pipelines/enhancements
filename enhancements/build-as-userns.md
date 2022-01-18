# Build containers images as user

## Goals

There is two different scenarios we want to achieve :

1) Allow user to run buildah as root inside the container but running as user on the host

2) Running as user inside the container.

### Scenario 1 Running as root on container as user on host

* Openshift 4.10 will introduces a new "builder" workload on cri-o config. This
  allows running container as user on the host but shows as root on host. The
  annotations looks like this :

  ```yaml
    annotations:
      io.kubernetes.cri-o.userns-mode: "auto"
      io.openshift.builder: "true"
  ```

### Scenario 2 Running as user on container

* The official buildah image :

```
quay.io/buildah/stable:v1.21.0
```

has everything we need to run as user inside the containers using the user `'build''`

* As long as we run the container as this user we should be able to build
  containers as user by default.

## Implementations

### Solution 1

* This should works by default on openshift >4.10 as long as we have the PR on machineconfig

* We would have to modify the buildah task to add the annotations into it.

* We would need this PR merged to get the subuids and subgids added on hosts :

  https://github.com/openshift/machine-config-operator/pull/2906

* There is currently a bug on pipeline where the annotation on task doesn't
  propogate properly when running from a pipelinerun, the PR is here :

  https://github.com/tektoncd/pipeline/pull/4478


### Solution 2

* we need to enforce running as the `build` user on container.

* To allow that we could have a new SCC with less right than the pipelines scc
  and only able to run as the user build instead of the anyuid rights we have
  now.

* the openshift-pipelines operator then would have to create two different SA and two different SCC in each namespaces.

* we would need to modify the buildah tekton tasks to use that SA.

* we would probably need to modify the s2i images to do the same as what the buildah container is doing.


## Examples

* Additional SCC:

```yaml
...
runAsUser:
  type: RunAsAny
  uid: 1000
seLinuxContext:
  type: MustRunAs
supplementalGroups:
  type: RunAsAny
```
