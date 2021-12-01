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

TBD

What is midstream used for?
- Apply patches to upstream code
  -Move patches where? Downstream? Upstream?
- Branches and tags are tracked in upstream_sources.yaml
- CI jobs used to be run to test midstream (not downstream builds) on OpenShift
