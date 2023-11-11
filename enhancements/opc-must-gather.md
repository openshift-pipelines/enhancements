---
status: proposed
title: Improvement in the must gather script
creation-date: "2023-04-03"
last-updated: "2023-04-03"
authors: ["shverma"]
---

# Must-Gather Enhancement

Must-Gather is a tool built on top of [Openshift must-gather](https://github.com/openshift/must-gather) that gathers openshift-pipelines debug information

<!-- toc -->

- [Summary](#summary)
- [Motivation](#motivation)
- [Goals](#goals)
- [Non-Goals](#non-goals)
- [Proposal](#proposal)
- [Alternatives](#alternatives)
- [References (optional)](#references-optional)
<!-- /toc -->

## Summary

`must-gather` is a tool(basically a script) built on top of [Openshift must-gather](https://github.com/openshift/must-gather) to gather openshift pipelines debug information.

Customers or users can attach or share openshift pipelines debug information to support the case and this debug information will be helpful for support team or developer to understand the issue of failure.

## Motivation

To gather pipelines debug information with all details including the errors from the cluster and will help customers and users to understand the issue easily.

### Goals

- Enhance the logs of must gather script
- Collects the errors of must gather script in a separate file
- Collect the cluster informations
- Collect the installed tekton components informations
- Integrate must-gather with [Openshift Pipeline CLI](https://github.com/openshift-pipelines/opc)

### Non-Goals

- Show debug information on Web console

### Proposal

#### Implement and Integrate must gather to Openshift Pipeline CLI

Users can also use opc cli to gather pipelines debug informations from cluster.
e.g: opc must-gather --image=<must-gather-image>

### Pros:

- We can customize piepline debug information and can add features which we might not do with Openshift CLI
  like to get pipeline debug information from only a particular namespace or a particular resource.

#### Gather pipeline debug information from target namespace

Current must gather script gathers pipelines debug information from all namespaces api resources of component across cluster and we can update must gather script to get the pipelines debug information
from the target namespace.

e.g. opc must-gather --image=<must-gather-image> -n {namespace}

#### Pros

- Will require less time and effort to find issue from debug information

## Alternatives

- `opc adm must-gather --image=<must gather image>` Openshift Pipelines CLI to gather pipelines debug information

## References (optional)

- [Openshift Pipelines must-gather](https://github.com/openshift-pipelines/must-gather)
