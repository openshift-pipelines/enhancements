---
status: implemented
title: Enhance Support Policy of OpenShit Pipelines
creation-date: '2022-02-09'
last-updated: '2023-07-31'
authors: ['vdemeester']
---

# Enhance Support Policy of OpenShit Pipelines

Link: [SRVKP-1738](https://issues.redhat.com/browse/SRVKP-1738)

<!-- toc -->
- [Summary](#summary)
- [Motivation](#motivation)
- [Challenges](#challenges)
- [OLM and contiguous supported ocp version](#olm-and-contiguous-supported-ocp-version)
- [Channel names](#channel-names)
- [Proposal](#proposal)
  - [Using &quot;range&quot; notation](#using-range-notation)
  - [Versioned channel names](#versioned-channel-names)
- [Alternatives](#alternatives)
  - [Keep using &quot;one version&quot; notation](#keep-using-one-version-notation)
  - [Keep current channel names (stable/preview)](#keep-current-channel-names-stablepreview)
<!-- /toc -->

## Summary

The plan is to be able to support Pipelines, GitOps, ServiceMesh and
Serverless using the same OpenShift profiles/lifecycles.

Pipelines 1.7:  N-1
Pipelines 1.8:  N-2
Pipelines 1.9:  N-3
Pipelines 1.10: N-3


Supporting Materials:
 * [OCP 4.y lifecycles](https://docs.google.com/document/d/1exNNJPdwCH3F5eEKaLVBcrj_qAHfJ3PlfnquP0-6TxM/edit#heading=h.539ohzh6t1k6)
 * [Proposal for Pipelines &
   GitOps](https://docs.google.com/spreadsheets/d/1Tn6BHNBJnEaTBRm22U0KrNCR_Zwu_qBn-IR-xFEa4Os/edit#gid=1430910332)

## Motivation

OpenShift Pipelines version are currently only shipped and supported
for one specific version of OpenShift. This mean that, for example,
OpenShift Pipelines 1.5 is only available and supported on 4.8,
OpenShift Pipelines 1.6 on 4.9 and so on. Although this makes our
"support life" easier, but it comes with some challenges:

- this puts pressure on our customers to upgrade to the latest
  OpenShift early if they want to benefit from the latest features that
  are coming in OpenShift Pipelines.
- this means our customers might lag behind for a long time and use
  old version of upstream tekton components that have possibly bugs
  already fixed.

In addition, it has no real technological reason for not supporting
multiple version. Tekton supports a relatively wide range of k8s
version, and so can OpenShift Pipelines.

Enabling customers to get the latest of feature from Tekton in their
supported cluster is a win-win situation : customers can use the
latest, and we can ship the latest (fixes, …) to our customers.

## Challenges

## OLM and contiguous supported ocp version

As written in [SRVCOM-1654](https://issues.redhat.com/browse/SRVCOM-1654):

> According to [Managing OpenShift
> Versions](https://redhat-connect.gitbook.io/certified-operator-guide/ocp-deployment/operator-metadata/bundle-directory/managing-openshift-versions),
> using a range will prevent the operator from being included in a
> version past the highest version in the range. For example, the
> Serverless 1.19.0 operator currently specifies "v4.6-v4.9". This
> means that the operator will not be included in the OCP v4.10
> index. If the operator had specified "v4.6" instead in the label,
> then the 1.19.0 operator would be included in the OCP 4.10 index.

If we switch to the “range” notation (as we started on nightlies), we
will end up with forcing ourselves to release ahead of OpenShift each
time if we want to not dig ourselves into a whole (aka, ending up with
no version at a new OCP version released).

The idea behind
[SRVCOM-1654](https://issues.redhat.com/browse/SRVCOM-1654) is to set
the ocp version at the lowest version that the team (serverless here)
would like to support. Usually this would probably be the minimum
supported version upstream. And each time that minimum supported
version upstream is incremented, the ocp version supported would
increase. On paper this could be seen as “too much” version to support
as this could mean a version supporting 4.8 would be available in all
future versions (if no further update). In practice, as we will
release more versions, with the same “lower ocp version supported”, we
would “hide” those versions and it would require a very explicit and
“deep” knowledge to install a previous version of OpenShift Pipelines.

## Channel names

A somewhat recent, but valid, request from our customer is : to be
able to auto-upgrade security fixes, but not new versions. This means,
for example, that 1.6.0 -> 1.6.1 would be automatic, but 1.6.1 ->
1.7.0 would require manual intervention (approval, …). This is not
currently supported by OLM and thus may require some “adjustments” on
the way we name/manage our channels.

See also [OpenShift Pipelines GA Operator
Channels](https://docs.google.com/document/d/1wktenlhM64o_38BhWgz8Am-YjKkeBj7thzjQ1aNXwUg/edit#heading=h.cliecwuv1cq).

## Proposal

### Using "range" notation


- **Pros**
  - Allow us to “technically” restrict on which OCP our product is shipped
  - (Similar) Document clearly what version are supported from the Bundle annotation
- **Cons**
  - (OLM shortcomings) won’t be added to newer OCP version
  - Forces us to do at least one release before a newer OCP version
  - Because of that, might hinder using Pipelines on newer OCP version (if we don’t have a release already “supporting it”)
    This is a little bit “taken care of” by our nightlies, but still.

The reason we would use the range notation, is to **make it clear and
explicit** what are the version of OpenShift this version will
support — even if those are future version.

| OSP version | Notation       |
|-------------|----------------|
| 1.6.x       | 4.9 (no range) |
| 1.7.x       | 4.9-4.11       |
| 1.8.x       | 4.10-4.12      |
| 1.9.x       | 4.11-4.13      |
| 1.10.x      | 4.12-4.14      |

*We can also decide to “augment” this even more, such as 1.10.x would
be 4.11 -> 4.15 for example, if we feel like it could be a thing.*

### Versioned channel names

As stated above, some customer want to go at their pace into upgrading
OpenShift Pipelines to a newer version (with new features) but keep
getting automatic upgrades from OLM. This would be achieve by the
following set of versionned channels.

For each supported version of OpenShift Pipelines, we would define a
`pipelines-{VERSION}` channel that get only that release and bugfixes
associated with it. In addition, to satisfy customers that want to be
always running the latest version, we would provide a `latest` channel
that gets the latest supported version on this OpenShift release.


|          | T (4.9, 1.6)            | T+1 (4.10, 1.7)                                                              | T+2 (4.11, 1.8)                                                                     | T+3 (4.12, 1.9)                                                                           | T+4 (4.13, 1.10)                                                                               |
|----------|-------------------------|------------------------------------------------------------------------------|-------------------------------------------------------------------------------------|-------------------------------------------------------------------------------------------|------------------------------------------------------------------------------------------------|
| OCP 4.9  | stable: 1.6.x           | stable: 1.6.x<br>**latest**: 1.7.x<br>pipelines-1.7: 1.7.x<br>preview: 1.6.x | stable: 1.6.x<br>**latest**: 1.7.x<br>pipelines-1.7: 1.7.x<br>preview: 1.6.x        | stable: 1.6.x<br>**latest**: 1.7.x<br>pipelines-1.7: 1.7.x<br>preview: 1.6.x              | stable: 1.6.x<br>**latest**: 1.7.x<br>pipelines-1.7: 1.7.x<br>preview: 1.6.x                   |
| OCP 4.10 | *RCs* stable: 1.6.x (?) | stable: 1.6.x<br>**latest**: 1.7.x<br>pipelines-1.7: 1.7.x<br>preview: 1.6.x | **latest**: 1.8.x<br>pipelines-1.7: 1.7.x<br>pipelines-1.8: 1.8.x<br>preview: 1.6.x | **latest**: 1.8.x<br>pipelines-1.7: 1.7.x<br>pipelines-1.8: 1.8.x<br>preview: 1.6.x       | **latest**: 1.8.x<br>pipelines-1.7: 1.7.x<br>pipelines-1.8: 1.8.x<br>preview: 1.6.x                              |
| OCP 4.11 | N/A                     | *RCs* **latest**: 1.7.x<br>pipelines-1.7: 1.7.x                              | **latest**: 1.8.x<br>pipelines-1.7: 1.7.x<br>pipelines-1.8: 1.8.x                   | **latest**: 1.9.x<br>pipelines-1.7: 1.7.x<br>pipelines-1.8: 1.8.x<br>pipelines-1.9: 1.9.x | **latest**: 1.9.x<br>pipelines-1.7: 1.7.x<br>pipelines-1.8: 1.8.x<br>pipelines-1.9: 1.9.x      |
| OCP 4.12 | N/A                     | N/A                                                                          | *RCs* **latest**: 1.8.x<br>pipelines-1.8: 1.8.x                                     | **latest**: 1.9.x<br>pipelines-1.8: 1.8.x<br>pipelines-1.9: 1.9.x                         | **latest**: 1.10.x<br>pipelines-1.8.x: 1.8.x<br>pipelines-1.9: 1.9.x<br>pipelines-1.10: 1.10.x |


- Stable channel is kept only on 4.9, because it’s already present,
  and stays on 1.6.x to match previous releases
- Latest channel always get the latest supported version on the given
  OpenShift
- For example, here, 1.9.x is only supported on 4.10 and above, and
  thus doesn’t appear on 4.9 Nothing prevent us, in the future, to
  support a wider range also
- The default channel is highlighted in **bold**


## Alternatives

### Keep using "one version" notation

- **Pros**
  - No change compared to today “flow”
  - We would just not increase the version each time (but only “on-demand”)
  - Discussing this with Alan Field, it seems like this is “how OLM envisioned fulfilling our use case” 😅.
  - It also is “close” to what we “support” upstream, as.. we set a minimum version but not “maximum” version, we support, potentially, all forward versions.
- **Cons**
  - What we support is not “technically” written (aka in code and in the bundle annotation)
  - A user with deep knowledge could install an older version on the cluster that we “wouldn’t” (want?) to support. We would need to document this somewhere (an OpenShift Pipelines support table of sort)


### Keep current channel names (stable/preview)

- **Pros**
  - Doesn’t change from today
  - Simple for the user to choose
- **Cons**
  - Doesn’t fix auto-approval for bug fixes but not for new version
