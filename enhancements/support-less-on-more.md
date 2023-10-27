---
status: propose
title: Update OpenShift Pipelines support policy to support less versions on more OpenShift versions
creation-date: '2023-07-31'
last-updated: '2023-07-31'
authors: ['vdemeester']
---

# Update OpenShift Pipelines support policy to support less versions on more OpenShift versions

<!-- toc -->
- [Summary](#summary)
- [Motivation](#motivation)
- [Proposal](#proposal)
  - [Support only the last 2 releases](#support-only-the-last-2-releases)
  - [Support down to <code>MIN_KUBERNETES_VERSION</code>](#support-down-to-)
- [Testing](#testing)
- [Dependencies](#dependencies)
- [Examples](#examples)
- [When ?](#when-)
<!-- /toc -->

---
type: project
deadline: 2023-09-02
tags:
  - project
---
## Summary

The plan is to reduce the number of OpenShift Pipelines version we
support at a given time while augmenting the number of OpenShift
version we support them in.


Supporting Materials:
 * [Enhance Support Policy of OpenShift Pipelines](./enhance-support-policy.md)
 * [OCP 4.y lifecycles](https://docs.google.com/document/d/1exNNJPdwCH3F5eEKaLVBcrj_qAHfJ3PlfnquP0-6TxM/edit#heading=h.539ohzh6t1k6)
 * [Proposal for Pipelines &
   GitOps](https://docs.google.com/spreadsheets/d/1Tn6BHNBJnEaTBRm22U0KrNCR_Zwu_qBn-IR-xFEa4Os/edit#gid=1430910332)

## Motivation

OpenShift Pipelines versions are currently only shipped and supported
for a small set of OpenShift version (n-1, n, n+1).  OpenShift support
can be very long (especially for EUS version), which makes us having
to support a lot of version at the same time. Adding to this support
exception that we may give to customers, it can become a burden for
maintenance, and a lot of bug-fix version to release.

For example, today (2023, 31st of July), we are *technically*
supporting the following:
- 1.8.x on 4.10 (EUS)
- 1.9.x on >= 4.10
- 1.10.x
- 1.11.x

In addition, it has no real technological reason for not supporting
multiple version. Tekton supports a relatively wide range of Kubernetes
version, and so can OpenShift Pipelines.

Enabling customers to get the latest of feature from Tekton in their
supported cluster is a win-win situation : customers can use the
latest, and we can ship the latest (fixes, …) to our customers.

## Proposal

### Support only the last 2 releases

The proposal is to only support the last 2 releases. As of today, this
would mean supporting 1.11 and 1.10.

In addition, as soon as release N is released, release N-1 while being
still supported, only gets important bugfixes and security
fixes. Quick note on this, it would also get any bugfix release if
upstream components tracked in that release do a bugfix release as
well.

The idea of the proposal is to make it more appealing to get the
latest release. The only challenge can be the
`MIN_KUBERNETES_VERSION`, as discussed below.

### Support down to `MIN_KUBERNETES_VERSION`

Each version we release is supported on **any** OpenShift version that
satisfy the following:

- OpenShift version is still supported.
- The pipeline `MIN_KUBERNETES_VERSION` is compatible with the
  kubernetes version of OpenShift.

In a gist this would mean, today, that:

- 1.11 (`MIN_KUBERNETES_VERSION` = 1.24) is supported on >= 4.11
- 1.10 (`MIN_KUBERNETES_VERSION` = 1.23) is supported on >= 4.10

As of today, no OpenShift version below 4.10 is supported, so we could
support only 1.10 and 1.11. In the case of an OpenShift version is
still supported but the `MIN_KUBERNETES_VERSION` is lower than any of
the 2 most recent OpenShift Pipeline's release, we would have to
support that one (in maintenance/security mode) until the given
OpenShift release is not supported anymore, but only on that given
OpenShift version.

For example, when we release 1.12 (`MIN_KUBERNETES_VERSION` = 1.24),
we will need to still support 1.10 *on* 4.10 (not necessarily on 4.11
or higher).

## Testing

This means we are testing OpenShift Pipelines on more OpenShift version. Although this increase a little bit the testing effort for one release, the assumption here are the following:

- We'll have more and more automated test making that effort less.
- We'll have less z-stream for older version to do, and thus reducing the overall effort.

## Dependencies

There is possible dependencies from/to our operator that could be impacted by this move.
One of them is the *devconsole*, aka the UI of openshift-pipelines. Supporting older version of OpenShift might mean incompatibilities with the UI and bugs that would require an z-stream, and that might be a problem. One assumption here is that **we would move forward to own our UI (using dynamic plugin) so that we are independent of the console/devconsole lifecycle**.

## Examples

- OSP 1.11 - released 2023-06
	- OCP >= 4.11
	- Not supported in 2023-12 (OSP 1.13 + 1 month)
- OSP 1.12 - released 2023-09
	- OCP >= 4.11
	- Not supported in 2024-04 (OSP 1.14 + 1 month)
- OSP 1.13 - release 2023-11
	- OCP >= 4.12 (EUS)
	- Not supported in 2024-06 (OSP-1.15 + 1 month) **or** until 4.12 EUS is not supported (which is, for 4.12, **2025-01**)
- OSP 1.14 - release 2024-03 (OSP 1.16 + 1 month)
	- (*example*) OCP >= 4.13
	- Not supported in 2024-09 (OSP 1.16 + 1 month)
- OSP 1.15 - release 2024-06 (OSP 1.17 + 1 month)
	- (*example*) OCP >= 4.13
	- Not supported in 2024-12 (OSP 1.18 + 1 month)
- … and so one

### End-User Support (EUS)

For a given EUS release, we support the latest shipped OpenShift Pipelines version that was shipped before the maintenance support window ended, and we will not ship new one (after that window ended).

For example, for OCP 4.13

- maintenance support ends: November 17, 2024
- we are releasing 1.17 on December 31, 2024, then:
  - we will ship it for OCP 4.12 but we will not ship it for OCP 4.13
  - we will ship it for OCP 4.14 onwards (whatever the supported OCP versions are)

In addition, we would release one version of OSP for 3 or 4 releases (from the lowest counting upward). But nothing should/would prevent us from adding support for a new version of OpenShift, by doing a bugfix release.

## When ?

We can put this *change* in place any time as we could retrofit this on older version — to support older OpenShift version — by doing bugfix releases.
