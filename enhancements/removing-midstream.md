---
status: proposed
title: Removing midstream for OpenShift Pipelines releases
creation-date: '2021-12-01'
last-updated: '2021-12-01'
authors: ['vdemeester', 'concaf']
---

# Removing midstream for OpenShift Pipelines releases

… or how we should simplify our release process from upstream (tekton
org and component releases) to downstream (internal Red Hat work on
releases).

<!-- toc -->
- [Summary](#summary)
- [Motivation](#motivation)
- [Goals](#goals)
- [Requirements](#requirements)
- [Proposal](#proposal)
<!-- /toc -->

## Summary

OpenShift Pipelines is build on top of Tekton, but with some
"not-so-secret sauce". OpenShift pipelines get upstream release of
component, might apply some patches and rebuild them in the Red Hat
Toolchain. Today, we have a set of git repository in the `openshift`
organization that are mirroring upstream releases and have potential
patch attached to them — this is what we call midstream. We them sync
those in our Red Hat internal infrastructure to build the final
OpenShift Pipeline product.

Having midstream worked well in the begining but is now duplicating
work (two sync), and as everything lives upstream for OpenShift
Pipelines, it is not necessary anymore. The aim is to remove it and
have this enhancement proposal describing what we propose to do.

## Motivation

As said above, the midstream repository are now more work than they
are worth. They also have CI attached but that doesn't reflect the
product and thus CI results is most not useful.

## Goals

- Remove the use of midstream repositories (any `openshift/tekton-` repositories)
- Document what is the new "release" workflow from upstream to
  downstream
- Explore possible alternatives

## Requirements

- It should be possible to apply "out-of-upstream" patches on the
  source code before building the product.
- Ideally those set of patches are publicly available (for
  transparency)
- Re-use as much as we can from upstream and/or downstream tooling
- The process should be automated as much as possible

## Proposal

There is two different case to handle:
- Components (pipeline, triggers, chains, …)
- Operator

Let's dig into what needs to be done for both here, and then we can try to define a set of
things to run (in Task/Pipeline) to handle both these cases.

### Components

For each components we ship as part of OpenShift Pipelines, there is two simple steps to do:
- Fetch the upstream repository at a given branch
- Apply patches if need be (if they didn't get in, …)
- Build the images


### Operator

For the operator, it is a bit trickier as we have an extra step to take:
- Fetch the upstream repository at a given branch
- Fetch the component's payloads at a given release / nightly
- Apply patches if need be (same as above, …)
- Build the images

### Combination of the two

If we combine both, it looks a little bit like the following

- Fetch the upstream repository
  - Get the latest commit of the tracked branch (or tag)
  - Update `upstream_sources.yml` with it
- Read the operator metadata and fetch payload version
  - latest means we take nightly payload
  - otherwise we get the released payload
- Apply patches if need be
- Build images
  - ahead of time to not submit a changeset if it doesn't build
  - or just send a changeset

This two pieces can happen when we track new commits in the branches we track. As of today, CPaaS
doesn't seem to support this kind of custom behavior inside the "poll upstream job", but we could
deactivate it and have our own, living in our long-living *dogfooding* cluster.

In addition, it should be possible to make this all work locally on your laptop. What this means
is that it should be possible to update upstream source from upstream repositories and this would
allow you to get the proper payload apply patches and build the images.

## Design detail

The idea is to replace the CPaaS `poll-ustream` job with our own "process", and in our case, our 
own Tekton Pipeline. So the first step is to de-activate the automated `poll-upstream` Job.
It is composed in several parts that will dig into in the next sub-parts.

We are also assuming some familiarities with our *productization* repository layout.

```bash
.:
drwxr-xr-x    - vincent  2 Dec 09:50 distgit
drwxr-xr-x    - vincent  6 Dec 11:08 hack
.rw-r--r-- 2.9k vincent  6 Dec 11:24 Makefile
drwxr-xr-x    - vincent  6 Dec 11:10 patches
.rw-r--r--  128 vincent  2 Dec 09:50 project.yml
.rw-r--r-- 3.4k vincent  6 Dec 11:08 README.md
.rw-r--r--  573 vincent  6 Dec 11:09 upstream_sources.yaml

distgit/containers:
drwxr-xr-x - vincent  3 Dec 16:30 openshift-pipelines-cli-tkn
drwxr-xr-x - vincent  3 Dec 16:30 openshift-pipelines-controller
drwxr-xr-x - vincent  3 Dec 16:30 openshift-pipelines-entrypoint
drwxr-xr-x - vincent  3 Dec 16:30 openshift-pipelines-git-init
drwxr-xr-x - vincent  3 Dec 16:30 openshift-pipelines-imagedigestexporter
drwxr-xr-x - vincent  3 Dec 16:30 openshift-pipelines-kubeconfigwriter
drwxr-xr-x - vincent  3 Dec 16:30 openshift-pipelines-nop
drwxr-xr-x - vincent  6 Dec 11:33 openshift-pipelines-operator
drwxr-xr-x - vincent  6 Dec 11:09 openshift-pipelines-operator-bundle
drwxr-xr-x - vincent  3 Dec 16:30 openshift-pipelines-operator-proxy
drwxr-xr-x - vincent  3 Dec 16:30 openshift-pipelines-operator-webhook
drwxr-xr-x - vincent  3 Dec 16:30 openshift-pipelines-pullrequest-init
drwxr-xr-x - vincent  3 Dec 16:30 openshift-pipelines-serve-tkn-cli
drwxr-xr-x - vincent  3 Dec 16:30 openshift-pipelines-triggers-controller
drwxr-xr-x - vincent  3 Dec 16:30 openshift-pipelines-triggers-core-interceptors
drwxr-xr-x - vincent  3 Dec 16:30 openshift-pipelines-triggers-eventlistenersink
drwxr-xr-x - vincent  3 Dec 16:30 openshift-pipelines-triggers-webhook
drwxr-xr-x - vincent  3 Dec 16:30 openshift-pipelines-webhook

distgit/rpms:
drwxr-xr-x - vincent  2 Dec 09:50 tektoncd-cli
```

- Containers "definition" live in `distgit/containers`. One folder equals one container image to build,
  they may depend on the same "upstream" source.
  - Each folders hold *templates* (`.in` files) and files that will be copied in the respective `dist-git`
    repository. `distgit/containers/openshift-pipelines-controllers` becomes http://pkgs.devel.redhat.com/cgit/containers/openshift-pipelines-controller.
  - One thing to note on **how** the container images are build. On `brew` (RH internal toolchain), each image
    is build with the context (folder sent for build) being the folder itself. This means, for example, that 
    `openshift-pipelines-controller` is build with a command *like* `docker build distgit/containers`.
- RPMs "definition" live in `distgit/rpms`.

### Fetch latest version from upstream

The assumptions made here is that, the "source of truth" when we want to fetch version from upstream
is the `upstream_sources.yaml` file, and more accurately the `url` and `branch fields`

```yaml
git:
- automerge: 'yes'
  branch: main
  commit: aa30266234055e5d8aa373551e27f7ca885390e7
  url: https://github.com/tektoncd/triggers
- automerge: 'yes'
  branch: main
  commit: 07adee706154ce7b79045dd109f1382e997b3163
  url: https://github.com/tektoncd/pipeline
- automerge: 'yes'
  branch: main
  commit: 030e7e5ff9e8461f6ab56287cee3a539dc67a938
  url: https://github.com/tektoncd/operator
- automerge: 'yes'
  branch: main
  commit: 3aa43bb188a4234245b8d8e65a719d43e864798d
  dest_formats:
    branch:
      gen_source_repos: true
  url: https://github.com/tektoncd/cli

```

In order to get the latest version of a given branch, we can use `git ls-remote`. For example,
to get the latest commit for the `main` branch, we would do:

```bash
$ git ls-remote https://github.com/tektoncd/pipeline.git main | awk '{ print $1 }'
07adee706154ce7b79045dd109f1382e997b3163
```

We can automate this with a `Makefile` target. The idea would be to able to upgrade a component
in particular, a set of component or all component. Something like the following:

```bash 
$ make sources/upgrade
# fetch the latest commit (git ls-remote) for each components
# mutate upstream_sources.yaml with the result
$ make sources/pipeline/upgrade
# fetch the latest commit for pipeline
# mutate upstream_sources.yaml with the result
$ make sources/pipeline/upgrade sources/triggers/upgrade
# fetch the latest commit for pipeline and triggers
# mutate upstream_sources.yaml with the result
```

### Read operator metadata and fetch payload versions

For this to work, we need to assume the upstream `tektoncd/operator` repository (or any personal fork)
hold the payload information somewhere. As of today this is the case in [`test/config.sh`](https://github.com/tektoncd/operator/blob/main/test/config.sh),
but this is subject to change if we feel like it.

Given that assumption, the goal here would be to fetch the payload from this information and copy the
content into the correct set of containers.

### Apply patches if need be

### Build images

### Create a gerrit changeset

## Alternatives

### Keep midstream, and automate midstream sync

- **Pros**
  - Patches can be managed (applied) there
  - Patches are visible to anyone
  - We can automate some stuff directly in GitHub at no cost (GitHub workflow)
  - We reduce downstream need
- **Cons**
  - It's one set of repo (and automation) to manage
  - If we apply patches, we need to rebase or squash upstream changes
    - Meaning downstream fetch is acting wonky

## Open questions

TBD

What is midstream used for?
- Apply patches to upstream code
  -Move patches where? Downstream? Upstream?
- Branches and tags are tracked in upstream_sources.yaml
- CI jobs used to be run to test midstream (not downstream builds) on OpenShift
- how frequently do we need to sync branches? every merge or nightly? this needs to be automated, right?
