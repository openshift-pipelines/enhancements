---
status: implementable
title: Private Git instances support in Hub
creation-date: "2022-03-01"
last-updated: "2022-03-01"
authors: ["vinamra28"]
---

# Private Git instances support in Hub

<!-- toc -->

- [Summary](#summary)
- [Motivation](#motivation)
  - [Goals](#goals)
  - [Non-Goals](#non-goals)
- [Requirements](#requirements)
- [Proposal](#proposal)
  - [Advantages](#advantages)
  - [Disadvantages](#disadvantages)
- [Design](#design)
  - [Cloning private repositories](#cloning-private-repositories)
  - [Serving README and YAML via Endpoints](#serving-readme-and-yaml-via-endpoints)
- [References (optional)](#references-optional)
<!-- /toc -->

## Summary

Tekton Hub currently supports catalog from public git repositories
such as `Bitbucket`, `Gitlab` or `Github` or from public git enterprises
where authentication is not required to clone a repository via `https` URL.
This document covers the way of supporting the private git repositories and instances.

## Motivation

Since customers are having their Github being behind VPN and various authentication
methods, this is blocking them to deploy their own instance of Hub as it wasn't able
to clone the repositories. Having the support of cloning these repositories will
help in more adoption of Hub by customers.

### Goals

1. Hub should be able to clone catalog and feed DB with all catalog details using username, password, or ssh
1. Hub UI should be able to render README and YAML for the resource

### Non-Goals

TBD

## Requirements

1. User should create the kubernetes secrets with ssh keys present in it
1. Public SSH key should be added in the git(github, bitbucket, gitlab, etc) account

## Proposal

1. User should be able to provide the sshUrl to TektonHub via [config.yaml][config.yaml]
   and Hub should find the ssh keys present in `~/.ssh` path and perform the clone
   into a directory which persists.
1. Since we are already cloning the repository in `/tmp` directory, why not to
   directly serve the README and YAML files directly from that cloned repository
   instead of reading from Github, Gitlab or Bitbucket. Hub should expose two
   endpoints `/resource/<catalog>/<kind>/<name>/<version>/readme` and
   `/resource/<catalog>/<kind>/<name>/<version>/yaml` which will do the needful.

### Advantages

1. It will always guarantee that the README and YAML would be
   available to the Hub UI even if the repository is private
1. Avoids API rate limiting issues
1. Even if Github or other git providers are down, Tekton Hub would still work
1. This will remove the inconsistency that is observed when a manifest
   gets updated remote but there was not catalog refresh so Hub isn't updated

### Disadvantages

1. If the clone of catalog is deleted, Hub will stop working until next catalog
   refresh is done
1. Increases the load on Hub server

## Design

### Cloning private repositories

In order to clone a private repository, modifying the [config.yaml][config.yaml]
by adding sshUrl like

```yaml
catalogs:
  - name: tekton
    org: tektoncd
    type: community
    provider: github
    url: https://github.com/tektoncd/catalog
    sshUrl: git@github.com:tektoncd/catalog
    revision: main
```

would tell the Hub API server to clone the repository using SSH Url
if this is isn't specified then use the https URL.

### Serving README and YAML via Endpoints

1. Doing catalog refresh will create a directory in the pod as
   `/tmp/<catalog_name>` where the catalog will be stored
1. `/resource/<catalog>/<kind>/<name>/<version>/readme` will search for
   the README present in the path `/tmp/<catalog>/<kind>/<task>/<version>/README.md`
   and return the response accordingly
1. `/resource/<catalog>/<kind>/<name>/<version>/yaml` will search for
   the README present in the path `/tmp/<catalog>/<kind>/<task>/<version>/<task>.yaml`
   and return the response accordingly

[config.yaml]: https://github.com/tektoncd/hub/blob/main/config.yaml
