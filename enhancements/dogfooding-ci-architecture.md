---
status: proposed
title: Dogfooding OpenShift Pipelines
creation-date: '2021-08-30'
last-updated: '2021-08-30'
authors: ['vdemeester']
---

# Dogfooding OpenShift Pipelines architecture

… or how would be our ideal CI setup to build OpenShift Pipelines with
OpenShift Pipelines.

<!-- toc -->
- [Summary](#summary)
- [Motivation](#motivation)
  - [Goals](#goals)
  - [Non-Goals](#non-goals)
- [Requirements](#requirements)
- [Proposal](#proposal)
- [Drawbacks](#drawbacks)
- [Alternatives](#alternatives)
- [References (optional)](#references-optional)
<!-- /toc -->

## Summary

OpenShift Pipelines is build on top of Tekton, and as such, should be
*agile* enough to be able to build and test itself. This document
should cover all part of building and testing OpenShift Pipelines that
are public (aka we won't dig into the specific *productization*
process of Red Hat as it is out of scope and not public).

## Motivation

It is important to use what you build 👼. In other words, if we are
building a CI/CD tool that we don't use ourselves for our own CI/CD
needs (as a team), this mean we are doing something wrong (either we
are not building the right thing, or we are duplicating effort somehow).

This document aims to serve as a direction and documentation of how
OpenShift Pipelines is building on top of itself (and Tekton) and how
the CI/CD setup is done, or at least what we are aiming for.

### Goals

TBD

### Non-Goals

TBD

## Requirements

TPD

## Proposal

TBD

## Drawbacks

TBD

## Alternatives

TBD

## References (optional)

TBD
