---
title: Integrate Pipelines as Code with Enterprise Contract
authors:
  - "@savita"
creation-date: 2023-07-10
status: implementable
---

# Integrate Pipelines as Code with Enterprise Contract

## Summary

By default Pipelines as Code runs all the pipelineruns based on the events coming from respective SCM and finaly display the status of run

The Supply Chain Security (Tekton Chains) in Openshift Pipelines works by observing all TaskRuns/PipelineRuns executions in the cluster. When TaskRuns/PipelineRuns complete, Chains takes a snapshot of them. Chains then converts this snapshot to one or more standard payload formats, signs them and stores them somewhere.

The Enterprise Contract is a set of tools for maintaining software supply chain security, and for the definition and enforcement of policies related to how container images are built and tested. and Its main purpose is to verify the security and provenance of builds created by Tekton Chains or Red Hat Trusted Application Pipeline (RHTAP).

So, Basically Tekton Chains helps to sign the image and Enterprise Contract helps to verify the signed image.

Now Pipelines as Code will be integrated to Enterprise Contract so that the build pipelines will be verified by Enterprise Contract and based on the verification status Pipelines as Code display the final status on the respective SCMs.

## Prerequisite

1. Make sure chains installed on the cluster
2. Follow [signed-provenance-tutorial.md](https://github.com/tektoncd/chains/blob/main/docs/tutorials/signed-provenance-tutorial.md) to configure chains

## Proposal

Tekton Chains 
1. sign the image and stores it in OCI registry
2. Attestate PipelineRun/TaskRun payload using slsa and stores is in local storage Tekton.
But Enterprise Contract only verify attestation of a signed image not the PipelineRun/TaskRun payload because Enterprise Contract understand the OCI not tekton storage.

Considering Enterprise Contract current support the Pipelines as Code will run Enterprise Contract tasks only for Build PipelineRuns(The PipelineRun which build the image).

story :

<https://issues.redhat.com/browse/SRVKP-3084>

### When PipelineRuns are not Building images

There won't be any changes in functionality behaviour 

### When There is a Build PipelineRuns (The PipelineRun which build the image)

If `.tekton` directory containes Build PipelineRuns then Pipelines as Code will understand that and triggers a Enterprise Contract task to verify the image sign

![Screenshot from 2023-07-10 13-12-43](https://github.com/savitaashture/enhancements/assets/9441662/48b38acf-5054-428f-8553-436ee3a2edc7)


### Design

First approach:

1. The PipelineRun provided by user under `.tekton` folder should have a label
`pipelinesascode.tekton.dev/cosign-pub: secret_name_where_cosign_pub_key_present`
2. Based on Pull/Push event Build PipelineRun will be created.
3. Once PipelineRun is success then PAC posts success message with inprogress state on GitHub checks with message like still verifying image sign.
4. PAC watches on event continously so whenever PAC finds [IMAGE_URL, IMAGE_DIGEST Results](https://github.com/tektoncd/chains/blob/main/docs/config.md#chains-type-hinting) PAC will understand its a Build PipelineRun and then checks for annotation `chains.tekton.dev/signed: "true"` if signing is success then PAC creates a [Enterprise Contract task](https://github.com/enterprise-contract/ec-cli/blob/main/tasks/verify-enterprise-contract/0.1/verify-enterprise-contract.yaml) to verify the image signing.

Inputs for Task:

The task requires information of IMAGE_URL and Public (cosign.pub) key
PAC will take public key from labels
  1. If its a scret it just uses as it is
  2. If its a string value PAC will create a temp secret and pass it to task

5. Once verification is success PAC updates summary message on same GitHub checks

Second approach:

1. Based on Pull/Push event Build PipelineRun will be created.
2. Once PipelineRun is success then PAC posts success message with inprogress state on GitHub checks with message like still verifying image sign.
3. PAC watches on event continously so whenever PAC finds [IMAGE_URL, IMAGE_DIGEST Results](https://github.com/tektoncd/chains/blob/main/docs/config.md#chains-type-hinting) PAC will understand its a Build PipelineRun and then checks for annotation `chains.tekton.dev/signed: "true"` if signing is success then PAC creates a [Enterprise Contract task](https://github.com/enterprise-contract/ec-cli/blob/main/tasks/verify-enterprise-contract/0.1/verify-enterprise-contract.yaml) to verify the image signing.

Inputs for Task:

The task requires information of IMAGE_URL and Public (cosign.pub) key
PAC will read chains secret from SYSTEM_NAMESPACE to get public key(cosign.pub).
Cons:
Non admin user won't have access to SYSTEM_NAMESPACE

4. Once verification is success PAC updates summary message on same GitHub checks
