---
title: Option to disable default creation of service account and rbac resources
authors:
  - "@savitaashture"
creation-date: 2021-09-19
last-updated: 2021-09-19
status: implementable
---

# TEP-0001: Option to disable default creation of service account and rbac resources on Openshift

<!-- toc -->
- [Summary](#summary)
- [Motivation](#motivation)
  - [Goals](#goals)
- [Proposal](#proposal)
  - [User Stories](#user-stories)
  - [Usage examples](#usage-examples)
- [Design Details](#design-details)
  - [Disable creation of RBAC resources at cluster level](#disable-creation-of-rbac-resources-at-cluster-level)
  - [Scenarios](#scenarios)
    - [Disable auto creation of RBAC resources](#disable-auto-creation-of-rbac-resources)
    - [Enable auto creation of RBAC resources (Default behavior)](#enable-auto-creation-of-rbac-resources)
- [Risk](#risk)
- [Alternative Approach](#alternative-approach)
<!-- /toc -->

## Summary

The proposal helps cluster admin to disable auto creation of RBAC resources
(ServiceAccount, RoleBinding, SCCRoleBinding, CABundlesConfigMap and openshift-pipelines-clusterinterceptors ClusterRoleBinding).
Related Stories and Epics are

https://issues.redhat.com/browse/SRVKP-1578

https://issues.redhat.com/browse/SRVKP-1670

https://issues.redhat.com/browse/SRVKP-1649

## Motivation

OpenShift Pipelines operator will create a RBAC resources (ServiceAccount(`pipeline`), RoleBinding, SCCRoleBinding, CABundlesConfigMap and openshift-pipelines-clusterinterceptors) on all namespace when installed.
The pipelines-scc-rolebinding has RunAsAny among other things.
This can be seen as a security issue as some customers have reported. It would be interesting if a cluster-admin could disable this feature on basis of requirement.

### Goals

Provide way to cluster admin to disable auto creation of rbac resources at cluster level. 

### Non-Goals

## Proposal

Installation of OpenShift Pipelines operator by default create RBAC resources on all namespaces.
cluster admin should have the permission to disable RBAC resource creation at cluster level using `TektonConfig CR`. 
   
### User Stories
As a cluster admin, I want to be able to disable auto creation of ServiceAccount and RBAC resources at cluster level because
some customers have reported that SCCRolebinding `pipelines-scc-rolebinding` can be seen as a security issue which has **RunAsAny** among other things.

### Usage examples

## Design Details

The main goal of this TEP is to provide ways to cluster admin to disable creation of RBAC resources at cluster level.

### Disable creation of RBAC resources at cluster level
Cluster admin can create/edit TektonConfig CR and set `createRbacResource` to `false` so that RBAC resources will not create in any of the namespaces in that cluster.
```yaml
apiVersion: operator.tekton.dev/v1alpha1
kind: TektonConfig
metadata:
  name: config
spec:
  params:
  - name: createRbacResource
    value: "false"
  profile: all
  targetNamespace: openshift-pipelines
  addon:
    params:
    - name: clusterTasks
      value: "true"
    - name: pipelineTemplates
      value: "true"

```

### Scenarios
##### Disable auto creation of RBAC resources
* Set Param 
```yaml
params:
- name: createRbacResource
  value: "false"
```
in TektonConfig CR then RBAC resources will not create on any namespace

##### Enable auto creation of RBAC resources (Default behavior)
By default auto creation of RBAC resources are enabled.
If its disabled and admin wants to enable it they can set param like below
```yaml
params:
- name: createRbacResource
  value: "true"
```
OR remove `createRbacResource` from TektonConfig CR then RBAC resources will auto create on all namespaces

## Advantages

## Test Plan

## Alternatives

## Risk
If auto creation of RBAC resources disabled then ClusterTask would not work by default anymore. 
This has to be clear to the end user that when they disables this, either it also disable default cluster task or they are expected to not work.

## Alternative Approach
When cluster admin disable auto creation of ServiceAccount and RBAC resources in all namespaces(at cluster level) 
then its the responsibility of user to create those resources manually if they require.

## Follow Up Story
As part of this TEP we are focusing on disabling auto creation at cluster level 
but going forward we will add disabling auto creation of RBAC resources for particular namespace(`namespace level`).