---
title: PipelinesAsCode in Tekton Operator
authors:
- "@sm43"
  creation-date: 2022-01-28
  last-updated: 2022-01-28
  status: implementable
---

# PipelinesAsCode in Tekton Operator

<!-- toc -->
  - [Final Decision:](#final-decision)
- [Proposed Solutions:](#proposed-solutions)
  - [Integrating with Upstream Tekton Operator](#integrating-with-upstream-tekton-operator)
    - [How?](#how)
    - [Pros](#pros)
    - [Cons](#cons)
  - [Adding PAC as a component in TektonAddon](#adding-pac-as-a-component-in-tektonaddon)
    - [How?](#how-1)
    - [Pros](#pros-1)
    - [Cons](#cons-1)
  - [Single Separate Code Repository for OpenShift specific extension and component](#single-separate-code-repository-for-openshift-specific-extension-and-component)
    - [How?](#how-2)
    - [Pros](#pros-2)
    - [Cons](#cons-2)
  - [Single Separate Deployment for OpenShift Component](#single-separate-deployment-for-openshift-component)
    - [How?](#how-3)
    - [Pros](#pros-3)
    - [Cons](#cons-3)
  - [One Deployment per OpenShift component](#one-deployment-per-openshift-component)
    - [How?](#how-4)
    - [Pros](#pros-4)
    - [Cons](#cons-4)
<!-- /toc -->

Goal: Installing and Managing PAC through Tekton Operator/OpenShift Pipelines Operator.

This document explores different ways to do that and to decide which way we want to proceed.

### Final Decision:
- After Discussion with team, it was decided to move on with Solution number 2 that is adding PAC as an Addon
- In future, if PAC gets complex and requires a separate lifecycle manager then we would think about adding a CRD and a reconciler.

## Proposed Solutions:

### Integrating with Upstream Tekton Operator

#### How?
- adding a New CRD eg. PipelinesAsCode and a reconciler which would install and manage PAC.
- Keeping it separate in a contrib directory

#### Pros
- All development happening upstream at single place, so we could directly productised from upstream and no additional work in midstream
- PAC can be released with Upstream Operator release without any additional work

#### Cons
- As this is coming from Red Hat, this wouldn’t be supported by Tekton community and need to be mentioned explicitly
- This could be solved with having a contrib-like folder/thing for things we may want to integrate as part of the Operator but outside of tekton org or experimental.
- This was mentioned in an upstream discussion, If something is packaged with Tekton Operator, for users it might mean it is part of Official Tekton, but pac is not so there was concern mentioned naming it TektonPipelinesAsCode initially but it could be renamed to OpenShiftPipelinesAsCode or just PipelinesAsCode and explicitly mention that not supported by Tekton Community but this would occur again in future if we add any more OpenShift specific components.

### Adding PAC as a component in TektonAddon

#### How?
- Currently, we install clusterTasks, Pipelines templates and other resources using TektonAddon CR.
- We could consider PAC as an Addon and install it through TektonAddon
- There could be a field in TektonConfig/TektonAddon to enable/disable PAC installation

#### Pros
- No new CRD required

#### Cons
- Currently, TektonAddon is enabled for OpenShift only. If in future if we enable it or k8s then this would go in OpenShift Extension
- TektonAddon consist of OpenShift specific resources so cannot be enabled on k8s at present


### Single Separate Code Repository for OpenShift specific extension and component

#### How?
- Create a new repo eg. OpenShift Pipeline operator for OpenShift specific components and extensions
- Pac would have a CRD and a reconciler to manage it’s installation and lifecycle
- Along with this we add all upstream OpenShift extension in this repo and main reconciler will be imported from upstream tektoncd repo

#### Pros
- Having multiple repositories give more freedom
- Single Image
- Auto installation of pac can be handled
  - in upstream openshift extension
  - May be using init container
  - Or by watching TektonCOnfig as Secondary Resources

#### Cons
- Local development would be different from upstream in this case
- Any changes to main reconciler will go in upstream repo and any changes in extension will go in newly created repo
- Will have to create release yaml from this repository
- Setup New CI

### Single Separate Deployment for OpenShift Component

#### How?
- Create a new repo eg. OpenShift Pipeline operator for OpenShift specific component
- Pac would have a CRD and a reconciler to manage it’s installation and lifecycle
- This would have a separate deployment which would be added in CSV along with upstream

#### Pros
- Having multiple repositories give more freedom
- Auto installation of pac can be handled
  - in upstream openshift extension
  - May be using init container
  - Or by watching TektonConfig as Secondary Resources

#### Cons
- Local development would be different from upstream in this case
- Upstream dev will happen as it is currently, for pac reconciler it would be in this repo
- Setup New CI


### One Deployment per OpenShift component

#### How?
- hub-> hub deployment, pac -> pac-deployment added directly in csv
- Each component will have one CRD and a reconciler which installs and manages its lifecycle
- Deployment for this component can be directly added to CSV along with Tekton Operator Deployment
- This could also mean pac, hub and any other component in future can have its own deployment and can be added csv directly

#### Pros
- Each component repo can have its operator code
- Pac, Hub can have their operator controller in their repositories and can be added to csv with their deployments
- Each component can have its own operator

#### Cons
- More images to productized
- Extra work to fetch all component deployments and package in csv
