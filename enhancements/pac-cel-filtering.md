---
title: Option to disable default creation of service account and rbac resources
authors:
  - "@chmouelito"
creation-date: 2021-03-14
status: implementable
---

# TEP-XXX: Pipelines as code event filtering with CEL

## Summary

This proposal helps user to do advanced filtering for PipelineRun selections.

## Motivation

We currently have a very basic filtering mechanism for Pac PipelineRuns.

Using the `pipelinesascode.tekton.dev/on-*` annotations user can specify wether it
targets a specific heads like branch or tags or an event (pull_request, issue).

```yaml
    pipelinesascode.tekton.dev/on-target-branch: "[%s]"
    pipelinesascode.tekton.dev/on-event: "[%s]"
```

This works for most cases but thi misses some flexibility for some advanced use cases.

For example when trying this user story :

<https://issues.redhat.com/browse/SRVKP-1746>

We want to be able to match events on files, which mean we want to express something like this :

- If the target branch is "main"
- If the event is a pull request
- And if the files `cli/**` has been modified

Then run the `cli` pipeline.

But the question is how to express negation, what if the user wants to match every files that are not in cli to that specific pipelinerun.

And even further the filtering would make it more difficult to express the same thing in a more readable way.

## Proposal

adding an annotation to the PipelineRun to express a CEL filtering.

for example :

```yaml
annotations:
    pipelinesascode.tekton.dev/on-cel-expression: |
        target_branch == "main" && event == "pull_request" && file.startswith("cli/**")
```

we can imagine any filters that can be expressed in CEL we want to grow with, including some custom function like triggers has.

We would still keep the basic filtering mechanism for now and only let the user use the `on-cel-expression` advanced for advance usage.

## Advantage

- We can address most of the Gitlab action use cases <https://docs.github.com/en/actions/using-workflows/workflow-syntax-for-github-actions#on>
- Use should be familiar with CEL syntax if they are familiar with triggers.

## Cons

- are we reinventing a `"trigger as code"` ? 🙃
- is user going to be confused between `on-target-branch/event` and `on-cel-expression`?
