---
status: implementable
title: Hub in OpenShift Pipelines
creation-date: "2021-09-07"
last-updated: "2021-02-27"
authors: ["vinamra28"]
---

# Hub in OpenShift Pipelines Operator

<!-- toc -->

- [Summary](#summary)
- [Motivation](#motivation)
  - [Goals](#goals)
  - [Non-Goals](#non-goals)
- [Requirements](#requirements)
- [Proposal](#proposal)
- [Design](#design)
  - [Handling Upgrades](#handling-upgrades)
- [Drawbacks](#drawbacks)
- [Alternatives](#alternatives)
- [References (optional)](#references-optional)
<!-- /toc -->

## Summary

Tekton Hub provides a central hub for searching and sharing Tekton resources
across many distributed Tekton catalogs hosted by various organizations and
teams. Hub currently displays a curated set of community contributed tasks
from the Community Catalog. It allows resources to be searched by name or
its “display name”, filtered by categories (cloud, cli, github etc…), and
rated by users. This document should cover the way how we can ship Tekton Hub
in a better way so that users can install their own personalized hub within their
cluster.

## Motivation

With time Hub is gaining more attention by many users and organizations
and want to deploy the Hub within their own environment. Keeping this
in mind we should simply the process of deployment so that with less
effort hub can be installed on anyone's cluster.

### Goals

Provide a way for users to...

1. Run their own instance of Tekton Hub on their own cluster.
2. Expose the OpenShift routes so that ODC can use the exposed APIs
   in `PipelineBuilder`.

### Non-Goals

This TEP will not cover how the user is going to expose the service
to external world via `Ingress`.

## Requirements

1. Images should be made available to the cluster in case of disconnected install.
2. Required secrets should be present in the `targetNamespace`.

## Proposal

Have a CRD in Tekton Operator and a controller/reconciler which can handle CRUD operations
for `Tekton Hub` resources such as API, UI and DB. Whenever a user installs a TektonHub CR
with required spec it's going to perform following operations:

- Create DB deployment
- Run DB migration
- API deployment, service and Route in case of OpenShift
- UI deployment, service and Route in case of OpenShift

## Design

```yaml
apiVersion: operator.tekton.dev/v1alpha1
kind: TektonHub
metadata:
  name: hub
spec:
  targetNamespace:
    # <namespace> in which you want to install Tekton Hub. Leave it blank if in case you want to install
    # in default installation namespace ie `openshift-pipelines` in case of OpenShift and `tekton-pipelines` in case of Kubernetes
  api:
    hubConfigUrl: https://raw.githubusercontent.com/tektoncd/hub/main/config.yaml
```

There would be separate manifests for deploying via Tekton Operator instead of
having a common `release.yaml`.

### Flow

The installation flow of `Tekton Hub` is as follows:

1. Create the `TektonHub` CR which will create the `targetNamespace` if it's not present.
1. Create the required secrets for API in the target namespace.
1. the `Tekton Hub` components in the following order:
   1. Create `db` secrets
   1. Create DB deployment, pvc and service
   1. Run the DB-Migration, if it fails then do not proceed
   1. After the successful completion of DB-Migration, search for the API secrets
   1. If API secrets are present then create the remaining components required for API
   1. Post successful running of API, do the same process for UI

## Handling Upgrades

Whenever there is a new version of OpenShift Pipelines available, all the components
of Tekton Hub are going to upgrade in the following order:

1. Run the new db-migration
1. After successful completion of db-migration run the new API and UI server.

## Handling Uninstall

Whenever the CR TektonHub is deleted from the cluster, all the components
related to Tekton Hub along with `tekton-hub` namespace should be deleted.

## Drawbacks

With current release cycle users may have to wait for the next release of
OpenShift Pipelines to make it available for OpenShift users.

## Alternatives

### Hub as it's own Operator

We can also create a separate operator for Hub and manage it's lifecycle
by own instead of relying on OpenShift Pipelines operator.

#### Drawbacks

There will be an overhead to manage one extra operator.