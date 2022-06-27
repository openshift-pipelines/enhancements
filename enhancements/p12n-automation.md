---
status: proposed
title: Desired state of OpenShift Pipelines P12n
creation-date: '2022-05-11'
last-updated: '2022-05-11'
authors: ['concaf']
---


# Desired state of OpenShift Pipelines P12n

This document defines the end goal / desired state of productization desired in OpenShift Pipelines.

## What is SLSA?

### What is SLSA?

SLSA is a set of incrementally adoptable security guidelines, established by industry consensus.

### Summary of levels

| Level | Description                                   | Example                                               |
| :---- | :-------------------------------------------- | :---------------------------------------------------- |
| 1     | Documentation of the build process            | Unsigned provenance                                   |
| 2     | Tamper resistance of the build service        | Hosted source/build, signed provenance                |
| 3     | Extra resistance to specific threats          | Security controls on host, non-falsifiable provenance |
| 4     | Highest levels of confidence and trust        | Two-party review + hermetic builds                    |
### Detailed explanation

| Level | Requirements |
| ----- | ------------ |
| 0     | **No guarantees.** SLSA 0 represents the lack of any SLSA level. |
| 1     | **The build process must be fully scripted/automated and generate provenance.** Provenance is metadata about how an artifact was built, including the build process, top-level source, and dependencies. Knowing the provenance allows software consumers to make risk-based security decisions. Provenance at SLSA 1 does not protect against tampering, but it offers a basic level of code source identification and can aid in vulnerability management. |
| 2     | **Requires using version control and a hosted build service that generates authenticated provenance.** These additional requirements give the software consumer greater confidence in the origin of the software. At this level, the provenance prevents tampering to the extent that the build service is trusted. SLSA 2 also provides an easy upgrade path to SLSA 3. |
| 3     | **The source and build platforms meet specific standards to guarantee the auditability of the source and the integrity of the provenance respectively.** We envision an accreditation process whereby auditors certify that platforms meet the requirements, which consumers can then rely on. SLSA 3 provides much stronger protections against tampering than earlier levels by preventing specific classes of threats, such as cross-build contamination. |
| 4     | **Requires two-person review of all changes and a hermetic, reproducible build process.** Two-person review is an industry best practice for catching mistakes and deterring bad behavior. Hermetic builds guarantee that the provenance’s list of dependencies is complete. Reproducible builds, though not strictly required, provide many auditability and reliability benefits. Overall, SLSA 4 gives the consumer a high degree of confidence that the software has not been tampered with. |

## So, what do we want?

- We want to use OpenShift Pipelines to ship OpenShift Pipelines while adhering to Red Hat p12n requirements
- We achieve, at least, SLSA level 2, i.e.
    - The build process must be fully scripted/automated and generate provenance
    - Requires using version control and a hosted build service that generates authenticated provenance
- We want a dashboard (via an API) that is [the source-of-truth and that displays the current state of our p12n](https://issues.redhat.com/browse/SRVKP-1918) - release dates, CI failures, overall health

## How does p12n + process automation look like?

### Prerequisites

- [Separate p12n (HB) pipeline for tkn?](https://issues.redhat.com/browse/SRVKP-2098)
- [Is CPaaS the right way for tkn?](https://issues.redhat.com/browse/SRVKP-2105)
- [Fix CPaaS CI](https://issues.redhat.com/browse/SRVKP-2126)


### Upstream changes

- As soon as a change gets merged upstream, CI updates upstream_sources.yaml downstream.
    - [Auto-trigger upstream sync task on upstream merges](https://issues.redhat.com/browse/SRVKP-2127)
    - [Auto-merge "sync upstream" changeset or PR](https://issues.redhat.com/browse/SRVKP-1947)
- CI runs e2e tests on the resulting downstream build.
    - [Consolidate e2e and tests scripts to run on build](https://issues.redhat.com/browse/SRVKP-1916)
- Then, CI publishes downstream artifacts which are posted in upstream PR and also visible in the p12n dashboard.
    - [Add downstream release artifacts to every PR](https://issues.redhat.com/browse/SRVKP-2021)
    - [Reduce the bugfix merge --> QE testing time](https://issues.redhat.com/browse/SRVKP-2027)

### Release

There is a lot of scripting that can be done to make the release process automated - and run as Tasks/Pipelines in OpenShift Pipelines.

#### Adding a new image

Currently we follow the steps [here](https://gitlab.cee.redhat.com/cpaas-products/openshift-pipelines#steps-to-add-a-new-image-in-pipelines-p12n) to add a new image - this could be automated via git and GitLab API.

#### New minor/patch releases

We follow the steps [here](https://gitlab.cee.redhat.com/cpaas-products/openshift-pipelines#steps-to-add-a-new-image-in-pipelines-p12n) and [here](https://gitlab.cee.redhat.com/cpaas-products/openshift-pipelines#new-patchsecurity-release-xyz-playbook) to setup the release machinery in HoneyBadger, CPaaS, Errata, etc. This can be updated via Errata API, GitLab API and git.

#### External team dependencies

We are dependent on other teams like:
- RCM - for binary signing, HoneyBadger, advisory processing, etc
- ART - to upload manifests to mirror.openshift.com

Some delays here are due to the nature of the beast (external failures), but from our end we could [automate creation of tickets on Jira](https://issues.redhat.com/browse/SRVKP-1616)

## References

- https://slsa.dev/spec/v0.1/levels