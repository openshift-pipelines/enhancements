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

## Alternatives

## Open questions

TBD

What is midstream used for?
- Apply patches to upstream code
  -Move patches where? Downstream? Upstream?
- Branches and tags are tracked in upstream_sources.yaml
- CI jobs used to be run to test midstream (not downstream builds) on OpenShift
- how frequently do we need to sync branches? every merge or nightly? this needs to be automated, right?